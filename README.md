# Affiliate Recipe Specification






## Overview

The Walmart Recipe service can be used to map partner recipes with products in the Walmart Grocery Catalog. The service provides real time mapping of recipe ingredients to available grocery catalog products at a Walmart store level. The document lists onboarding steps along with API specification required to perform the Partner Recipes to Walmart Grocery Catalog integration.

The sequence diagram below represents the high-level architecture of the overall flow. The onboarding section lists the steps required to: onboard a new partner, register recipes, and invoke mapping APIs. API documentation lists the API details along with sample request/response body.

## Recipe Service Flow

![image-20200624145459940](/Users/j0l05qj/Library/Application Support/typora-user-images/image-20200624145459940.png)

External client will receive a client secret and id issued by Walmart

Given the client secret, external client can make a IAM call using client id and secret


### OAuth API

| Request/Response format | JSON                     |
| ----------------------- | ------------------------ |
| Method                  | POST                     |
| URI                     | /identity/oauth/v1/token |
| Authentication          | Client Id, Client Secret |

**Request Header Parameters**

| Header Parameter | Description                       | Mandatory |
| ---------------- | --------------------------------- | --------- |
| cache-control    | no-cache                          | Yes       |
| WM_CONSUMER.ID   | Walmart Consumer Id               | Yes       |
| Content-Type     | application/x-www.form-urlencoded | Yes       |

**Request Payload Parameters**

| Payload Parameter | Description          | Mandatory |
| ----------------- | -------------------- | --------- |
| grant_type        | client_credentials   | Yes       |
| client_id         | Walmat Consumer Id   | Yes       |
| client_secret     | IAM Generated Secret | Yes       |



### Register Recipe API

### Update Recipe API

### Recipe Products API

### Similar Products API



