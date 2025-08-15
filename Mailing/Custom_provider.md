# Custom Email Provider

Learn how to create a custom email provider in supastarter's FastAPI backend.

## Creating a Custom Provider

You can implement any email service by creating a custom provider that extends the EmailProvider interface.

### Provider Interface

**File**: `api-main/app/services/email.py`

```python
from abc import ABC, abstractmethod
from typing import List, Optional

class EmailProvider(ABC):
    @abstractmethod
    async def send(
        self,
        to: List[str],
        subject: str,
        html: str,
        text: Optional[str] = None
    ) -> bool:
        pass
```

### Custom Provider Implementation

**File**: `api-main/app/services/providers/custom.py`

```python
import aiohttp
from typing import List, Optional
from app.services.email import EmailProvider
from app.core.config import settings
import logging

logger = logging.getLogger(__name__)

class CustomProvider(EmailProvider):
    """Custom email provider implementation."""
    
    def __init__(self):
        self.api_key = settings.CUSTOM_EMAIL_API_KEY
        self.base_url = settings.CUSTOM_EMAIL_BASE_URL
        
        if not self.api_key:
            raise ValueError("CUSTOM_EMAIL_API_KEY not configured")
    
    async def send(
        self,
        to: List[str],
        subject: str,
        html: str,
        text: Optional[str] = None
    ) -> bool:
        """Send email via custom API."""
        
        async with aiohttp.ClientSession() as session:
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            # Customize payload format for your provider
            data = {
                "from": settings.EMAIL_FROM,
                "to": to,
                "subject": subject,
                "html_body": html
            }
            
            if text:
                data["text_body"] = text
            
            try:
                async with session.post(
                    f"{self.base_url}/send",
                    json=data,
                    headers=headers
                ) as response:
                    if response.status in [200, 201, 202]:
                        result = await response.json()
                        logger.info(f"Email sent via custom provider. ID: {result.get('id')}")
                        return True
                    else:
                        error = await response.text()
                        logger.error(f"Custom provider API error: {error}")
                        return False
                        
            except Exception as e:
                logger.error(f"Failed to send email via custom provider: {str(e)}")
                return False
```
 
export const send: SendEmailHandler = async ({ to, subject, text, html }) => {
  // handle your custom email sending logic here
};
Then, make sure to export your custom provider in the /api-main/app/services/src/provider/index.ts:


export * from './custom';
