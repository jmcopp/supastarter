Permissions and access control
Learn how use permissions and access control in your supastarter frontend application.

We have already guided you through the process of how implement access control in the API routes of your application. In this guide we will show you how you can protect pages and display UI based on the users role or permissions.

Protect a route (server side)
For authenticated users
To protect a route to be only accessible for authenticated users, you can simply get the session in the RSC component and check if the user is authenticated.

Note: When you are inside the /apps/web/app/(saas)/app directory, you don't need to check if the user is authenticated, because the session is verified in the middleware.


import { getSession } from "@saas/auth/lib/server";
export async function MyProtectedPage() {
    const { session } = await getSession();
 
    if (!session) {
        return redirect("/auth/login");
    }
 
    return <div>My protected page</div>;
}
For specific roles
More interesting is to check if the user has the necessary permissions to access the page. For example, you can make a page only accessible for users with the admin role.


import { getSession } from "@saas/auth/lib/server";
 
export async function MyAdminPage() {
    const { session } = await getSession();
 
    if (!session?.user.role === 'admin') {
        return redirect("/app");
    }
 
    return <div>This page is only accessible for admins</div>;
}
For active or specifc subscription
Or if you want to check for an active subscription, you can do the following:


export async function MyPremiumPage() {
    const purchases = await getPurchases();
    
    const { activePlan, hasSubscription } = createPurchasesHelper(purchases);
 
    if (!activePlan) {
        return redirect("/app");
        // or show a message to the user that they need to subscribe to the premium plan
    }
 
    // or check for a specific subscription - you don't need to check for the active plan if you use this
    if (!hasSubscription('pro')) {
        return <div>This page is only accessible for users with a pro subscription</div>;
    }
 
    return <div>This page is only accessible for users with an active subscription</div>;
}
For organization role
You can also check if a user has a specific role inside the current organization. For example, you might want to add features that are only available for organization owners or admins.


import { isOrganizationAdmin } from "@repo/auth/lib/helper";
import { getActiveOrganization, getSession } from "@saas/auth/lib/server";
 
export async function MyOrganizationPage({
	params,
}: PropsWithChildren<{
	params: Promise<{ organizationSlug: string }>;
}>) {
    const { session } = await getSession();
    const organizationSlug = await params;
	const organization = await getActiveOrganization(organizationSlug);
 
    if (!organization) {
		redirect("/app");
	}
 
	const userIsOrganizationAdmin = isOrganizationAdmin(
		organization,
		session?.user,
	);
 
    if (!userIsOrganizationAdmin) {
        return <div>This page is only accessible for organization admins</div>;
    }
 
    return <div>This page is only accessible for organization admins</div>;
}
Display UI based on permissions (client side)
On client side, you can use the useSession hook to get the session and then check if the user has the necessary permissions.

For authenticated users

import { useSession } from "@saas/auth/lib/client";
 
export function MyComponent() {
    const { session } = useSession();
 
    if (!session) {
        return <div>You need to be logged in to access this page</div>;
    }
 
    return <div>You are logged in</div>;
}
Note: You always want to check the permission on the server side first to avoid any security issues.

For specific roles

import { useSession } from "@saas/auth/lib/client";
 
export function MyComponent() {
    const { session } = useSession();
 
    if (session?.user.role !== 'admin') {
        return <div>This page is only accessible for admins</div>;
    }
 
    return <div>This page is only accessible for admins</div>;
}
For active or specifc subscription

import { useSession } from "@saas/auth/lib/client";
 
export function MyComponent() {
    const { activePlan, hasSubscription } = usePurchases(); // or usePurchases(organizationId) if you have enabled billing for organizations
 
    if (!activePlan) {
        return <div>You don't have an active subscription</div>;
    }
 
    if (!hasSubscription('pro')) {
        return <div>You need to subscribe to the pro plan to access this page</div>;
    }
 
    return <div>You have an active subscription</div>;
}
For organization role

import { useSession } from "@saas/auth/lib/client";
 
export function MyComponent() {
	const { activeOrganization, activeOrganizationUserRole, isOrganizationAdmin } = useActiveOrganization();
 
    // isOrganizationAdmin is a helper that includes the roles 'admin' and 'owner'
    if (!isOrganizationAdmin) {
        return <div>This page is only accessible for organization admins</div>;
    }
 
    if (activeOrganizationUserRole !== 'some-role') {
        return <div>This page is only accessible for organization members with the some-role role</div>;
    }
 
    return <div>This page is only accessible for organization admins</div>;
}
Previous

User and session

Next

oAuth

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




