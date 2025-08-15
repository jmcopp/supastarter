Overview
Learn how to create background tasks & cron jobs in your supastarter application.

With every larger application, you'll get to a point where you need to run tasks in the background or want to queue up or schedule tasks to be executed. This could be sending emails, processing images, or any other task that doesn't need to be done immediately, take too long for an API call, or need to be run at a specific time.

There are multiple approaches to implement tasks in your supastarter application.

Serverless environments
If you deploy your application in a serverless environment, you want to use a third-party service to schedule and trigger your tasks like trigger.dev or upstash:

trigger.dev
Upstash QStash
Self-hosted environments
If you have your backend deployed on a long-running server like described in our standalone API documentation, you can also use libraries like bullmq to create and schedule background tasks.

Previous

Standalone API

Next

trigger.dev

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

