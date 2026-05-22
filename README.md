\# Pathao Courier Laravel Integration



A custom Laravel integration for \*\*Pathao Courier Merchant API\*\*, built without any third-party package. This gives you full control over the API layer, making it ideal for SaaS applications or projects with specific requirements.



\---



\## ✨ Features



\- Issue \& refresh access tokens

\- Create a single order

\- Bulk order creation

\- Get order short info by consignment ID

\- Fetch city, zone, and area lists

\- Calculate delivery price

\- Manage merchant stores

\- Multi-tenant / custom config support (no `.env` dependency)



\---



\## 📋 Requirements



\- PHP >= 8.0

\- Laravel >= 9.x

\- Pathao Courier Merchant API credentials



\---



\## 🌐 Environment Reference



\### 🧪 Sandbox / Test Environment



Use this environment for development and testing. No real orders are placed.





| Field           | Value                                      |

| --------------- | ------------------------------------------ |

| `base\_url`      | `https://courier-api-sandbox.pathao.com`   |

| `client\_id`     | `7N1aMJQbWm`                               |

| `client\_secret` | `wRcaibZkUdSNz2EI9ZyuXLlNrnAv0TdPUPXMnD39` |

| `username`      | `test@pathao.com`                          |

| `password`      | `lovePathao`                               |

| `grant\_type`    | `password`                                 |





\### 🚀 Production / Live Environment



Use this environment for real orders. Obtain credentials from your Pathao merchant dashboard.





| Field           | Value                                             |

| --------------- | ------------------------------------------------- |

| `base\_url`      | `https://api-hermes.pathao.com`                   |

| `client\_id`     | Available in \*\*Merchant API Credentials\*\* section |

| `client\_secret` | Available in \*\*Merchant API Credentials\*\* section |

| `username`      | Your merchant account email                       |

| `password`      | Your merchant account password                    |





\---



\## ⚙️ Configuration



\### 1. Add credentials to `.env`



\*\*For Sandbox:\*\*



```env

PATHAO\_BASE\_URL=https://courier-api-sandbox.pathao.com

PATHAO\_CLIENT\_ID=7N1aMJQbWm

PATHAO\_CLIENT\_SECRET=wRcaibZkUdSNz2EI9ZyuXLlNrnAv0TdPUPXMnD39

PATHAO\_USERNAME=test@pathao.com

PATHAO\_PASSWORD=lovePathao

```



\*\*For Production:\*\*



```env

PATHAO\_BASE\_URL=https://api-hermes.pathao.com

PATHAO\_CLIENT\_ID=your-client-id

PATHAO\_CLIENT\_SECRET=your-client-secret

PATHAO\_USERNAME=your-email@example.com

PATHAO\_PASSWORD=your-password

```



\### 2. Create the config file



Create `config/pathao.php`:



```php

<?php



return \[

&#x20;   'base\_url'      => env('PATHAO\_BASE\_URL', 'https://api-hermes.pathao.com'),

&#x20;   'client\_id'     => env('PATHAO\_CLIENT\_ID', ''),

&#x20;   'client\_secret' => env('PATHAO\_CLIENT\_SECRET', ''),

&#x20;   'username'      => env('PATHAO\_USERNAME', ''),

&#x20;   'password'      => env('PATHAO\_PASSWORD', ''),

];

```



\---



\## 🛠️ Service Class



Create `app/Services/PathaoCourierService.php`:



