User and session
Learn how to access the user and session in your supastarter application.

Accessing the user and session
You can access the user and session in your application by from the useSession hook.


const { user, session } = useSession();
Both can be null if the user is not authenticated, but if you use the hook inside an /app/... route, they should always be defined.

The user object contains the information of the authenticated user and the session object contains the session data.


type Session = {
    id: string;
    userId: string;
    createdAt: Date;
    updatedAt: Date;
    expiresAt: Date;
    token: string;
    ipAddress?: string | null | undefined;
    userAgent?: string | null | undefined;
    impersonatedBy?: string | null | undefined;
}
 
type User = {
    id: string;
    createdAt: Date;
    updatedAt: Date;
    email: string;
    emailVerified: boolean;
    name: string;
    image?: string | null | undefined;
    role: "admin" | "user";
    onboardingComplete: boolean;
}
Wait until the session has been loaded
In some cases you might want to wait until the session has been loaded before accessing the user and session. For this there is a loaded property that you can use.


const { user, session, loaded } = useSession();
 
if (!loaded) return <div>Loading...</div>;
Reload session
If for some reason you need to reload the session, for example when you changed some property of the user like it's name or role, you can use the reloadSession function.


const { reloadSession } = useSession();
 
await reloadSession();
Get session on server
To use the session on the server, e.g. in a RSC, you can use the getSession function.


import { getSession } from "@saas/auth/lib/server";
 
async function ServerComponent() {
    const session = await getSession();
 
    return <div>User name: {JSON.stringify(session?.user.name)}</div>;
}
Previous

Overview

Next

Permissions and access control

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

