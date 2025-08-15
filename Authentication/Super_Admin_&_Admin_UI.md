Super Admin & Admin UI
supastarter comes with an admin role and a UI for managing users and organizations out of the box. The only thing you need to do is to create a new user and assign it the admin role.

Create admin user via CLI
You can use a simple CLI script to create a new admin user in the database:


pnpm --filter scripts create:user
Enter the email, the name and select the role Admin to create a new admin user:



You will see an automatically generated password in the terminal, which you can use to login to your application.

Assign admin role to existing user in the database
If you have already created a user and want to make it an admin, but don't have another admin account yet to change the role in the application UI, you can simply change the database entry of that user. To assign the admin role, first start Prisma Studio by running the following command in your terminal:


pnpm --filter database studio
A new browser window will open with Prisma Studio. Select the User table and find the user you just createed. Then, click on the role field and enter admin. Lastly hit the button saying Save 1 change to save the new role.

Access the Admin UI in your application
Now, when you log in with the admin user, you will see a new item in the navigation called Admin:



Previous

oAuth

Next

Overview

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




