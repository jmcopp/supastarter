# Resend Email Provider

Learn how to use Resend for sending emails in supastarter's FastAPI backend.

## Setup

### 1. Create Resend Account

1. Sign up at [resend.com](https://resend.com)
2. Verify your domain
3. Get your API key from the dashboard

### 2. Configure Environment

**File**: `api-main/.env`

```bash
EMAIL_PROVIDER=resend
EMAIL_FROM=hello@yourdomain.com
RESEND_API_KEY=re_123456789...
```

### 3. Install Dependencies

**File**: `api-main/requirements.txt`

```
aiohttp>=3.8.0
```

## Resend Provider Implementation

**File**: `api-main/app/services/providers/resend.py`

```python
import aiohttp
from typing import List, Optional, Dict
from app.services.email import EmailProvider
from app.core.config import settings
import logging

logger = logging.getLogger(__name__)

class ResendProvider(EmailProvider):
    """Resend email provider implementation."""
    
    def __init__(self):
        self.api_key = settings.RESEND_API_KEY
        self.base_url = "https://api.resend.com/emails"
        
        if not self.api_key:
            raise ValueError("RESEND_API_KEY not configured")
    
    async def send(
        self,
        to: List[str],
        subject: str,
        html: str,
        text: Optional[str] = None
    ) -> bool:
        """Send email via Resend API."""
        
        async with aiohttp.ClientSession() as session:
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            data = {
                "from": settings.EMAIL_FROM,
                "to": to,
                "subject": subject,
                "html": html
            }
            
            if text:
                data["text"] = text
            
            if settings.EMAIL_REPLY_TO:
                data["reply_to"] = settings.EMAIL_REPLY_TO
            
            try:
                async with session.post(
                    self.base_url,
                    json=data,
                    headers=headers
                ) as response:
                    if response.status == 200:
                        result = await response.json()
                        logger.info(f"Email sent successfully. ID: {result.get('id')}")
                        return True
                    else:
                        error = await response.text()
                        logger.error(f"Resend API error: {error}")
                        return False
                        
            except Exception as e:
                logger.error(f"Failed to send email via Resend: {str(e)}")
                return False
```

## Domain Configuration

Add Resend DNS records in your domain provider:

- **SPF**: `v=spf1 include:amazonses.com ~all`
- **DKIM**: Provided by Resend
- **DMARC**: `v=DMARC1; p=quarantine`

Verify in Resend dashboard
