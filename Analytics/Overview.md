Overview
Learn how to use analytics in supastarter.

supastarter comes with a generic analytics module that enables you to integrate analytics providers and track custom events.

Setup analytics
In the apps/web/modules/analytics/index.ts file, export the provider file of the provider you want to use:


export * from "./provider/google";
// or export * from './provider/mixpanel';
// or export * from './provider/pirsch';
// or export * from './provider/plausible';
// or export * from './provider/umami';
// or export * from './provider/vercel';
// or export * from './provider/posthog';
// or export * from './provider/custom';
To learn how to set up your analytics provider, check out the respective documentation:

Google Analytics
Mixpanel
Pirsch
Plausible
Umami
Vercel
Vemetric
PostHog
Custom
Analytics script component
The analytics script component that is part of the provider file you exported in the first step is automatically included in the application:

apps/web/modules/shared/components/Document.tsx

export function Document({ children }: PropsWithChildren) {
	return (
		<html>
			<body>
				{children}
				<AnalyticsScript />
			</body>
		</html>
	);
}
Now your app will include the analytics script and track page views.

Track custom events
To track custom events, import the useAnalytics hook and use the trackEvent function:


import { useAnalytics } from "@analytics";
 
export function YourComponent() {
  const { trackEvent } = useAnalytics();
 
  return (
    <button onClick={() => trackEvent("button-clicked", { value: "asdf" })}>
      Click me
    </button>
  );
}
The trackEvent function takes two arguments:

event: The name of the event
data: An object containing additional data about the event
Previous

Going to production

Next

Google Analytics

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




