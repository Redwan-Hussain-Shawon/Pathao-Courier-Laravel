# Pathao Courier Laravel Integration

A custom Laravel integration for **Pathao Courier Merchant API**, built without any third-party package. This gives you full control over the API layer, making it ideal for SaaS applications or projects with specific requirements.

---

## ✨ Features

- Issue & refresh access tokens
- Create a single order
- Bulk order creation
- Get order short info by consignment ID
- Fetch city, zone, and area lists
- Calculate delivery price
- Manage merchant stores
- Multi-tenant / custom config support (no `.env` dependency)

---

## 📋 Requirements

- PHP >= 8.0
- Laravel >= 9.x
- Pathao Courier Merchant API credentials

---

## 🌐 Environment Reference

### 🧪 Sandbox / Test Environment

Use this environment for development and testing. No real orders are placed.


| Field           | Value                                      |
| --------------- | ------------------------------------------ |
| `base_url`      | `https://courier-api-sandbox.pathao.com`   |
| `client_id`     | `7N1aMJQbWm`                               |
| `client_secret` | `wRcaibZkUdSNz2EI9ZyuXLlNrnAv0TdPUPXMnD39` |
| `username`      | `test@pathao.com`                          |
| `password`      | `lovePathao`                               |
| `grant_type`    | `password`                                 |


### 🚀 Production / Live Environment

Use this environment for real orders. Obtain credentials from your Pathao merchant dashboard.


| Field           | Value                                             |
| --------------- | ------------------------------------------------- |
| `base_url`      | `https://api-hermes.pathao.com`                   |
| `client_id`     | Available in **Merchant API Credentials** section |
| `client_secret` | Available in **Merchant API Credentials** section |
| `username`      | Your merchant account email                       |
| `password`      | Your merchant account password                    |


---

## ⚙️ Configuration

### 1. Add credentials to `.env`

**For Sandbox:**

```env
PATHAO_BASE_URL=https://courier-api-sandbox.pathao.com
PATHAO_CLIENT_ID=7N1aMJQbWm
PATHAO_CLIENT_SECRET=wRcaibZkUdSNz2EI9ZyuXLlNrnAv0TdPUPXMnD39
PATHAO_USERNAME=test@pathao.com
PATHAO_PASSWORD=lovePathao
```

**For Production:**

```env
PATHAO_BASE_URL=https://api-hermes.pathao.com
PATHAO_CLIENT_ID=your-client-id
PATHAO_CLIENT_SECRET=your-client-secret
PATHAO_USERNAME=your-email@example.com
PATHAO_PASSWORD=your-password
```

### 2. Create the config file

Create `config/pathao.php`:

```php
<?php

return [
    'base_url'      => env('PATHAO_BASE_URL', 'https://api-hermes.pathao.com'),
    'client_id'     => env('PATHAO_CLIENT_ID', ''),
    'client_secret' => env('PATHAO_CLIENT_SECRET', ''),
    'username'      => env('PATHAO_USERNAME', ''),
    'password'      => env('PATHAO_PASSWORD', ''),
];
```

---

## 🛠️ Service Class

Create `app/Services/PathaoCourierService.php`:

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Cache;

class PathaoCourierService
{
    protected string $baseUrl;
    protected string $clientId;
    protected string $clientSecret;
    protected string $username;
    protected string $password;

    public function __construct(
        string $clientId     = null,
        string $clientSecret = null,
        string $username     = null,
        string $password     = null,
        string $baseUrl      = null
    ) {
        $this->baseUrl      = $baseUrl      ?? config('pathao.base_url');
        $this->clientId     = $clientId     ?? config('pathao.client_id');
        $this->clientSecret = $clientSecret ?? config('pathao.client_secret');
        $this->username     = $username     ?? config('pathao.username');
        $this->password     = $password     ?? config('pathao.password');
    }

