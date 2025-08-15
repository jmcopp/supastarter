Store data for organizations
Learn how to store data for organizations in your supastarter application.

When working with organizations, you most likely want to store data for each organization that can be accessed by the organization members.

We are re-using the post example from the API documentation and extend it to store the organiaztion with it. Let's assume you want to enable all members of an organization to edit the posts of the organization instead of just the author.

You probably still want the individual author to be defined on the post, so you can still see who created the post.

Adjust the database schema
To do this, you can add the organization_id field in the post model using SQLModel.

api/app/models/post.py

```python
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List
import uuid
from datetime import datetime

class PostBase(SQLModel):
    title: str
    content: str
    organization_id: str

class Post(PostBase, table=True):
    id: str = Field(default_factory=lambda: str(uuid.uuid4()), primary_key=True)
    author_id: str = Field(foreign_key="user.id")
    organization_id: str = Field(foreign_key="organization.id")
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    
    # Relationships
    author: Optional["User"] = Relationship(back_populates="posts")
    organization: Optional["Organization"] = Relationship(back_populates="posts")

class PostCreate(PostBase):
    pass

class PostResponse(PostBase):
    id: str
    author_id: str
    created_at: datetime
    updated_at: datetime
```
Add organization_id to the post creation endpoint
To create a post now, you need to pass the organization_id to the endpoint.

api/app/routers/posts.py

```python
from fastapi import APIRouter, HTTPException, Depends
from sqlmodel import Session, select
from app.core.auth import get_current_user
from app.core.database import get_session
from app.models.user import User
from app.models.post import Post, PostCreate, PostResponse
from app.models.organization import Organization, OrganizationMember
from typing import List

router = APIRouter(prefix="/posts", tags=["posts"])

@router.post("/", response_model=PostResponse)
async def create_post(
    post_data: PostCreate,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """
    Create a new post within an organization.
    Requires user to be a member of the organization.
    """
    
    # Verify that the user is a member of the organization
    membership_query = select(OrganizationMember).where(
        OrganizationMember.organization_id == post_data.organization_id,
        OrganizationMember.user_id == current_user.id
    )
    membership = session.exec(membership_query).first()
    
    if not membership:
        raise HTTPException(
            status_code=403, 
            detail="You are not a member of this organization"
        )
    
    # Optionally check the role of the user
    # if membership.role not in ["admin", "member"]:
    #     raise HTTPException(status_code=403, detail="Insufficient permissions")
    
    post = Post(
        title=post_data.title,
        content=post_data.content,
        author_id=current_user.id,
        organization_id=post_data.organization_id
    )
    
    session.add(post)
    session.commit()
    session.refresh(post)
    
    return post
```
This endpoint is protected by authentication which means only authenticated users can access it. We're checking if the user is a member of the organization using the OrganizationMember model. If the user is not a member, the function will throw an HTTP 403 error.

Create a post from the UI
To create a post from the UI, create a custom hook or use a direct API call.

```tsx
export const useCreateOrganizationPostMutation = () => {
    return useMutation({
        mutationKey: ["organization", "create-posts", "create"],
        mutationFn: async (data: { title: string; content: string; organization_id: string }) => {
            const response = await fetch('/api/posts/', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}` // Use your auth token here
                },
                body: JSON.stringify(data)
            });
 
            if (!response.ok) {
                throw new Error("Failed to create post");
            }
            
            return response.json();
        },
    });
};
```
When using this mutation, you need to pass the organizationId that you can get from the useActiveOrganization hook. Make sure to only call this mutation when there is an active organization.


const { activeOrganization } = useActiveOrganization();
const createOrganizationPostMutation = useCreateOrganizationPostMutation();
 
const onSubmit = async (data) => {
	if (!activeOrganization) {
		throw new Error("No active organization found");
	}
 
	await createOrganizationPostMutation.mutateAsync({
		...data,
		organization_id: activeOrganization.id,
	});
}
Add organization_id to the post query endpoint
Now you probably want to query the posts of an organization to list them in the UI. To do this, you can add the organization_id to the endpoint which lists all posts.

```python
@router.get("/", response_model=List[PostResponse])
async def get_posts(
    organization_id: str,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """
    Get all posts for an organization.
    Requires user to be a member of the organization.
    """
    
    # Always verify that the user is a member of the organization
    membership_query = select(OrganizationMember).where(
        OrganizationMember.organization_id == organization_id,
        OrganizationMember.user_id == current_user.id
    )
    membership = session.exec(membership_query).first()
    
    if not membership:
        raise HTTPException(
            status_code=403,
            detail="You are not a member of this organization"
        )
    
    posts_query = select(Post).where(Post.organization_id == organization_id)
    posts = session.exec(posts_query).all()
    
    return posts
```
This endpoint will fetch all posts of the organization and return them as a JSON response.

Query posts by organization in the UI
Lastly, to query the posts by organization in the UI, you can create a query that takes an organization_id:

```tsx
export const organizationPostsQueryKey = (organizationId?: string) => 
    ["organization", organizationId, "posts"] as const;

export const useOrganizationPostsQuery = (organizationId?: string) => {
    return useQuery({
        queryKey: organizationPostsQueryKey(organizationId),
        queryFn: async () => {
            const response = await fetch(`/api/posts/?organization_id=${organizationId}`, {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${token}` // Use your auth token here
                }
            });
 
            if (!response.ok) {
                throw new Error("Failed to fetch posts");
            }
 
            return response.json();
        },
        enabled: !!organizationId // Only run query if organization_id is provided
    });
};
```
Now inside your component, you can use the useOrganizationPostsQuery hook to fetch the posts of an organization.


const { data: posts, isPending } = useOrganizationPostsQuery(organizationId);
 
if (isPending) return <div>Loading...</div>;
 
if (!posts?.length) return <div>No posts found</div>;
 
return <div>{posts.map((post) => <div key={post.id}>{post.title}</div>)}</div>;
Previous

Use organizations in your application

Next

Overview

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




