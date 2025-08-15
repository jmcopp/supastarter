Check for purchases or subscriptions
Learn how to check for purchases or subscriptions in your application to provide access to premium features.

One of the most common use cases for payments is to provide access to premium features.

Plan ID
The plan id is used to identify a plan. It is defined in the key property of the plan object from the config file like free, pro, enterprise or lifetime from the example config.

Check for purchases or subscriptions on client side
You can check if a user has a purchase or subscription by creating custom hooks that call the FastAPI endpoints.

Create a custom hook to fetch purchase data:

```tsx
import { useQuery } from '@tanstack/react-query';

interface Purchase {
    id: string;
    plan_id: string;
    type: 'subscription' | 'one_time';
    status: 'active' | 'cancelled' | 'expired';
    // ... other properties
}

export function usePurchases(organizationId?: string) {
    const { data: purchases = [], ...query } = useQuery({
        queryKey: ['purchases', organizationId],
        queryFn: async () => {
            const url = organizationId 
                ? `/api/purchases?organization_id=${organizationId}`
                : '/api/purchases';
            
            const response = await fetch(url, {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });
            
            if (!response.ok) {
                throw new Error('Failed to fetch purchases');
            }
            
            return response.json();
        }
    });

    const activePlan = purchases.find((p: Purchase) => p.status === 'active')?.plan_id || 'free';
    
    const hasSubscription = (planIds?: string | string[]) => {
        if (!planIds) {
            return purchases.some((p: Purchase) => p.type === 'subscription' && p.status === 'active');
        }
        
        const plans = Array.isArray(planIds) ? planIds : [planIds];
        return purchases.some((p: Purchase) => 
            p.type === 'subscription' && 
            p.status === 'active' && 
            plans.includes(p.plan_id)
        );
    };
    
    const hasPurchase = (planIds: string | string[]) => {
        const plans = Array.isArray(planIds) ? planIds : [planIds];
        return purchases.some((p: Purchase) => plans.includes(p.plan_id));
    };

    return {
        activePlan,
        purchases,
        hasSubscription,
        hasPurchase,
        ...query
    };
}

// Usage in component
function SomeComponent() {
    const { activePlan, hasSubscription, hasPurchase } = usePurchases();
 
    // check if the user has an active subscription
    const hasActiveSubscription = hasSubscription();
 
    // check if the user has a purchase for the pro plan
    const hasProPurchase = hasPurchase("pro");
    
    // check if the user has a purchase for the pro plan or the enterprise plan
    const hasProOrEnterprisePurchase = hasPurchase(["pro", "enterprise"]);
 
    // check if the user has a purchase of the lifetime plan
    const hasLifetimeAccess = hasPurchase("lifetime");
}
```
Check for purchases on server side
You can also check for purchases and get the active plan on the server side using FastAPI endpoints and services.

Create a service to handle purchase logic:

api/app/services/purchases.py

```python
from sqlmodel import Session, select
from app.models.purchase import Purchase
from typing import List, Optional, Union

class PurchasesHelper:
    def __init__(self, purchases: List[Purchase]):
        self.purchases = purchases
    
    @property
    def active_plan(self) -> str:
        """Get the active plan ID, defaults to 'free' if no active purchases."""
        active_purchase = next(
            (p for p in self.purchases if p.status == 'active'), 
            None
        )
        return active_purchase.plan_id if active_purchase else 'free'
    
    def has_subscription(self, plan_ids: Optional[Union[str, List[str]]] = None) -> bool:
        """Check if user has an active subscription."""
        if not plan_ids:
            return any(p.type == 'subscription' and p.status == 'active' 
                      for p in self.purchases)
        
        plans = [plan_ids] if isinstance(plan_ids, str) else plan_ids
        return any(
            p.type == 'subscription' and 
            p.status == 'active' and 
            p.plan_id in plans
            for p in self.purchases
        )
    
    def has_purchase(self, plan_ids: Union[str, List[str]]) -> bool:
        """Check if user has a purchase for specific plan(s)."""
        plans = [plan_ids] if isinstance(plan_ids, str) else plan_ids
        return any(p.plan_id in plans for p in self.purchases)

async def get_purchases(
    session: Session, 
    user_id: Optional[str] = None, 
    organization_id: Optional[str] = None
) -> List[Purchase]:
    """Get purchases for a user or organization."""
    query = select(Purchase)
    
    if organization_id:
        query = query.where(Purchase.organization_id == organization_id)
    elif user_id:
        query = query.where(Purchase.user_id == user_id)
    
    return list(session.exec(query).all())

def create_purchases_helper(purchases: List[Purchase]) -> PurchasesHelper:
    """Create a purchases helper with the given purchases."""
    return PurchasesHelper(purchases)
```

Using in FastAPI endpoints:

```python
from fastapi import APIRouter, Depends
from app.core.auth import get_current_user
from app.core.database import get_session
from app.services.purchases import get_purchases, create_purchases_helper

@router.get("/some-endpoint")
async def some_endpoint(
    organization_id: Optional[str] = None,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    purchases = await get_purchases(
        session, 
        organization_id=organization_id or None,
        user_id=current_user.id if not organization_id else None
    )
    
    helper = create_purchases_helper(purchases)
    
    # Use the helper methods
    active_plan = helper.active_plan
    has_subscription = helper.has_subscription()
    has_pro = helper.has_purchase("pro")
    
    return {
        "active_plan": active_plan,
        "has_subscription": has_subscription,
        "has_pro": has_pro
    }
```
Check for purchases of organization
Both the React hook and FastAPI service support organizations. Simply pass the organization_id as an argument.

**Client side:**
```tsx
const { activeOrganization } = useActiveOrganization();
const { activePlan, hasSubscription, hasPurchase } = usePurchases(activeOrganization?.id);
```

**Server side:**
```python
# Get organization purchases
purchases = await get_purchases(
    session, 
    organization_id=organization_id
)

helper = create_purchases_helper(purchases)
active_plan = helper.active_plan
```
Previous

Manage plans and products

Next

Set up a paywall

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




