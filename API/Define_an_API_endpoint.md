Define an API endpoint
Learn how to define an API endpoint in your supastarter application.

To define a new API endpoint you can either create a new router or add a new endpoint to an existing router.

This guide assumes you have already created a model in your database schema for the feature you want to create an API endpoint for. To learn how to create a model in your schema and update your database, please refer to the database documentation.

Create a new router
A router is basically a sub-path of your API. It is used to group related endpoints together.

For this example we will create a new router for the posts feature, which will contain the common CRUD endpoints for the posts resource.

If you expect your router to have a lot of endpoints, we recommend to create a new router file in a sub-folder for the feature. For example /packages/api/src/routes/posts/router.ts.

If you expect your router to have a few endpoints, you can also create a new router file in the /packages/api/src/routes/posts.ts file.

We are going to go with the first option and create a new router file in a sub-folder for the feature.

packages/api/src/routes/posts/router.ts

import { Hono } from "hono";
 
export const postsRouter = new Hono().basePath("/posts");
You can see we are using the basePath method to set the base path for the router. This is optional, but it is a good practice to separate the API endpoints by feature. It will cause all endpoints we define in this router to be mounted at /api/posts.

Now we can add a new endpoint to the router.

packages/api/src/routes/posts/router.ts

import { z } from "zod";
import { describeRoute } from "hono-openapi";
import { resolver, validator } from "hono-openapi/zod";
import { HTTPException } from "hono/http-exception";
import { z } from "zod";
import { PostSchema } from "database";
 
export const postsRouter = new Hono()
  .basePath("/posts")
  .get("/",
    describeRoute({
		tags: ["Posts"],
		summary: "Get all posts",
		responses: {
			200: {
				description: "Returns all posts",
				content: {
					"application/json": {
						schema: resolver(z.array(PostSchema)),
					},
				},
			},
		},
	}),
   async (c) => {
    const posts = await db.post.findMany();
    return c.json(posts);
  });
This endpoint will fetch all posts from the database and return them as a JSON response.

We have also defined the necessary properties to generate the OpenAPI specification for this endpoint.

Mount feature router
To make this endpoint available in the API, we need to add it to the main router in the /packages/api/src/api.ts file.

packages/api/src/api.ts

// ...
import { postsRouter } from "./routes/posts/router";
 
// ...
 
const appRouter = app
 // ...
	.route("/posts", postsRouter);
Now you can use the endpoint in your application. It is available at /api/posts.

CRUD endpoints
Here are examples of the typical CRUD endpoints for the posts resource.

Create a post
packages/api/src/routes/posts/router.ts

import { z } from "zod";
import { describeRoute } from "hono-openapi";
import { resolver, validator } from "hono-openapi/zod";
import { HTTPException } from "hono/http-exception";
import { z } from "zod";
import { PostSchema } from "database";
 
export const postsRouter = new Hono()
  .basePath("/posts")
  .post("/",
    validator("json", z.object({
      title: z.string(),
      content: z.string(),
    })),
    describeRoute({
		tags: ["Posts"],
		summary: "Create a new post",
		responses: {
			200: {
				description: "Returns the created post",
				content: {
					"application/json": {
						schema: resolver(PostSchema),
					},
				},
			},
		},
	}),
   async (c) => {
    const { title, content } = await c.req.valid("json");
    const post = await db.post.create({
      data: {
        title,
        content,
      },
    });
    return c.json(post);
  });
If you want to protect this endpoint or add the current user as the author of the post, you can use the authMiddleware in your router or on a specific endpoint.

This example assumes that you have added the authorId field and the relation to the User model to the post model.


// ...
import { authMiddleware } from "../middleware/auth";
 
export const postsRouter = new Hono()
    .basePath("/posts")
    .post("/",
        authMiddleware,
        // ...
        async (c) => {
            const { title, content } = await c.req.valid("json");
			const user = c.get("user");
            const post = await db.post.create({
                data: {
                    title,
                    content,
                    authorId: user.id,
                },
				include: {
					author: true,
				},
            });
            return c.json(post);
        });
Get single post
packages/api/src/routes/posts/router.ts

import { z } from "zod";
import { describeRoute } from "hono-openapi";
import { resolver, validator } from "hono-openapi/zod";
import { HTTPException } from "hono/http-exception";
import { z } from "zod";
import { PostSchema } from "database";
 
export const postsRouter = new Hono()
  .basePath("/posts")
  .get("/:id",
    describeRoute({
		tags: ["Posts"],
		summary: "Get a single post",
		responses: {
			200: {
				description: "Returns the post",
				content: {
					"application/json": {
						schema: resolver(PostSchema),
					},
				},
			},
		},
	}),
   async (c) => {
    const id = c.req.param("id");
    const post = await db.post.findUnique({
      where: {
        id,
      },
    });
    return c.json(post);
  });
Update a post
packages/api/src/routes/posts/router.ts

import { z } from "zod";
import { describeRoute } from "hono-openapi";
import { resolver, validator } from "hono-openapi/zod";
import { HTTPException } from "hono/http-exception";
import { z } from "zod";
import { PostSchema } from "database";
 
export const postsRouter = new Hono()
  .basePath("/posts")
  .put("/:id",
    describeRoute({
		tags: ["Posts"],
		summary: "Update a post",
		responses: {
			200: {
				description: "Returns the updated post",
				content: {
					"application/json": {
						schema: resolver(PostSchema),
					},
				},
			},
		},
	}),
   async (c) => {
	const id = c.req.param("id");
    const { title, content } = await c.req.valid("json");
    const post = await db.post.update({
      where: {
        id,
      },
      data: {
        title,
        content,
      },
    });
    return c.json(post);
  });
Delete a post
packages/api/src/routes/posts/router.ts

import { z } from "zod";
import { describeRoute } from "hono-openapi";
import { resolver, validator } from "hono-openapi/zod";
import { HTTPException } from "hono/http-exception";
import { z } from "zod";
import { PostSchema } from "database";
 
export const postsRouter = new Hono()
  .basePath("/posts")
  .delete("/",
    describeRoute({
		tags: ["Posts"],
		summary: "Delete a post",
		responses: {
			204: {
				description: "No content returned",
			},
		},
	}),
   async (c) => {
    const id = c.req.param("id");
    await db.post.delete({
      where: {
        id,
      },
    });
    return c.body(null, 204);
  });
Previous

Overview

Next

Protect API endpoints

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

