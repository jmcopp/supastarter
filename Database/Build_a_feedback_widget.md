Build a Feedback Widget
Learn how to build a feedback widget in supastarter – a complete guide from database to UI

In this guide, we'll take you through the process of implementing a feature in supastarter from start to finish. This will a lot of the things from the previous guides and documentation, but we'll be doing it in a more complete way and with a more concrete example of creating a feedback widget for your app.

Overview
The feedback widget consists of:

Database Schema: Prisma model to store feedback data
Database Queries: Dedicated query functions for feedback operations
API Endpoint: REST API to handle feedback submissions with session integration
Frontend Component: React component with form and UI
Translations: Internationalization support for the widget
Step 1: Database Schema
First, we added a Feedback model to the Prisma schema:


// packages/database/prisma/schema.prisma
 
model Feedback {
    id        String   @id @default(cuid())
    userId    String?
    user      User?    @relation(fields: [userId], references: [id], onDelete: SetNull)
    email     String?
    name      String?
    message   String
    type      String
    ipAddress String?
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
 
    @@map("feedback")
}
We also added the relation to the User model:


model User {
    // ... existing fields ...
    feedbacks          Feedback[]
}
Step 2: Database Queries
Created dedicated query functions for feedback operations:


// packages/database/prisma/queries/feedback.ts
import { db } from "../client";
 
export async function createFeedback({
    message,
    type,
    email,
    name,
    ipAddress,
    userId,
}: {
    message: string;
    type: string;
    email?: string;
    name?: string;
    ipAddress?: string;
    userId?: string;
}) {
    return await db.feedback.create({
        data: {
            message,
            type,
            email,
            name,
            ipAddress,
            userId,
        },
    });
}
Updated the queries index to export the feedback functions:


// packages/database/prisma/queries/index.ts
export * from "./ai-chats";
export * from "./feedback";
export * from "./organizations";
export * from "./purchases";
export * from "./users";
Step 3: API Endpoint
Created the feedback API route with validation and session integration:


// packages/api/src/routes/feedback/types.ts
import { z } from "zod";
 
export const feedbackSchema = z.object({
    message: z.string().min(10).max(1000),
    type: z.enum(["bug", "feature", "general"]).default("general"),
    email: z.string().email().optional(),
    name: z.string().min(2).max(100).optional(),
});
 
export type FeedbackFormValues = z.infer<typeof feedbackSchema>;

// packages/api/src/routes/feedback/router.ts
import { auth } from "@repo/auth";
import { createFeedback } from "@repo/database";
import { logger } from "@repo/logs";
import { Hono } from "hono";
import { describeRoute } from "hono-openapi";
import { resolver, validator } from "hono-openapi/zod";
import { z } from "zod";
import { localeMiddleware } from "../../middleware/locale";
import { feedbackSchema } from "./types";
 
export const feedbackRouter = new Hono().basePath("/feedback").post(
    "/",
    validator("json", feedbackSchema),
    describeRoute({
        tags: ["Feedback"],
        summary: "Submit user feedback",
        description: "Submit feedback with optional contact information",
        responses: {
            201: {
                description: "Feedback submitted successfully",
                content: {
                    "application/json": {
                        schema: resolver(z.object({ 
                            id: z.string(),
                            message: z.string() 
                        })),
                    },
                },
            },
            400: {
                description: "Invalid feedback data",
                content: {
                    "application/json": {
                        schema: resolver(z.object({ error: z.string() })),
                    },
                },
            },
        },
    }),
    async (c) => {
        const feedbackData = c.req.valid("json");
        const ipAddress = c.req.header("x-forwarded-for") || c.req.header("x-real-ip");
        const session = await auth.api.getSession({
            headers: c.req.raw.headers,
        });
 
        try {
            // Store feedback in database using the query function
            const feedback = await createFeedback({
                message: feedbackData.message,
                type: feedbackData.type,
                email: feedbackData.email,
                name: feedbackData.name,
                ipAddress,
                userId: session?.user.id,
            });
 
            return c.json({ 
                id: feedback.id,
                message: "Feedback submitted successfully" 
            }, 201);
        } catch (error) {
            logger.error("Failed to submit feedback:", error);
            return c.json({ error: "Could not submit feedback" }, 500);
        }
    },
);
Added the router to the main API app:


