oAuth
Learn how to setup up new oAuth provider in your supastarter application.

Setup up new provider
As an example, we will add Facebook as an oAuth provider.

First go to the Facebook developer console and create a new app.

Add the credentials to your .env.local file:


FACEBOOK_CLIENT_ID=your-client-id
FACEBOOK_CLIENT_SECRET=your-client-secret
In the /packages/auth/auth.ts file you need to add the new provider add the new provider to the following places:


export const auth = betterAuth({
  // ...
	account: {
		accountLinking: {
   // ...
			trustedProviders: [
        //..
        "facebook"
      ],
		},
	},
	socialProviders: {
    // ...
		facebook: {
			clientId: process.env.FACEBOOK_CLIENT_ID as string,
			clientSecret: process.env.FACEBOOK_CLIENT_SECRET as string,
		},
	},
});
The last thing you need to do is to add the new provider to the oAuthProviders object in the apps/web/modules/saas/auth/constants/oauth-providers.ts file:


export const oAuthProviders = {
  // ...
  facebook: {
    name: "Facebook",
    icon: ({ ...props }: IconProps) => (
      // grab the svg from https://simpleicons.org/
      <svg {...props} viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
        <path d="M9.101 23.691v-7.98H6.627v-3.667h2.474v-1.58c0-4.085 1.848-5.978 5.858-5.978.401 0 .955.042 1.468.103a8.68 8.68 0 0 1 1.141.195v3.325a8.623 8.623 0 0 0-.653-.036 26.805 26.805 0 0 0-.733-.009c-.707 0-1.259.096-1.675.309a1.686 1.686 0 0 0-.679.622c-.258.42-.374.995-.374 1.752v1.297h3.919l-.386 2.103-.287 1.564h-3.246v8.245C19.396 23.238 24 18.179 24 12.044c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.628 3.874 10.35 9.101 11.647Z" />
      </svg>
    ),
  },
};
Now you should see the new provider on the login page.

Learn more about oAuth providers in the better-auth documentation.

Previous

Permissions and access control

Next

Super Admin & Admin UI

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

