Going to production
Learn how to make your app ready for production and launch it.

This section gives you a checklist of things you need to do before launching your apps and includes common pitfalls to avoid.

We will not cover the deployment process here in detail, please refer to the deployment section for more information.

Mails
Mail templates
supastarter includes a hand full of mail templates for things like magic links, email verification, password reset, etc.



You can find them in the /api-main/app/services/emails directory. Each mail template is a React component that you can customize to your needs.

You definitely want to change the logo inside the mail template Wrapper.tsx to your own logo. The colors will be automatically adjusted to your brand colors from the tailwind config.

Check the mailing section for more information on how to customize the mail templates.

Preview all mail templates by running pnpm --filter mail preview and opening http://localhost:3005 in your browser.

If you added a new language to your supastarter application, also make sure that all mail templates are translated to the new language, as otherwise the mails might not be displayed correctly.

Things to check:


All templates are working

Design is consistent with your brand

All templates are translated

Logo is changed to your own logo
Mail provider and domain
Make sure you have set up your mail provider and domain correctly, so that the mails are sent out correctly and not marked as spam.

Also make sure that you have set up the from property in the config/index.ts file in the mail section to your own email address.

Things to check:


Mail provider is set up correctly

Domain for sending mails is verified in mail provider

from config is set to an email address of your domain
Internationalization
supastarter supports multiple languages and allows to switch between them with a language switcher. Per default English and German are provided and enabled.

A common pitfall is to keep the language enabled that you don't want to support, which can unwanted behavior as the site will automatically redirect the user to the language that fits the browser language. If you don't want to support a language, make sure to disable it in the config/index.ts file in the i18n section.

Things to check:


Only the languages you want to support are enabled

All keys for the enabled languages are translated
Payments
Products and subscriptions
Make sure the products or subscriptions you want to offer are set up correctly in the payment provider. Also make sure to use the production mode of your payment provider as Stripe and Lemonsqueezy both offer test modes, that you should not use in production.

That means all the prices are set correctly (monthly, yearly, etc.) and if you want to offer your productions in different currencies, make sure to set up the currencies correctly.

Things to check:


All products and subscriptions are set up correctly in production mode

All prices are set correctly (monthly, yearly, etc.)

All currencies supported currencies are defined and configured for each language
Webhoooks
If you have set up billing for your app, you need to set up webhooks for the payment provider to sync the purchases to your database.

If you have done this already, make sure you are using the production keys and environment variables. Also make sure that the webhook is pointing to your production url.

Things to check:


You are using the production keys and environment variables from your payment provider

Webhook is pointing to your production url
SEO
Especially for the marketing pages you want to make sure that the SEO is set up correctly, so that the pages can be found and indexed by the search engines.

Read the guides on meta tags and sitemap to learn more.

Things to check:


All pages have a meta title and description

All pages that should be indexed are added to the sitemap
Legal pages
supastarter has defined a few placeholder pages for legal pages, like privacy policy which you edit in the /frontend/content/legal/*.md files.

Common legal pages are:

Cookie policy
Imprint
Create the legal pages that make sense for your kind of application.

The legal pages are added to the sitemap automatically, so you don't need to worry about that.

Things to check:


Define all necessary legal pages
Deployment
Environment variables
Make sure that all the environment variables that are required for your app to work are set in your deployment environment.

Things to check:


All environment variables are set correctly

Make sure you are using production values for the environment variables
Region
When your have deployed your app to a cloud provider that runs your app (or at least the serverless functions) in a specific region (like Vercel does), you need to make sure that you have selected a region that is:

Close to your database location
Close to your target audience
This is important as the latency between the user, the server and the database can have a big impact on the performance of your app.

When you target a global audience, prioritize the first point or choose a platform that has multiple regions. If you deploy to multiple regions, you want to have a globally distributed database too, as otherwise the latency can be too high from some regions.

Things to check:


App is deployed to a region close to your database location

App is deployed to a region close to your target audience
