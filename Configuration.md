Configuration
Learn how to configure your supastarter application.

supastarter is a highly flexible and customizable starter kit and allow you to configure the project to your needs. You can find the main config file in the config/index.ts file in the repository.

This section will cover all configuration options and when to use them.

Config file
You can find the configuration object in the config/index.ts file in the repository.


export const config = {
    // Internationalization
	i18n: {
        // Whether internationalization should be enabled (if disabled, you still need to define the locale you want to use below and set it as the default locale)
		enabled: true,
        // Define all locales here that should be available in the app
        // You need to define a label that is shown in the language selector and a currency that should be used for pricing with this locale
		locales: {
			en: {
				currency: "USD",
				label: "English",
			},
			de: {
				currency: "USD",
				label: "Deutsch",
			},
		},
        // The default locale is used if no locale is provided
		defaultLocale: "en",
        // The default currency is used for pricing if no currency is provided
		defaultCurrency: "USD",
        // The name of the cookie that is used to determine the locale
		localeCookieName: "NEXT_LOCALE",
	},
    // Organizations
	organizations: {
        // Whether organizations are enabled in general
		enable: true,
        // Whether billing for organizations should be enabled (below you can enable it for users instead)
		enableBilling: true,
        // Whether the organization should be hidden from the user (use this for multi-tenant applications)
		hideOrganization: false,
        // Should users be able to create new organizations? Otherwise only admin users can create them
		enableUsersToCreateOrganizations: true,
        // Whether users should be required to be in an organization. This will redirect users to the organization page after sign in
		requireOrganization: false,
        // These colors are used for placeholder avatar if the organization has no logo uploaded
		avatarColors: ["#4e6df5", "#e5a158", "#9dbee5", "#ced3d9"],
        // Define forbidden organization slugs. Make sure to add all paths that you define as a route after /app/... to avoid routing issues
		forbiddenOrganizationSlugs: [
			"new-organization",
			"admin",
			"settings",
			"ai-demo",
		],
	},
    // Users
	users: {
        // Whether billing should be enabled for users (above you can enable it for organizations instead)
		enableBilling: true,
        // Whether you want the user to go through an onboarding form after signup (can be defined in the OnboardingForm.tsx)
		enableOnboarding: false,
	},
    // Authentication
	auth: {
        // Whether users should be able to create accounts (otherwise users can only be by admins)
		enableSignup: true,
  // Whether users should be able to sign in with a magic link
        enableMagicLink: true,
  // Whether users should be able to sign in with a social provider
		enableSocialLogin: true,
  // Whether users should be able to sign in with a passkey
		enablePasskeys: true,
  // Whether users should be able to sign in with a password
		enablePasswordLogin: true,
        // where users should be redirected after the sign in
		redirectAfterSignIn: "/app",
        // where users should be redirected after logout
		redirectAfterLogout: "/",
        // how long a session should be valid
		sessionCookieMaxAge: 60 * 60 * 24 * 30,
	},
    // Mails
	mails: {
        // the from address for mails
		from: "hello@your-domain.com",
	},
    // Frontend
	ui: {
        // the themes that should be available in the app
		enabledThemes: ["light", "dark"],
        // the default theme
		defaultTheme: "light",
        // the saas part of the application
		saas: {
            // whether the saas part should be enabled (otherwise all routes will be redirect to the marketing page)
			enabled: true,
            // whether the sidebar layout should be used
			useSidebarLayout: true,
		},
        // the marketing part of the application
		marketing: {
            // whether the marketing features should be enabled (otherwise all routes will be redirect to the saas part)
			enabled: true,
		},
	},
    // Storage
	storage: {
        // define the name of the buckets for the different types of files
		bucketNames: {
			avatars: process.env.NEXT_PUBLIC_AVATARS_BUCKET_NAME ?? "avatars",
		},
	},
    // Contact form
    contactForm: {
        // whether the contact form should be enabled
        enabled: true,
        // the email address to which the contact form should be sent
        to: "hello@your-domain.com",
        // the subject of the email
        subject: "New contact form submission",
    },
    // Payments
	payments: {
        // define the products that should be available in the checkout
        // read the payments documentation for more information on how to define plans
		plans: {
            // ...
		},
	},
};
Use cases
The configuration options enable you to set up lots of different use cases. Here are some examples:

Deploy marketing page only
If you want to deploy the marketing page only, you can set the ui.saas.enabled option to false. This will disable the saas part of the application and redirect requests to the marketing page.


export const config = {
    // ...
    ui: {
        saas: {
            enabled: false,
        },
    },
};
To not have to manually set this option while developing locally, you can define an environment variable NEXT_PUBLIC_SAAS_ENABLED in your .env file and use this in the file file.


export const config = {
    // ...
    ui: {
        saas: {
            enabled: process.env.NEXT_PUBLIC_SAAS_ENABLED === "true",
        },
    },
};
This way you can set the deployed version to false while enabling the development version to true.

Disable marketing page
If you want to ship the marketing page separately and just use the SaaS part of supastarter, you can simply disable the marketing part of the application. This will redirect all requests to the SaaS part or the login page if the user is not authenticated.


export const config = {
    // ...
    ui: {
        marketing: {
            enabled: false,
        },
    },
};
Multi-tenant application
If you want to use supastarter as a multi-tenant application, you can use the following configuration. This will

require users to be in an organization
hide the organization from the user
redirect users to the organization page after sign in

export const config = {
    // ...
    organizations: {
        hideOrganization: true,
        requireOrganization: true,
        enableUsersToCreateOrganizations: false,
    },
};
With this setup, you can have two different behaviors for creating organizations.

Users can create an initial organizations (and no further)
Only admins can create organizations
To enable the second behavior, you can set the auth.enableSignup option to false. This way users can only join if they are invited to an organization and organizations can only be created by admins.


export const config = {
    // ...
    auth: {
        enableSignup: false,
    },
};
Enable/disable onboarding
There is a prepared onboarding form, which allows users to set up their profile after sign up. If you want to disable it, you can set the users.enableOnboarding option to false.


export const config = {
    // ...
    users: {
        enableOnboarding: false,
    },
};
Attach billing to users or organizations
You can attach billing to users or organizations. To enable this for either one, you can set the enableBilling option to true.


export const config = {
    // for users
    users: {
        enableBilling: true,
    },
    // for organizations
    organizations: {
        enableBilling: true,
    },
};
In theory, you can have both enabled at the same time, but for most use cases you want to attach it to only one of them.

Disable organizations
If you don't want to use organizations, you can set the organizations.enable option to false. This will disable the organizations feature totally.


export const config = {
    // ...
    organizations: {
        enable: false,
    },
};
Customize authentication
You can easily (de)activate authentication methods like social login, passkeys, password login and magic links.


export const config = {
    // ...
    auth: {
        enableSocialLogin: false,
        enablePasskeys: false,
        enablePasswordLogin: false,
        enableMagicLink: false,
    },
};
Previous

Setup

Next

Troubleshooting

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

