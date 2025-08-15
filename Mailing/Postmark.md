Postmark
Learn how to use Postmark for sending emails.

To use postmark with supastarter, first create a postmark account and grab your server token.

Add the following environment variables to your .env.local file and your deployment environment:

.env.local

POSTMARK_SERVER_TOKEN="your-server-token"
Then, make sure to activate Postmark in the /packages/mail/src/provider/index.ts:


export * from './postmark';
To customize the provider, edit the packages/mail/src/provider/postmark/index.ts.

For further instructions, follow the general mail instructions.

Previous

Resend

Next

Nodemailer

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




