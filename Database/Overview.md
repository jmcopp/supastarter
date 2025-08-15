Overview
Learn how to use the database in supastarter.

With supastarter you can choose between Prisma and Drizzle as your database ORM.

You can find the schema and configuration in the packages/database directory either in the prisma or drizzle folder.

Update schema
Use database client
Use database studio
Configure the ORM
Prisma
You can skip this if you are using Prisma with PostgreSQL, as this is the default setup.

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
Previous

Local Development

Next

Use database client

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




