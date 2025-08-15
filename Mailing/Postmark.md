# Postmark Email Provider

Learn how to use Postmark for sending emails in supastarter's FastAPI backend.

## Setup

### 1. Create Postmark Account

1. Sign up at [postmarkapp.com](https://postmarkapp.com)
2. Create a new server
3. Get your server token from the dashboard
4. Verify your sending domain

### 2. Configure Environment

**File**: `api-main/.env`

```bash
EMAIL_PROVIDER=postmark
EMAIL_FROM=hello@yourdomain.com
POSTMARK_SERVER_TOKEN=your-server-token
```

### 3. Install Dependencies

**File**: `api-main/requirements.txt`

```
aiohttp>=3.8.0
```

## Postmark Provider Implementation

**File**: `api-main/app/services/providers/postmark.py`

```python
import aiohttp
from typing import List, Optional, Dict
from app.services.email import EmailProvider
from app.core.config import settings
import logging

logger = logging.getLogger(__name__)

class PostmarkProvider(EmailProvider):
    """Postmark email provider implementation."""
    
    def __init__(self):
        self.server_token = settings.POSTMARK_SERVER_TOKEN
        self.base_url = "https://api.postmarkapp.com/email"
        
        if not self.server_token:
            raise ValueError("POSTMARK_SERVER_TOKEN not configured")
    
    async def send(
        self,
        to: List[str],
        subject: str,
        html: str,
        text: Optional[str] = None
    ) -> bool:
        """Send email via Postmark API."""
        
        async with aiohttp.ClientSession() as session:
            headers = {
                "X-Postmark-Server-Token": self.server_token,
                "Content-Type": "application/json"
            }
            
            data = {
                "From": settings.EMAIL_FROM,
                "To": ",".join(to),
                "Subject": subject,
                "HtmlBody": html
            }
            
            if text:
                data["TextBody"] = text
            
            if settings.EMAIL_REPLY_TO:
                data["ReplyTo"] = settings.EMAIL_REPLY_TO
            
            try:
                async with session.post(
                    self.base_url,
                    json=data,
                    headers=headers
                ) as response:
                    if response.status == 200:
                        result = await response.json()
                        logger.info(f"Email sent successfully. ID: {result.get('MessageID')}")
                        return True
                    else:
                        error = await response.text()
                        logger.error(f"Postmark API error: {error}")
                        return False
                        
            except Exception as e:
                logger.error(f"Failed to send email via Postmark: {str(e)}")
                return False
```
Then, make sure to activate Postmark in the /api-main/app/services/src/provider/index.ts:


export * from './postmark';
To customize the provider, edit the api-main/app/services/src/provider/postmark/index.ts.

For further instructions, follow the general mail instructions.