    // Override credentials at runtime (SaaS / multi-tenant)
    public static function withConfig(
        string $clientId,
        string $clientSecret,
        string $username,
        string $password,
        string $baseUrl = null
    ): static {
        return new static($clientId, $clientSecret, $username, $password, $baseUrl);
    }

    /*
    |--------------------------------------------------------------------------
    | Get Access Token (auto-cached)
    |--------------------------------------------------------------------------
    */
protected function getAccessToken(): string
{
    $cacheKey = 'pathao_access_token_' . md5(
        $this->clientId . '_' . $this->username
    );

    return Cache::remember($cacheKey, now()->addDays(5), function () {

        $response = $this->issueToken();

        if (!isset($response['access_token'])) {
            throw new \Exception('Pathao token generation failed');
        }

        return $response['access_token'];
    });
}

    protected function headers(): array
    {
        return [
            'Authorization' => 'Bearer ' . $this->getAccessToken(),
            'Content-Type'  => 'application/json',
            'Accept'        => 'application/json',
        ];
    }

    /*
    |--------------------------------------------------------------------------
    | Issue Access Token
    |--------------------------------------------------------------------------
    */
    public function issueToken(): array
    {
        return Http::post($this->baseUrl . '/aladdin/api/v1/issue-token', [
            'client_id'     => $this->clientId,
            'client_secret' => $this->clientSecret,
            'grant_type'    => 'password',
            'username'      => $this->username,
            'password'      => $this->password,
        ])->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Refresh Access Token
    |--------------------------------------------------------------------------
    */
    public function refreshToken(string $refreshToken): array
    {
        return Http::post($this->baseUrl . '/aladdin/api/v1/issue-token', [
            'client_id'     => $this->clientId,
            'client_secret' => $this->clientSecret,
            'grant_type'    => 'refresh_token',
            'refresh_token' => $refreshToken,
        ])->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Create a New Store
    |--------------------------------------------------------------------------
    */
    public function createStore(array $data): array
    {
        return Http::withHeaders($this->headers())
            ->post($this->baseUrl . '/aladdin/api/v1/stores', $data)
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Get Merchant Stores
    |--------------------------------------------------------------------------
    */
    public function getStores(): array
    {
        return Http::withHeaders($this->headers())
            ->get($this->baseUrl . '/aladdin/api/v1/stores')
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Place Single Order
    |--------------------------------------------------------------------------
    */
    public function placeOrder(array $data): array
    {
        return Http::withHeaders($this->headers())
            ->post($this->baseUrl . '/aladdin/api/v1/orders', $data)
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Bulk Order Creation
    |--------------------------------------------------------------------------
    */
    public function bulkCreateOrders(array $orders): array
    {
        return Http::withHeaders($this->headers())
            ->post($this->baseUrl . '/aladdin/api/v1/orders/bulk', ['orders' => $orders])
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Get Order Short Info by Consignment ID
    |--------------------------------------------------------------------------
    */
    public function getOrderInfo(string $consignmentId): array
    {
        return Http::withHeaders($this->headers())
            ->get($this->baseUrl . '/aladdin/api/v1/orders/' . $consignmentId . '/info')
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Get City List
    |--------------------------------------------------------------------------
    */
    public function getCities(): array
    {
        return Http::withHeaders($this->headers())
            ->get($this->baseUrl . '/aladdin/api/v1/city-list')
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Get Zones by City ID
    |--------------------------------------------------------------------------
    */
    public function getZones(int $cityId): array
    {
        return Http::withHeaders($this->headers())
            ->get($this->baseUrl . '/aladdin/api/v1/cities/' . $cityId . '/zone-list')
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Get Areas by Zone ID
    |--------------------------------------------------------------------------
    */
    public function getAreas(int $zoneId): array
    {
        return Http::withHeaders($this->headers())
            ->get($this->baseUrl . '/aladdin/api/v1/zones/' . $zoneId . '/area-list')
            ->json();
    }

    /*
    |--------------------------------------------------------------------------
    | Price Calculation
    |--------------------------------------------------------------------------
    */
    public function calculatePrice(array $data): array
    {
        return Http::withHeaders($this->headers())
            ->post($this->baseUrl . '/aladdin/api/v1/merchant/price-plan', $data)
            ->json();
    }
}
```

---

## 📖 Usage Examples

### 🔑 1. Issue an Access Token

```php
use App\Services\PathaoCourierService;

$pathao = new PathaoCourierService();

$response = $pathao->issueToken();
```

**Response:**

```json
{
    "token_type": "Bearer",
    "expires_in": 432000,
    "access_token": "ISSUED_ACCESS_TOKEN",
    "refresh_token": "ISSUED_REFRESH_TOKEN"
}
```

---

### 🔄 2. Refresh Access Token

```php
$response = $pathao->refreshToken('YOUR_REFRESH_TOKEN');
```

**Response:**

```json
{
    "token_type": "Bearer",
    "expires_in": 432000,
    "access_token": "NEW_ACCESS_TOKEN",
    "refresh_token": "NEW_REFRESH_TOKEN"
}
```

---

### 🏪 3. Create a New Store

> **💡 Tip:** You need valid `city_id`, `zone_id`, and `area_id` before creating a store.
> Use `getCities()` → `getZones($city_id)` → `getAreas($zone_id)` to fetch them in order.

```php
use App\Services\PathaoCourierService;

$pathao = new PathaoCourierService();

$response = $pathao->createStore([
    'name'              => 'Demo Store',    // required  | string  | Store name, 3–50 characters
    'contact_name'      => 'Test Merchant', // required  | string  | Contact person name, 3–50 characters
    'contact_number'    => '017XXXXXXXX',   // required  | string  | Must be exactly 11 characters
    'secondary_contact' => '015XXXXXXXX',   // nullable  | string  | Must be exactly 11 characters
    'otp_number'        => '017XXXXXXXX',   // nullable  | string  | Order OTP will be sent to this number
    'address'           => 'House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh', // required | string | 15–120 characters
    'city_id'           => 1,               // required  | integer | From getCities()
    'zone_id'           => 298,             // required  | integer | From getZones($city_id)
    'area_id'           => 37,              // required  | integer | From getAreas($zone_id)
]);
```

**Response:**

```json
{
    "message": "Store created successfully, Please wait one hour for approval.",
    "type": "success",
    "code": 200,
    "data": {
        "store_name": "Demo Store"
    }
}
```

> **⏳ Note:** After creation, the store requires up to **1 hour** for admin approval before it becomes active.

---

### 📬 4. Place a Single Order

```php
$response = $pathao->placeOrder([
    'store_id'                  => 12345,        // required  | integer  | Your merchant store ID (sets pickup location)
    'merchant_order_id'         => 'ORD-001',    // nullable  | string   | Your own order tracking ID
    'recipient_name'            => 'John Doe',   // required  | string   | 3–100 characters
    'recipient_phone'           => '017XXXXXXXX',// required  | string   | Must be 11 characters
    'recipient_secondary_phone' => '015XXXXXXXX',// nullable  | string   | Must be 11 characters
    'recipient_address'         => 'House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh', // required | string | 10–220 characters
    'recipient_city'            => 1,            // nullable  | integer  | city_id from getCities(); auto-detected if omitted
    'recipient_zone'            => 298,          // nullable  | integer  | zone_id from getZones(); auto-detected if omitted
    'recipient_area'            => 37,           // nullable  | integer  | area_id from getAreas(); auto-detected if omitted
    'delivery_type'             => 48,           // required  | integer  | 48 = Normal Delivery, 12 = On Demand Delivery
    'item_type'                 => 2,            // required  | integer  | 1 = Document, 2 = Parcel
    'item_quantity'             => 1,            // required  | integer  | Number of parcels
    'item_weight'               => 0.5,          // required  | float    | Min: 0.5 kg — Max: 10 kg
    'amount_to_collect'         => 900,          // required  | integer  | COD amount; use 0 for non-COD orders
    'item_description'          => 'Clothing item, price 900', // nullable | string | Brief description of the parcel
    'special_instruction'       => 'Handle with care',         // nullable | string | Delivery instructions for the courier
]);
```

**Response:**

```json
{
    "message": "Order Created Successfully",
    "type": "success",
    "code": 200,
    "data": {
        "consignment_id": "ORDER_CONSIGNMENT_ID",
        "merchant_order_id": "ORD-001",
        "order_status": "Pending",
        "delivery_fee": 80
    }
}
```

> **Delivery Type:** `48` = Normal Delivery, `12` = On Demand Delivery
> **Item Type:** `1` = Document, `2` = Parcel

---

### 📦 5. Bulk Order Creation

```php
// Each object in the array follows the same field rules as a single order.
$orders = [
    [
        'store_id'                  => 12345,
        'merchant_order_id'         => 'ORD-001',
        'recipient_name'            => 'John Doe',
        'recipient_phone'           => '017XXXXXXXX',
        'recipient_secondary_phone' => null,
        'recipient_address'         => 'House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh',
        'recipient_city'            => 1,
        'recipient_zone'            => 298,
        'recipient_area'            => 37,
        'delivery_type'             => 48,
        'item_type'                 => 2,
        'item_quantity'             => 2,
        'item_weight'               => 0.5,
        'amount_to_collect'         => 100,
        'item_description'          => 'Clothing item, price 3000',
        'special_instruction'       => 'Do not put water',
    ],
    [
        'store_id'                  => 12345,
        'merchant_order_id'         => 'ORD-002',
        'recipient_name'            => 'Jane Smith',
        'recipient_phone'           => '015XXXXXXXX',
        'recipient_secondary_phone' => null,
        'recipient_address'         => 'House 3, Road 14, Dhanmondi, Dhaka-1205, Bangladesh',
        'recipient_city'            => null,
        'recipient_zone'            => null,
        'recipient_area'            => null,
        'delivery_type'             => 48,
        'item_type'                 => 2,
        'item_quantity'             => 1,
        'item_weight'               => 0.5,
        'amount_to_collect'         => 200,
        'item_description'          => 'Food item, price 1000',
        'special_instruction'       => 'Deliver before 5 pm',
    ],
];

$response = $pathao->bulkCreateOrders($orders);
```

**Response:**

```json
{
    "message": "Your bulk order creation request is accepted, please wait some time to complete order creation.",
    "type": "success",
    "code": 202,
    "data": true
}
```

---

### 🔍 6. Get Order Info

```php
$response = $pathao->getOrderInfo('YOUR_CONSIGNMENT_ID');
```

**Response:**

```json
{
    "message": "Order info",
    "type": "success",
    "code": 200,
    "data": {
        "consignment_id": "YOUR_CONSIGNMENT_ID",
        "merchant_order_id": "ORD-001",
        "order_status": "Pending",
        "order_status_slug": "Pending",
        "updated_at": "2024-11-20 15:11:40",
        "invoice_id": null
    }
}
```

---

### 🌆 7. Get City List

```php
$response = $pathao->getCities();
```

**Response:**

```json
{
    "message": "City successfully fetched.",
    "type": "success",
    "code": 200,
    "data": {
        "data": [
            { "city_id": 1, "city_name": "Dhaka" },
            { "city_id": 2, "city_name": "Chittagong" },
            { "city_id": 4, "city_name": "Rajshahi" }
        ]
    }
}
```

---

### 🗺️ 8. Get Zones by City

```php
$response = $pathao->getZones(1); // city_id = 1 (Dhaka)
```

**Response:**

```json
{
    "message": "Zone list fetched.",
    "type": "success",
    "code": 200,
    "data": {
        "data": [
            { "zone_id": 298, "zone_name": "60 feet" },
            { "zone_id": 1070, "zone_name": "Abdullahpur Uttara" }
        ]
    }
}
```

---

### 📍 9. Get Areas by Zone

```php
$response = $pathao->getAreas(298); // zone_id = 298
```

**Response:**

```json
{
    "message": "Area list fetched.",
    "type": "success",
    "code": 200,
    "data": {
        "data": [
            { "area_id": 37, "area_name": "Bonolota", "home_delivery_available": true, "pickup_available": true },
            { "area_id": 3, "area_name": "Road 03", "home_delivery_available": true, "pickup_available": true }
        ]
    }
}
```

---

### 💰 10. Price Calculation

```php
$response = $pathao->calculatePrice([
    'store_id'       => 12345, // required  | integer | Your merchant store ID (sets pickup location)
    'item_type'      => 2,     // required  | integer | 1 = Document, 2 = Parcel
    'delivery_type'  => 48,    // required  | integer | 48 = Normal Delivery, 12 = On Demand Delivery
    'item_weight'    => 0.5,   // required  | float   | Min: 0.5 kg — Max: 10 kg
    'recipient_city' => 1,     // required  | integer | city_id from getCities()
    'recipient_zone' => 298,   // required  | integer | zone_id from getZones($city_id)
]);
```

**Response:**

```json
{
    "message": "price",
    "type": "success",
    "code": 200,
    "data": {
        "price": 80,
        "discount": 0,
        "promo_discount": 0,
        "plan_id": 69,
        "cod_enabled": 1,
        "cod_percentage": 0.01,
        "additional_charge": 0,
        "final_price": 80
    }
}
```

---

### 🏬 11. Get Merchant Store Info

```php
$response = $pathao->getStores();
```

**Response:**

```json
{
    "message": "Store list fetched.",
    "type": "success",
    "code": 200,
    "data": {
        "data": [
            {
                "store_id": 12345,
                "store_name": "Demo Store",
                "store_address": "House 123, Road 4, Sector 10, Uttara, Dhaka-1230, Bangladesh",
                "is_active": 1,
                "city_id": 1,
                "zone_id": 298,
                "hub_id": 5,
                "is_default_store": false,
                "is_default_return_store": false
            }
        ],
        "total": 1,
        "current_page": 1,
        "per_page": 1000,
        "last_page": 1
    }
}
```

---

## 🏢 Multi-Tenant / SaaS Usage

Override credentials at runtime without touching `.env`:

```php
$pathao = PathaoCourierService::withConfig(
    clientId:     'tenant-client-id',
    clientSecret: 'tenant-client-secret',
    username:     'tenant@email.com',
    password:     'tenant-password'
);

$response = $pathao->placeOrder([...]);
```

To target the sandbox environment for a specific tenant:

```php
$pathao = PathaoCourierService::withConfig(
    clientId:     '7N1aMJQbWm',
    clientSecret: 'wRcaibZkUdSNz2EI9ZyuXLlNrnAv0TdPUPXMnD39',
    username:     'test@pathao.com',
    password:     'lovePathao',
    baseUrl:      'https://courier-api-sandbox.pathao.com'
);
```

---

## 📦 Order Status Reference


| Status                | Description                         |
| --------------------- | ----------------------------------- |
| `Pending`             | Order placed, not yet picked up     |
| `Pickup_Requested`    | Pickup has been requested           |
| `Picked`              | Parcel picked up from merchant      |
| `In_Transit`          | Parcel is on its way                |
| `Delivered`           | Successfully delivered to recipient |
| `Partially_Delivered` | Partially delivered                 |
| `Cancelled`           | Order cancelled                     |
| `Hold`                | Order is on hold                    |
| `Return`              | Parcel is being returned            |


---

## 🚀 Delivery Type Reference


| Code | Description        |
| ---- | ------------------ |
| `48` | Normal Delivery    |
| `12` | On Demand Delivery |


---

## 📦 Item Type Reference


| Code | Description |
| ---- | ----------- |
| `1`  | Document    |
| `2`  | Parcel      |

