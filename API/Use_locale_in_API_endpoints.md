# Use Locale in API Endpoints

Learn how to use the users locale in your supastarter API endpoints.

Sometimes you need to use the users locale in your API endpoints. For example, you might want to fetch data from a database that is localized or if you want to send an email in the users locale.

To make the users locale available in your FastAPI endpoint handler, you can create a dependency to extract the locale:

```python
from fastapi import APIRouter, Depends, Request
from app.core.auth import get_current_user

def get_user_locale(request: Request) -> str:
    """Extract user locale from request headers or cookies."""
    # Try to get locale from cookie first
    locale = request.cookies.get("locale")
    
    # If not found, try Accept-Language header
    if not locale:
        accept_language = request.headers.get("accept-language")
        if accept_language:
            # Extract first locale from Accept-Language header
            locale = accept_language.split(",")[0].split("-")[0]
    
    # Default to English if nothing found
    return locale or "en"

router = APIRouter(prefix="/email", tags=["email"])

@router.get("/send-localized-email")
async def send_localized_email(
    locale: str = Depends(get_user_locale),
    current_user: User = Depends(get_current_user)
):
    """Send an email in the user's locale."""
    # Use the locale for localized content
    # For example: translate email content, format dates, etc.
    
    return {"message": f"Email sent in locale: {locale}"}
```

This dependency will provide the locale from the locale cookie of the user, the Accept-Language header, or default to the application's default locale.
