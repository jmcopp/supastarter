Nodemailer
Learn how to use Nodemailer for sending emails.

To use Nodemailer with supastarter, make sure you have an SMTP server available, that you can send mails from.

Add the following environment variables to your .env.local file and your deployment environment:

.env.local

MAIL_HOST="your-mail-host"
MAIL_PORT="your-mail-port"
MAIL_USER="your-mail-user"
MAIL_PASS="your-mail-password"
Then, make sure to activate Nodemailer in the /packages/mail/src/provider/index.ts:


export * from './nodemailer';
To customize the provider, edit the packages/mail/src/provider/nodemailer/index.ts.

For further instructions, follow the general mail instructions.

Previous

Postmark

Next

Custom provider

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

