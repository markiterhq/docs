# API
The Markiter API provides a unified programmability model that you can use to manage your collaboration environment such as Microsoft Teams and build powerful apps easily.

  Key features:
  - Create powerful self-service templates for Microsoft Teams
  - Deploy Microsoft Planner at scale with plans templates
  - Create a custom approval process for your teams provisioning requests
  - Apply automaticaly governance policies across all your teams
  - Integrate a custom in-house and LoB apps with Microsoft Teams

To see all the available operations, see the [API reference](/api)

# Authentication
To access the Markiter API, a valid Azure Active Directory access token is required, as a user or as an application.

## Supported access tokens
The Markiter API expects a valid access token in the HTTP `Authorization` request header with a `bearer` token such as:
```json
{
  "Authorization": "bearer <JWT_TOKEN>"
}
```

Markiter supports [access tokens](https://docs.microsoft.com/en-us/azure/active-directory/develop/access-tokens) retreived from the following `OAuth 2.0` grant flows:
- [Auth Code](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow)
- [On-behalf-of](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow)
- [Client Credentials](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)

## Required roles
The Markiter API implements role-based access control for each operation:
| Role | Code | Origin |
|------|------|--------|
| Anonymous Access | `ANONYMOUS_ACCESS` | Markiter |
| End-User | `AUTHENTICATED_USER` | Markiter |
| Authorized App | `AUTHORIZED_APP` | Markiter |
| Catalog Manager | `CATALOG_MANAGER` | Markiter |
| Governance Manager | `GOVERNANCE_MANAGER` | Markiter |
| Compliance Manager | `COMPLIANCE_MANAGER` | Markiter |
| Integration Manager | `INTEGRATION_MANAGER` | Markiter |
| Change Manager | `CHANGE_MANAGER` | Markiter |
| Teams Service Administrator | `TEAMS_SERVICE_ADMIN` | Microsoft 365 |
| Global Administrator | `GLOBAL_ADMIN` | Microsoft 365 |

Each operation in this documentation specifies its required roles (You'll need at least ONE of them) through the `x-Markiter-required-roles` extension.

# Rate limits

The Markiter API allows you to access data and to perform operations on multiple services. These services impose their own rate limits that affect applications that use Markiter API to access them.

## Overview of rate limit tiers
Rate limits are defined as "tiers" applied on a "per user (or app) per tenant" basis.

⚠️ The specific limits described here are subject to change.

| Tier | Code | Limit | Window |
|------|------|-------|--------|
| Tier 1 | `tier-1` | 6 | 1 minute |
| Tier 2 | `tier-2` |  20 | 1 minute |
| Tier 3 | `tier-3` |  60 | 1 minute |
| Fair Use | `fair-use` | 600 | 1 minute |

Each operation in this documentation specifies its rate limits as a tier through the `x-markiter-rate-limit` extension.

## What happens when your is exceeding a limit?

When a threshold is exceeded, Markiter API limits any further requests from that client for a period of time. When throttling occurs, Markiter API returns HTTP status code 429 (Too many requests), and the requests fail.

Throttling limits the number of concurrent calls to a service to prevent overuse of resources. Markiter API is designed to handle a high volume of requests. If an overwhelming number of requests occurs, throttling helps maintain optimal performance and reliability of the Markiter API service.

Throttling limits vary based on the scenario. For example, if you are performing a large volume of writes, the possibility for throttling is higher than if you are only performing reads.

Markiter API is conforming to the [IETF ratelimit standardization proposal](https://tools.ietf.org/id/draft-polli-ratelimit-headers-01.html).

💡 Note:
Throttling behavior can depend on the type and number of requests. For example, if you have a high volume of requests, all requests types are throttled. Threshold limits vary based on the request type. Therefore, you could encounter a scenario where writes are throttled but reads are still permitted.

## Common throttling scenarios

The most common causes of throttling of clients include:
- A large number of requests across all applications in a our environments.
- A large number of requests from a particular application across all environments.

## Best practices to avoid throttling

Programming patterns like continuously polling a resource to check for updates and regularly scanning resource collections to check for new or deleted resources are more likely to lead to applications being throttled and degrade overall performances.

Before any throttling, Markiter API provides two useful headers included in every responses so that you can monitor your own activity level:
- `X-RateLimit-Limit`: The limit of requests in a perdiod of time (aka "window")
- `X-RateLimit-Remaining`: The current number of requests that could be made during the current window.

## Best practices to handle throttling

The following are best practices for handling throttling:

- Reduce the number of operations per request.
- Reduce the frequency of calls.
- Avoid immediate retries, because all requests accrue against your usage limits.

When you implement error handling, use the HTTP error code 429 to detect throttling. The failed response includes the `Retry-After` response header. Backing off requests using the `Retry-After` delay is the fastest way to recover from throttling because Markiter API continues to log resource usage while a client is being throttled.

1. Wait the number of seconds specified in the `Retry-After` header.
2. Retry the request.
3. If the request fails again with a 429 error code, you are still being throttled. Continue to use the recommended `Retry-After` delay and retry the request until it succeeds.

💡 Note:
If no `Retry-After` header is provided by the response, we recommend implementing an exponential backoff retry policy.

In addition to the `Retry-After` header, Markiter API includes `X-RateLimit-Limit` and `X-RateLimit-Remaining` infos in the body of the throttled response:
```yaml
{
  message: 'Too many requests, please try again later...',
  rateLimitExceeded: {
    tier: 'tier-1', # Tier code
    rateLimitWindow: 60000, # In ms
    rateLimitMax: 6 # In # of requests
  }
}
```
# Errors

While your application should handle all error responses (in the 400 and 500 ranges), pay special attention to certain expected errors and responses, listed in the following table.

| Topic | HTTP code | Best practice|
|:------|:----------------|:-------------|
| User does not have access | 403 | If your application is up and running, it could encounter this error even if it has been granted the necessary permissions. In this case, it's most likely that the signed-in user or virtual app does not have privileges to access the resource requested. Your application should provide a generic "Access denied" error back to the signed-in user. |
|Not found| 404 | In certain cases, a requested resource might not be found. For example a resource might not exist, because it has not yet been provisioned or because it has been deleted. |
|Throttling|429|APIs might throttle at any time for a variety of reasons, so your application must **always** be prepared to handle 429 responses. This error response includes the `Retry-After` field in the HTTP response header. Backing off requests using the `Retry-After` delay is the fastest way to recover from throttling. For more information, see the [Rate-limit](https://docs.markiter.ai/api#tag/Rate-limits) article.|
|Service unavailable| 503 | This is likely because the services is busy. You should employ a back-off strategy similar to 429. Additionally, you should **always** make new retry requests over a new HTTP connection.|

## Reliability and support
To ensure reliability and facilitate support for your application, generate a unique GUID and send it on each Markiter API REST request. This will help Markiter investigate any errors more easily if you need to report an issue with Markiter API.
To do so, on every request to Markiter API, generate a unique GUID, send it in the `client-request-id` HTTP request header, and also log it in your application's logs.

# Webhooks

Webhooks enable organizations to trigger automated operations outside of the Markiter platform, such as in a custom application, or in an automation tool such as Microsoft Power Automate, Azure Logic Apps or Zapier.

⚠️ Note:  
Due to the fact that data may be exchanged outside of your Microsoft 365 environment, webhooks have to be considered as **highly sensitive**. To protect these operations, the Markiter platform controls webhooks management through RBAC, making any operation related to webhooks accessible only to users granted with one of the following roles:
- `Global admin` (Microsoft 365) role
- `Teams service admin` (Microsoft 365) role
- `Integration Manager` (Markiter) role

## Anatomy of a Webhook
A wehbook is defined by the following properties:
```json
{
    "id": "7f105c7d-2dc5-4532-97cd-4e7ae6534c07", // {string} Webhook UUID automatically generated during its creation - ReadOnly
    "name": "Example Webhook", // {string} Webhook name - Read/Write
    "description": "This is a new webhook", // {string} Webhook description - Read/Write
    "active": true, // {boolean} Webhook status - Read/Write
    "events": [ // {array} Array of events codes that will trigger the webhook - Read/Write
        "team_provisioning_completed", // {string} Event code - Read/Write
        ...
    ],
    "config": { // {object} Webhook http configuration
        "verb": "post", // {string} Http verb used by the the webhook - ReadOnly as currently only `post` is supported
        "url": "https://example.com/webhook", // {string} Target URL of the webhook - Read/Write
        "content_type": "json", // {string} Http content-type used by the webhook - ReadOnly as currently only `json` (matching to `application/json`) is   supported
        "secret":"secretClientValue" // {string} Secret value used to authentify the wehbook emitter by the consumer - Read/Write
    }
}
```

## Anatomy of a Request
When triggered, the webhook generates an http `POST` request to its configured url.

### Headers
The following headers are included in the request:
```json
{
    "X-Markiter-Webhook": "", // {string} UUID of the webhook that triggered the request.
    "X-Markiter-Event": "", // {string} Code of the event that triggered the request.
    "X-Markiter-Delivery": "", // {string} An automatically generated UUID to identify the request.
    "X-Markiter-Signature": "" // {string} This header is sent if the webhook is configured with a secret.
}
```

### Payload
A webhook payload contains at least the following properties:
```json
{
    "tenant": {
        "id": "" // {string} The tenant ID from where the event originates
    }
}
```

### User-Agent
The User-Agent for the requests will have the prefix `Markiter-Webhook/` and include the Markiter current version number.  
For instance `Markiter-Webhook/2.1.193`

## Endpoints Requirements

### Security
Your endpoint must be an HTTPS URL with a valid SSL certificate that can correctly process event notifications.

### Responses
The expected success response codes from the target endpoint are `200`, `201`, `202`. If any other code is received, our webhook engine will retry the request following this retry policy:
```js
MAX_RETRY = 2 // Maximum number of retry before flagging the delivery as `failed`
RETRY_INTERVAL = 10000 // Number of milliseconds between each retry
```

### Verifying Webhooks
Webhooks sent by Markiter can be verified by calculating a digital signature. If a secret has been defined, the webhook requests will includes a `X-Markiter-Signature` header.  
To verify that the request came from Markiter, compute the HMAC hex digest of the request body, generated using the SHA-256 hash function and the secret as the HMAC key. If they match, then you can be sure that the webhook was sent from Markiter.

Here are a comprehensive list of examples for multiple languages:

#### Node
```js
const crypto = require('crypto');
const hmac = crypto.createHmac('sha256', 'secret');
hmac.update('Message');
console.log(hmac.digest('hex'));
```

#### PHP
```php
<?php
echo hash_hmac('sha256', 'Message', 'secret');
?>
```

#### Java
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class Test {
public static void main(String[] args) {
    try {
        String secret = "secret";
        String message = "Message";

        Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes(), "HmacSHA256");
        sha256_HMAC.init(secret_key);

        byte[] hash = (sha256_HMAC.doFinal(message.getBytes()));
        StringBuffer result = new StringBuffer();
        for (byte b : hash) {
            result.append(String.format("%02x", b)); 
        }
        System.out.println(result.toString());
    }
    catch (Exception e){
        System.out.println("Error");
    }
}
}
```

#### C#
```csharp
using System.Security.Cryptography;

namespace Test
{
public class MyHmac
{
    public static void Main(string[] args){
    var hmac = new MyHmac ();
    System.Console.WriteLine(hmac.CreateToken ("Message", "secret"));
    }
    private string CreateToken(string message, string secret)
    {
    secret = secret ?? "";
    var encoding = new System.Text.ASCIIEncoding();
    byte[] keyByte = encoding.GetBytes(secret);
    byte[] messageBytes = encoding.GetBytes(message);
    using (var hmacsha256 = new HMACSHA256(keyByte))
    {
        byte[] hashmessage = hmacsha256.ComputeHash(messageBytes);

        var sb = new System.Text.StringBuilder();
        for (var i = 0; i <= hashmessage.Length - 1; i++)
        {
        sb.Append(hashmessage[i].ToString("x2"));
        }
        return sb.ToString();
    }
    }
}
}
```

#### Go
```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
)

func ComputeHmac256(message string, secret string) string {
    key := []byte(secret)
    h := hmac.New(sha256.New, key)
    h.Write([]byte(message))
    return hex.EncodeToString(h.Sum(nil))
}

func main() {
    fmt.Println(ComputeHmac256("Message", "secret"))
}
```

#### Ruby
```ruby
require 'openssl'
OpenSSL::HMAC.hexdigest('sha256', "secret", "Message")
```

#### Python (3.x)
```python
import hashlib
import hmac

KEY = "secret"
KEY_BYTES=KEY.encode('ascii')
MESSAGE = "Message"
MESSAGE_BYTES=MESSAGE.encode('ascii')
result = hmac.new(KEY_BYTES, MESSAGE_BYTES, hashlib.sha256).hexdigest()

print (result)
```

# Use Postman

You can use a Postman collection to get started with the Markiter API in minutes:
1. Download and register for [Postman](https://www.getpostman.com/).
2. Choose **File | Import ...**.
3. Select **Import From Link**.
4. Paste the following URL and choose **Import**.

```
https://api.markiter.app/v1.0/openapi
```

---
# Versions

## API v1.0
Markiter API under the `v1.0` endpoint (under `https://api.markiter.app/v1.0`) are in general availability (`GA`) status.

**💡 Recommendation: Use v1.0 APIs for your production apps.**
