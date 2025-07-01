

---

### 📁 `README-Backend-Testing.md`

````markdown
# 📦 Smart Retail Inventory System - Backend API Testing Guide

This document provides an overview for testing the backend API endpoints using Postman or cURL.  
Each section lists the **route**, **method**, **URL**, and expected **response**.

---

## ✅ Health Check

### 🔹 `/health`
- **Method**: `GET`
- **URL**: `http://localhost:5000/health`
- **Expected**:
```json
{ "status": "healthy", "database": "connected" }
````

### 🔹 `/ready`

* **Method**: `GET`
* **URL**: `http://localhost:5000/ready`
* **Expected**:

```json
{ "status": "ready" }
```

---

## 📦 Products (Admin)

### 🔹 Get All Products

* **Method**: `GET`
* **URL**: `http://localhost:5000/api/products`
* **Expected**: List of product objects.

### 🔹 Create New Product

* **Method**: `POST`
* **URL**: `http://localhost:5000/api/products`
* **Body (JSON)**:

```json
{
  "name": "HDMI Cable",
  "sku": "HDMI-001",
  "stock_level": 30,
  "category": "Cables",
  "price": 25.0,
  "cost": 8.0,
  "low_stock_threshold": 5
}
```

* **Expected**: Created product object with ID.

---

### 🔹 Update Product

* **Method**: `PUT`
* **URL**: `http://localhost:5000/api/products/1`
* **Body (JSON)**: Same as above.
* **Expected**: Updated product object.

### 🔹 Delete Product

* **Method**: `DELETE`
* **URL**: `http://localhost:5000/api/products/1`
* **Expected**:
  Status `204 No Content`.

---

### 🔹 Restock Product

* **Method**: `POST`
* **URL**: `http://localhost:5000/api/products/1/restock`
* **Body (JSON)**:

```json
{ "quantity": 10 }
```

* **Expected**: Updated product object with new stock.

---

## 👤 User API

### 🔹 Get Products (User)

* **Method**: `GET`
* **URL**: `http://localhost:5000/api/user/products`
* **Expected**: List of available products.

### 🔹 Purchase Product

* **Method**: `POST`
* **URL**: `http://localhost:5000/api/user/products/1/purchase`
* **Body (JSON)**:

```json
{ "quantity": 2 }
```

* **Expected**:

```json
{ "message": "Purchase successful", "remaining_stock": 28 }
```

---

## 📊 Analytics

### 🔹 Dashboard Summary

* **Method**: `GET`
* **URL**: `http://localhost:5000/api/dashboard/summary`
* **Expected**:

```json
{
  "totalProducts": 5,
  "totalValue": 1275.0,
  "lowStockProducts": 1,
  "restocksPending": 3
}
```

### 🔹 Inventory Trend (30 days total)

* **Method**: `GET`
* **URL**: `http://localhost:5000/api/analytics/inventory-trend`
* **Expected**:
  List of 30 days with total stock each day.

### 🔹 Product Trend

* **Method**: `GET`
* **URL**: `http://localhost:5000/api/analytics/product-trend/1`
* **Expected**:
  Stock changes for specific product over 30 days.

### 🔹 Inventory Metrics

* **Method**: `GET`
* **URL**: `http://localhost:5000/api/analytics/metrics`
* **Expected**:
  List of products with:

  * currentStock
  * minStock / maxStock
  * changeAmount
  * changePercent

---

## 📥 Restock Logs

### 🔹 Get Last 5 Restocks

* **Method**: `GET`
* **URL**: `http://localhost:5000/api/restocks`
* **Expected**:
  List of last 5 restock events (timestamp, quantity, product\_id).

---

## 📈 Prometheus Monitoring

### 🔹 Metrics Endpoint

* **Method**: `GET`
* **URL**: `http://localhost:5000/metrics`
* **Expected**:
  Prometheus-formatted metrics for requests, latency, etc.

---

## 🧪 Example Postman Collection

* Optional: Create and export your Postman tests using this structure.
* You can organize requests by folders: `Health`, `Products`, `User`, `Analytics`, `Monitoring`.

---

> 🧠 For testing, make sure your backend is running (e.g. `docker-compose up`) and DB is populated with test data.

```

---

### ✅ טיפ נוסף:
אם תרצה גם `README.md` לקובץ הראשי של הפרויקט – עם הסבר כללי, הרצה, Docker, CI/CD, ועוד – רק תבקש ואבנה אותו גם.

בהצלחה בתיעוד 💪 זה חלק חשוב בפרויקט מקצועי!
```
