Upstash QStash
Learn how to set up background jobs, cron jobs and queues with Upstash in your supastarter application.

Upstash QStash is a reliable message queue and job scheduler that's perfect for serverless environments. It allows you to create background jobs, schedule tasks, and handle webhook processing with ease.

Prerequisites
Before getting started, you'll need:

An Upstash account
A QStash service created in your Upstash dashboard
Your QStash credentials (URL and token)
1. Install Dependencies
First, install the QStash SDK:


pnpm add --filter api @upstash/qstash
2. Environment Variables
Add the following environment variables to your .env.local file:


QSTASH_URL=https://qstash.upstash.io
QSTASH_TOKEN=your_qstash_token_here
QSTASH_CURRENT_SIGNING_KEY=your_current_signing_key_here
QSTASH_NEXT_SIGNING_KEY=your_next_signing_key_here
You can find these values in your Upstash dashboard under the QStash service.

3. Create the QStash Client
Create a utility file to initialize the QStash client:

packages/api/src/lib/qstash.ts

import { Client as QStashClient } from "@upstash/qstash";
 
export const qstashClient = new QStashClient({
	baseUrl: process.env.QSTASH_URL,
	token: process.env.QSTASH_TOKEN,
});
4. Create a task router
Create a new router to handle your tasks:

packages/api/src/routes/tasks/router.ts

import { Receiver } from "@upstash/qstash";
import { Hono } from "hono";
import { createMiddleware } from "hono/factory";
 
const qStashVerifyMiddleware = createMiddleware(async (c, next) => {
	const currentSigningKey = process.env.QSTASH_CURRENT_SIGNING_KEY as string;
	const nextSigningKey = process.env.QSTASH_NEXT_SIGNING_KEY as string;
 
	if (!currentSigningKey || !nextSigningKey) {
		return c.json({ error: "QStash signing keys not configured" }, 500);
	}
 
	const signature = c.req.header("upstash-signature");
	const body = await c.req.text();
 
	if (!signature) {
		return c.json({ error: "Missing signature" }, 401);
	}
 
	try {
		const receiver = new Receiver({
			currentSigningKey,
			nextSigningKey,
		});
 
		const isValid = receiver.verify({
			body,
			signature,
		});
 
		if (!isValid) {
			return c.json({ error: "Invalid signature" }, 401);
		}
 
		await next();
	} catch (error) {
		console.error("QStash signature verification failed:", error);
		return c.json({ error: "Invalid signature" }, 401);
	}
});
 
export const tasksRouter = new Hono()
	.basePath("/tasks")
	.use(qStashVerifyMiddleware)
	.post("/test", async (c) => {
		const { message } = await c.req.json();
 
		console.log(message);
 
		return c.json({ message: "Task received" });
	});
5. Trigger a task
To trigger your task, you can use the qstashClient to send a request to the task router:


import { qstashClient } from "../lib/qstash";
 
await qstashClient.publishJSON({
	url: `${getBaseUrl()}/api/tasks/test`,
	body: {
		message: "Hello, world!"
	}
});
6. Usage Examples
Cron jobs
To schedule a task to run at a specific time, you can use the qstashClient to create a cron job:


import { qstashClient } from "@/lib/qstash";
 
await client.schedules.create({
  destination: `${getBaseUrl()}/api/tasks/test`,
  cron: "*/1 * * * *",
});
Queues
To create a queue, you can use the qstashClient to create a queue and create a sequence of tasks to be executed.


import { qstashClient } from "@/lib/qstash";
 
const queue = await qstashClient.queues.create({
	name: "my-queue",
});
 
await queue.enqueueJSON({
	url: `${getBaseUrl()}/api/tasks/step-1`,
	body: {
		message: "Hello, world!"
	}
});
 
await queue.enqueueJSON({
	url: `${getBaseUrl()}/api/tasks/step-2`,
	body: {
		message: "Hello, world!"
	}
});
Previous

trigger.dev

Next

Going to production

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




