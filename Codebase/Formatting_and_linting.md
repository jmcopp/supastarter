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
