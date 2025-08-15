Setup
Learn how to setup your supastarter application.

This guide will walk you through setting up supastarter. We will go through the process of cloning the project, installing dependencies, setting up your database and running the local development server.

Prerequisites
Before you can get started, you will need to have the following installed on your machine.

Node.js (v20 or higher)
Git
pnpm
VSCode (recommended, or any other code editor)
Project setup
Create a new database
supastarter uses Prisma as an ORM (database access layer). This means you can use any database supported by Prisma, including PostgreSQL, MySQL and MongoDB. You can find all supported databases here.

Before creating a new supastarter project, make sure to have created a new database and have the connection string ready. For example when using PostgreSQL, the connection string will look something like this:


postgresql://user:password@host:port/database
Recommended database providers are Supabase, PlanetScale and Neon.

You can also find setup guides for certain providers in the supastarter blog:

Supabase
Initialize a new supastarter project
During the setup process you will be asked to provide the following information:

Project name - The name of your project
Database provider - The database provider you are using
Database connection string - The connection string of your database
To create a new supastarter project all you need to do is run the following command (replace my-awesome-project with the name of your project):


npx supastarter new my-awesome-project
This will clone and configure the supastarter repository, install all the dependencies and set up the database for you.

Manual setup
The following steps are only necessary if you encounter any errors during the setup process, otherwise you can skip to step 3.

Clone the supastarter repository

git clone https://github.com/supastarter/supastarter-nextjs.git
Install the dependencies
Make sure you have installed pnpm before running the following command:


pnpm install
Set up the environment variables
To do this, copy the .env.local.example file in the root of your project and rename it to .env.local.

Then open the .env.local file and set at least the DATABASE_URL:

.env.local

# Database
DATABASE_URL="YOUR_DATABASE_CONNECTION_STRING"
If you are using Supabase, you want to follow our Supabase setup guide as you will need a second connection string for the database migration.

Configure the database
Prisma
You can skip this step if you are using PostgreSQL.

If you are using a different database than PostgreSQL, you will need to configure the database provider in the /packages/database/prisma/schema.prisma file.

packages/database/prisma/schema.prisma

datasource db {
  provider = "mysql" // or "mongodb"
}
You will also need to change the database provider in the /packages/auth/auth.ts file:

packages/auth/auth.ts

export const auth = betterAuth({
	database: prismaAdapter(db, {
		provider: "mysql", // or "mongodb"
	}),
});
In case you are using MySQL, you will also need to remove all occurrences of mode: "insensitive" from the project. This is a feature that Prisma does not support for MySQL, so it will cause errors on build. Simply search in the project for mode: "insensitive" and remove it.

Drizzle
If you want to use Drizzle instead of Prisma, you will need to change the export in the /packages/database/index.ts file:

packages/database/index.ts

export * from "./drizzle";
Next make sure to use the drizzle adapter in the /packages/auth/auth.ts file:

packages/auth/auth.ts

export const auth = betterAuth({
	database: drizzleAdapter(db, {
		provider: "mysql", // or "sqlite" or "pg"
	}),
});
If you are using a different database than PostgreSQL, you will need to export the schema for either mysql or sqlite from the /packages/database/drizzle/schemaindex].ts file.

packages/database/drizzle/schema/index.ts

export * from "./postgres";
// or export * from "./mysql";
// or export * from "./sqlite";
Lastly, you will need to set up the correct database client in the client.ts file. Follow the instructions in the Drizzle documentation for the database provider you are using. For PostgreSQL you can use the following example:

packages/database/drizzle/client.ts

import { drizzle } from "drizzle-orm/node-postgres";
import * as schema from "./schema/postgres";
 
const databaseUrl = process.env.DATABASE_URL as string;
 
if (!databaseUrl) {
	throw new Error("DATABASE_URL is not set");
}
 
export const db = drizzle(databaseUrl, {
	schema,
});
Read more about how to use the ORMs in the database documentation.

Select your mail provider
Select your mail provider which is used for sending transactional emails like the magic link for login.

In the /packages/mail/src/providers/index.ts file, set the export to the provider you want to use:


export * from "./plunk";
// or export * from './resend';
// or export * from './postmark';
// or export * from './nodemailer';
Then make sure to set the mandatory environment variables for your provider in the .env.local file:

Plunk -> PLUNK_API_KEY
Resend -> RESEND_API_KEY
Postmark -> POSTMARK_SERVER_TOKEN
Nodemailer -> MAIL_HOST, MAIL_PORT, MAIL_USER, MAIL_PASS
If you want to use a custom mail provider, use the /packages/mail/src/providers/custom.ts file and export from custom.ts.

Set up the payment provider
Next, we need to set the payment provider which is used for handling payments::

packages/payments/provider/index.ts

export * from "./stripe";
// or export * from './lemonsqueezy';
// or export * from './creem';
// or export * from './polar';
Push and generate the database schema
Make sure to check your schema in packages/database/prisma/schema.prisma before continuing.

To push the database schema to your database, run the following command:


pnpm --filter database push
Then, to generate the Prisma client, run the following command:


pnpm --filter database generate
Set up the supastarter repository as the upstream origin
The last step is to set the supastarter repository as the upstream origin for your project, so you can pull in updates in the future.

Run the following commands:


rm -rf .git
git init
git remote add upstream https://github.com/supastarter/supastarter-nextjs.git
git add .
git commit -m "Initial commit"
Set up your storage provider
Storage is necessary to upload and serve files like images for example for the avatars of users and teams. Setting up a storage provider is optional, but recommended and necessary if you want to enable avatar and logo uploads.

supastarter supports all S3-compatible storage providers like AWS S3, DigitalOcean Spaces, MinIO, etc. and Supabase Storage.

Storage setup guide

Set up your local development environment (optional)
If you want to set up a complete local development environment, you can follow the local development guide.

Start your development server
Now your app should be ready to go. To start the local development server, navigate into your project root folder and run the following command.


pnpm dev
Open http://localhost:3000 in your browser to see the your app.

Create admin user
To use your application, you want to create an admin user. You can use a simple CLI script to create a new admin user in the database:


pnpm --filter scripts create:user
Enter the email, the name and select the role Admin to create a new admin user:



You will see an automatically generated password in the terminal, which you can use to login to your application.

Previous

Tech Stack

Next

Configuration

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

