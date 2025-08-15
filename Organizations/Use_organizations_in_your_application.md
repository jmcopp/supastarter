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

To get the organization data for a specific organization, you can call the FastAPI endpoint:

**Client side:**
```tsx
const getOrganization = async (organizationId: string) => {
    const response = await fetch(`/api/organizations/${organizationId}`, {
        method: 'GET',
        headers: {
            'Authorization': `Bearer ${token}` // Use your auth token here
        }
    });

    if (!response.ok) {
        throw new Error('Failed to fetch organization');
    }

    return response.json();
};
```

**Server side (FastAPI endpoint):**
```python
@router.get("/{organization_id}", response_model=OrganizationResponse)
async def get_organization(
    organization_id: str,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """
    Get organization data for a specific organization.
    User must be a member of the organization.
    """
    
    # Verify user is a member of the organization
    membership_query = select(OrganizationMember).where(
        OrganizationMember.organization_id == organization_id,
        OrganizationMember.user_id == current_user.id
    )
    membership = session.exec(membership_query).first()
    
    if not membership:
        raise HTTPException(status_code=403, detail="Access denied")
    
    org_query = select(Organization).where(Organization.id == organization_id)
    organization = session.exec(org_query).first()
    
    if not organization:
        raise HTTPException(status_code=404, detail="Organization not found")
    
    return organization
```

To learn more about how to use organizations with Supabase Auth, please refer to the Supabase documentation and the Authentication section of this guide.
