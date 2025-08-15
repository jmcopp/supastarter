Google Analytics
Learn how to use Google Analytics with supastarter.

To use Google Analytics with supastarter, first create a Google Analytics account and grab your analytics id.

Then, add the app code to your .env.local file and your deployment environment:


NEXT_PUBLIC_GOOGLE_ANALYTICS_ID=your_id_here
Next, we need to activate Google Analytics as the analytics provider. To do this, open apps/web/modules/analytics/index.tsx and change the content to:


export * from "./provider/google";
To customize the Google Analytics configuration, edit the apps/web/modules/analytics/provider/google/index.tsx.

For further instructions, follow the general analytics instructions.

Previous

Overview

Next

Mixpanel

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

