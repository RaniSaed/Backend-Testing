

-----

## **I. Health & Readiness Checks**

These endpoints are crucial for monitoring the application's basic operational status.

### 1\. **Get Prometheus Metrics**

  * **URL:** `/metrics`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a plain text response containing Prometheus metrics in their exposition format. This includes application-specific metrics like request counters and latencies.

### 2\. **Health Check**

  * **URL:** `/health`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON object indicating the application's health and database connection status.
        ```json
        {
          "status": "healthy",
          "database": "connected"
        }
        ```
      * **Failure (500 Internal Server Error):** Returns a JSON object indicating an unhealthy status and details about the database connection error.
        ```json
        {
          "status": "unhealthy",
          "database": "disconnected",
          "error": "..."
        }
        ```

### 3\. **Readiness Check**

  * **URL:** `/ready`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON object indicating the application is ready to accept traffic.
        ```json
        {
          "status": "ready"
        }
        ```

-----

## **II. User API**

These endpoints are designed for general user interactions, primarily for viewing products and making purchases.

### 1\. **Get All Products (User View)**

  * **URL:** `/api/user/products`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON array of all available products, including their details.
        ```json
        [
          {
            "id": 1,
            "name": "Laptop",
            "sku": "LP123",
            "stock_level": 50,
            "category": "Electronics",
            "price": 1200.0,
            "cost": 800.0,
            "low_stock_threshold": 10
          },
          // ... more products
        ]
        ```

### 2\. **Purchase Product**

  * **URL:** `/api/user/products/<int:product_id>/purchase`
  * **Method:** `POST`
  * **Request Body (JSON):**
    ```json
    {
      "quantity": 2
    }
    ```
  * **Expected Result:**
      * **Success (200 OK):** Returns a success message and the remaining stock level after a successful purchase. The `RestockLog` and `LowStockProduct` tables will be updated accordingly.
        ```json
        {
          "message": "Purchase successful",
          "remaining_stock": 48
        }
        ```
      * **Failure (404 Not Found):** If the `product_id` does not exist.
        ```json
        {
          "error": "Product not found"
        }
        ```
      * **Failure (400 Bad Request):** If the requested `quantity` exceeds the available `stock_level`.
        ```json
        {
          "error": "Not enough stock"
        }
        ```
      * **Failure (400 Bad Request):** If `quantity` is missing or invalid in the request body.

-----

## **III. Admin API**

These endpoints are for administrative tasks, including managing products, handling stock, and viewing detailed logs.

### 1\. **Manage Products (Get All / Create New)**

  * **URL:** `/api/products`

  * **Method: `GET`**

      * **Expected Result:**
          * **Success (200 OK):** Returns a JSON array of all products, similar to the user's `GET /api/user/products` endpoint.

  * **Method: `POST`**

      * **Request Body (JSON):**
        ```json
        {
          "name": "New Product",
          "sku": "NP001",
          "stock_level": 100,
          "category": "Office Supplies",
          "price": 25.50,
          "cost": 15.00,
          "low_stock_threshold": 20
        }
        ```
          * **Note:** `stock_level`, `category`, `price`, `cost`, and `low_stock_threshold` are optional. `name` and `sku` are required.
      * **Expected Result:**
          * **Success (201 Created):** Returns the JSON representation of the newly created product. If the `stock_level` is at or below the `low_stock_threshold`, an entry will also be created in the `LowStockProduct` table.
            ```json
            {
              "id": 2,
              "name": "New Product",
              "sku": "NP001",
              "stock_level": 100,
              "category": "Office Supplies",
              "price": 25.5,
              "cost": 15.0,
              "low_stock_threshold": 20
            }
            ```
          * **Failure (400 Bad Request):** If required fields (`name`, `sku`) are missing from the request body.
            ```json
            {
              "error": "Missing field: 'name'"
            }
            ```

