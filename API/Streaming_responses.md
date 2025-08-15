Streaming responses
Learn how to stream responses from your API.

As supastarter integrates Hono as the API framework, you can easily stream responses from the API which is useful if you are working with AI providers.

Streaming with the API
With Vercel AI SDK
When using the Vercel AI SDK, you can use the streamText function to call the AI provider and then call the toDataStreamResponse function to stream the response:


export const aiRouter = new Hono()
	.basePath("/ai")
	.post(
		"/chats-response",
		async (c) => {
			const { id } = c.req.param();
			const { messages } = c.req.valid("json");
			const user = c.get("user");
 
			const response = streamText({
				model: textModel,
				messages,
			});
 
			return response.toDataStreamResponse({
				sendUsage: true,
			});
		},
	);
With Hono streamText
If you want to stream any data from your API, you can use the stream function from Hono.


export const aiRouter = new Hono()
	.basePath("/ai")
	.post(
		"/stream",
		async (c) => {
			return streamText(c, async (stream) => {
				await stream.writeln('Hello')
				await stream.sleep(1000)
				await stream.write(`Hono!`)
			});
		},
	);
To learn more about streaming with Hono, you can read the Hono documentation.

Previous

Use locale in API endpoints

Next

OpenAPI documentation

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

