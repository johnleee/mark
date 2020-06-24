# Affiliate Recipe Specification



[Overview](#overview) <br/>
[Recipe Service Flow](#recipe-service-flow)
1. [Onboarding Steps](#1-onboarding-steps) <br/>
2. [API](#2-api) <br/>
2.1 [OAuth API](#21-oauth-api) <br/>
2.2 [Register Recipe API](#22-register-recipe-api) <br/>
2.3 [Update Recipe API](#23-update-recipe-api) <br/>
2.4 [Store Locator API](#23-store-locator-api) <br/>
2.5 [Recipe Products API](#25-recipe-products-api) <br/>
2.6 [Similar Products API](#26-similar-products-api) <br/>
3. [CURL Calls] (#3-curl-calls) <br/>
3.1 [OAuth API CURL Call](#31-oauth-api-curl) <br/>
3.2 [Register Recipe API CURL Call](#32-register-recipe-api-curl-call) <br/>
3.3 [Update Recipe API CURL Call](#33-update-recipe-api-curl-call) <br/>
3.3 [Store Locator API CURL Call](#34-store-locator-api-curl-call) <br/>
3.5 [Recipe Products API CURL Call](#35-recipe-products-api-curl-call) <br/>
3.6 [Similar Products API CURL Call](#36-similar-products-api-curl-call) <br/>

## Overview

The Walmart Recipe service can be used to map partner recipes with products in the Walmart Grocery Catalog. The service provides real time mapping of recipe ingredients to available grocery catalog products at a Walmart store level. The document lists onboarding steps along with API specification required to perform the Partner Recipes to Walmart Grocery Catalog integration.

The sequence diagram below represents the high-level architecture of the overall flow. The onboarding section lists the steps required to: onboard a new partner, register recipes, and invoke mapping APIs. API documentation lists the API details along with sample request/response body.

## Recipe Service Flow

![image-20200624145459940](/Users/j0l05qj/Library/Application Support/typora-user-images/image-20200624145459940.png)


## 1. Onboarding Steps

**A. One-time step during partner registration:**

   **1.1** Walmart will onboard Partner and provide client access token to invoke recipe APIs.
   - Getting Access
   - External client will receive a client secret and id issued by Walmart. 
   - Givent the client secret, external client can make an IAM API call using client id and secret.

   **1.2** Partner registers all recipes by specifying recipes in structured format. Partners will upload/email recipes in a JSON file that will have this format:

```json
{
  "recipes": [
    {
      "externalId": "14",
      "title": "Bacon Quiche",
      "description": "Quiche is the perfect food.",
      "servingSize": 4,
      "ingredients": [
        {
          "externalId": "ing2643",
          "text": "1.0 Pie Crust",
          "product": "Pie Crust",
          "unit": "",
          "unitValue": "1.0"
        },
        {
          "externalId": "ing153",
          "text": "150.0 gram Bacon",
          "product": "Bacon",
          "unit": "gram",
          "unitValue ": "150.0"
        }
      ]
    }
  ]
}
```

**B. One-time step during user setup:**

   **1.3** Partner invokes Store Locator API to get store details by passing customer zip code.

**C. Active User Flow:**

   **1.4** Partner invokes Get Recipe Products API to get Ingredient to Product mapping details for given recipe identifiers.
   **1.5** Partner invokes Get Similar Products API to get Similar Products for given product identifiers.

## 2 API

### 2.1 OAuth API

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



### 2.2 Register Recipe API

### 2.3 Update Recipe API

### 2.4 Store Locator API

### 2.5 Recipe Products API

### 2.5 Similar Products API

## 3 CURL Calls

### 3.1 OAuth API CURL Call

### 3.2 Register Recipe API CURL Call

### 3.3 Update Recipe CURL Call

### 3.4 Store Locator API CURL Call

### 3.5 Recipe Products API CURL Call

### 3.6 Similar Products API CURL Call





