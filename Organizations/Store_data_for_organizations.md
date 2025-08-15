Store data for organizations
Learn how to store data for organizations in your supastarter application.

When working with organizations, you most likely want to store data for each organization that can be accessed by the organization members.

We are re-using the post example from the API documentation and extend it to store the organiaztion with it. Let's assume you want to enable all members of an organization to edit the posts of the organization instead of just the author.

You probably still want the individual author to be defined on the post, so you can still see who created the post.

Adjust the database schema
To do this, you can add the organizationId field in the post schema.


model Post {
  id        String   @id @default(cuid())
  title     String
  content   String
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  organizationId String
  organization Organization @relation(fields: [organizationId], references: [id]) 
}
 
model Organization {
  // ...
  posts Post[]
}
 
model User {
  // ...
  posts Post[]
}
Add organizationId to the post creation endpoint
To create a post now, you need to pass the organizationId to the endpoint.

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
	authMiddleware,
    validator("json", z.object({
      title: z.string(),
      content: z.string(),
      organizationId: z.string(),
    })),
 // ...
   async (c) => {
    const { title, content, organizationId } = await c.req.valid("json");
	const user = c.get("user");
 
 // verify that the user is a member of the organization
 // will throw an error if the user is not a member
	const { organization, role } = await verifyOrganizationMembership(organizationId, user.id);
 
 // optionally you can check the role of the user
 // if (role !== "admin") {
 // 	throw new HTTPException(403, { message: "You are not an admin of this organization" });
 // }
 
    const post = await db.post.create({
      data: {
        title,
        content,
		authorId: user.id,
		organizationId,
      },
    });
 
    return c.json(post);
  });
This resolver is protected by the authMiddleware which means only authenticated users can access it. Beyond that, we are checking if the user is a member of the organization with the verifyOrganizationMembership helper function. If the user is not a member, the function will throw an error and the request will not be processed further.

Create a post from the UI
To create a post from the UI, create a useCreateOrganizationPostMutation hook.


export const createOrganizationPostMutationKey = ["organization", "create-posts", "create"] as const;
export const useCreateOrganizationPostMutation = () => {
	return useMutation({
		mutationKey: createOrganizationPostMutationKey,
		mutationFn: async (
			data: InferRequestType<typeof apiClient.posts.$post>["json"],
		) => {
			const response = await apiClient.posts.$post({ data });
 
			if (!response.ok) {
				throw new Error("Failed to create post");
			}
		},
	});
};
When using this mutation, you need to pass the organizationId that you can get from the useActiveOrganization hook. Make sure to only call this mutation when there is an active organization.


const { activeOrganization } = useActiveOrganization();
const createOrganizationPostMutation = useCreateOrganizationPostMutation();
 
const onSubmit = async (data) => {
	if (!activeOrganization) {
		throw new Error("No active organization found");
	}
 
	await createOrganizationPostMutation.mutateAsync({
		...data,
		organizationId: activeOrganization.id,
	});
}
Add organizationId to the post query endpoint
Now you probably want to query the posts of an organization to list them in the UI. To do this, you can add the organizationId to the endpoint which lists all posts.

packages/api/src/routes/posts/router.ts

export const postsRouter = new Hono()
	.basePath("/posts")
	.get("/",
		validator("query", z.object({
			organizationId: z.string(),
		})),
		async (c) => {
			const { organizationId } = await c.req.valid("query");
 
   // always verify that the user is a member of the organization
			await verifyOrganizationMembership(organizationId, user.id);
 
			const posts = await db.post.findMany({
				where: {
					organizationId,
				},
			});
 
			return c.json(posts);
		}
	);
This endpoint will fetch all posts of the organization and return them as a JSON response.

Query posts by organization in the UI
Lastly, to query the posts by organization in the UI, you can create a query that takes an organizationId:


export const organizationPostsQueryKey = (organizationId?: string) => ["organization", organizationId, "posts"] as const;
export const useOrganizationPostsQuery = (
	organizationId?: string,
) => {
	return useQuery({
		queryKey: organizationPostsQueryKey(organizationId),
		queryFn: async () => {
			const response = await apiClient.posts.$get({
				query: {
					organizationId,
				},
			});
 
			if (!response.ok) {
				throw new Error("Failed to fetch posts");
			}
 
			return response.json();
		}
	});
};
Now inside your component, you can use the useOrganizationPostsQuery hook to fetch the posts of an organization.


const { data: posts, isPending } = useOrganizationPostsQuery(organizationId);
 
if (isPending) return <div>Loading...</div>;
 
if (!posts?.length) return <div>No posts found</div>;
 
return <div>{posts.map((post) => <div key={post.id}>{post.title}</div>)}</div>;
Previous

Use organizations in your application

Next

Overview

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




