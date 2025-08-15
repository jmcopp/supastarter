# Mailing Overview

Learn how to send emails in supastarter's multi-container architecture.

## Architecture

Supastarter implements email functionality in the **FastAPI backend** with support for multiple email providers and template rendering.

## Email Stack

- **FastAPI** - Email service integration
- **Jinja2** - Email template rendering
- **Multiple Providers** - Resend, SendGrid, Postmark, SMTP
- **Async Support** - Non-blocking email sending

## Email Service Setup

### Core Email Service

**File**: `api-main/app/services/email.py`

```python
from typing import Dict, List, Optional
from abc import ABC, abstractmethod
import aiohttp
from jinja2 import Environment, FileSystemLoader
from app.core.config import settings
import logging

logger = logging.getLogger(__name__)

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

class ResendProvider(EmailProvider):
    def __init__(self):
        self.api_key = settings.RESEND_API_KEY
        self.base_url = "https://api.resend.com/emails"
    
    async def send(
        self,
        to: List[str],
        subject: str,
        html: str,
        text: Optional[str] = None
    ) -> bool:
        async with aiohttp.ClientSession() as session:
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            data = {
                "from": settings.EMAIL_FROM,
                "to": to,
                "subject": subject,
                "html": html,
                "text": text or ""
            }
            
            async with session.post(
                self.base_url,
                json=data,
                headers=headers
            ) as response:
                return response.status == 200

class EmailService:
    def __init__(self):
        # Select provider based on configuration
        if settings.EMAIL_PROVIDER == "resend":
            self.provider = ResendProvider()
        elif settings.EMAIL_PROVIDER == "sendgrid":
            self.provider = SendGridProvider()
        elif settings.EMAIL_PROVIDER == "postmark":
            self.provider = PostmarkProvider()
        else:
            self.provider = SMTPProvider()
        
        # Setup Jinja2 for templates
        self.template_env = Environment(
            loader=FileSystemLoader("app/templates/emails")
        )
    
    async def send_email(
        self,
        to: str | List[str],
        template: str,
        context: Dict,
        subject: Optional[str] = None
    ) -> bool:
        """Send email using template."""
        # Ensure to is a list
        to_list = [to] if isinstance(to, str) else to
        
        # Render template
        template_obj = self.template_env.get_template(f"{template}.html")
        html_content = template_obj.render(**context)
        
        # Get subject from template or use provided
        if not subject:
            subject = self._extract_subject(html_content)
        
        # Send email
        try:
            result = await self.provider.send(
                to=to_list,
                subject=subject,
                html=html_content
            )
            
            if result:
                logger.info(f"Email sent to {to_list} using template {template}")
            else:
                logger.error(f"Failed to send email to {to_list}")
            
            return result
        except Exception as e:
            logger.error(f"Email sending error: {str(e)}")
            return False
    
    def _extract_subject(self, html: str) -> str:
        """Extract subject from HTML title tag."""
        import re
        match = re.search(r'<title>(.*?)</title>', html)
        return match.group(1) if match else "No Subject"

# Global email service instance
email_service = EmailService()
```

## Email Templates

### Base Template

**File**: `api-main/app/templates/emails/base.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block subject %}{% endblock %}</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            line-height: 1.6;
            color: #333;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        .header {
            text-align: center;
            padding: 20px 0;
            border-bottom: 1px solid #e5e5e5;
        }
        .content {
            padding: 30px 0;
        }
        .footer {
            text-align: center;
            padding: 20px 0;
            border-top: 1px solid #e5e5e5;
            color: #666;
            font-size: 14px;
        }
        .button {
            display: inline-block;
            padding: 12px 24px;
            background-color: #4F46E5;
            color: white;
            text-decoration: none;
            border-radius: 6px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="header">
        <img src="{{ logo_url }}" alt="Logo" style="height: 40px;">
    </div>
    
    <div class="content">
        {% block content %}{% endblock %}
    </div>
    
    <div class="footer">
        <p>© 2024 {{ company_name }}. All rights reserved.</p>
        <p>
            <a href="{{ unsubscribe_url }}">Unsubscribe</a> |
            <a href="{{ privacy_url }}">Privacy Policy</a>
        </p>
    </div>
</body>
</html>
```

### Welcome Email Template

**File**: `api-main/app/templates/emails/welcome.html`

