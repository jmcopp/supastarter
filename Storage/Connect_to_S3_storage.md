Connect to S3 storage
Learn how to connect to a S3 compatible storage service in supastarter.

All you need to do to set it up is to add the following environment variables to your .env.local file (and later your production environment):


S3_ACCESS_KEY_ID="your-access-key"
S3_SECRET_ACCESS_KEY="your-secret-key"
S3_ENDPOINT="your-endpoint"
S3_REGION="your-region" # Optional, defaults to auto
Some providers like DigitalOcean Spaces require you to set allowed origins for CORS requests. You can set the allowed origins to * to allow all origins, but it's recommended to set it to the domain of your application.

Previous

Overview

Next

Uploading files

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




