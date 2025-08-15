Update the codebase
Learn how to update your codebase from the supastarter repository to get the latest updates and features.

Updating your code to the latest updates and features of supastarter is in general possible, we have to mention though that with every change you make to your application, it will become harder to update your code base, because you are essentially rebasing your code on top of the latest supastarter code with Git. That means that you will have to resolve merge conflicts and that you will have to be careful not to overwrite your own code.

Before you update your code base, make sure your git repository is clean and that you have no uncommitted changes.

Add the supastarter repository as a remote
To be able to update your code base, you need to add the supastarter repository as a remote to your git repository. You can do this by running the following command in your terminal:


git remote add upstream https://github.com/supastarter/supastarter-nextjs.git
If you have done this in the setup before, you can skip this step.

Update your repository from the supastarter repository

git pull upstream main --allow-unrelated-histories --rebase
If you have any merge conflicts, you will have to resolve them manually. If you are not sure how to do this, please refer to the Git documentation.

Previous

VS Code setup

Next

Manage dependencies

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




