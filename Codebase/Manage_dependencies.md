Manage dependencies
Learn how to manage dependencies in supastarter.

As the package manager we chose pnpm.

Why pnpm

It is a fast, disk space efficient package manager that uses hard links and symlinks to save one version of a module only ever once on a disk. It also has a great monorepo support.

Install a package
To install a package to the supastarter monorepo you need to decide whether you want to install it to the root of the monorepo or to a specific workspace. Installing it to the root makes it available to all packages, while installing it to a specific workspace makes it available only to that workspace.

To install a package globally, run:


pnpm add -w <package-name>
To install a package to a specific workspace, run:


pnpm add --filter <workspace-name> <package-name>
Here is a list of the workspaces of supastarter:

web
api
auth
database
mail
eslint-config-custom
tsconfig
tailwind
utils
Remove a package
Removing a package is the same as installing but with the remove command.

To remove a package globally, run:


pnpm remove -w <package-name>
To remove a package to a specific workspace, run:


pnpm remove --filter <workspace-name> <package-name>
Update a package
Updating is a bit easier since there is a nice way to update a package in all workspaces at once:


pnpm update -r <package-name>
Previous

Update the codebase

Next

Formatting and linting

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




