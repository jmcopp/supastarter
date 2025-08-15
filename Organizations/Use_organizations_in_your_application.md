Use organizations in your application
Learn how to use organizations in your supastarter application.

In the supastarter application, the active organization is determined from the organizationSlug path parameter of the URL.

Whenever you are on a path that starts with /app/my-organization, the organization my-organization is active.

There is another common pattern to store the active organization in the session of the user, but we decided to use the path parameter as it enables a few benefits:

It's easier to understand and reason about the active organization from the URL
You can easily share a link to a specific organization or some organization page
You can easily navigate to a different organization by changing the path parameter
You can use different organizations in the same browser session in different tabs
To use the active organization in your application, you can use the useActiveOrganization hook.

It provides the following properties:

activeOrganization: The active organization
setActiveOrganization: A function to set the active organization by the organization id
loaded: A boolean indicating if the active organization has been loaded yet
isOrganizationAdmin: A boolean indicating if the user is an admin or the owner of the active organization
refetchActiveOrganization: A function to refetch the active organization
Usually when you are inside a page that is part of the /app/[organizationSlug] path, you the active organization is already loaded and should be set.


const { activeOrganization, setActiveOrganization, loaded, isOrganizationAdmin, refetchActiveOrganization } = useActiveOrganization();
 
// Check if the active organization is loaded
if (!loaded) {
  return <div>Loading...</div>;
}
 
// Check if the active organization is loaded
if (!activeOrganization) {
  return <div>No active organization found</div>;
}
 
// Check if the user is an admin of the active organization
if (!isOrganizationAdmin) {
  return <div>You are not an admin of the active organization</div>;
}
 
// Refetch the active organization
await refetchActiveOrganization();
Get active organization on server side
To get the active organization on the server side, you can use the getActiveOrganization function and pass it the organization slug from the url.


export default async function OrganizationPage({
	children,
	params,
}: PropsWithChildren<{
	params: Promise<{
		organizationSlug: string;
	}>;
}>) {
	const { organizationSlug } = await params;
	const organization = await getActiveOrganization(organizationSlug);
 
	return <div>Active organization: {organization.name}</div>;
}
Get organization without slug
In some cases you want to get organization data without a slug being present in the URL.

In general the user can access the organization data for the organizations they are a member of.

To get the organization data for a specific organization, you can use the authClient on client side:


const { data, error } = await authClient.organization.getFullOrganization(
	{
		query: {
			organizationId: '1234lasjdfwlj34l', // replace with the organization id
		},
	},
);
On server side you can use the auth class to get the organization data. Make sure to pass the headers to the request, as the cookies are not automatically attached to the request.


const organization = await auth.api.getFullOrganization({
	headers: await headers(),
	query: {
		organizationId: '1234lasjdfwlj34l', // replace with the organization id
	},
});
To learn more about how to use organizations, please refer to the better-auth documentation.

Previous

Configure organizations

Next

Store data for organizations

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




