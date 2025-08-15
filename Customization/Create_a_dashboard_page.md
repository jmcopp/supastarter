Create a dashboard page
Learn how to create a dashboard page in supastarter.

1. Create a new page
For this example we are going to create the AI Demo page in the dashboard. We start by creating the new page in the /apps/web/app/(saas)/app folder. To do so, we need to create a subfolder with the name of the page (in this case ai-demo) and in there a page.tsx file.

apps/web/app/(saas)/app/ai-demo/page.tsx

export default function AiDemoPage() {
  return (
    <div>
      <h1>AI Demo</h1>
    </div>
  );
}
2. Add menu item for the new page
In the NavBar component we need to add a new menu item for the new page:

apps/web/modules/saas/shared/components/NavBar.tsx

const menuItems = [
  // ...
  {
    title: "AI Demo",
    href: "/app/ai-demo",
    icon: WandIcon,
  },
];
Now we can navigate to the new page by clicking on the menu item.

Previous

Styling the application

Next

Add a new onboarding step

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




