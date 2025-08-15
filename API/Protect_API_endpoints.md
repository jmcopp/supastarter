Protect API endpoints
Learn how to protect your API endpoints in your supastarter application.

Check if the user is authenticated
To protect your API endpoints, you can use the authMiddleware in your router or on a specific endpoint.

We'll use the posts example from the Define an API endpoint guide and make it only available for authenticated users.


// ...
import { authMiddleware } from "../middleware/auth";
 
export const postsRouter = new Hono()
    .basePath("/posts")
    .get("/",
        authMiddleware,
        // ...
        async (c) => {
            const posts = await db.post.findMany();
            return c.json(posts);
        });
The middlware will throw an error if the user is not authenticated.

The authMiddleware will also provide the session and user infromation to the endpoint. If the handler is reached, both objects are available in the context.


async (c) => {
    const user = c.get("user");
    const session = c.get("session");
 
    // do something with the user and session
});
Admin-only endpoints
There is a second middleware which does the same as the authMiddleware, but it also checks if the user has the admin role.


import { adminMiddleware } from "../middleware/admin";
 
export const adminRouter = new Hono()
    .get("/some-admin-route",
        adminMiddleware,
        // ...
        async (c) => {
            // do something only admins can do
        });
Previous

Define an API endpoint

Next

Use API in application

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




