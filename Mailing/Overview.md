Overview
Learn how to send emails in your supastarter application.

For mails supastarter integrates React Email which enables you to write your mails in React.

Why React Email:
It gives you the ability to style your mails with Tailwind CSS just like in your application. Of course the config from your app is reused for your mails. It also gives you the ability to use components in your mails, so you can build a generic template and use it in all your mails.

Providers
In the mail package in your repository you can find the provider folder.

There are multiple providers available:

Plunk
Postmark
Resend
Nodemailer
Console
Custom provider
Set from mail address
Next up, set the from mail address in the config/index.ts file. This is the mail address that will be used as the sender of all mails. Please make sure that the mail address and domain are verified in your mail provider.


export const config = {
  mails: {
    from: "example@example.com",
  },
};
Mail templates
In the mail package in your repository you can find the templates folder. In here all your mail templates are located.



To preview your templates while developing you can run the following command in your terminal:


pnpm --filter mail preview
This will start a local server on port 3005 where you can preview your mails.

If this isn't working for you, try running the following command in the packages/mail/.react-email subfolder:


# run this in packages/mail/.react-email
npm i
Create a mail template
To create a new mail template, simply create a new .tsx in the templates folder that has a default export of a React component.

To use variables, just define them as props of your component.

The translation strings for your mail templates are defined in the packages/i18n/translatinons folder.


import { Link, Text } from "@react-email/components";
import { createTranslator } from "use-intl/core";
import type { BaseMailProps } from "../types";
import PrimaryButton from "./components/PrimaryButton";
import Wrapper from "./components/Wrapper";
 
export function MagicLink({
  url,
  name,
  otp,
  locale,
  translations,
}: {
  url: string;
  name: string;
  otp: string;
} & BaseMailProps): JSX.Element {
  const t = createTranslator({
    locale,
    messages: translations,
    namespace: "mail",
  });
 
  return (
    <Wrapper>
      <Text>{t("magicLink.body", { name })}</Text>
 
      <Text>
        {t("common.otp")}
        <br />
        <strong className="text-2xl font-bold">{otp}</strong>
      </Text>
 
      <Text>{t("common.useLink")}</Text>
 
      <PrimaryButton href={url}>{t("magicLink.login")} &rarr;</PrimaryButton>
 
      <Text className="text-muted-foreground text-sm">
        {t("common.openLinkInBrowser")}
        <Link href={url}>{url}</Link>
      </Text>
    </Wrapper>
  );
}
 
export default MagicLink;
Check out the offical docs of React Email for more information on how to use it and the available components.

Register the mail template
Before you can use your new mail template, you have to register it in the lib/templates.ts file:


import { NewMail } from "../templates/NewMail";
 
export const mailTemplates = {
  //...
  newMail: NewMail,
};
Use the mail template
Now you can send a mail with your template using the sendMail method in your application:


import { sendMail } from "mail";
 
await sendMail({
  to: "tim@apple.com",
  template: "newMail",
  context: {
    name: "Tim Cook",
  },
});
Edit mail template wrapper
All mail templates are wrapped in the Wrapper.tsx component which provides the base layout for the emails and also includes the logo. You most likely want to change the logo to your own logo and if you want to adjust the common layout of the emails you can do so here. In here you can also adjust the Tailwind CSS config that will be used across the email templates.


export default function Wrapper({ children }: PropsWithChildren) {
	return (
		<Html lang="en">
			<Head>
				<Font
					fontFamily="Inter"
					fallbackFontFamily="Arial"
					fontWeight={400}
					fontStyle="normal"
				/>
			</Head>
			<Tailwind
				config={{
     // ...
				}}
			>
				<Section className="bg-background p-4">
					<Container className="rounded-lg bg-card p-6 text-card-foreground">
						<Logo />
						{children}
					</Container>
				</Section>
			</Tailwind>
		</Html>
	);
}
The logo can be adjusted in the packages/mail/src/components/Logo.tsx component.

Translations
All mail templates can be translated using the use-intl (the core library of next-intl) package. The translations are defined in the packages/i18n/translations folder.

To use translations in your mail templates, you can use the createTranslator function from use-intl/core.

Each mail template is passed the locale and translations props. The locale is the current locale of the user and the translations are the translations for the current locale.


import { createTranslator } from "use-intl/core";
 
export function MyMailTemplate({
  locale,
  translations,
}: BaseMailProps) {
  const t = createTranslator({
    locale,
    messages: translations,
    namespace: "mail",
  });
 
  return (
    <Wrapper>
      <Text>{t("myMailTemplate.body")}</Text>
    </Wrapper>
  );
}
Per default the sendEmail function will use the default locale you defined in your config/index.ts.

The API uses the sendEmail function to send various mails like the magic link mail or the password reset mail. In the API we have a locale variable available in the context which will be automatically read from the users locale cookie, so it will match the selected language of the user for each request.

If you want to send a mail in a specific language you can pass the locale prop to the sendEmail function:


await sendEmail({
  to: "example@example.com",
  template: "myMailTemplate",
  locale: "de",
  context: {
    // ...
  },
});
Previous

Accessing stored files

Next

Plunk

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

