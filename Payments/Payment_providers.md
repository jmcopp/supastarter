Stripe
Learn how to set up Stripe with supastarter.

Get the api key
After you have created your account for Stripe, you will need to get the API key. You can do this by going to the API page in the dashboard. Here you will find the Secret key and the Publishable key. You will need the Secret key for the integration to work.

Add environment variables
To use the Stripe integration, you need to define the following environment variables to your .env.local as well as your production environment:

.env.local

STRIPE_SECRET_KEY="" # Your Stripe secret key
STRIPE_WEBHOOK_SECRET="" # The secret key of the webhook you created (see below)
Create products
For your users to choose from the available subscription plans, you need to create those Products first on the Products page. You can create as many products as you want, but you will need to create at least one product for the integration to work.

Create one product per plan you want to offer. You can add multiple prices within this product to offer multiple currencies or different billing intervals.



Create a webhook
To sync the current subscription status and other information to your database, you need to set up a webhook.

The webhook code comes ready to use with supastarter, you just have to create the webhook in the Stripe dashboard and insert the URL for your project.

To configure a new webhook, go to the Webhooks page in the Stripe settings and click the Add endpoint button.



Select the following events:

checkout.session.completed
customer.subscription.created
customer.subscription.updated
customer.subscription.deleted
To get the URL for the webhook, you can either use a local development URL or the URL of your deployed app:

Local development
If you want to test the webhook locally, you can use ngrok to create a tunnel to your local machine. Ngrok will then give you a URL that you can use to test the webhook locally.

To do so, install ngrok and run it with the following command (while your FastAPI development server is running):

```bash
ngrok http 8000
```

This will give you a URL (see the Forwarding output) that you can use to create a webhook in Stripe. Just use that URL and add `/webhooks/stripe` to it.

Production / preview deployment
When you have already deployed a version of your project, you can use the actual URL to create the webhook with. This will be necessary for a production version your app later anyway.

Make sure you have a deployed version that has the STRIPE_WEBHOOK_SECRET environment variable set.

Then you can use the URL of your deployed FastAPI app and add `/webhooks/stripe` to it.

Example URL: https://your-api.com/webhooks/stripe

Set up products in app
The created products have to be defined in the FastAPI configuration file:

api/app/core/config.py

```python
from pydantic_settings import BaseSettings
from typing import Dict, List, Any

class Settings(BaseSettings):
    # Stripe configuration
    stripe_secret_key: str
    stripe_webhook_secret: str
    
    # Payment plans configuration
    payments_plans: Dict[str, Any] = {
        "free": {
            "is_free": True,
        },
        "pro": {
            "recommended": True,
            "prices": [
                {
                    "type": "recurring",
                    "product_id": "",  # Set via environment variable
                    "interval": "month",
                    "amount": 29,
                    "currency": "USD",
                    "seat_based": True,
                    "trial_period_days": 7,
                },
                {
                    "type": "recurring",
                    "product_id": "",  # Set via environment variable  
                    "interval": "year",
                    "amount": 290,
                    "currency": "USD",
                    "seat_based": True,
                    "trial_period_days": 7,
                },
            ],
        },
    }
    
    # Price IDs from environment
    price_id_pro_monthly: str = ""
    price_id_pro_yearly: str = ""
    
    class Config:
        env_file = ".env"

settings = Settings()

# Update the plans with actual price IDs
if settings.price_id_pro_monthly:
    settings.payments_plans["pro"]["prices"][0]["product_id"] = settings.price_id_pro_monthly
if settings.price_id_pro_yearly:
    settings.payments_plans["pro"]["prices"][1]["product_id"] = settings.price_id_pro_yearly
```
We are using envs for the product ids, so you can define different product ids for each environment, as you probably want to use the test mode for development.

Define the product ids in your .env file and in the environment variables of your production environment:

.env

```bash
PRICE_ID_PRO_MONTHLY=""
PRICE_ID_PRO_YEARLY=""
```
Learn more about the payment plans configuration in Manage plans and products documentation.

Set currency for locales in your app
Lastly you need to configure the currency that should be used for each locale in your app.

You can do this by setting the currency property for each locale in the FastAPI configuration file.

```python
class Settings(BaseSettings):
    # ... other settings
    
    # Internationalization configuration
    i18n_locales: Dict[str, Dict[str, str]] = {
        "en": {
            "currency": "USD",
            "language": "en",
        },
        "de": {
            "currency": "EUR", 
            "language": "de",
        },
    }
    
    class Config:
        env_file = ".env"
```
Previous

Overview

Next

Lemonsqueezy

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




