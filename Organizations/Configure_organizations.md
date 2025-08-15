Configure organizations
Learn how to configure organizations in your supastarter application.

Disable organizations
To disable organizations, you can set the organizations.enabled flag to false in the config/index.ts. This will completely disable organizations and all related pages in the application.


const config = {
  organizations: {
    enabled: false,
  },
};
Require an organization
To require an organization for a user to be able to access the application, you can set the organizations.requireOrganization flag to true in the config/index.ts. This will lead to the user being asked to create an organization after signing up, if he hasn't been invited to an organization yet.


const config = {
  organizations: {
    requireOrganization: true,
  },
};
Hide organization selection
You can hide the organization selection in the UI by setting the organizations.hideOrganization flag to true in the config/index.ts. This can be useful if you want to build a multi-tenant application where users should only be member of one organization.


const config = {
  organizations: {
    hideOrganization: true,
  },
};
Disable organization creation
If users should be able to be member of multiple organizations, but not to create new organizations, you can set the organizations.enableUsersToCreateOrganizations flag to false.


const config = {
  organizations: {
    enableUsersToCreateOrganizations: false,
  },
};
Invite-only organizations
To get an invite-only organization setup, you can disable the signup of users:


const config = {
  auth: {
    enableSignup: false,
  },
};
Previous

Overview

Next

Use organizations in your application

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