// packages/api/src/app.ts
import { feedbackRouter } from "./routes/feedback/router";
 
const appRouter = app
    .route("/", authRouter)
    .route("/", webhooksRouter)
    .route("/", aiRouter)
    .route("/", uploadsRouter)
    .route("/", paymentsRouter)
    .route("/", contactRouter)
    .route("/", feedbackRouter)  // Added this line
    .route("/", newsletterRouter)
    .route("/", organizationsRouter)
    .route("/", adminRouter)
    .route("/", healthRouter);
Step 4: Frontend Component
Created a React component with form validation, session integration, and internationalization:


// apps/web/modules/shared/components/FeedbackWidget.tsx
"use client";
 
import { zodResolver } from "@hookform/resolvers/zod";
import { useSession } from "@saas/auth/hooks/use-session";
import { Button } from "@ui/components/button";
import {
    Dialog,
    DialogContent,
    DialogHeader,
    DialogTitle,
    DialogTrigger,
} from "@ui/components/dialog";
import {
    Form,
    FormControl,
    FormField,
    FormItem,
    FormLabel,
    FormMessage,
} from "@ui/components/form";
import { Input } from "@ui/components/input";
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectValue,
} from "@ui/components/select";
import { Textarea } from "@ui/components/textarea";
import { cn } from "@ui/lib";
import { MessageSquare, Send } from "lucide-react";
import { useTranslations } from "next-intl";
import { useState } from "react";
import { useForm } from "react-hook-form";
import { z } from "zod";
import { feedbackSchema, type FeedbackFormValues } from "@repo/api/src/routes/feedback/types";
import { apiClient } from "@repo/api/src/lib/api-client";
import { toast } from "sonner";
 
