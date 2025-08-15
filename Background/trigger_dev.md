trigger.dev
Learn how to set up background jobs, cron jobs and queues with trigger.dev in your supastarter application.

Before we get started, make sure to create a new trigger.dev and project.

You can find a full example of this recipe in the feat/trigger.dev branch of the supastarter-nextjs repository.

1. Create a new package in your repository
Create a new folder tasks in the /packages directory and add the following files:

packages/tasks/package.json

{
	"dependencies": {
		"@repo/database": "workspace:*",
		"@trigger.dev/sdk": "3.3.17"
	},
	"devDependencies": {
		"@biomejs/biome": "2.1.0",
		"@trigger.dev/build": "3.3.17",
		"@repo/tsconfig": "workspace:*"
	},
	"main": "./index.ts",
	"name": "@repo/tasks",
	"scripts": {
		"type-check": "tsc --noEmit",
		"dev": "pnpm dlx trigger.dev@latest dev --env-file ../../.env.local",
		"deploy": "pnpm dlx trigger.dev@latest deploy"
	},
	"version": "0.0.0"
}
packages/tasks/tsconfig.json

{
	"extends": "@repo/tsconfig/base.json",
	"include": ["**/*.ts"],
	"exclude": ["dist", "build", "node_modules"]
}
packages/tasks/trigger.config.ts

import { additionalPackages } from "@trigger.dev/build/extensions/core";
import { prismaExtension } from "@trigger.dev/build/extensions/prisma";
import { defineConfig } from "@trigger.dev/sdk/v3";
 
export default defineConfig({
	project: "your_project_id",
	runtime: "node",
	tsconfig: "./tsconfig.json",
	logLevel: "log",
	maxDuration: 300,
	dirs: ["./trigger"],
	build: {
		extensions: [
			additionalPackages({ packages: ["zod-prisma-types"] }),
			prismaExtension({
				schema: "../database/prisma/schema.prisma",
				clientGenerator: "client",
				typedSql: true,
				directUrlEnvVarName: "DATABASE_URL_UNPOOLED",
			}),
		],
		external: [
			"@react-email/render",
			"@react-email/components",
			"react-dom",
			"react",
		],
	},
});
2. Create your first task
Now to create your first task, create a new file test-task.ts in the packages/tasks/trigger directory and add the following code:

packages/tasks/trigger/test-task.ts

import { task } from "@trigger.dev/sdk/v3";
 
export const testTask = task({
	id: "test-task",
	run: async () => {
		console.log("test task");
	},
});
Read the trigger.dev documentation for more information on how to create tasks or cron jobs.

3. Test your task
You can easily test your task locally by running pnpm --filter tasks dev from the root of your repository.

This will deploy your task to trigger.dev in the development environment, so you can trigger it from there. When you run the task in the development environment, it will be executed on your local machine.

4. Deploy your task
To deploy your task to trigger.dev, run pnpm --filter tasks deploy from the root of your repository.

You can also add this command as an automated deployment step in your CI/CD pipeline, by creating a new github action.

You need to add the TRIGGER_ACCESS_TOKEN secret to your repositories secrets, which you can create in the trigger.dev dashboard.

.github/workflows/trigger-deploy.yml

name: Deploy to trigger.dev (prod)
 
on:
  push:
    branches:
      - main
 
jobs:
  deploy:
    runs-on: ubuntu-latest
 
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: lts/*
      - uses: pnpm/action-setup@v4
      - name: Install dependencies
        run: pnpm install
      - name: Deploy trigger tasks
        env:
          TRIGGER_ACCESS_TOKEN: ${{ secrets.TRIGGER_ACCESS_TOKEN }}
        run: |
          pnpm --filter tasks deploy
5. Programmatically trigger a task
If you want to trigger one of your tasks from your application, for example as an action from an API call, you can import the task to your API resolver and trigger it.

Make sure to add the @repo/tasks package to your api package as a dependency.

api/actions/test-task.ts

import { testTask } from "@repo/tasks/trigger/test-task";
 
import { logger } from "@repo/logs";
import { sendEmail } from "@repo/mail";
import { Hono } from "hono";
import { describeRoute } from "hono-openapi";
import { resolver, validator } from "hono-openapi/zod";
import { z } from "zod";
import { localeMiddleware } from "../middleware/locale";
 
export const newsletterRouter = new Hono().basePath("/newsletter").post(
	"/signup",
 // ...
	async (c) => {
  // ...
 
        await testTask.trigger();
 
        // ...
	},
);
Previous

Overview

Next

Upstash QStash

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

