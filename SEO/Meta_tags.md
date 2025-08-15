Meta tags
Learn how to set meta tags for pages in your supastarter app.

Meta tags are used to describe the content of a page. It's important to set these tags for each page, as it helps search engines understand what the page is about and can improve the SEO of your app.

Per default, your application has set a base title for each page, which is defined in the /apps/web/app/layout.tsx file.

apps/web/app/layout.tsx

export const metadata: Metadata = {
	title: {
		absolute: "supastarter.nextjs - Application",
		default: "supastarter.nextjs- Application",
		template: "%s | supastarter.nextjs - Application",
	},
};
This setup will result in the title supastarter.nextjs - Application for the home page and each page that has no title set.

If a title is set for a page, it the template will be applied to the title, so if your page title is Changelog, the title will be Changelog | supastarter.nextjs - Application.

To set a meta tags for a specific page, use the generateMetadata function in that page:

apps/web/app/changelog/page.tsx

export const generateMetadata = () => {
	return {
		title: "Changelog",
		description: "The description of the changelog page",
	};
};
To have an internationalized title, you can also use translations in the generateMetadata function:

apps/web/app/changelog/page.tsx

export async function generateMetadata() {
	const t = await getTranslations();
	return {
		title: t("changelog.title"),
		description: t("changelog.description"),
	};
}
Learn more about how to set meta tags in the offical Next.js documentation.

Previous

Documentation

Next

Sitemap

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