export function FeedbackWidget({ className }: { className?: string }) {
    const t = useTranslations();
    const { user } = useSession();
    const [isOpen, setIsOpen] = useState(false);
 
    const createFeedbackMutation = useMutation({
        mutationFn: async (data: FeedbackFormValues) => {
            const response = await apiClient.feedback.create({
                body: data,
            });
 
            if (!response.ok) {
                throw new Error("Failed to submit feedback");
            }
 
            return response.data;
        }
    });
 
    const form = useForm<FeedbackFormValues>({
        resolver: zodResolver(feedbackSchema),
        defaultValues: {
            message: "",
            type: "general",
            email: "",
            name: "",
        },
    });
 
    const onSubmit = async (data: FeedbackFormValues) => {
        setIsSubmitting(true);
        try {
            await createFeedbackMutation.mutateAsync(data);
 
            setIsOpen(false);
            form.reset();
            toast.success(t("feedback.success.message"));
        } catch (error) {
            console.error("Error submitting feedback:", error);
            toast.error(t("feedback.error.message"));
        }
    };
 
    return (
        <Dialog open={isOpen} onOpenChange={setIsOpen}>
            <DialogTrigger asChild>
                <Button
                    variant="outline"
                    size="sm"
                    className={cn(
                        "fixed bottom-4 right-4 z-50 shadow-lg",
                        className,
                    )}
                >
                    <MessageSquare className="h-4 w-4 mr-2" />
                    {t("feedback.button")}
                </Button>
            </DialogTrigger>
            <DialogContent className="sm:max-w-md">
                <DialogHeader>
                    <DialogTitle>{t("feedback.title")}</DialogTitle>
                </DialogHeader>
 
                    <Form {...form}>
                        <form
                            onSubmit={form.handleSubmit(onSubmit)}
                            className="space-y-4"
                        >
                            <FormField
                                control={form.control}
                                name="type"
                                render={({ field }) => (
                                    <FormItem>
                                        <FormLabel>
                                            {t("feedback.form.type.label")}
                                        </FormLabel>
                                        <Select
                                            onValueChange={field.onChange}
                                            defaultValue={field.value}
                                        >
                                            <FormControl>
                                                <SelectTrigger>
                                                    <SelectValue
                                                        placeholder={t(
                                                            "feedback.form.type.placeholder",
                                                        )}
                                                    />
                                                </SelectTrigger>
                                            </FormControl>
                                            <SelectContent>
                                                <SelectItem value="general">
                                                    {t(
                                                        "feedback.form.type.options.general",
                                                    )}
                                                </SelectItem>
                                                <SelectItem value="bug">
                                                    {t("feedback.form.type.options.bug")}
                                                </SelectItem>
                                                <SelectItem value="feature">
                                                    {t(
                                                        "feedback.form.type.options.feature",
                                                    )}
                                                </SelectItem>
                                            </SelectContent>
                                        </Select>
                                        <FormMessage />
                                    </FormItem>
                                )}
                            />
 
                            {!user && (
                                <>
                                    <FormField
                                        control={form.control}
                                        name="name"
                                        render={({ field }) => (
                                            <FormItem>
                                                <FormLabel>
                                                    {t("feedback.form.name.label")}
                                                </FormLabel>
                                                <FormControl>
                                                    <Input
                                                        placeholder={t(
                                                            "feedback.form.name.placeholder",
                                                        )}
                                                        {...field}
                                                    />
                                                </FormControl>
                                                <FormMessage />
                                            </FormItem>
                                        )}
                                    />
 
                                    <FormField
                                        control={form.control}
                                        name="email"
                                        render={({ field }) => (
                                            <FormItem>
                                                <FormLabel>
                                                    {t("feedback.form.email.label")}
                                                </FormLabel>
                                                <FormControl>
                                                    <Input
                                                        placeholder={t(
                                                            "feedback.form.email.placeholder",
                                                        )}
                                                        {...field}
                                                    />
                                                </FormControl>
                                                <FormMessage />
                                            </FormItem>
                                        )}
                                    />
                                </>
                            )}
 
                            <FormField
                                control={form.control}
                                name="message"
                                render={({ field }) => (
                                    <FormItem>
                                        <FormLabel>
                                            {t("feedback.form.message.label")}
                                        </FormLabel>
                                        <FormControl>
                                            <Textarea
                                                placeholder={t(
                                                    "feedback.form.message.placeholder",
                                                )}
                                                className="min-h-[100px]"
                                                {...field}
                                            />
                                        </FormControl>
                                        <FormMessage />
                                    </FormItem>
                                )}
                            />
 
                            <Button
                                type="submit"
                                className="w-full"
                                loading={createFeedbackMutation.isPending}
                            >
                                {t("feedback.form.submit")}
                            </Button>
                        </form>
                    </Form>
            </DialogContent>
        </Dialog>
    );
}
Step 5: Translations
Added translation keys for the feedback widget:


// packages/i18n/translations/en.json
{
  "feedback": {
    "button": "Feedback",
    "title": "Send Feedback",
    "success": {
      "title": "Thank you!",
      "message": "Your feedback has been submitted successfully."
    },
    "error": {
      "title": "Error",
      "message": "Failed to submit feedback"
    },
    "form": {
      "type": {
        "label": "Feedback Type",
        "placeholder": "Select feedback type",
        "options": {
          "general": "General",
          "bug": "Bug Report",
          "feature": "Feature Request"
        }
      },
      "name": {
        "label": "Name",
        "placeholder": "Your name"
      },
      "email": {
        "label": "Email",
        "placeholder": "your.email@example.com"
      },
      "message": {
        "label": "Message",
        "placeholder": "Tell us what you think..."
      },
      "submit": "Send Feedback"
    }
  }
}

