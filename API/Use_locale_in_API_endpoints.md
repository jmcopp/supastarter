Use locale in API endpoints
Learn how to use the users locale in your supastarter API endpoints.

Sometimes you need to use the users locale in your API endpoints. For example, you might want to fetch data from a database that is localized or if you want to send an email in the users locale.

To make the users locale available in your endpoint handler, you can use the localeMiddleware.


import { localeMiddleware } from "../middleware/locale";
 
export const emailRouter = new Hono()
    .basePath("/send-localized-email")
    .get("/",
        localeMiddleware,
        // ...
        async (c) => {
            const locale = c.get("locale");
            // do something with the locale
        });
This middleware will either provide the locale from the locale cookie of the user or the default locale of the application.

Previous

Use API in application

Next

Streaming responses

© 2025 supastarter. All rights reserved.

Featured on Startup Fame



Blog
Documentation
Demo
Tools
SaaS ideas generator
Best SaaS ideas 2025
Boilerplates and Stacks
Showcase
Changelog
Roadmap
Become an affiliate
Privacy policy
Terms of service
Acceptable use
Disclaimer
License
Next.js SaaS starter kit
Next.js SaaS boilerplate
Nuxt SaaS starter kit
Next.js SaaS boilerplate
Nuxt SaaS boilerplate
SvelteKit SaaS starter kit
Next.js starter template
Nuxt starter template
SvelteKit starter template

