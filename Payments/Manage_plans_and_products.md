Manage plans and products
Learn how to manage plans and products in your application.



Define plans and products
You can manage the plans and products in the FastAPI configuration file of your supastarter project.

There are different types of plans you can define:

Free plan
The free plan is the default plan for users who have not purchased any plans or can be used to access a limited version of your product.

As this is no paid plan, you don't need to define any prices or attach a product id to it.

api/app/core/config.py

```python
from pydantic_settings import BaseSettings
from typing import Dict, List, Any

class Settings(BaseSettings):
    payments_plans: Dict[str, Any] = {
        "free": {
            "is_free": True,
        },
    }
    
    class Config:
        env_file = ".env"
```
Enterprise plan
The enterprise plan is not a real plan, but will show up in the pricing table with a link to a contact form, so customers can contact you to get access to your product.

As this is no paid plan, you don't need to define any prices or attach a product id to it.

```python
class Settings(BaseSettings):
    payments_plans: Dict[str, Any] = {
        "enterprise": {
            "is_enterprise": True,
        },
    }
    
    class Config:
        env_file = ".env"
```
Subscription plans and one-time purchase plans
A plan represents a product or service of your application and each is a column in your pricing table. It has the following properties:

recommended: if this plan should be highlighted as recommended
hidden: hide the plan from the pricing table, can be used if you want to grandfather old plans or prepare for a new plan
prices: define the prices for this plan
One plan can have multiple prices, for example a monthly and yearly price or/and for each currency you support.

A price has the following properties:

- **type**: the type of the price, can be recurring or one_time
- **product_id**: the id of the product from the payment provider
- **interval**: the interval of the price, can be month, year, week or day
- **interval_count**: the number of intervals to bill, defaults to 1
- **amount**: the amount of the price
- **currency**: the currency of the price, for example USD
- **trial_period_days**: the number of days of the trial period, leave out if you don't want to offer a trial period
- **seat_based**: if price is per seat (this will only work for organizations and will multiply the price by the number of members), defaults to false
You can publish your site with a placeholder product_id, if you need a landing page with pricing table to verify your store in Stripe or Lemonsqueezy. Just note that this will not work in production.

```python
class Settings(BaseSettings):
    payments_plans: Dict[str, Any] = {
        "pro": {
            "recommended": True,
            "prices": [
                {
                    "type": "recurring",
                    "product_id": "price_as34asdflkh134kh",
                    "interval": "month",
                    "amount": 29,
                    "currency": "USD",
                    "trial_period_days": 7,
                    "seat_based": True,
                },
                {
                    "type": "recurring",
                    "product_id": "price_as34asdflkh134kh",
                    "interval": "year",
                    "amount": 290,
                    "currency": "USD",
                    "trial_period_days": 7,
                    "seat_based": True,
                },
            ],
        },
        "lifetime": {
            "prices": [
                {
                    "type": "one_time",
                    "product_id": "price_as34asdflkh134kh",
                    "amount": 999,
                    "currency": "USD",
                },
            ],
        },
    }
    
    class Config:
        env_file = ".env"
```
Support multiple currencies
To support multiple currencies, make sure to define the different currencies for each locale in the i18n section of the config:

```python
class Settings(BaseSettings):
    i18n_locales: Dict[str, Dict[str, str]] = {
        "en": {
            "currency": "USD",
        },
        "de": {
            "currency": "EUR",
        },
    }
    
    class Config:
        env_file = ".env"
```

Then define the prices for each locale in the payments_plans section:

```python
class Settings(BaseSettings):
    payments_plans: Dict[str, Any] = {
        "pro": {
            "prices": [
                {
                    "type": "recurring",
                    "product_id": "price_as34asdflkh134kh",
                    "interval": "month",
                    "amount": 29,
                    "currency": "USD"
                },
                {
                    "type": "recurring",
                    "product_id": "price_as34asdflkh134kh",
                    "interval": "month",
                    "amount": 29,
                    "currency": "EUR"
                },
            ],
        },
    }
    
    class Config:
        env_file = ".env"
```
Plan information for pricing table
You can define the information for the pricing table for each plan in the usePlanData hook in /frontend/modules/saas/payments/hooks/plan-data.ts.

You can define a title, a description and a features array for each plan.

We recommend you to use the t() function to get the translations for the plan information and then define the translations in the /packages/i18n/translations/ folder.

/frontend/modules/saas/payments/hooks/plan-data.ts

export function usePlanData() {
	const t = useTranslations();
 
	const planData: Record<
		ProductReferenceId,
		{
			title: string;
			description: ReactNode;
			features: ReactNode[];
		}
	> = {
		free: {
			title: t("pricing.products.free.title"),
			description: t("pricing.products.free.description"),
			features: [
				t("pricing.products.free.features.anotherFeature"),
				t("pricing.products.free.features.limitedSupport"),
			],
		},
		pro: {
			title: t("pricing.products.pro.title"),
			description: t("pricing.products.pro.description"),
			features: [
				t("pricing.products.pro.features.anotherFeature"),
				t("pricing.products.pro.features.fullSupport"),
			],
		},
		enterprise: {
			title: t("pricing.products.enterprise.title"),
			description: t("pricing.products.enterprise.description"),
			features: [
				t("pricing.products.enterprise.features.unlimitedProjects"),
				t("pricing.products.enterprise.features.enterpriseSupport"),
			],
		},
	};
 
	return { planData };
}
Note: When you remove one of the existing plans, make sure to remove the corresponding entry in the /frontend/modules/saas/payments/hooks/plan-data.ts file, otherwise you will get a type error.