// packages/i18n/translations/de.json
{
  "feedback": {
    "button": "Feedback",
    "title": "Feedback senden",
    "success": {
      "title": "Vielen Dank!",
      "message": "Ihr Feedback wurde erfolgreich übermittelt."
    },
    "error": {
      "title": "Fehler",
      "message": "Feedback konnte nicht gesendet werden"
    },
    "form": {
      "type": {
        "label": "Feedback-Typ",
        "placeholder": "Feedback-Typ auswählen",
        "options": {
          "general": "Allgemein",
          "bug": "Fehlermeldung",
          "feature": "Feature-Anfrage"
        }
      },
      "name": {
        "label": "Name",
        "placeholder": "Ihr Name"
      },
      "email": {
        "label": "E-Mail",
        "placeholder": "ihre.email@beispiel.com"
      },
      "message": {
        "label": "Nachricht",
        "placeholder": "Sagen Sie uns, was Sie denken..."
      },
      "submit": "Feedback senden"
    }
  }
}
Step 6: Integration
Add the feedback widget to your layout or pages:


// apps/web/app/(marketing)/layout.tsx
import { FeedbackWidget } from "@modules/shared/components/FeedbackWidget";
 
export default function MarketingLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <>
            {children}
            <FeedbackWidget />
        </>
    );
}
Step 7: Evaluation
The next logical step would be to add some way to evaluate the feedback. You could either send it an admin email or build a small page in the admin dashboard to view the feedback.

We'll cover this in a future guide.

Features
The feedback widget includes the following features:

Session Integration: Automatically associates feedback with logged-in users
Conditional Fields: Name and email fields are hidden for logged-in users
Internationalization: Full i18n support with English and German translations
Form Validation: Client-side validation with Zod schema
Responsive Design: Mobile-friendly UI with Tailwind CSS
Database Storage: Persistent storage with Prisma ORM
API Integration: RESTful API with proper error handling
Type Safety: Full TypeScript support throughout the stack
Usage
The feedback widget will appear as a floating button in the bottom-right corner of the page. Users can:

Click the feedback button to open the dialog
Select a feedback type (General, Bug Report, or Feature Request)
Enter their name and email (only for non-logged-in users)
Write their feedback message
Submit the feedback
The widget automatically handles:

Form validation
Loading states
Success messages
Error handling
Session management
Database storage
Further Considerations
Here are some additional features you might want to consider adding to your feedback widget:

Analytics and Insights: Track feedback patterns and user behavior
Email Notifications: Send admin notifications for new feedback submissions
Rate Limiting: Prevent spam by limiting submissions per IP address
Feedback Management Dashboard: Create an admin interface to view and manage feedback
Sentiment Analysis: Integrate AI services to analyze feedback sentiment
Export and Reporting: Add CSV export functionality for feedback data
Accessibility Improvements: Enhance screen reader support and keyboard navigation
Customization Options: Allow configuration of widget position, theme, and visibility
Multi-language Support: Add more translation languages beyond English and German
Feedback Categories: Add more specific feedback types or custom categories
File Attachments: Allow users to attach screenshots or files with their feedback
Follow-up Communication: Enable two-way communication with feedback submitters
Conclusion
This guide demonstrates how to build a complete feedback widget in supastarter, covering all aspects from database design to user interface. The implementation follows supastarter's best practices:

Type Safety: Full TypeScript integration throughout the stack
Database Design: Proper Prisma schema with relationships
API Design: RESTful endpoints with validation and error handling
UI/UX: Modern, accessible components with Shadcn UI
Internationalization: Multi-language support
Session Integration: Seamless user experience for logged-in users
Error Handling: Comprehensive error management and user feedback
The feedback widget serves as an excellent example of how to implement features in supastarter, showcasing the framework's capabilities for building production-ready applications. The modular approach allows for easy extension and customization based on your specific needs.

Key takeaways:

Start with database schema and work your way up to the UI
Use TypeScript for type safety across the entire stack
Implement proper validation and error handling
Consider user experience and accessibility
Plan for scalability and maintenance
Follow the established patterns in your codebase
This pattern can be applied to build other features like contact forms, support tickets, or any user input system. The feedback widget demonstrates the power and flexibility of the supastarter framework for building real-world applications.

Previous

Supabase setup

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




