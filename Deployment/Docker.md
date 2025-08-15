Docker
Learn how to deploy your supastarter application as a Docker container.

In this guide we are going to show you how to deploy your supastarter application as a Docker container, so you can deploy it to any platform or server that supports docker images.

You can find a working example of this on the feature/docker-deploy branch in the supastarter repository.

Why deploy your SaaS as a Docker container?
Deploying your SaaS as a Docker container allows you to have complete control over your server environment. It ensures better privacy, contributes to cost savings if managed correctly, and offers you the flexibility to customize your server setup to suit your specific needs. It can also give your application a performance boost compared to hosting on a serverless platform like Vercel, as it removes cold starts.

Setup Next.js app for docker deployment
To get started we first need to configure our Next.js app to be built as a standalone app, so we can later run the application in a docker container. To do so, add the following next.config.ts file to your project:

apps/web/next.config.ts

const nextConfig: NextConfig = {
  //...
  output: "standalone",
};
Setup docker configuration for the Next.js app
Next, create a Dockerfile in the root of your Next.js app (apps/web/Dockerfile) with the following content:


FROM node:22-alpine AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable
 
FROM base AS builder
RUN apk add --no-cache libc6-compat
RUN apk update
WORKDIR /app
RUN pnpm add -g turbo
COPY . .
RUN turbo prune @repo/web --docker
 
FROM base AS installer
RUN apk add --no-cache libc6-compat
RUN apk update
WORKDIR /app
 
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
ENV CYPRESS_INSTALL_BINARY=0
RUN pnpm install
 
COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json
 
RUN pnpm turbo run build --filter=@repo/web...
 
FROM base AS runner
WORKDIR /app
 
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs
 
COPY --from=installer /app/apps/web/next.config.ts .
COPY --from=installer /app/apps/web/package.json .
 
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public
 
USER nextjs
 
EXPOSE 3000
 
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"
 
CMD ["node", "apps/web/server.js"]
In the root of the supastarter project, also add a .dockerignore file with the following content:


Dockerfile
.dockerignore
node_modules
**/node_modules
npm-debug.log
README.md
.next
.git
And that's all the changes necessary! Now you can build your app as a docker image and deploy it anywhere you can run docker images.

Running your app locally with Docker
If you have Docker installed on your local machine and want to run your Next.js app there for testing the docker image, simply run the following commands from your project’s root:


docker build -f apps/web/Dockerfile . --no-cache -t supastarter-docker
docker run -p 3000:3000 supastarter-docker
To fully use the application, you need to pass the necessary environment variables you have defined in your .env.local file to the container.

Now you can deploy your app to any server that supports docker images.

Troubleshooting
If you are getting a SSL error like ERR_SSL_PACKET_LENGTH_TOO_LONG or ERR_SSL_WRONG_VERSION_NUMBER, see our troubleshooting guide for a quick fix.

Previous

Netlify

Next

Coolify

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

