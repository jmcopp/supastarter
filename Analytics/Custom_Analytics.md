# Custom Analytics

Learn how to use a custom analytics provider with supastarter.

To use a custom analytics provider with supastarter, first make sure you have the url of the tracking script.

Then, if you need to pass any environment variables with the script, add them to the .env.local file and your deployment environment:

```bash
NEXT_PUBLIC_MY_CUSTOM_ANALYTICS_KEY=your_key_here
```

Next, we need to activate your analytics provider. To do this, open `frontend/modules/analytics/index.tsx` and change the content to:

```typescript
export * from "./provider/custom";
```
To customize the custom analytics configuration, edit the frontend/modules/analytics/provider/custom/index.tsx.

For further instructions, follow the general analytics instructions.