### 2\. **Product Detail (Get / Update / Delete)**

  * **URL:** `/api/products/<int:product_id>`

  * **Method: `GET`**

      * **Expected Result:**
          * **Success (200 OK):** Returns the JSON representation of the specific product.
            ```json
            {
              "id": 1,
              "name": "Laptop",
              "sku": "LP123",
              "stock_level": 50,
              "category": "Electronics",
              "price": 1200.0,
              "cost": 800.0,
              "low_stock_threshold": 10
            }
            ```
          * **Failure (404 Not Found):** If the `product_id` does not exist.

  * **Method: `PUT`**

      * **Request Body (JSON):**
        ```json
        {
          "name": "Updated Laptop",
          "sku": "LP123-NEW",
          "stock_level": 55,
          "category": "High-Tech",
          "price": 1250.0,
          "cost": 820.0,
          "low_stock_threshold": 15
        }
        ```
          * **Note:** `name` and `sku` are required. Other fields are optional and will update if provided.
      * **Expected Result:**
          * **Success (200 OK):** Returns the JSON representation of the updated product. The `RestockLog` and `LowStockProduct` tables will be updated if `stock_level` changes.
            ```json
            {
              "id": 1,
              "name": "Updated Laptop",
              "sku": "LP123-NEW",
              "stock_level": 55,
              "category": "High-Tech",
              "price": 1250.0,
              "cost": 820.0,
              "low_stock_threshold": 15
            }
            ```
          * **Failure (404 Not Found):** If the `product_id` does not exist.
          * **Failure (400 Bad Request):** If required fields (`name`, `sku`) are missing or invalid in the request body.

  * **Method: `DELETE`**

      * **Expected Result:**
          * **Success (204 No Content):** Returns an empty response with a success status. Associated `RestockLog` and `LowStockProduct` entries for this product will also be deleted.
          * **Failure (404 Not Found):** If the `product_id` does not exist.

### 3\. **Restock Product**

  * **URL:** `/api/products/<int:product_id>/restock`
  * **Method:** `POST`
  * **Request Body (JSON):**
    ```json
    {
      "quantity": 10
    }
    ```
  * **Expected Result:**
      * **Success (200 OK):** Returns the JSON representation of the restocked product with the updated `stock_level`. A `RestockLog` entry will be created, and the `LowStockProduct` table will be updated if the stock level crosses the threshold.
        ```json
        {
          "id": 1,
          "name": "Laptop",
          "sku": "LP123",
          "stock_level": 60,
          "category": "Electronics",
          "price": 1200.0,
          "cost": 800.0,
          "low_stock_threshold": 10
        }
        ```
      * **Failure (404 Not Found):** If the `product_id` does not exist.
      * **Failure (400 Bad Request):** If `quantity` is missing, not a valid number, or less than or equal to 0.
        ```json
        {
          "error": "Quantity must be positive"
        }
        ```
        ```json
        {
          "error": "Invalid request"
        }
        ```

-----

## **IV. Monitoring / Dashboard API**

These endpoints provide insights into the inventory status and trends, useful for dashboards and reporting.

### 1\. **Get Low Stock Products**

  * **URL:** `/api/products/low-stock`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON array of products currently marked as low in stock, ordered by timestamp (most recent first).
        ```json
        [
          {
            "id": 1,
            "product_id": 101,
            "name": "Pens",
            "sku": "PEN001",
            "stock_level": 5,
            "low_stock_threshold": 10,
            "timestamp": "2024-07-01T10:30:00.000Z"
          },
          // ... more low stock products
        ]
        ```

### 2\. **Get Recent Restock Logs**

  * **URL:** `/api/restocks`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON array of the 5 most recent restock (or purchase) logs.
        ```json
        [
          {
            "id": 1,
            "product_id": 101,
            "quantity": 20,
            "timestamp": "2024-07-01T11:00:00.000Z"
          },
          // ... up to 5 logs
        ]
        ```

### 3\. **Get Dashboard Summary**

  * **URL:** `/api/dashboard/summary`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON object summarizing key inventory metrics.
        ```json
        {
          "totalProducts": 10,
          "totalValue": 15000.75,
          "lowStockProducts": 3,
          "restocksPending": 2
        }
        ```

### 4\. **Get Inventory Trend**

  * **URL:** `/api/analytics/inventory-trend`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON array showing the total inventory stock level for the last 30 days.
        ```json
        [
          {
            "date": "2024-06-02",
            "stock": 1200
          },
          // ... for 30 days
          {
            "date": "2024-07-01",
            "stock": 1250
          }
        ]
        ```

### 5\. **Get Product Trend**

  * **URL:** `/api/analytics/product-trend/<int:product_id>`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON array showing the stock level for a specific product over the last 30 days.
        ```json
        [
          {
            "date": "2024-06-02",
            "stock": 50
          },
          // ... for 30 days
          {
            "date": "2024-07-01",
            "stock": 48
          }
        ]
        ```
      * **Failure (404 Not Found):** If the `product_id` does not exist.

### 6\. **Get Inventory Metrics**

  * **URL:** `/api/analytics/metrics`
  * **Method:** `GET`
  * **Expected Result:**
      * **Success (200 OK):** Returns a JSON array of detailed metrics for each product, including current, min, max stock, and stock change over the last 30 days.
        ```json
        [
          {
            "id": 1,
            "name": "Laptop",
            "sku": "LP123",
            "currentStock": 48,
            "minStock": 45,
            "maxStock": 55,
            "changeAmount": -2,
            "changePercent": "-4.0%"
          },
          // ... metrics for all products
        ]
        ```

-----

