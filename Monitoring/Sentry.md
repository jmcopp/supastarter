Sentry
Learn how to set up Sentry for monitoring your supastarter app.

Sentry is a tool that helps you monitor your application and identify issues from backend to frontend. In this guide, we will set up Sentry for your supastarter app.

Create a Sentry project
Before we start integrating Sentry into your application, you need to sign up for Sentry and create a new organization and project in the Sentry dashboard.



Follow the setup wizard to create a new project by selecting Next.js as the platform and entering the project name:





Set up Sentry in your application
1. Install the Sentry SDK
Add the Sentry SDK to your web application by running the following command:


pnpm --filter web add @sentry/nextjs
3. Add Sentry to Next.js config
In your /apps/web/next.config.ts file, add the Sentry plugin:

apps/web/next.config.ts

import { withSentryConfig } from "@sentry/nextjs";
 
// ...
 
export default withNextIntl(
	withContentCollections(
		withSentryConfig(nextConfig, {
			org: "supastarter",
			project: "supastarter-web",
			authToken: process.env.SENTRY_AUTH_TOKEN,
			tunnelRoute: "/monitoring-tunnel",
			silent: false,
		}),
	),
);
_Note Make sure to call the with* functions in the correct order, as there might be issues with a different order.

As we are using the tunnel route, we need to make sure that the middleware is not blocking the request to the tunnel route. For this we need to extend the matcher to the middleware config:

apps/web/middleware.ts

// ...
export const config = {
	matcher: [
		"/((?!api|image-proxy|images|fonts|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt|monitoring-tunnel).*)",
	],
};
3. Create sentry config files
Create the following files in the /apps/web folder:

Note: Replace the DSN url with the one from your Sentry project.

apps/web/sentry.client.config.ts

import * as Sentry from "@sentry/nextjs";
 
Sentry.init({
	dsn: "https://a19de6cb0beb3b0f5b5874eb59901f84@o4508742845857792.ingest.de.sentry.io/4508742854574160",
	integrations: [Sentry.replayIntegration()],
	tracesSampleRate: 1,
	replaysSessionSampleRate: 0.1,
	replaysOnErrorSampleRate: 1.0,
	debug: false,
});
apps/web/sentry.server.config.ts

import * as Sentry from "@sentry/nextjs";
 
Sentry.init({
	dsn: "https://a19de6cb0beb3b0f5b5874eb59901f84@o4508742845857792.ingest.de.sentry.io/4508742854574160",
	tracesSampleRate: 1,
	debug: false,
});
apps/web/sentry.server.config.ts

import * as Sentry from "@sentry/nextjs";
 
Sentry.init({
	dsn: "https://a19de6cb0beb3b0f5b5874eb59901f84@o4508742845857792.ingest.de.sentry.io/4508742854574160",
	tracesSampleRate: 1,
	debug: false,
});
4. Set up Next.js intstrumentation
Next, we will set up Next.js intstrumentation to capture errors and performance data for the different types of requests.

Create a new file in the /apps/web folder:

apps/web/instrumentation.ts

import * as Sentry from "@sentry/nextjs";
 
export async function register() {
	if (process.env.NEXT_RUNTIME === "nodejs") {
		await import("./sentry.server.config");
	}
 
	if (process.env.NEXT_RUNTIME === "edge") {
		await import("./sentry.edge.config");
	}
}
 
export const onRequestError = Sentry.captureRequestError;
5. Create global error handler
Create a new file in the /apps/web/app folder:

apps/web/app/global-error.tsx

"use client";
 
import * as Sentry from "@sentry/nextjs";
import NextError from "next/error";
import { useEffect } from "react";
 
export default function GlobalError({
	error,
}: {
	error: Error & { digest?: string };
}) {
	useEffect(() => {
		Sentry.captureException(error);
	}, [error]);
 
	return (
		<html lang="en">
			<body>
				<NextError statusCode={0} />
			</body>
		</html>
	);
}
6. Set Sentry envs
In the Sentry dashboard, go to Settings > Auth Tokens and create a new token.

Add the new token as the SENTRY_AUTH_TOKEN environment variable to your .env file.

We also need to set the SENTRY_SUPPRESS_TURBOPACK_WARNING environment variable to 1 to suppress the warning about the Turbopack integration.


SENTRY_AUTH_TOKEN=<your-sentry-auth-token>
SENTRY_SUPPRESS_TURBOPACK_WARNING=1
And that's it! You can now start your application and see the errors and performance data in the Sentry dashboard.

You can find a full example of the Sentry setup in the repository supastarter-nextjs/feat/sentry-integration repository.

For more information and further use cases, check out the offical Sentry setup guide for Next.js.

Previous

Overview

Next

E2E testing

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




