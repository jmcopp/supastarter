Troubleshooting
Find answers to questions that have been asked by other developers and might help you too.

My environment variables from .env.local are not being loaded
Make sure you are running the pnpm dev command from the root directory of your project (where the pnpm-workspace.yaml file is located)

supastarter uses the dotenv-cli to load environment variables from a .env.local file. The dotenv-cli is automatically used when running the pnpm dev command from the root directory.

Also make sure that the environment variable you are trying to access in your application is listed in the globalEnv object in the turbo.json file. Only then will turbo make the environment variable available to the runtime.

turbo.json

{
  "globalEnv": {
    "NEXT_PUBLIC_SUPABASE_URL": "https://your-supabase-url.supabase.co"
  }
}
How do I use the debugger in VSCode with supastarter?
To use the debugger in VSCode with supastarter, you need to add a new configuration to your .vscode/launch.json file:


{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Server DEBUG",
      "runtimeExecutable": "pnpm",
      "runtimeArgs": ["run", "dev"],
      "restart": true,
      "console": "integratedTerminal",
      "cwd": "${workspaceFolder}/apps/web",
      "envFile": "${workspaceFolder}/.env.local"
    }
  ]
}
The changes in the tailwind theme configuration are not being applied
There is a known issue with Tailwind CSS in monorepos which requires you to restart the development server after changing the theme configuration. If this does not work, try also saving the globals.css file in the web package and do a hard refresh in the browser.

My application is very slow in production
The most common reason for a slow application in production is the physical distance between the server or serverless functions and the database.

Make sure to deploy your application to a region that is close to your database. For example when you are using Supabase and Vercel, you can select the region of the Supabase instance in the setup process and the region of the Vercel serverless functions can be selected in the project settings under the Functions tab.



I'm getting a SSL routines
If you are deploying your app to platforms like DigitalOcean or fly.io, you might get a SSL error like ERR_SSL_PACKET_LENGTH_TOO_LONGor ERR_SSL_WRONG_VERSION_NUMBER.

This is because of the use of req.nextUrl.origin in the middleware.ts file, which is not working as expected on these platforms.

The simple fix is to replace all occurrences of req.nextUrl.origin with getBaseUrl() in the middleware.ts and the middleware-helpers.ts file:

apps/web/middleware.ts

import { getBaseUrl } from "@repo/utils"; 
 
export default function middleware(req: NextRequest) {
  const { pathname, origin } = req.nextUrl;
  const baseUrl = getBaseUrl();
 
  if (req.nextUrl.pathname === "/") {
    return NextResponse.redirect(new URL("/", origin)); 
    return NextResponse.redirect(new URL("/", baseUrl)); 
  }
}
middleware-helpers.ts

import { getBaseUrl } from "@repo/utils"; 
 
export const getSession = async (req: NextRequest): Promise<Session | null> => {
	const response = await fetch(
		new URL(
			"/api/auth/get-session?disableCookieCache=true",
			req.nextUrl.origin, 
			getBaseUrl(), 
		),
	);
 
 // ...
};
Previous

Configuration

Next

Project structure

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




