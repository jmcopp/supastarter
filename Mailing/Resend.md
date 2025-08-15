Resend
Learn how to use Resend for sending emails.

To use Resend with supastarter, first create a Resend account and grab your API key.

Add the following environment variables to your .env.local file and your deployment environment:

.env.local

RESEND_API_KEY="your-api-key"
Then, make sure to activate Resend in the /packages/mail/src/provider/index.ts:


export * from './resend';
For further instructions, follow the general mail instructions.

Previous

Plunk

Next

Postmark

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




