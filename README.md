# 📦 Smart Retail Inventory System - Backend API Testing Guide

This document provides a full testing reference for all backend API routes using Postman or curl.

---

## 🩺 Health Check

### `/health`
- **Method**: `GET`
- **URL**: `http://localhost:5000/health`
- **Expected**:
```json
{ "status": "healthy", "database": "connected" }
```

### `/ready`
- **Method**: `GET`
- **URL**: `http://localhost:5000/ready`
- **Expected**:
```json
{ "status": "ready" }
```

---

## 📦 Products (Admin)

### `GET /api/products`
- List all products

### `POST /api/products`
- Create a product
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

### `GET /api/products/<id>`
- Get product by ID

### `PUT /api/products/<id>`
- Update product

### `DELETE /api/products/<id>`
- Delete product

### `POST /api/products/<id>/restock`
- Add quantity to product stock
```json
{ "quantity": 10 }
```

---

## 👤 User API

### `GET /api/user/products`
- List products for users

### `POST /api/user/products/<id>/purchase`
- Purchase a product
```json
{ "quantity": 2 }
```

---

## 📥 Restock Logs

### `GET /api/restocks`
- Get last 5 restock actions

---

## 📊 Analytics

### `GET /api/dashboard/summary`
- General stats: product count, inventory value, low stock count, recent restocks

### `GET /api/analytics/inventory-trend`
- 30-day trend of total inventory stock

### `GET /api/analytics/product-trend/<id>`
- 30-day trend of specific product

### `GET /api/analytics/metrics`
- Per-product metrics (min/max stock, percent change)

---

## 📉 Monitoring

### `GET /metrics`
- Prometheus metrics

---

> Make sure your backend is running (via Docker or Flask) and test with Postman or curl.
