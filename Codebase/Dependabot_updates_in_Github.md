Dependabot updates in Github
Learn how to use Dependabot updates in Github to keep your dependencies up to date.

Per default, supastarter has a setup for Dependabot in Github. It's a tool that will automatically check for updates of your dependencies and create a pull request for you to merge.

You can find the Dependabot configuration in the .github/dependabot.yml file.

Dependabot configuration
We have configured Dependabot to check for updates daily and create up to 20 pull requests.

We have also grouped some dependencies together, so that Dependabot will only create a single pull request for each group like all prisma dependencies or react dependencies.


version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    open-pull-requests-limit: 20
    schedule:
      interval: "daily"
    groups:
      content-collections:
        patterns:
          - "@content-collections/*"
      prisma:
        patterns:
          - "@prisma/*"
          - "prisma"
      radix-ui:
        patterns:
          - "@radix-ui/*"
      next-intl:
        patterns:
          - "next-intl"
          - "use-intl"
      react:
        patterns:
          - "react"
          - "react-dom"
          - "@types/react"
          - "@types/react-dom"
Dependabot pull requests
When Dependabot creates a pull request, it will include a summary of the changelog of the dependencies that are being updated.

The PR will also run the linter and formatter as well as the tests to make sure that the codebase is still in a working state after the update.

If you have Vercel connected to your repository, it will also deploy the changes to the staging environment, so you can test the changes before merging them to the main branch.

It is recommended always preview and test the changes in the staging environment before merging the PR to the main branch to not break the application.

Deactivate Dependabot
If you don't want to use Dependabot (which we do not recommend, as you want to stay up to date with the latest package versions), you can deactivate it by simply deleting the .github/dependabot.yml file.

If you want to reactivate it later, you can simply add it back.
