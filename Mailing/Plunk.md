Plunk
Learn how to use Plunk for sending emails.

To use Plunk with supastarter, first create a Plunk account and grab your API key.

Add the following environment variables to your .env.local file and your deployment environment:

.env.local

PLUNK_API_KEY="your-api-key"
Then, make sure to activate Plunk in the /packages/mail/src/provider/index.ts:


export * from './plunk';
To customize the provider, edit the packages/mail/src/provider/plunk/index.ts.

For further instructions, follow the general mail instructions.

Previous

Overview

Next

Resend

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




