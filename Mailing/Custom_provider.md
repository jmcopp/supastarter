Custom provider
Learn how to use a custom provider for sending emails.

You can also use any other provider you like, by creating a new file in the provider folder and implementing the SendEmailHandler interface:


import { config } from "@repo/config";
import { SendEmailHandler } from "../types";
 
const { from } = config;
 
export const send: SendEmailHandler = async ({ to, subject, text, html }) => {
  // handle your custom email sending logic here
};
Then, make sure to export your custom provider in the /packages/mail/src/provider/index.ts:


export * from './custom';
Previous

Nodemailer

Next

Internationalization

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




