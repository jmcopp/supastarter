# Analytics Overview

Learn how to implement analytics in supastarter's multi-container architecture.

## Architecture

Analytics in supastarter is implemented at the **backend level** with tracking endpoints and frontend integration for event capture.

```
Frontend (React) → API (FastAPI) → Analytics Provider
                                ↓
                         Database Storage
```

## Analytics Service

### Core Analytics Service

**File**: `api-main/app/services/analytics.py`

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
from datetime import datetime
import aiohttp
import hashlib
from app.core.config import settings
import logging

logger = logging.getLogger(__name__)

class AnalyticsProvider(ABC):
    @abstractmethod
    async def track_event(
        self,
        event_name: str,
        properties: Dict[str, Any],
        user_id: Optional[str] = None,
        anonymous_id: Optional[str] = None
    ) -> bool:
        pass
    
    @abstractmethod
    async def identify_user(
        self,
        user_id: str,
        traits: Dict[str, Any]
    ) -> bool:
        pass

class GoogleAnalyticsProvider(AnalyticsProvider):
    """Google Analytics 4 provider."""
    
    def __init__(self):
        self.measurement_id = settings.GA_MEASUREMENT_ID
        self.api_secret = settings.GA_API_SECRET
        self.endpoint = "https://www.google-analytics.com/mp/collect"
    
    async def track_event(
        self,
        event_name: str,
        properties: Dict[str, Any],
        user_id: Optional[str] = None,
        anonymous_id: Optional[str] = None
    ) -> bool:
        """Track event to Google Analytics."""
        
        async with aiohttp.ClientSession() as session:
            params = {
                "measurement_id": self.measurement_id,
                "api_secret": self.api_secret
            }
            
            payload = {
                "client_id": anonymous_id or self._generate_client_id(user_id),
                "events": [{
                    "name": event_name,
                    "params": properties
                }]
            }
            
            if user_id:
                payload["user_id"] = user_id
            
            try:
                async with session.post(
                    self.endpoint,
                    params=params,
                    json=payload
                ) as response:
                    return response.status == 204
            except Exception as e:
                logger.error(f"GA tracking error: {str(e)}")
                return False

class AnalyticsService:
    """Main analytics service."""
    
    def __init__(self):
        if settings.ANALYTICS_PROVIDER == "google":
            self.provider = GoogleAnalyticsProvider()
        elif settings.ANALYTICS_PROVIDER == "mixpanel":
            self.provider = MixpanelProvider()
        else:
            self.provider = None
    
    async def track(
        self,
        event_name: str,
        properties: Optional[Dict[str, Any]] = None,
        user_id: Optional[str] = None
    ) -> bool:
        """Track analytics event."""
        
        if not self.provider:
            return True
        
        return await self.provider.track_event(
            event_name=event_name,
            properties=properties or {},
            user_id=user_id
        )

# Global analytics instance
analytics = AnalyticsService()
```

## Analytics API Endpoints

**File**: `api-main/app/routes/analytics.py`

```python
from fastapi import APIRouter, Request, Depends
from typing import Optional, Dict, Any
from app.services.analytics import analytics
from app.core.auth import get_current_user_optional

router = APIRouter(prefix="/analytics", tags=["analytics"])

@router.post("/track")
async def track_event(
    request: Request,
    event: str,
    properties: Optional[Dict[str, Any]] = None,
    current_user: Optional[str] = Depends(get_current_user_optional)
):
    """Track analytics event."""
    
    success = await analytics.track(
        event_name=event,
        properties=properties,
        user_id=current_user
    )
    
    return {"success": success}
```

## Frontend Integration

### Analytics Hook

**File**: `frontend/lib/hooks/useAnalytics.ts`

```typescript
import { useCallback } from 'react'
import { useAuth } from './useAuth'

export function useAnalytics() {
  const { user } = useAuth()
  
  const trackEvent = useCallback(async (
    event: string,
    properties?: Record<string, any>
  ) => {
    try {
      const response = await fetch('/api/analytics/track', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          event,
          properties
        })
      })
      
      const data = await response.json()
      return data.success
    } catch (error) {
      console.error('Analytics tracking error:', error)
      return false
    }
  }, [])
  
  return {
    trackEvent
  }
}
```

## Configuration

**File**: `api-main/.env`

```bash
# Analytics Configuration
ANALYTICS_PROVIDER=google  # or mixpanel, posthog, custom

# Google Analytics
GA_MEASUREMENT_ID=G-XXXXXXXXXX
GA_API_SECRET=your-api-secret

# Mixpanel
MIXPANEL_PROJECT_TOKEN=your-project-token

# PostHog
POSTHOG_API_KEY=your-api-key
POSTHOG_HOST=https://app.posthog.com
```
