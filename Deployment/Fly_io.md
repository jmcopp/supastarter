Fly.io
Learn how to deploy your supastarter application to Fly.io.

This guide will show you how to deploy your supastarter SaaS to fly.io in minutes. And the best part? You can actually start for free thanks to the generous free tier of fly.io.

Why host your SaaS on fly.io?
Self-hosting your SaaS app as a docker image allows you to have complete control over your server environment. It ensures better privacy, contributes to cost savings if managed correctly, and offers you the flexibility to customize your server setup to suit your specific needs. It can also give your application a performance boost compared to hosting on a serverless platform like Vercel, since it removes cold starts.

Get your supastarter app ready for Docker deployment
To get your supastarter app ready for Docker deployment, follow the steps in our Docker deployment guide.

Deploying your application to Fly.io
Make sure you have an account on Fly.io before continuing. You can sign up for free at fly.io.

To deploy your application with Docker to fly.io simply install the Fly CLI and run the following command from your project’s root:


fly launch --dockerfile apps/web/Dockerfile
Follow the steps in the CLI and configure the used port to 3000 in the deployment configuration and you should have a running instance of your app within a few minutes on fly.io.



Troubleshooting
If you are getting a SSL error like ERR_SSL_PACKET_LENGTH_TOO_LONG or ERR_SSL_WRONG_VERSION_NUMBER, see our troubleshooting guide for a quick fix.

Previous

Render

Next

Netlify

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




