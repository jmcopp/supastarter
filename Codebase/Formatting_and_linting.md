Formatting and linting
Learn how to format and lint your supastarter codebase.

supastarter uses Biome for formatting and linting.

Auto-format and fix code on save
Per default, Biome is configured to format and lint the codebase while you are developing.

If you want to disable this behavior, open the .vscode/settings.json file and set the following options:


{
  "editor.formatOnSave": false,
  "editor.codeActionsOnSave": {
    "quickfix.biome": "never",
    "source.organizeImports.biome": "never"
  }
}
Manual formatting and linting
To manually format your codebase run:


pnpm format
To manually lint your codebase run:


pnpm lint
To fix all the linting errors in your codebase run:


pnpm lint --fix
Previous

Manage dependencies

Next

Dependabot updates in Github

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

