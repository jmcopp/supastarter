Set up a paywall
Learn how to set up a paywall for your supastarter application.



Per default, supastarter has includes a free plan which means that users can access your application after signing up without any payment.

If your SaaS should not be accessible for free or only with a trial of a paid plan, you can set up a paywall.

All you need to do is to remove the plan with isFree: true from the configuration file.

config/index.ts

export const config = {
  payments: {
    plans: {
        free: { 
            isFree: false, 
        }, 
        // ...
    }
  },
}
This will remove the free plan and redirect the users to the /app/choose-plan page after the signup (and the onboarding flow).

Previous

Check for purchases or subscriptions

Next

Overview

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