```php

<?php



namespace App\\Services;



use Illuminate\\Support\\Facades\\Http;

use Illuminate\\Support\\Facades\\Cache;



class PathaoCourierService

{

&#x20;   protected string $baseUrl;

&#x20;   protected string $clientId;

&#x20;   protected string $clientSecret;

&#x20;   protected string $username;

&#x20;   protected string $password;



&#x20;   public function \_\_construct(

&#x20;       string $clientId     = null,

&#x20;       string $clientSecret = null,

&#x20;       string $username     = null,

&#x20;       string $password     = null,

&#x20;       string $baseUrl      = null

&#x20;   ) {

&#x20;       $this->baseUrl      = $baseUrl      ?? config('pathao.base\_url');

&#x20;       $this->clientId     = $clientId     ?? config('pathao.client\_id');

&#x20;       $this->clientSecret = $clientSecret ?? config('pathao.client\_secret');

&#x20;       $this->username     = $username     ?? config('pathao.username');

&#x20;       $this->password     = $password     ?? config('pathao.password');

&#x20;   }



&#x20;   // Override credentials at runtime (SaaS / multi-tenant)

&#x20;   public static function withConfig(

&#x20;       string $clientId,

&#x20;       string $clientSecret,

&#x20;       string $username,

&#x20;       string $password,

&#x20;       string $baseUrl = null

&#x20;   ): static {

&#x20;       return new static($clientId, $clientSecret, $username, $password, $baseUrl);

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Get Access Token (auto-cached)

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   protected function getAccessToken(): string

&#x20;   {

&#x20;       return Cache::remember('pathao\_access\_token', 432000, function () {

&#x20;           $response = $this->issueToken();

&#x20;           return $response\['access\_token'] ?? '';

&#x20;       });

&#x20;   }



&#x20;   protected function headers(): array

&#x20;   {

&#x20;       return \[

&#x20;           'Authorization' => 'Bearer ' . $this->getAccessToken(),

&#x20;           'Content-Type'  => 'application/json',

&#x20;           'Accept'        => 'application/json',

&#x20;       ];

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Issue Access Token

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function issueToken(): array

&#x20;   {

&#x20;       return Http::post($this->baseUrl . '/aladdin/api/v1/issue-token', \[

&#x20;           'client\_id'     => $this->clientId,

&#x20;           'client\_secret' => $this->clientSecret,

&#x20;           'grant\_type'    => 'password',

&#x20;           'username'      => $this->username,

&#x20;           'password'      => $this->password,

&#x20;       ])->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Refresh Access Token

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function refreshToken(string $refreshToken): array

&#x20;   {

&#x20;       return Http::post($this->baseUrl . '/aladdin/api/v1/issue-token', \[

&#x20;           'client\_id'     => $this->clientId,

&#x20;           'client\_secret' => $this->clientSecret,

&#x20;           'grant\_type'    => 'refresh\_token',

&#x20;           'refresh\_token' => $refreshToken,

&#x20;       ])->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Create a New Store

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function createStore(array $data): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->post($this->baseUrl . '/aladdin/api/v1/stores', $data)

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Get Merchant Stores

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function getStores(): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->get($this->baseUrl . '/aladdin/api/v1/stores')

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Place Single Order

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function placeOrder(array $data): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->post($this->baseUrl . '/aladdin/api/v1/orders', $data)

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Bulk Order Creation

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function bulkCreateOrders(array $orders): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->post($this->baseUrl . '/aladdin/api/v1/orders/bulk', \['orders' => $orders])

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Get Order Short Info by Consignment ID

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function getOrderInfo(string $consignmentId): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->get($this->baseUrl . '/aladdin/api/v1/orders/' . $consignmentId . '/info')

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Get City List

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function getCities(): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->get($this->baseUrl . '/aladdin/api/v1/city-list')

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Get Zones by City ID

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function getZones(int $cityId): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->get($this->baseUrl . '/aladdin/api/v1/cities/' . $cityId . '/zone-list')

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Get Areas by Zone ID

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function getAreas(int $zoneId): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->get($this->baseUrl . '/aladdin/api/v1/zones/' . $zoneId . '/area-list')

&#x20;           ->json();

&#x20;   }



&#x20;   /\*

&#x20;   |--------------------------------------------------------------------------

&#x20;   | Price Calculation

&#x20;   |--------------------------------------------------------------------------

&#x20;   \*/

&#x20;   public function calculatePrice(array $data): array

&#x20;   {

&#x20;       return Http::withHeaders($this->headers())

&#x20;           ->post($this->baseUrl . '/aladdin/api/v1/merchant/price-plan', $data)

&#x20;           ->json();

&#x20;   }

}

```



\---



\## 📖 Usage Examples



\### 🔑 1. Issue an Access Token



```php

use App\\Services\\PathaoCourierService;



$pathao = new PathaoCourierService();



$response = $pathao->issueToken();

```



\*\*Response:\*\*



```json

{

&#x20;   "token\_type": "Bearer",

&#x20;   "expires\_in": 432000,

&#x20;   "access\_token": "ISSUED\_ACCESS\_TOKEN",

&#x20;   "refresh\_token": "ISSUED\_REFRESH\_TOKEN"

}

```



