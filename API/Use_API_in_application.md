Use API in application
Learn how to fetch data from your API in your supastarter application.

To query this endpoint from your application in a type-safe way, you can use the apiClient in your Next.js application.

Use API in a client component

import { apiClient } from "@shared/lib/api-client";
 
const response = await apiClient.posts.$get();
 
if (!response.ok) {
	throw new Error("Failed to fetch posts");
}
 
const posts = await response.json();
You can use the apiClient directly in a useEffect for example, but we recommend using it in combination with Tanstack Query for a better developer and user experience.

It gives you a lot of features like caching, error handling, loading states, and more.


const { data, isLoading, error } = useQuery({
	queryKey: ["posts"],
	queryFn: () => {
    const response = await apiClient.posts.$get();
 
    if (!response.ok) {
      throw new Error("Failed to fetch posts");
    }
 
    const posts = await response.json();
  }
});
You now have a fully type-safe API client that you can use in your application.

You can find many examples of how to use API endpoints with Tanstack Query in the supastarter application code. Just search for apiClient, useQuery and useMutation in the codebase.

Use API in a server component
The API client can be used in the same way in a server component, but for the cookies to automatically be attached to the request, you can use the getServerApiClient helper:


import { getServerApiClient } from "@shared/lib/server";
 
const apiClient = await getServerApiClient();
 
const purchases = await apiClient.payments.purchases.$get(
  {
    query: {
      organizationId,
    },
  }
);
Previous

Protect API endpoints

Next

Use locale in API endpoints

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




