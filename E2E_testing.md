E2E testing
Learn how to write e2e tests with Playwright for your supastarter app.

For E2E testing supastarter integrates a basic setup of Playwright.

Why Playwright

Playwright is a great open-source tool for e2e testing. It's easy to use and has a great developer experience. It's fast, mature and reliable. You can also use it for component (unit) testing.

Running e2e tests
To run the e2e tests in your local environment, run the following command:


pnpm --filter web e2e
This will start up the development server of your application (if it's not already running) and also open the Playwright UI.

In the UI you can see all available tests and run them.



Running e2e tests in CI
supastarter has a GitHub action integrated by default to run the e2e tests in CI. It will automtically run on every PR to test your changes, but for it work correctly you need to provide the necessary secrects / envs to your Github action workflow.

To do this, open your Github repository and navigate to Settings > Secrets and variables > Actions. There you need to add the following secrets:

DATABASE_URL: The database url of your application. You can create a different database to run test against if you like.
Depending on your applications setup you also need to provide further envs, just like they are defined in your .env.local file.

For example when you are using Stripe, you need to provide the following secrets:

STRIPE_SECRET_KEY: Your secret key for the Stripe api
Writing e2e tests
The e2e tests are located in the frontend/tests folder. To create a new test suite, create a new *.spec.ts file in this folder:


import { expect, test } from "@playwright/test";
 
test.describe("some feature", () => {
  test("should do something", async ({ page }) => {
    await page.goto("/app/feature-page");
 
    await expect(page.getByRole("heading", { name: "Amazing feature" })).toBeVisible();
  });
});
Playwright will then automatically pick up the test suite and run it.

Learn more about writing e2e tests in the Playwright documentation.