\---



\### 🔄 2. Refresh Access Token



```php

$response = $pathao->refreshToken('YOUR\_REFRESH\_TOKEN');

```



\*\*Response:\*\*



```json

{

&#x20;   "token\_type": "Bearer",

&#x20;   "expires\_in": 432000,

&#x20;   "access\_token": "NEW\_ACCESS\_TOKEN",

&#x20;   "refresh\_token": "NEW\_REFRESH\_TOKEN"

}

```



\---



\### 🏪 3. Create a New Store



> \*\*💡 Tip:\*\* You need valid `city\_id`, `zone\_id`, and `area\_id` before creating a store.

> Use `getCities()` → `getZones($city\_id)` → `getAreas($zone\_id)` to fetch them in order.



```php

use App\\Services\\PathaoCourierService;



$pathao = new PathaoCourierService();



$response = $pathao->createStore(\[

&#x20;   'name'              => 'Demo Store',    // required  | string  | Store name, 3–50 characters

&#x20;   'contact\_name'      => 'Test Merchant', // required  | string  | Contact person name, 3–50 characters

&#x20;   'contact\_number'    => '017XXXXXXXX',   // required  | string  | Must be exactly 11 characters

&#x20;   'secondary\_contact' => '015XXXXXXXX',   // nullable  | string  | Must be exactly 11 characters

&#x20;   'otp\_number'        => '017XXXXXXXX',   // nullable  | string  | Order OTP will be sent to this number

&#x20;   'address'           => 'House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh', // required | string | 15–120 characters

&#x20;   'city\_id'           => 1,               // required  | integer | From getCities()

&#x20;   'zone\_id'           => 298,             // required  | integer | From getZones($city\_id)

&#x20;   'area\_id'           => 37,              // required  | integer | From getAreas($zone\_id)

]);

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "Store created successfully, Please wait one hour for approval.",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "store\_name": "Demo Store"

&#x20;   }

}

```



> \*\*⏳ Note:\*\* After creation, the store requires up to \*\*1 hour\*\* for admin approval before it becomes active.



\---



\### 📬 4. Place a Single Order



```php

$response = $pathao->placeOrder(\[

&#x20;   'store\_id'                  => 12345,        // required  | integer  | Your merchant store ID (sets pickup location)

&#x20;   'merchant\_order\_id'         => 'ORD-001',    // nullable  | string   | Your own order tracking ID

&#x20;   'recipient\_name'            => 'John Doe',   // required  | string   | 3–100 characters

&#x20;   'recipient\_phone'           => '017XXXXXXXX',// required  | string   | Must be 11 characters

&#x20;   'recipient\_secondary\_phone' => '015XXXXXXXX',// nullable  | string   | Must be 11 characters

&#x20;   'recipient\_address'         => 'House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh', // required | string | 10–220 characters

&#x20;   'recipient\_city'            => 1,            // nullable  | integer  | city\_id from getCities(); auto-detected if omitted

&#x20;   'recipient\_zone'            => 298,          // nullable  | integer  | zone\_id from getZones(); auto-detected if omitted

&#x20;   'recipient\_area'            => 37,           // nullable  | integer  | area\_id from getAreas(); auto-detected if omitted

&#x20;   'delivery\_type'             => 48,           // required  | integer  | 48 = Normal Delivery, 12 = On Demand Delivery

&#x20;   'item\_type'                 => 2,            // required  | integer  | 1 = Document, 2 = Parcel

&#x20;   'item\_quantity'             => 1,            // required  | integer  | Number of parcels

&#x20;   'item\_weight'               => 0.5,          // required  | float    | Min: 0.5 kg — Max: 10 kg

&#x20;   'amount\_to\_collect'         => 900,          // required  | integer  | COD amount; use 0 for non-COD orders

&#x20;   'item\_description'          => 'Clothing item, price 900', // nullable | string | Brief description of the parcel

&#x20;   'special\_instruction'       => 'Handle with care',         // nullable | string | Delivery instructions for the courier

]);

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "Order Created Successfully",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "consignment\_id": "ORDER\_CONSIGNMENT\_ID",

&#x20;       "merchant\_order\_id": "ORD-001",

&#x20;       "order\_status": "Pending",

&#x20;       "delivery\_fee": 80

&#x20;   }

}

```