```html
{% extends "base.html" %}

{% block subject %}Welcome to {{ app_name }}!{% endblock %}

{% block content %}
<h2>Welcome, {{ name }}!</h2>

<p>Thank you for joining {{ app_name }}. We're excited to have you on board!</p>

<p>Here's what you can do next:</p>
<ul>
    <li>Complete your profile</li>
    <li>Explore our features</li>
    <li>Connect with other users</li>
</ul>

<center>
    <a href="{{ dashboard_url }}" class="button">Go to Dashboard</a>
</center>

<p>If you have any questions, feel free to reach out to our support team.</p>

<p>Best regards,<br>The {{ app_name }} Team</p>
{% endblock %}
```

## Configuration

### Environment Variables

**File**: `api-main/.env`

```bash
# Email Configuration
EMAIL_PROVIDER=resend  # or sendgrid, postmark, smtp
EMAIL_FROM=hello@yourdomain.com
EMAIL_REPLY_TO=support@yourdomain.com

# Provider-specific
RESEND_API_KEY=re_...
# or
SENDGRID_API_KEY=SG...
# or
POSTMARK_SERVER_TOKEN=...
# or
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-password
```

### Configuration Schema

**File**: `api-main/app/core/config.py`

```python
class Settings(BaseSettings):
    # Email settings
    email_provider: str = "resend"
    email_from: str = "hello@example.com"
    email_reply_to: Optional[str] = None
    
    # Provider credentials
    resend_api_key: Optional[str] = None
    sendgrid_api_key: Optional[str] = None
    postmark_server_token: Optional[str] = None
    
    # SMTP settings
    smtp_host: Optional[str] = None
    smtp_port: int = 587
    smtp_user: Optional[str] = None
    smtp_password: Optional[str] = None
    smtp_use_tls: bool = True
    
    class Config:
        env_file = ".env"
```

## Using Email Service

### Send Welcome Email

```python
from app.services.email import email_service

@router.post("/users/signup")
async def signup(user_data: UserCreate):
    # Create user logic...
    
    # Send welcome email
    await email_service.send_email(
        to=user.email,
        template="welcome",
        context={
            "name": user.name,
            "app_name": settings.APP_NAME,
            "dashboard_url": f"{settings.FRONTEND_URL}/dashboard",
            "company_name": settings.COMPANY_NAME,
            "logo_url": f"{settings.FRONTEND_URL}/logo.png",
            "unsubscribe_url": f"{settings.FRONTEND_URL}/unsubscribe",
            "privacy_url": f"{settings.FRONTEND_URL}/privacy"
        }
    )
    
    return {"message": "User created"}
```

### Send Password Reset Email

```python
@router.post("/auth/forgot-password")
async def forgot_password(email: str):
    # Generate reset token...
    
    await email_service.send_email(
        to=email,
        template="password_reset",
        context={
            "reset_url": f"{settings.FRONTEND_URL}/reset-password?token={token}",
            "expires_in": "1 hour"
        }
    )
    
    return {"message": "Reset email sent"}
```

## Localized Emails

### Multi-language Support

```python
async def send_localized_email(
    to: str,
    template: str,
    context: Dict,
    locale: str = "en"
):
    """Send email in user's language."""
    
    # Use localized template
    template_name = f"{template}_{locale}"
    
    # Add locale to context
    context["locale"] = locale
    
    # Load translations
    translations = load_translations(locale)
    context.update(translations)
    
    await email_service.send_email(
        to=to,
        template=template_name,
        context=context
    )
```

## Testing Emails

### Development Email Preview

```python
# api-main/scripts/preview_emails.py
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/preview/{template}")
async def preview_email(template: str):
    """Preview email template in browser."""
    
    from app.services.email import email_service
    
    # Sample context
    context = {
        "name": "John Doe",
        "app_name": "Supastarter",
        # ... other context
    }
    
    template_obj = email_service.template_env.get_template(f"{template}.html")
    html = template_obj.render(**context)
    
    return HTMLResponse(html)

# Run with: uvicorn scripts.preview_emails:app --reload
# Access at: http://localhost:8000/preview/welcome
```

## Available Providers

### [Resend](Resend.md)
Production-ready email delivery with great developer experience and reliable infrastructure.

### [Plunk](Plunk.md)
Simple email marketing and transactional emails with built-in analytics and automation features.

### [Postmark](Postmark.md)
Fast and reliable transactional email service with excellent deliverability and detailed analytics.

### [Custom Provider](Custom_provider.md)
Learn how to implement your own email provider to integrate with any email service.

## Best Practices

- Use templates for consistent branding
- Implement retry logic for failed sends
- Monitor email deliverability rates
- Test emails across different clients
- Handle bounces and unsubscribes properly
- Log email sending for debugging
- Use appropriate rate limiting
