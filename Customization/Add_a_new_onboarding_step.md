Add a new onboarding step
Learn how to adjust the onboarding flow in your supastarter app and add a new step.

The onboarding flow in supastarter is designed to be modular and easily extensible. This guide explains how to add new steps to the onboarding process.

Overview
The onboarding system consists of two main components:

OnboardingForm.tsx - The container component that manages step navigation
Individual step components (e.g., OnboardingStep1.tsx)
Adding a New Step
1. Create the Step Component
Create a new file for your step component in frontend/modules/saas/onboarding/components/. For example, OnboardingStep2.tsx (you could also name it semantically, e.g. OnboardingStepTeam.tsx):


"use client";
 
import { zodResolver } from "@hookform/resolvers/zod";
import { Button } from "@ui/components/button";
import { Form } from "@ui/components/form";
import { useTranslations } from "next-intl";
import { useForm } from "react-hook-form";
import { z } from "zod";
 
// 1. Define your form schema
const formSchema = z.object({
  // Add your form fields here
});
 
type FormValues = z.infer<typeof formSchema>;
 
export function OnboardingStep2({ onCompleted }: { onCompleted: () => void }) {
  const t = useTranslations();
  const form = useForm<FormValues>({
    resolver: zodResolver(formSchema),
  });
 
  const onSubmit = async (values: FormValues) => {
    try {
      // Handle form submission
      // Update user data if needed
      onCompleted();
    } catch (e) {
      form.setError("root", {
        type: "server",
        message: t("onboarding.notifications.error"),
      });
    }
  };
 
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        {/* Add your form fields here */}
      </form>
    </Form>
  );
}
2. Update OnboardingForm.tsx
Add your new step to the steps array in OnboardingForm.tsx.


const steps = [
  {
    component: <OnboardingStep1 onCompleted={() => setStep(2)} />,
  },
  {
    component: <OnboardingStep2 onCompleted={() => onCompleted()} />,
  },
];
3. Add Translations
Add the necessary translations for your step in your translation files:


{
  "onboarding": {
    "step2": {
      "title": "Step 2 Title",
      "description": "Step 2 Description"
      // Add other translations
    }
  }
}
Best Practices
State Management

Use the onCompleted prop to handle navigation between steps
For the final step, call the main onCompleted function to finish onboarding
Form Validation

Always use Zod schemas for form validation
Include proper error handling and error messages
UI Components

Use Shadcn UI components for consistency
Follow the existing pattern of using Form, FormField, FormItem, etc.
Responsive Design

Ensure your step component works well on both mobile and desktop
Use Tailwind's responsive classes when needed
Progress Indicator

The progress bar will automatically update based on the number of steps in the steps array
Example Flow
User starts at step 1 (?step=1)
After completing step 1, they're moved to step 2 (?step=2)
After completing the final step, they're redirected to the app dashboard
Notes
The onboarding state is tracked in the URL using the step query parameter
If you want to make the process more robust, you could add a onboardingStep cookie to track the step in case the user navigates away from the page and comes back later.
The final onCompleted function updates the user's onboardingComplete status and redirects to the app
Each step component should be a client component (use "use client" directive)
