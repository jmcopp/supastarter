# Plunk Email Provider

Learn how to use Plunk for sending emails in supastarter's FastAPI backend.

## Setup

### 1. Create Plunk Account

1. Sign up at [useplunk.com](https://useplunk.com)
2. Get your API key from the dashboard
3. Verify your sending domain

### 2. Configure Environment

**File**: `api-main/.env`

```bash
EMAIL_PROVIDER=plunk
EMAIL_FROM=hello@yourdomain.com
PLUNK_API_KEY=sk_...
PLUNK_PROJECT_ID=proj_...
```

### 3. Install Dependencies

**File**: `api-main/requirements.txt`

```
aiohttp>=3.8.0
```

## Plunk Provider Implementation

**File**: `api-main/app/services/providers/plunk.py`

```python
import aiohttp
from typing import List, Optional, Dict
from app.services.email import EmailProvider
from app.core.config import settings
import logging

logger = logging.getLogger(__name__)

class PlunkProvider(EmailProvider):
    """Plunk email provider implementation."""
    
    def __init__(self):
        self.api_key = settings.PLUNK_API_KEY
        self.project_id = settings.PLUNK_PROJECT_ID
        self.base_url = "https://api.useplunk.com/v1"
        
        if not self.api_key:
            raise ValueError("PLUNK_API_KEY not configured")
    
    async def send(
        self,
        to: List[str],
        subject: str,
        html: str,
        text: Optional[str] = None,
        metadata: Optional[Dict] = None
    ) -> bool:
        """Send email via Plunk API."""
        
        async with aiohttp.ClientSession() as session:
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            # Plunk expects single recipient per call
            for recipient in to:
                data = {
                    "to": recipient,
                    "subject": subject,
                    "body": html,
                    "from": settings.EMAIL_FROM,
                    "name": settings.APP_NAME
                }
                
                if text:
                    data["text"] = text
                
                if metadata:
                    data["metadata"] = metadata
                
                try:
                    async with session.post(
                        f"{self.base_url}/send",
                        json=data,
                        headers=headers
                    ) as response:
                        if response.status != 200:
                            error = await response.text()
                            logger.error(f"Plunk API error for {recipient}: {error}")
                            return False
                            
                except Exception as e:
                    logger.error(f"Failed to send email to {recipient}: {str(e)}")
                    return False
            
            logger.info(f"Emails sent successfully to {to}")
            return True
```

## Contact Management

### Create/Update Contacts

```python
class PlunkContactService:
    """Manage contacts in Plunk."""
    
    def __init__(self):
        self.api_key = settings.PLUNK_API_KEY
        self.base_url = "https://api.useplunk.com/v1"
    
    async def create_or_update_contact(
        self,
        email: str,
        properties: Dict,
        lists: Optional[List[str]] = None
    ) -> bool:
        """Create or update a contact in Plunk."""
        
        async with aiohttp.ClientSession() as session:
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            data = {
                "email": email,
                "properties": properties
            }
            
            if lists:
                data["lists"] = lists
            
            async with session.post(
                f"{self.base_url}/contacts",
                json=data,
                headers=headers
            ) as response:
                return response.status == 200
```

## Best Practices

- Use templates for consistent branding
- Segment contacts into targeted lists
- Track events for better automation
- Handle bounces to maintain list health
- Respect unsubscribes immediately
- Test emails before sending campaigns
- Monitor delivery rates and engagement
