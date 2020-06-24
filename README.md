# Affiliate Recipe Specification



## Overview

The Walmart Recipe service can be used to map partner recipes with products in the Walmart Grocery Catalog. The service provides real time mapping of recipe ingredients to available grocery catalog products at a Walmart store level. The document lists onboarding steps along with API specification required to perform the Partner Recipes to Walmart Grocery Catalog integration.

The sequence diagram below represents the high-level architecture of the overall flow. The onboarding section lists the steps required to: onboard a new partner, register recipes, and invoke mapping APIs. API documentation lists the API details along with sample request/response body.

## Table of Contents

[Recipe Service Flow](#recipe-service-flow)
1. [Onboarding Steps](#1-onboarding-steps) <br/>
2. [API](#2-api) <br/>
2.1 [OAuth API](#21-oauth-api) <br/>
2.2 [Register Recipe API](#22-register-recipe-api) <br/>
2.3 [Update Recipe API](#23-update-recipe-api) <br/>
2.4 [Store Locator API](#23-store-locator-api) <br/>
2.5 [Recipe Products API](#25-recipe-products-api) <br/>
2.6 [Similar Products API](#26-similar-products-api) <br/>
3. [CURL Calls](#3-curl-calls) <br/>
3.1 [OAuth API CURL Call](#31-oauth-api-curl) <br/>
3.2 [Register Recipe API CURL Call](#32-register-recipe-api-curl-call) <br/>
3.3 [Update Recipe API CURL Call](#33-update-recipe-api-curl-call) <br/>
3.3 [Store Locator API CURL Call](#34-store-locator-api-curl-call) <br/>
3.5 [Recipe Products API CURL Call](#35-recipe-products-api-curl-call) <br/>
3.6 [Similar Products API CURL Call](#36-similar-products-api-curl-call) <br/>

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

| 						  |                      |
| ----------------------- | ------------------------ |
| Request/Response format | JSON                     |
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

Register partner recipes with Walmart.

|  						  |                          |
| ----------------------- | ------------------------ |
| Request/Response format | JSON                     |
| Method                  | PUT                      |
| URI                     | /api/<partnerId>/recipes |
| Authentication          | Client Id, Client Secret |

**Request Header Parameters**

| Header Parameter | Description                       | Mandatory |
| ---------------- | --------------------------------- | --------- |
| WM_SVC.VERSION   | 1.0.0                        	   | No        |
| WM_CONSUMER.ID   | Walmart Consumer Id               | Yes       |
| WM_SVC.ENV   	   | stg, prod          			   | No        |
| WM_SVC.NAME      | affiliates-recipe	               | No        |
| AUTHORIZATION	   | Bearer <oauth_token>              | Yes       |
| Content-Type     | application/x-www.form-urlencoded | Yes       |

**Request Body Schema**

**RegisterRecipeRequest** Object

Register Recipe request will contain an array of recipes to be registered.

| Field Name | Data Type              | Description      | Mandatory |
| ---------- | ---------------------- | ---------------- | --------- |
| recipes    | array of Recipe object | Array of Recieps | Yes       |

**Recipe** Object

**RegisterRecipeRequest.recipes[]**

Each Recipe object will contain metadata of the recipe along with list of ingredients.

| **Field Name** | **Data Type**                | **Description**                                              | **Mandatory** |
| -------------- | :--------------------------- | ------------------------------------------------------------ | ------------- |
| externalId     | string                       | A unique Id for  the object, which partner uses to identify the recipe. | No            |
| title          | string                       | Title of the  recipe                                         | Yes           |
| description    | string                       | Description of  the recipe                                   | No            |
| servingSize    | integer                      | Serving size of  the recipe                                  | Yes           |
| ingredients    | array of  Ingredients object | Array of  Ingredients                                        | Yes           |

**Ingredient** Object

**RegisterRecipeRequest.recipes[].ingredients[]**

Each Ingredient object will contain external identifier along with the ingredient text.



| **Field Name** | **Data Type** | **Description**                                              | **Mandatory**                             |
| -------------- | ------------- | ------------------------------------------------------------ | ----------------------------------------- |
| externalId     | string        | A unique Id for  the object, which partner uses to identify the ingredient. | No                                        |
| text           | string        | Ingredient text   e.g.150.0 gram Bacon                       | Yes                                       |
| product        | string        | Ingredient name  e.g. Bacon                                  | Yes                                       |
| unit           | string        | Ingredient unit  e.g. gram                                   | Yes (pass empty  string if not available) |
| unitValue      | string        | Ingredient  quantity  e.g. 150.0                             | Yes                                       |

**Response Body Schema**

**MetaData** Object

**metaData**

| **Field Name** | **Data Type** | **Description**                                              |
| -------------- | ------------- | ------------------------------------------------------------ |
| code           | integer       | HTTP response  code, e.g. 200 for success                    |
| message        | string        | Response message.                                            |
| processedAt    | integer       | Timestamp of the  request.                                   |
| processedId    | string        | Unique ID of the  request. Can be used e.g. for technical support. |

**RegisterRecipeResponse** Object

| **Field Name** | **Data Type**                   | **Description**  |
| -------------- | ------------------------------- | ---------------- |
| recipes        | array of  RecipeResponse object | Array of Recipes |

**Recipe** Object

**RegisterRecipeResponse.recipes[]** 

| **Field Name** | **Data Type**                | **Description**                                              |
| -------------- | ---------------------------- | ------------------------------------------------------------ |
| externalId     | string                       | A unique Id for  the object, which partner uses to identify the recipe. |
| recipeId       | integer                      | A unique Id for  the object, which WMT uses to identity the recipe |
| updateTime     | Integer                      | Update Timestamp                                             |
| title          | string                       | Title of the  recipe                                         |
| description    | string                       | Description of  the recipe                                   |
| servingSize    | integer                      | Serving size of  the recipe                                  |
| ingredients    | array of  Ingredients object | Array of  Ingredients                                        |

**Ingredient** object

**RegisterRecipeResponse.recipes[].ingredients**

Each Ingredient object will contain external identifier along with the ingredient text.

| **Field Name** | **Data Type** | **Description**                                              |
| -------------- | ------------- | ------------------------------------------------------------ |
| ingredientId   | string        | A unique Id for  the object, which WMT uses to identity the ingredient |
| externalId     | string        | A unique Id for  the object, which partner uses to identify the ingredient. |
| text           | string        | Ingredient text                                              |
| product        | string        | Ingredient name                                              |
| unit           | string        | Ingredient unit                                              |
| unitValue      | string        | Ingredient quantity                                          |

### 2.3 Update Recipe API

Update partner recipes with WMT. Updating a recipe is atomic. You need to override the whole recipe. If some fields are skipped they will be deleted from a recipe.

| Request/Response  format | JSON                     |
| ------------------------ | ------------------------ |
| Method                   | POST                     |
| URI                      | /api/<partnerId>/recipes |
| Authentication           | Client access  token     |

**Request Header Parameters**

| **Header  Parameter** | **Description**      | **Mandatory** |
| --------------------- | -------------------- | ------------- |
| WM_SVC.VERSION        | 1.0.0                | No            |
| WM_CONSUMER.ID        | Walmart Consumer  ID | Yes           |
| WM_SVC.ENV            | stg, prod            | No            |
| WM_SVC.NAME           | affiliates-recipe    | No            |
| AUTHORIZATION         | Bearer  <oauthToken> | Yes           |
| Content-Type          | application/json     | Yes           |

**Request Body Schema** 

**UpdateRecipeRequest** Object

 Update Recipe request will contain an array of recipes to be registered.

| **Field Name** | **Data Type**           | **Description**  |
| -------------- | ----------------------- | ---------------- |
| recipes        | array of Recipe  object | Array of Recipes |

**Recipe** object

**UpdateRecipeRequest.recipes[]**

Each Recipe object will contain metadata of the recipe along with list of ingredients.

| **Field Name** | **Data Type**                | **Description**                                              | **Mandatory** |
| -------------- | ---------------------------- | ------------------------------------------------------------ | ------------- |
| recipeId       | integer                      | WMT unique Id for  recipe                                    | Yes           |
| externalId     | string                       | A unique Id for  the object, which partner uses to identify the recipe. | No            |
| title          | string                       | Title of the  recipe                                         | Yes           |
| description    | string                       | Description of  the recipe                                   | No            |
| servingSize    | integer                      | Serving size of  the recipe                                  | Yes           |
| ingredients    | array of  Ingredients object | Array of Ingredients                                         | Yes           |

**Ingredient** object

**UpdateRecipeRequest.recipes[].ingredients**

Each Ingredient object will contain external identifier along with the ingredient text.

 

| **Field Name** | **Data Type** | **Description**                                              | **Mandatory**                             |
| -------------- | ------------- | ------------------------------------------------------------ | ----------------------------------------- |
| ingredientId   | integer       | WMT unigue Id for  ingredient                                | Yes                                       |
| externalId     | string        | A unique Id for  the object, which partner uses to identify the ingredient. | No                                        |
| text           | string        | Ingredient text   e.g.150.0 gram Bacon                       | Yes                                       |
| product        | string        | Ingredient name  e.g. Bacon                                  | Yes                                       |
| unit           | string        | Ingredient unit  e.g. gram                                   | Yes (pass empty  string if not available) |
| unitValue      | string        | Ingredient  quantity  e.g. 150.0                             | Yes                                       |

**Response** is same as **RegisterRecipeResponse** above**.**

### 2.4 Store Locator API

Access Point Locator Service returns a list of Delivery/Pickup access point.

| Request/Response  format | JSON                    |
| ------------------------ | ----------------------- |
| Method                   | POST                    |
| URI                      | /v1/storeServiceLocator |
| Authentication           | Client access  token    |

| **Field Name** | **Data Type**  | **Description**                                              | **Valid  Values/Format**                                     | **Mandatory**                                                |
| :------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| postalCode     | string         | Postal Code is  Mandatory   Must match regex ^(\d{5}(-)?(\d{4})?)$   5 digit zip or 9 digit zip with optional hyphen | 95134     e.g.   { "payload": {     "postalCode":"95054"     }   } | Yes                                                          |
| geoCode        | GeoCode Object | GeoCode object  containing latitude and longtitude           | {       "latitude":  "64.856378",   "longitude":  "- 147.68955"   }        e.g.       {    "payload": {    "postalCode": "99701",    "geoCode": {       "latitude":  "64.856378",   "longitude":  "- 147.68955"   }   }  } | No.     Zipcode  is mandatory. Lat &   long  is   Optional.  Can pass either Zipcode or Zipcode + lat-long in the payload. |
| filter         | string         | Specify the  fulfillment types inside response               | ["DELIVERY]  /  ["INSTORE_PICKUP]  /  ["DELIVERY",  "INSTORE_PICKUP"] | Yes                                                          |

**Request Query Parameters**

| **Query/Path Parameter** | **Description**               | **Mandatory**                                                |
| ------------------------ | ----------------------------- | ------------------------------------------------------------ |
| storeLimit               | Number of  stores per request | No<br />*Defaulted to **10,** if not sent in the request*  <br />*Maximum Limit  is **20*** |

### 2.5 Recipe Products API

API to retrieve WMT Grocery Products for a given recipe.

| Request/Response  format | JSON                              |
| ------------------------ | --------------------------------- |
| Method                   | POST                              |
| URI                      | /api/<partnerId>/recipes/products |
| Authentication           | Client access  token              |

**Request Header Parameters** 

| **Header  Parameter** | **Description**      | **Mandatory** |
| --------------------- | -------------------- | ------------- |
| WM_SVC.VERSION        | 1.0.0                | No            |
| WM_CONSUMER.ID        | Walmart Consumer  ID | Yes           |
| WM_SVC.ENV            | stg, prod            | No            |
| WM_SVC.NAME           | affiliates-recipe    | No            |
| AUTHORIZATION         | Bearer  <oauthToken> | Yes           |
| Content-Type          | application/json     | Yes           |

**Request Parameters**

| **Parameter** | **Description**   | **Mandatory** |
| ------------- | ----------------- | ------------- |
| appleifa      | iOS Device ID     | No            |
| googleaid     | Android Device ID | No            |

**Request Body Schema**

Store Object: Store (or warehouse) that the user will be buying from. Determines product prices and availability.

**Request.store**

| **Field Name** | **Description**                                     | **Mandatory** |
| -------------- | --------------------------------------------------- | ------------- |
| identifier     | Unique ID  identifying WMT Grocery store  e.g. 7233 | Yes           |

Options Object: Request options

**Request.options**

| **Field Name**     | **Data Type**                        | **Description**                                              | **Mandatory** |
| ------------------ | ------------------------------------ | ------------------------------------------------------------ | ------------- |
| calculation        | string                               | Enum: “price”,  “waste”.    How product selection should be optimized, for lowest price or minimum waste. | No            |
| productSubstitutes | array of Product  Substitute objects | Products that the  user has substituted to after a previous product request. These products will  always be returned in the response. | No            |

Product Substitutes Object: 

**Request.options.productSubstitutes[]**

| **Field Name**      | **Data Type** | **Description**                                              | **Mandatory** |
| ------------------- | ------------- | ------------------------------------------------------------ | ------------- |
| offerId             | string        | WMT Offer ID that  should be returned.                       | Yes           |
| similarProductsId   | string        | Similar Product  ID for that ingredient/product from a previous product response. (e.g. [10295576]) | Yes           |
| recipeIngredientIds | integer []    | Array of Recipe Ingredient  Id(s) (e.g. [102253]) to be substituted with similarProductsId above. (e.g. [10295576]) | Yes           |

Recipe Object

**Request.recipes[]**

| **Field Name** | **Data Type** | **Description**                                              | **Mandatory**                                                |
| -------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| recipeId       | string        | WMT Recipe ID.                                               | Either recipeId  or externalId is required. RecipeId will take precedence if both are passed.     Use this field or  externalId below. |
| externalId     | string        | A unique Id for  the object, which partner uses to identify the recipe. | Either recipeId  or externalId is required. RecipeId will take precedence if both are passed.     Use this field or  recipeId above. |
| portions       | integer       | Number of  portions that products should be calculated for.      E.g. If Recipe serving size is 4 and  portions is 8. Products returned will have factored in 2x the recipe  ingredients. | Yes                                                          |

**Response Body Schema**

**MetaData** Object

**metaData**

| **Field Name** | **Data Type** | **Description**                                              |
| -------------- | ------------- | ------------------------------------------------------------ |
| code           | integer       | HTTP response  code, e.g. 200 for success                    |
| message        | string        | Response message                                             |
| processedAt    | integer       | Timestamp of the  request                                    |
| processedId    | string        | Unique ID of the  request. Can be used e.g. for technical support. |

**Payload** Object

**Products** Object Array of Array

**payload.products[].[]**

| **Field Name**            | **Data Type**            | **Description**                                              |
| ------------------------- | ------------------------ | ------------------------------------------------------------ |
| id                        | integer                  | Grocery catalog  unique identifier                           |
| averageWeight             | double                   | Product weight ,  e.g. 0.25                                  |
| brand                     | string                   | Product brand                                                |
| categories                | Categories Object  Array | Categories  Object                                           |
| common                    | boolean                  | Indicated  whether the product is considered a common/pantry product that the user  likely already has at home. These products are typically not added to cart  automatically. |
| department                | string                   | Product  department                                          |
| description               | string                   | Short description  for the item. Contains escaped html formatting tags. |
| imageURL                  | string                   | Image URL                                                    |
| thumbnailImageURL         | string                   | Thumbnail Image  URL                                         |
| maxQuantity               | integer                  | Maximum quantity  of the item that can be added to the cart  |
| offerId                   | string                   | WMT grocery offer  identifier                                |
| offerPrice                | double                   | Price for one  piece.                                        |
| ppu                       | PricePerUnit  object     | Price per unit object                                        |
| productPageURL            | string                   | Product page URL                                             |
| quantity                  | Integer                  | Quantity of  product needed                                  |
| recipeIngredientIds       | Array[] integer          | Ingredient Id(s) that  Walmart product is fulfilling (e.g. 102253) |
| salesUnit                 | double                   | Product size,  e.g. weight of one banana                     |
| salesUnitOfMeasure        | string                   | Product size  unit, e.g. grams for bananas. Can be various piece, weight or volume units,  like 'ml', 'kg', 'lbs', 'ounce', 'fl oz', etc. Units returned are from  product source data. |
| similarProductsId         | string                   | This identifier  should be used with the Similar Products endpoint to fetch alternative  products that the user can substitute to. |
| status                    | string                   | Status of the  product returned  ·     in_stock:  Product is in stock  ·     out_of_stock:  All matched products are out of stock, return ingredient details.  ·     replaced:  Product has been substituted.  ·     replaced_out_of_stock:  Product has been substituted by the user but is no longer in stock.  ·     no_match:  No products are found for this ingredient, return ingredient details. |
| title                     | string                   | Title of the  ingredient                                     |
| unitOfOrder               | String                   | Product Order  Unit e.g. each, weight                        |
| unitPriceDisplayCondition | string                   | Price per unit  e.g. ($3.96/OZ)                              |

Categories Object Array

**payload.products[].[].categories[]**

| **Field Name** | **Data Type** | **Description** |
| -------------- | ------------- | --------------- |
| id             | string        | Category  ID    |
| name           | string        | Category  Name  |

PricePerUnit Object

**payload.products[].[].ppu**

| **Field Name** | **Data Type** | **Description**   |
| -------------- | ------------- | ----------------- |
| unit           | string        | Unit of price     |
| amount         | string        | Amount value      |
| currency       | string        | Currency of Price |

Recipe Object Array

**payload.recipes[]**

| **Field Name** | **Data Type**             | **Description**                                              |
| -------------- | ------------------------- | ------------------------------------------------------------ |
| recipeId       | string                    | WMT Recipe Id                                                |
| description    | string                    | Recipe  Description                                          |
| externalId     | integer                   | A unique Id for  the object, which partner uses to identify the recipe. |
| servingSize    | string                    | Number of  portions that the recipe ingredients are specified for. |
| title          | string                    | Recipe Title                                                 |
| images         | Images Object  Array      | Recipe Images                                                |
| ingredients    | Ingredients  Object Array | Recipe  Ingredients                                          |

Images Object Array

**payload.recipes[].images[]**

| **Field Name** | **Data Type** | **Description** |
| -------------- | ------------- | --------------- |
| imageId        | integer       | Image Id        |
| type           | string        | Image Type      |
| url            | string        | Image URL       |

Ingredients Object Array

**payload.recipes[].ingredients[]**

| **Field Name** | **Data Type** | **Description**                                       |
| -------------- | ------------- | ----------------------------------------------------- |
| ingredientId   | string        | Ingredient Id                                         |
| externalId     | string        | Customer  Ingredient ID (if provided in recipe data). |
| product        |               | Ingredient name  e.g. Bacon                           |
| text           |               | Ingredient text   e.g.150.0 gram Bacon                |
| unit           |               | Ingredient Unit  e.g. gram                            |
| unitValue      |               | Ingredient Quantity  e.g. 150.0                       |

### 2.6 Similar Products API

API to retrieve Similar WMT Grocery Item Products for a given product.

| Request/Response  format | JSON                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Method                   | GET                                                          |
| URI                      | /api/<partnerId>/stores/<storeId>/products/<similarProductsId  >/similarProducts    * similarProductsId is from Recipe Products API response |
| Authentication           | Client access  token                                         |

**Request Header Parameters**

| **Header  Parameter** | **Description**       | **Mandatory** |
| --------------------- | --------------------- | ------------- |
| WM_SVC.VERSION        | 1.0.0                 | No            |
| WM_CONSUMER.ID        | Walmart Consumer  ID  | Yes           |
| WM_SVC.ENV            | stg, prod             | No            |
| WM_SVC.NAME           | affiliates-recipe     | No            |
| AUTHORIZATION         | Bearer  <oauth_token> | Yes           |
| Content-Type          | application/json      | Yes           |

**Request Parameters**

| **Parameter** | **Description**   | **Mandatory** |
| ------------- | ----------------- | ------------- |
| appleifa      | iOS Device ID     | No            |
| googleaid     | Android Device ID | No            |

**Response Body Schema**

MetaData Object

**metaData**

| **Field Name** | **Data Type** | **Description**                                              |
| -------------- | ------------- | ------------------------------------------------------------ |
| code           | integer       | HTTP response  code, e.g. 200 for success                    |
| message        | string        | Response message                                             |
| processedAt    | integer       | Timestamp of the  request                                    |
| processedId    | string        | Unique ID of the  request. Can be used e.g. for technical support. |

**Payload** Object

**Similar Products** Object Array

**payload.[]**

| **Field Name**     | **Data Type**        | **Description**                                              |
| ------------------ | -------------------- | ------------------------------------------------------------ |
| id                 | integer              | Grocery catalog  unique identifier                           |
| brand              | string               | Product brand                                                |
| offerId            | string               | WMT grocery offer  identifier                                |
| images             | Images Object  Array | Product  Images                                              |
| inStock            | boolean              | State of  the product returned.                              |
| title              | string               | Title of the  ingredient                                     |
| offerPrice         | double               | Product  Price for one piece                                 |
| salesUnit          | double               | Product  size, e.g. weight of one banana                     |
| sortOrder          | integer              | Sort Order                                                   |
| salesUnitOfMeasure | string               | Product  size unit, e.g. grams for bananas. Can be various piece, weight or volume  units, like 'ml', 'kg', 'lbs', 'ounce', 'fl oz', etc. Units returned are from  product source data. |

 Images Object Array

**payload.[].images[]**

| **Field Name** | **Data Type** | **Description** |
| -------------- | ------------- | --------------- |
| imageId        | integer       | Image Id        |
| url            | string        | Image URL       |
| type           | String        | Image Type      |

## 3 CURL Calls

### 3.1 OAuth API CURL Call

**Stage call to get OAuth token**

```shell
curl -X POST \
  https://developer.api.stg.walmart.com/api-proxy/service/identity/oauth/v1/token \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/x-www-form-urlencoded' \
  -H WM_CONSUMER.ID: <clientId>' \
  -d 'grant_type=client_credentials&client_id=<client_id>&client_secret=<client_secret>'
```

**Production call to get OAuth token**

```shell
curl -X POST \
  https://developer.api.walmart.com/api-proxy/service/identity/oauth/v1/token \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/x-www-form-urlencoded' \
  -H 'wm_consumer.id: <client_id>' \
  -d 'grant_type=client_credentials&client_id=<client_id>&client_secret=<client_secret>'
```

**OAuth Token Response will look like this:**

```json
HTTP/1.1 200 OK 
{ 
    "access_token": "2YotnFZFEjr1zCsicMWpAA9k21msk00jkkjk", 
    "token_type": "Bearer", 
    "expires_in": 3600 
} 
```

### 3.2 Register Recipe API CURL Call

Get the access_token from OAuth API and pass as API Token in API calls below:

**Register Recipe API Request**

```json
curl -L -X PUT   https://developer.api.stg.walmart.com/api-proxy/service /affiliates/recipe/v1/sidechef/recipes' \
-H 'WM_CONSUMER.ID: <clientId>' \
-H 'Content-Type: application/json' \
-H 'authorization: Bearer <oauthToken>'\ 

-d '{
  "recipes": [
    {
      "externalId": "4",
      "title": "Quiche Lorraine",
      "description": "Quiche is the perfect food for a picnic or a party as it is served cold and can be conveniently prepared ahead of time.",
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
          "unitValue": "150.0"
        },
        {
          "externalId": "ing49",
          "text": "2.0 Egg",
          "product": "Egg",
          "unit": "",
          "unitValue": "2.0"
        },
        {
          "externalId": "ing27",
          "text": "0.33 cup Parmesan Cheese",
          "product": "Parmesan Cheese",
          "unit": "cup",
          "unitValue": "0.33"
        }
      ]
    },
    {
      "externalId": "37",
      "title": "Sundried Tomato and Feta Quiche",
      "description": "This Sundried Tomato and Feta Quiche is perfect for a picnic or for a light lunch.",
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
          "externalId": "ing171",
          "text": "0.33 cup Sun-Dried Tomatoes",
          "product": "Sun-Dried Tomatoes",
          "unit": "cup",
          "unitValue": "0.33"
        },
        
        {
          "externalId": "ing27",
          "text": "0.25 cup Parmesan Cheese",
          "product": "Parmesan Cheese",
          "unit": "cup",
          "unitValue": "0.25"
        },
        {
          "externalId": "ing882",
          "text": "2.0 Large Egg",
          "product": "Large Egg",
          "unit": "",
          "unitValue": "2.0"
        }
      ]
    }
  ]
}’

```

**Response:**

```json
{
  "metaData": {
    "code": 200,
    "message": "success",
    "processedAt": 1590095884653,
    "processedId": "8a8e97b4-a1d0-4ab6-b053-b40d085ee802"
  },
  "payload": {
    "recipes": [
      {
        "description": "Quiche is the perfect food for a picnic or a party as it is served cold and can be conveniently prepared ahead of time.",
        "externalId": "4",
        "ingredients": [
          {
            "externalId": "ing2643",
            "ingredientId": 102250,
            "product": "Pie Crust",
            "text": "1.0 Pie Crust",
            "unit": "",
            "unitValue": "1.0"
          },
          {
            "externalId": "ing153",
            "ingredientId": 102251,
            "product": "Bacon",
            "text": "150.0 gram Bacon",
            "unit": "gram",
            "unitValue": "150.0"
          },
          {
            "externalId": "ing49",
            "ingredientId": 102252,
            "product": "Egg",
            "text": "2.0 Egg",
            "unit": "",
            "unitValue": "2.0"
          },
          {
            "externalId": "ing27",
            "ingredientId": 102253,
            "product": "Parmesan Cheese",
            "text": "0.33 cup Parmesan Cheese",
            "unit": "cup",
            "unitValue": "0.33"
          }
        ],
        "recipeId": 9625,
        "servingSize": 4,
        "title": "Quiche Lorraine",
        "updateTime": 1590095884082
      },
      {
        "description": "This Sundried Tomato and Feta Quiche is perfect for a picnic or for a light lunch.",
        "externalId": "37",
        "ingredients": [
          {
            "externalId": "ing2643",
            "ingredientId": 102254,
            "product": "Pie Crust",
            "text": "1.0 Pie Crust",
            "unit": "",
            "unitValue": "1.0"
          },
          {
            "externalId": "ing171",
            "ingredientId": 102255,
            "product": "Sun-Dried Tomatoes",
            "text": "0.33 cup Sun-Dried Tomatoes",
            "unit": "cup",
            "unitValue": "0.33"
          },
          {
            "externalId": "ing27",
            "ingredientId": 102256,
            "product": "Parmesan Cheese",
            "text": "0.25 cup Parmesan Cheese",
            "unit": "cup",
            "unitValue": "0.25"
          },
          {
            "externalId": "ing882",
            "ingredientId": 102257,
            "product": "Large Egg",
            "text": "2.0 Large Egg",
            "unit": "",
            "unitValue": "2.0"
          }
        ],
        "recipeId": 9626,
        "servingSize": 4,
        "title": "Sundried Tomato and Feta Quiche",
        "updateTime": 1590095884217
      }
    ]
  }
}
```



### 3.3 Update Recipe CURL Call

Get the access_token from OAuth API and pass as API Token in API calls below:

**Update Recipe API Request**

```json
curl -L -X POST  https://developer.api.stg.walmart.com/api-proxy/service /affiliates/recipe/v1/sidechef/recipes' \
-H 'WM_CONSUMER.ID: <clientId>' \
-H 'Content-Type: application/json' \
-H 'authorization: Bearer <oauthToken>'\ 
-d '{
  "recipes": [
    {
      "description": "Quiche David is the perfect food for a picnic or a party as it is served cold and can be conveniently prepared ahead of time.",
      "externalId": "4",
      "ingredients": [
        {
          "externalId": "ing2643",
          "ingredientId": 102250,
          "product": "Pie Crust",
          "text": "1.0 Pie Crust",
          "unit": "",
          "unitValue": "1.0"
        },
        {
          "externalId": "ing153",
          "ingredientId": 102251,
          "product": "Bacon",
          "text": "150.0 gram Bacon",
          "unit": "gram",
          "unitValue": "150.0"
        },
        {
          "externalId": "ing49",
          "ingredientId": 102252,
          "product": "Egg",
          "text": "2.0 Egg",
          "unit": "",
          "unitValue": "2.0"
        },
        {
          "externalId": "ing27",
          "ingredientId": 102253,
          "product": "Parmesan Cheese",
          "text": "0.33 cup Parmesan Cheese",
          "unit": "cup",
          "unitValue": "0.33"
        }
      ],
      "recipeId": 9625,
      "servingSize": 4,
      "title": "Quiche David",
      "updateTime": 1590095884082
    }
  ]
}
```

**Response:**

```json
{
  "metaData": {
    "code": 200,
    "message": "success",
    "processedAt": 1590096551796,
    "processedId": "31c7a993-e043-47a0-83d7-49b9e417aba7"
  },
  "payload": {
    "recipes": [
      {
        "description": "Quiche David is the perfect food for a picnic or a party as it is served cold and can be conveniently prepared ahead of time.",
        "externalId": "4",
        "ingredients": [
          {
            "externalId": "ing2643",
            "ingredientId": 102250,
            "product": "Pie Crust",
            "text": "1.0 Pie Crust",
            "unit": "",
            "unitValue": "1.0"
          },
          {
            "externalId": "ing153",
            "ingredientId": 102251,
            "product": "Bacon",
            "text": "150.0 gram Bacon",
            "unit": "gram",
            "unitValue": "150.0"
          },
          {
            "externalId": "ing49",
            "ingredientId": 102252,
            "product": "Egg",
            "text": "2.0 Egg",
            "unit": "",
            "unitValue": "2.0"
          },
          {
            "externalId": "ing27",
            "ingredientId": 102253,
            "product": "Parmesan Cheese",
            "text": "0.33 cup Parmesan Cheese",
            "unit": "cup",
            "unitValue": "0.33"
          }
        ],
        "recipeId": 9625,
        "servingSize": 4,
        "title": "Quiche David",
        "updateTime": 1590096551559
      }
    ]
  }
}
```



### 3.4 Store Locator API CURL Call

Get the access_token from OAuth API and pass as API Token in API calls below:

**Store Locator API – Stage**

**With zipcode only**

```json
curl -X POST \ 
  https://developer.api.stg.walmart.com/api-proxy/service/affil/storeservice/v1/storeServiceLocator?storesLimit=<storeLimit> \ 
  -H 'content-type: application/json'\ 
  -H 'wm_consumer.id: <client id>'\ 
  -H 'authorization: Bearer <API Token>'\ 
  -d ' 
{ 
  "payload": { 
    "postalCode":"95054" 
  } 
}' 
```

**With zipcode & lat-long**

```json
curl -X POST \
  https://developer.api.stg.walmart.com/api-proxy/service/affil/storeservice/v1/storeServiceLocator?storesLimit=<storeLimit> \
  -H 'content-type: application/json'\
  -H 'wm_consumer.id: <client id>'\
  -H 'authorization: Bearer <API Token>'\
  -d '
{
    "payload": {
        "postalCode": "99701",
        "geoCode": {
            "latitude": "64.856378",
            "longitude": "-147.68955"
        }
    }
}
```

**Store Locator API - Production**

**With zipcode only**

```json
curl -X POST \ 
  https://developer.api.walmart.com/api-proxy/service/affil/storeservice/v1/storeServiceLocator?storesLimit=<storeLimit> \ 
  -H 'content-type: application/json'\ 
  -H 'wm_consumer.id: <client id>'\ 
  -H 'authorization: Bearer <API Token>'\ 
  -d ' 
{ 
  "payload": { 
    "postalCode":"95054" 
  } 
}' 
```

**With zipcode & lat-long**

```json
curl -X POST \ 
  https://developer.api.walmart.com/api-proxy/service/affil/storeservice/v1/storeServiceLocator?storesLimit=<storeLimit> \ 
  -H 'content-type: application/json'\ 
  -H 'wm_consumer.id: <client id>'\ 
  -H 'authorization: Bearer <API Token>'\ 
  -d ' 
{ 
    "payload": {
        "postalCode": "99701",
        "geoCode": {
            "latitude": "64.856378",
            "longitude": "-147.68955"
        }
    }
}'
```

**Response:**

```json
{
  "status": "OK",
  "payload": {
    "accessPointList": [
      {
        "accessPointId": "b6b67c47-4cc7-46be-bc1e-ecb3801d7301",
        "accessPointName": "2119 - Home Delivery",
        "supportedTimezone": "US/Pacific",
        "distance": {
          "measurementValue": 3.5416639469977946,
          "unitOfMeasure": "Miles"
        },
        "fulfillmentStoreId": "2119",
        "fulfillmentType": "DELIVERY",
        "serviceAddress": {
          "isApoFpo": "N",
          "isPoBox": "N",
          "country": "US",
          "geoPoint": {
            "latitude": "37.431268",
            "longitude": "-121.920205"
          },
          "city": "Milpitas",
          "postalCode": "950355100",
          "addressLineOne": "301 Ranch Dr",
          "stateOrProvinceName": "CA",
          "state": "CA"
        }
      }
    ]
  }
}
```



### 3.5 Recipe Products API CURL Call

Get the access_token from OAuth API and pass as API Token in API calls below:

**Recipe Products API Request**

```json
curl -L -X POST https://developer.api.stg.walmart.com/api-proxy/service/affiliates/recipe/v1/tasty/recipes/products' \
-H 'Content-Type: application/json' \
-H 'WM_CONSUMER.ID: <clientId>' \
-H 'Authorization: Bearer <oauthToken>'\ 
-H 'X-Custom-Auth: <nfApiKey>' \
-H 'X-Custom-User: <customUser>' \
-d '{
  "store": {
    "identifier": "7233"
  },
  "options": {
    "calculation": "price"
  },
  "recipes": [
    {
      "recipeId": "9625",
      "portions": 4
    }
  ]
}'
```

**Response:**

```json
{
  "metaData": {
    "code": 200,
    "message": "success",
    "processedAt": 1590097953480,
    "processedId": "c0aba1ca-b4ef-4b37-98fe-e7a91c37b081"
  },
  "payload": {
    "products": [
      [
        {
          "averageWeight": null,
          "brand": "Kraft",
          "categories": [
            {
              "id": 1255027787061,
              "name": "Eggs & Dairy"
            },
            {
              "id": 1255027787521,
              "name": "Cheese"
            },
            {
              "id": 1255027817452,
              "name": "Crumbled, Grated & Parmesan Cheese"
            }
          ],
          "common": false,
          "consumption": null,
          "department": "Eggs & Dairy",
          "description": "*  Parmesan   cheese  ",
          "id": 10295576,
          "imageUrl": "http://i5.walmartimages.com/asr/9e3d63ba-5886-4ecb-9946-729578c77f0a_2.39c4d1bb6a55a50548c8f30ec8761dff.jpeg",
          "maxQuantity": 12,
          "offerId": "04ED98C32A2047BF89C836FE8004C422",
          "offerPrice": 3.98,
          "ppu": {
            "amount": 0.569,
            "currency": "USD",
            "unit": "each"
          },
          "productPageURL": "https://grocery.walmart.com/ip/Kraft-Shredded-Parmesan-Cheese-7-oz-Jar/10295576",
          "promoted": false,
          "promotion": null,
          "quantity": 1.0,
          "recipeIngredientIds": [
            102253
          ],
          "salesUnit": 7.0,
          "salesUnitOfMeasure": "Ounce",
          "similarProductsId": "10295576",
          "status": "in_stock",
          "thumbnailImageUrl": "http://i5.walmartimages.com/asr/9e3d63ba-5886-4ecb-9946-729578c77f0a_2.39c4d1bb6a55a50548c8f30ec8761dff.jpeg?odnHeight=100&odnWidth=100&odnBg=ffffff",
          "title": "Kraft Shredded  Parmesan   Cheese , 7  oz  Jar",
          "unitOfOrder": Each",
          "unitPriceDisplayCondition": "(56.9 cents/OZ)"
        }
      ],
      [
        {
          "averageWeight": null,
          "brand": "Just Crack an Egg",
          "categories": [
            {
              "id": 1255027787061,
              "name": "Eggs & Dairy"
            },
            {
              "id": 1255027787921,
              "name": "Eggs"
            },
            {
              "id": 1255027800521,
              "name": "Substitute Eggs"
            }
          ],
          "common": false,
          "consumption": null,
          "department": "Eggs & Dairy",
          "description": "Ore-Ida Just Crack an  Egg  Protein Packed Scramble Kit, 2.25 oz Bowl",
          "id": 960502280,
          "imageUrl": "http://i5.walmartimages.com/asr/99ab94dc-2f95-460f-ba10-89ba2c09d270_2.09590a7640fdb5353316b1e3178324f5.jpeg",
          "maxQuantity": 12,
          "offerId": "19EB3CDBAF9542E8A06BFBAD0F816F8E",
          "offerPrice": 1.98,
          "ppu": {
            "amount": 0.88,
            "currency": "USD",
            "unit": "each"
          },
          "productPageURL": "https://grocery.walmart.com/ip/Ore-Ida-Just-Crack-an-Egg-Protein-Packed-Scramble-Kit-Breakfast-Bowls-2-25-oz-Bowl/960502280",
          "promoted": false,
          "promotion": null,
          "quantity": 1.0,
          "recipeIngredientIds": [
            102252
          ],
          "salesUnit": 2.25,
          "salesUnitOfMeasure": "Ounce",
          "similarProductsId": "960502280",
          "status": "in_stock",
          "thumbnailImageUrl": "http://i5.walmartimages.com/asr/99ab94dc-2f95-460f-ba10-89ba2c09d270_2.09590a7640fdb5353316b1e3178324f5.jpeg?odnHeight=100&odnWidth=100&odnBg=ffffff",
          "title": "Ore-Ida Just Crack an  Egg  Protein Packed Scramble Kit Breakfast Bowls, 2.25 oz Bowl",
          "unitOfOrder": "Each",
          "unitPriceDisplayCondition": "(88.0 cents/OZ)"
        }
      ],
      [
        {
          "averageWeight": null,
          "brand": "Keebler",
          "categories": [
            {
              "id": 1255027787111,
              "name": "Pantry"
            },
            {
              "id": 1255027787261,
              "name": "Baking"
            },
            {
              "id": 1256653759464,
              "name": "Pie Crusts & Fillings"
            }
          ],
          "common": true,
          "consumption": null,
          "department": "Pantry",
          "description": "* America's No. 1 crumb  crust  * 2 extra servings than original Ready  Crust  ",
          "id": 10818466,
          "imageUrl": "http://i5.walmartimages.com/asr/596348e6-9606-4d85-b59c-fbafd994c0a2_1.efeb9cf3d076f720a5fcabc1e4dbe77b.jpeg",
          "maxQuantity": 12,
          "offerId": "E515B0D786024498BF944F4C016433F6",
          "offerPrice": 2.24,
          "ppu": null,
          "productPageURL": "https://grocery.walmart.com/ip/Keebler-Ready-Crust-10-Inch-Graham-Pie-Crust-9-oz/10818466",
          "promoted": false,
          "promotion": null,
          "quantity": 1.0,
          "recipeIngredientIds": [
            102250
          ],
          "salesUnit": 9.0,
          "salesUnitOfMeasure": "Ounce",
          "similarProductsAmount": 0,
          "similarProductsId": "10818466",
          "status": "in_stock",
          "thumbnailImageUrl": "http://i5.walmartimages.com/asr/596348e6-9606-4d85-b59c-fbafd994c0a2_1.efeb9cf3d076f720a5fcabc1e4dbe77b.jpeg?odnHeight=100&odnWidth=100&odnBg=ffffff",
          "title": "Keebler Ready  Crust  10 Inch Graham  Pie   Crust  9 oz",
          "unitOfOrder": "Each",
          "unitPriceDisplayCondition": "(24.9 cents/OZ)"
        }
      ],
      [
        {
          "averageWeight": null,
          "brand": "Great Value",
          "categories": [
            {
              "id": 1255027787101,
              "name": "Meat"
            },
            {
              "id": 1255027787241,
              "name": "Bacon, Hot Dogs & Sausage"
            },
            {
              "id": 1255027791129,
              "name": "Bacon"
            }
          ],
          "common": false,
          "consumption": null,
          "department": "Meat",
          "description": "* Ready to eat * U.S. inspected and passed by the Department of Agriculture * Gluten free ",
          "id": 10533738,
          "imageUrl": "http://i5.walmartimages.com/asr/b6600c1e-7be0-429c-8e2a-26a333093887_1.e04a5bac96aea6d133b3419993f5e070.jpeg",
          "maxQuantity": 12,
          "offerId": "1B3D426A35254F1EBB896320165601D5",
          "offerPrice": 3.48,
          "ppu": {
            "amount": 1.66,
            "currency": "USD",
            "unit": "each"
          },
          "productPageURL": "https://grocery.walmart.com/ip/Great-Value-Fully-Cooked-Naturally-Hickory-Smoked-Bacon-2-1-Oz/10533738",
          "promoted": false,
          "promotion": null,
          "quantity": 3.0,
          "recipeIngredientIds": [
            102251
          ],
          "salesUnit": 2.1,
          "salesUnitOfMeasure": "Ounce",
          "similarProductsId": "10533738",
          "status": "in_stock",
          "thumbnailImageUrl": "http://i5.walmartimages.com/asr/b6600c1e-7be0-429c-8e2a-26a333093887_1.e04a5bac96aea6d133b3419993f5e070.jpeg?odnHeight=100&odnWidth=100&odnBg=ffffff",
          "title": "Great Value Fully Cooked Naturally Hickory Smoked  Bacon , 2.1  Oz. ",
          "unitOfOrder": "Each",
          "unitPriceDisplayCondition": "($1.66/OZ)"
        }
      ]
    ],
    "recipes": [
      {
        "description": "Quiche David is the perfect food for a picnic or a party as it is served cold and can be conveniently prepared ahead of time.",
        "externalId": "4",
        "images": null,
        "ingredients": [
          {
            "externalId": "ing2643",
            "ingredientId": 102250,
            "product": "Pie Crust",
            "sortOrder": 0,
            "text": "1.0 Pie Crust",
            "unit": "",
            "unitValue": "1.0"
          },
          {
            "externalId": "ing153",
            "ingredientId": 102251,
            "product": "Bacon",
            "sortOrder": 0,
            "text": "150.0 gram Bacon",
            "unit": "gram",
            "unitValue": "150.0"
          },
          {
            "externalId": "ing49",
            "ingredientId": 102252,
            "product": "Egg",
            "sortOrder": 0,
            "text": "2.0 Egg",
            "unit": "",
            "unitValue": "2.0"
          },
          {
            "externalId": "ing27",
            "ingredientId": 102253,
            "product": "Parmesan Cheese",
            "sortOrder": 0,
            "text": "0.33 cup Parmesan Cheese",
            "unit": "cup",
            "unitValue": "0.33"
          }
        ],
        "recipeId": 9625,
        "servingSize": 4,
        "title": "Quiche David"
      }
    ]
  }
}
```

### 3.6 Similar Products API CURL Call

Get the access_token from OAuth API and pass as API Token in API calls below:

**Similar Products API Request**

```json
curl -L -X GET   https://developer.api.stg.walmart.com/api-proxy/service/affiliates/recipe/v1/<partnerId>/stores/<storeId>/products/<productId>/similarProducts' \
-H 'Content-Type: application/json' \
-H 'WM_CONSUMER.ID: <clientId>' \
-H 'authorization: Bearer <oauthToken>'\ 
-H 'X-Custom-Auth: <nfApiKey >' \
-H 'X-Custom-User: <customUser>
```

**Response:**

```json
{
  "metaData": {
    "code": 200,
    "message": "Success",
    "processedAt": 1590098153319,
    "processedId": "6d83f439-996f-4c89-a62e-8256bc1a86e3"
  },
  "payload": [
    {
      "brand": "Frigo",
      "id": 10307238,
      "images": [
        {
          "imageId": 0,
          "type": "default",
          "url": "https://i5.walmartimages.com/asr/a842d176-fd34-4179-8925-7001b35423d4_1.3af4f804a0e605b22b14de49a322a873.jpeg"
        }
      ],
      "inStock": true,
      "offerId": "544A9F1C967B4D5B95EBFCDFC0874720",
      "offerPrice": 2.54,
      "salesUnit": 5.0,
      "salesUnitOfMeasure": "EA",
      "sortOrder": 1,
      "title": "Frigo Shredded Parmesan 5 Oz. Cup"
    },
    {
      "brand": "Kraft",
      "id": 10295582,
      "images": [
        {
          "id": 0,
          "type": "default",
          "url": "https://i5.walmartimages.com/asr/3d986952-7f9a-4139-86f4-7db77f80344d_1.c3cf2edfc4449a85bf70bc3b6af8d21c.jpeg"
        }
      ],
      "inStock": true,
      "offerId": "A7D190C06D734CCEA59F9A8D846961AC",
      "offerPrice": 3.47,
      "salesUnit": 8.0,
      "salesUnitOfMeasure": "EA",
      "sortOrder": 2,
      "title": "Kraft Grated Parmesan and Romano Cheese, 8 oz Jar"
    },
    {
      "brand": "Great Value",
      "id": 10315402,
      "images": [
        {
          "id": 0,
          "type": "default",
          "url": "https://i5.walmartimages.com/asr/f9c97aae-f1b5-449e-8945-a7935c62c91a_1.e74dc79ddec5f38ac9b17e91364e7ef6.jpeg"
        }
      ],
      "inStock": true,
      "offerId": "60DFDCC5CC59428DBEA2D58E113BCCF9",
      "offerPrice": 2.36,
      "salesUnit": 8.0,
      "salesUnitOfMeasure": "EA",
      "sortOrder": 3,
      "title": "Great Value Grated Parmesan Cheese, 8 oz"
    }
  ]
} 
```