> \*\*Delivery Type:\*\* `48` = Normal Delivery, `12` = On Demand Delivery

> \*\*Item Type:\*\* `1` = Document, `2` = Parcel



\---



\### 📦 5. Bulk Order Creation



```php

// Each object in the array follows the same field rules as a single order.

$orders = \[

&#x20;   \[

&#x20;       'store\_id'                  => 12345,

&#x20;       'merchant\_order\_id'         => 'ORD-001',

&#x20;       'recipient\_name'            => 'John Doe',

&#x20;       'recipient\_phone'           => '017XXXXXXXX',

&#x20;       'recipient\_secondary\_phone' => null,

&#x20;       'recipient\_address'         => 'House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh',

&#x20;       'recipient\_city'            => 1,

&#x20;       'recipient\_zone'            => 298,

&#x20;       'recipient\_area'            => 37,

&#x20;       'delivery\_type'             => 48,

&#x20;       'item\_type'                 => 2,

&#x20;       'item\_quantity'             => 2,

&#x20;       'item\_weight'               => 0.5,

&#x20;       'amount\_to\_collect'         => 100,

&#x20;       'item\_description'          => 'Clothing item, price 3000',

&#x20;       'special\_instruction'       => 'Do not put water',

&#x20;   ],

&#x20;   \[

&#x20;       'store\_id'                  => 12345,

&#x20;       'merchant\_order\_id'         => 'ORD-002',

&#x20;       'recipient\_name'            => 'Jane Smith',

&#x20;       'recipient\_phone'           => '015XXXXXXXX',

&#x20;       'recipient\_secondary\_phone' => null,

&#x20;       'recipient\_address'         => 'House 3, Road 14, Dhanmondi, Dhaka-1205, Bangladesh',

&#x20;       'recipient\_city'            => null,

&#x20;       'recipient\_zone'            => null,

&#x20;       'recipient\_area'            => null,

&#x20;       'delivery\_type'             => 48,

&#x20;       'item\_type'                 => 2,

&#x20;       'item\_quantity'             => 1,

&#x20;       'item\_weight'               => 0.5,

&#x20;       'amount\_to\_collect'         => 200,

&#x20;       'item\_description'          => 'Food item, price 1000',

&#x20;       'special\_instruction'       => 'Deliver before 5 pm',

&#x20;   ],

];



$response = $pathao->bulkCreateOrders($orders);

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "Your bulk order creation request is accepted, please wait some time to complete order creation.",

&#x20;   "type": "success",

&#x20;   "code": 202,

&#x20;   "data": true

}

```



\---



\### 🔍 6. Get Order Info



```php

$response = $pathao->getOrderInfo('YOUR\_CONSIGNMENT\_ID');

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "Order info",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "consignment\_id": "YOUR\_CONSIGNMENT\_ID",

&#x20;       "merchant\_order\_id": "ORD-001",

&#x20;       "order\_status": "Pending",

&#x20;       "order\_status\_slug": "Pending",

&#x20;       "updated\_at": "2024-11-20 15:11:40",

&#x20;       "invoice\_id": null

&#x20;   }

}

```



\---



\### 🌆 7. Get City List



```php

$response = $pathao->getCities();

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "City successfully fetched.",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "data": \[

&#x20;           { "city\_id": 1, "city\_name": "Dhaka" },

&#x20;           { "city\_id": 2, "city\_name": "Chittagong" },

&#x20;           { "city\_id": 4, "city\_name": "Rajshahi" }

&#x20;       ]

&#x20;   }

}

```



\---



\### 🗺️ 8. Get Zones by City



```php

$response = $pathao->getZones(1); // city\_id = 1 (Dhaka)

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "Zone list fetched.",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "data": \[

&#x20;           { "zone\_id": 298, "zone\_name": "60 feet" },

&#x20;           { "zone\_id": 1070, "zone\_name": "Abdullahpur Uttara" }

&#x20;       ]

&#x20;   }

}

```



\---



\### 📍 9. Get Areas by Zone



