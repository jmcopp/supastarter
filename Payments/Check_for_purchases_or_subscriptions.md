Check for purchases or subscriptions
Learn how to check for purchases or subscriptions in your application to provide access to premium features.

One of the most common use cases for payments is to provide access to premium features.

Plan ID
The plan id is used to identify a plan. It is defined in the key property of the plan object from the config file like free, pro, enterprise or lifetime from the example config.

Check for purchases or subscriptions on client side
You can check if a user has a purchase or subscription by using the usePurchases hook.

The usePurchases hook returns the following properties:

activePlan: The active plan of the user
purchases: An array of all the purchases of the user
hasSubscription: A function to check if the user has an active subscription. Optionally takes a plan id or an array of plan ids as argument to check for specific plans.
hasPurchase: A function to check if the user has a specifc purchase for a given plan id

 
function SomeComponent() {
    const { activePlan, hasSubscription, hasPurchase } = usePurchases();
 
    // check if the user has an active subscription
    const hasActiveSubscription = hasSubscription();
 
    // check if the user has a purchase for the pro plan
    const hasProPurchase = hasPurchase("pro");
    // or check if the user has a purchase for the pro plan or the enterprise plan
    const hasProOrEnterprisePurchase = hasPurchase(["pro", "enterprise"]);
 
    // check if the user has a purchase of the lifetime plan
    const hasLifetimeAccess = hasPurchase("lifetime");
}
Check for purchases on server side
You can also check for purchases and get the active plan on the server side by utilizing the createPurchasesHelper function, which will generate a bunch of helper functions based on the purchases you pass to it.

The createPurchasesHelper can be used in both RSC and API routes, but the way to fetch the purchases is different:

RSC
When you are inside your Next.js application in a RSC, you can use the getPurchases function from the @saas/payments/lib/server file.


import { getPurchases } from "@saas/payments/lib/server";
import { createPurchasesHelper } from "@repo/payments/lib/helper";
 
const purchases = await getPurchases();
 
const { activePlan, hasSubscription, hasPurchase } = createPurchasesHelper(purchases);
API routes
When you are inside an API route, you can use the relative import of getPurchases function from the packages/api/src/routes/payments/lib/purchases file. You also need to pass the organization id or the user id as an argument to the getPurchases function.


import { getPurchases } from "../../payments/lib/purchases";
import { createPurchasesHelper } from "@repo/payments/lib/helper";
import { authMiddleware } from "../../lib/middleware";
 
const someRouter = new Hono()
    .get('/', authMiddleware, async (c) => {
        const { organizationId } = c.req.param();
        const user = c.get('user');
 
        const purchases = await getPurchases({ organizationId }); // or getPurchases({ userId: user.id })
        const { activePlan, hasSubscription, hasPurchase } = createPurchasesHelper(purchases);
 
        // do something with the helpers here...
    })
Check for purchases of organization
Both usePurchases and getPurchases also support organizations. Simply pass the organization id as an argument.


// ... get the organization slug from the url
const organization = await getActiveOrganization(organizationSlug);
const purchases = await getPurchases(organizationId);

const { activeOrganization } = useActiveOrganization();
const { activePlan, hasSubscription, hasPurchase } = usePurchases(activeOrganization!.id);
Previous

Manage plans and products

Next

Set up a paywall

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

