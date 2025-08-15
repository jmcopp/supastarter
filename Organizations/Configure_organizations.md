Configure organizations
Learn how to configure organizations in your supastarter application.

Disable organizations
To disable organizations, you can set the organizations.enabled flag to false in the FastAPI configuration. This will completely disable organizations and all related API endpoints.

api/app/core/config.py

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # Organization configuration
    organizations_enabled: bool = True
    organizations_require_organization: bool = False
    organizations_hide_organization: bool = False
    organizations_enable_users_to_create: bool = True
    
    # Authentication
    auth_enable_signup: bool = True
    
    class Config:
        env_file = ".env"

settings = Settings()
```
Require an organization
To require an organization for a user to be able to access the application, set the `organizations_require_organization` flag to true. This will ensure users must be part of an organization to access the application.

```python
# In your .env file
ORGANIZATIONS_REQUIRE_ORGANIZATION=true
```

Hide organization selection
You can hide the organization selection in the UI by setting the `organizations_hide_organization` flag to true. This is useful for building multi-tenant applications where users should only be members of one organization.

```python
# In your .env file
ORGANIZATIONS_HIDE_ORGANIZATION=true
```

Disable organization creation
If users should be able to join multiple organizations but not create new ones, set the `organizations_enable_users_to_create` flag to false.

```python
# In your .env file
ORGANIZATIONS_ENABLE_USERS_TO_CREATE=false
```

Invite-only organizations
To create an invite-only organization setup, disable user signup:

```python
# In your .env file
AUTH_ENABLE_SIGNUP=false
```