```php

$response = $pathao->getAreas(298); // zone\_id = 298

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "Area list fetched.",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "data": \[

&#x20;           { "area\_id": 37, "area\_name": "Bonolota", "home\_delivery\_available": true, "pickup\_available": true },

&#x20;           { "area\_id": 3, "area\_name": "Road 03", "home\_delivery\_available": true, "pickup\_available": true }

&#x20;       ]

&#x20;   }

}

```



\---



\### 💰 10. Price Calculation



```php

$response = $pathao->calculatePrice(\[

&#x20;   'store\_id'       => 12345, // required  | integer | Your merchant store ID (sets pickup location)

&#x20;   'item\_type'      => 2,     // required  | integer | 1 = Document, 2 = Parcel

&#x20;   'delivery\_type'  => 48,    // required  | integer | 48 = Normal Delivery, 12 = On Demand Delivery

&#x20;   'item\_weight'    => 0.5,   // required  | float   | Min: 0.5 kg — Max: 10 kg

&#x20;   'recipient\_city' => 1,     // required  | integer | city\_id from getCities()

&#x20;   'recipient\_zone' => 298,   // required  | integer | zone\_id from getZones($city\_id)

]);

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "price",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "price": 80,

&#x20;       "discount": 0,

&#x20;       "promo\_discount": 0,

&#x20;       "plan\_id": 69,

&#x20;       "cod\_enabled": 1,

&#x20;       "cod\_percentage": 0.01,

&#x20;       "additional\_charge": 0,

&#x20;       "final\_price": 80

&#x20;   }

}

```



\---



\### 🏬 11. Get Merchant Store Info



```php

$response = $pathao->getStores();

```



\*\*Response:\*\*



```json

{

&#x20;   "message": "Store list fetched.",

&#x20;   "type": "success",

&#x20;   "code": 200,

&#x20;   "data": {

&#x20;       "data": \[

&#x20;           {

&#x20;               "store\_id": 12345,

&#x20;               "store\_name": "Demo Store",

&#x20;               "store\_address": "House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh",

&#x20;               "is\_active": 1,

&#x20;               "city\_id": 1,

&#x20;               "zone\_id": 298,

&#x20;               "hub\_id": 5,

&#x20;               "is\_default\_store": false,

&#x20;               "is\_default\_return\_store": false

&#x20;           }

&#x20;       ],

&#x20;       "total": 1,

&#x20;       "current\_page": 1,

&#x20;       "per\_page": 1000,

&#x20;       "last\_page": 1

&#x20;   }

}

```



\---



\## 🏢 Multi-Tenant / SaaS Usage



Override credentials at runtime without touching `.env`:



```php

$pathao = PathaoCourierService::withConfig(

&#x20;   clientId:     'tenant-client-id',

&#x20;   clientSecret: 'tenant-client-secret',

&#x20;   username:     'tenant@email.com',

&#x20;   password:     'tenant-password'

);



$response = $pathao->placeOrder(\[...]);

```



To target the sandbox environment for a specific tenant:



```php

$pathao = PathaoCourierService::withConfig(

&#x20;   clientId:     '7N1aMJQbWm',

&#x20;   clientSecret: 'wRcaibZkUdSNz2EI9ZyuXLlNrnAv0TdPUPXMnD39',

&#x20;   username:     'test@pathao.com',

&#x20;   password:     'lovePathao',

&#x20;   baseUrl:      'https://courier-api-sandbox.pathao.com'

);

```



\---



\## 📦 Order Status Reference





| Status                | Description                         |

| --------------------- | ----------------------------------- |

| `Pending`             | Order placed, not yet picked up     |

| `Pickup\_Requested`    | Pickup has been requested           |

| `Picked`              | Parcel picked up from merchant      |

| `In\_Transit`          | Parcel is on its way                |

| `Delivered`           | Successfully delivered to recipient |

| `Partially\_Delivered` | Partially delivered                 |

| `Cancelled`           | Order cancelled                     |

| `Hold`                | Order is on hold                    |

| `Return`              | Parcel is being returned            |





\---



\## 🚀 Delivery Type Reference





| Code | Description        |

| ---- | ------------------ |

| `48` | Normal Delivery    |

| `12` | On Demand Delivery |





\---



\## 📦 Item Type Reference





| Code | Description |

| ---- | ----------- |

| `1`  | Document    |

| `2`  | Parcel      |







