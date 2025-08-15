# SMTP Email Provider (Nodemailer)

Learn how to use SMTP for sending emails in supastarter's FastAPI backend.

## Setup

### 1. SMTP Server Configuration

Ensure you have access to an SMTP server (Gmail, Outlook, or custom SMTP).

### 2. Configure Environment

**File**: `api-main/.env`

```bash
EMAIL_PROVIDER=smtp
EMAIL_FROM=your-email@domain.com
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_USE_TLS=true
```

### 3. Install Dependencies

**File**: `api-main/requirements.txt`

```
aiosmtplib>=1.1.0
email-validator>=2.0.0
```

## SMTP Provider Implementation

**File**: `api-main/app/services/providers/smtp.py`

```python
import aiosmtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from typing import List, Optional
from app.services.email import EmailProvider
from app.core.config import settings
import logging

logger = logging.getLogger(__name__)

class SMTPProvider(EmailProvider):
    """SMTP email provider implementation."""
    
    def __init__(self):
        self.host = settings.SMTP_HOST
        self.port = settings.SMTP_PORT
        self.username = settings.SMTP_USER
        self.password = settings.SMTP_PASSWORD
        self.use_tls = settings.SMTP_USE_TLS
        
        if not all([self.host, self.username, self.password]):
            raise ValueError("SMTP configuration incomplete")
    
    async def send(
        self,
        to: List[str],
        subject: str,
        html: str,
        text: Optional[str] = None
    ) -> bool:
        """Send email via SMTP."""
        
        try:
            # Create message
            msg = MIMEMultipart('alternative')
            msg['From'] = settings.EMAIL_FROM
            msg['To'] = ', '.join(to)
            msg['Subject'] = subject
            
            # Add text part
            if text:
                text_part = MIMEText(text, 'plain', 'utf-8')
                msg.attach(text_part)
            
            # Add HTML part
            html_part = MIMEText(html, 'html', 'utf-8')
            msg.attach(html_part)
            
            # Send email
            await aiosmtplib.send(
                msg,
                hostname=self.host,
                port=self.port,
                username=self.username,
                password=self.password,
                use_tls=self.use_tls
            )
            
            logger.info(f"Email sent successfully via SMTP to {to}")
            return True
            
        except Exception as e:
            logger.error(f"Failed to send email via SMTP: {str(e)}")
            return False
```
MAIL_PORT="your-mail-port"
MAIL_USER="your-mail-user"
MAIL_PASS="your-mail-password"
Then, make sure to activate Nodemailer in the /api-main/app/services/src/provider/index.ts:


export * from './nodemailer';
To customize the provider, edit the api-main/app/services/src/provider/nodemailer/index.ts.

For further instructions, follow the general mail instructions.
