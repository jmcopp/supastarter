Update schema & migrate changes
Learn how to update your database schema and migrate changes with Prisma in supastarter.

Prisma
To update your database schema, you can simply edit the schema.prisma file in the /packages/database library. You can find more information about the Prisma schema file here.

To add an entity for "posts" for example, you can add the following to your schema:


model Post {
  id        String      @id @default(cuid())
  title     String
  content   String
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
Migrate changes (optional)
If you want to to keep a migration history of your database changes, you can run the following command to create a new migration:


pnpm --filter database migrate
Push your changes to the database
To push your changes to the database, you can run the following command:


pnpm --filter database push
Generate prisma client
After you have pushed your schema changes, you need to re-generate the prisma client. You can do this by running the following command:

This command will automatically be run when you start the development server.


pnpm --filter database generate
Drizzle
To update your database schema, you can simply edit the packages/database/drizzle/schema/[postgres|mysql|sqlite].ts file. You can find more information about the Drizzle schema file in the official documentation.

Migrate changes (optional)
To migrate your changes, you can run the following command:


pnpm --filter database migrate
Push your changes to the database
To push your changes to the database, you can run the following command:


pnpm --filter database push
Previous

Use database studio

Next

Overview

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

