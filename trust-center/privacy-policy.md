# Privacy policy

Customer trust and data security are critical to everything we do, and we understand that our customers need to be confident using our application, and be aware of our data collection practices.

## Privacy principles

Our primary privacy principles:
- Control: We will put you in control of your privacy with easy-to-use tools and clear choices.
- Transparency: We will be transparent about data collection and use so you can make informed decisions.
- Security: We protect your data with strong security and encryption.
- Benefit to you: When we do collect data, we will use it to benefit you and to make your experiences better.
- You own your data: Customer data is only used to provide agreed upon services and if you leave the data is removed.

## Data management policy

***This section is subject to change and we recommend that you check back quarterly for updates.***

### Application data

TODO: Describe collected data

### Census data
Census data is acquired solely to provide, support, and improve the application. It includes environmental information such as device and operating system versions, and regional and language settings. Here are some specific examples of the census data that's collected:

| Data type | Example | Notes |
|-----------|---------|-------|
| DeviceModel | iPhone | |
| OSName | iPhoneiOS | |
| OSVersion | 8.3 | |
| UserLanguage | EN-US | |
| UserID | E296D735-4F36-4E18-7C3B-52E1A02A0164 | The application automatically generates anonymous user IDs, then populates telemetry events with these IDs as they're sent from the app. A hashing ensures the ID cannot be linked to a specific user. |
| Session ID | 5E872200-F546-4CCD-8F23-AF5F507AA2DD | The application automatically generates anonymous session IDs, then populates telemetry events with these IDs as they're sent from your app. A hashing ensures the ID cannot be linked to a specific user. |
The application relies entirely on the Microsoft Azure platform.  
Here is a summary of the key architecture components and their associated data residency:

| Service | Role | Data Location |
|---------|------|---------------|
| Azure Traffic Manager | Azure Traffic Manager is a DNS-based traffic load balancer that enables us to distribute traffic optimally to services across global Azure regions. | By definition, this service is located in multiple regions globally, but doesn't store any data. |
| Azure App Service | The application is hosted as a containerized app on Linux containers, enabling vertical and horizontal scale-up based on application needs and reach high availability. | **France Central** |
| Azure Application Insights | Application Insights is an extensible Application Performance Management (APM). We're using it to monitor our live production environments, gather telemetry such as performance counters, Azure diagnostics, and diagnostic trace logs. | **France Central** |
| Azure Cache for Redis | Azure Cache for Redis is based on the popular software [Redis](https://redis.io/). It is used as a cache mechanism to improve the performance and scalability of the application, especially for back-end data store access and external APIs requests. | **France Central** |
| Azure Cosmos DB | Azure Cosmos DB is a globally distributed, multi-model database service that supports document, key-value, wide-column, and graph databases. The application relies on it as the main back-end data store. | Read Locations: **France Central**.  Write Locations: **France Central** |
| Azure Key Vault | Microsoft Azure Key Vault is a cloud-hosted management service that allows the application to encrypt keys and small secrets by using keys that are protected by hardware security modules (HSMs). The application relies on it to store securely its encryption keys. | **France Central** |
| Azure Blob Storage | Azure Blob Storage is a massively scalable object storage for unstructured data that allows the application to store securely blobs contents such as templates pictures. | **France Central** |

:::tip Reference
Learn more about [Azure Regions](https://azure.microsoft.com/en-us/global-infrastructure/regions/)
:::


* We are processing anonymous usage data to understand how our customers are using the application.
* We are processing non-anonymous usage data to create in-app usage dashboards and audit trails.

### Support data

Support data includes information such as user UPN, tenant ID, app version. Access to these data is restricted to our support team, and is only used in case of a direct support request to our support team.

### Error reporting data

Error reporting data can include information about performance and reliability, device configuration, network connection quality, error codes, error logs, and exceptions.

#### Log files lifecycle

Every 15 days, logs files are rotated and older log files deleted.

***Error reporting data may also contain personally identifiable information such as the user's IP address.***

### Data Residency

TODO: Add and describe est US residency
