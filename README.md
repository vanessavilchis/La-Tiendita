# La Tiendita - E-Commerce REST API

A comprehensive Spring Boot REST API for an online grocery store, inspired by traditional Mexican mercados. This project implements a complete e-commerce backend with product management, category organization, user authentication, and shopping cart functionality.

---

## Project Overview

La Tiendita is a **full-featured e-commerce REST API** built with Spring Boot that powers an online grocery store. The project demonstrates enterprise-level development practices including:
- RESTful API design
- Database integration with MySQL
- JWT authentication & authorization
- Role-based access control
- Shopping cart management (some cart features)
- Full CRUD operations
---

## Features Implemented

###  Phase 1: Categories Management 
**What I Built:**
- Complete CRUD operations for product categories
- RESTful endpoints following industry standards
- Admin-only access control for create/update/delete
- **BONUS:** Get products by category endpoint
- Full integration with frontend website

**Technical Details:**
- Created `CategoriesController.java` with 6 REST endpoints
- Implemented `MySqlCategoryDao.java` with JDBC database operations
- Used `@PreAuthorize` annotations for security
- Proper error handling with `ResponseStatusException`

**Endpoints:**
```
GET    /categories           → Get all categories
GET    /categories/{id}      → Get single category
GET    /categories/{id}/products  → Get products in category (BONUS)
POST   /categories           → Create category (Admin only)
PUT    /categories/{id}      → Update category (Admin only)
DELETE /categories/{id}      → Delete category (Admin only)
```

---

### Phase 2: Critical Bug Fixes 

#### **Bug 1: Product Search Missing minPrice Filter** 
**The Problem:**
- Product search only checked maximum price
- Users couldn't filter by minimum price range
- SQL query missing `price >= ?` condition

**My Solution:**
```java
// BEFORE (Buggy):
String sql = "WHERE (price <= ? OR ? = -1)";  // Only maxPrice!

// AFTER (Fixed):
String sql = "WHERE (price >= ? OR ? = -1) AND (price <= ? OR ? = -1)";
```

**Impact:**
- Product search now accurately filters by price range
- Users can find products between min and max prices
- Tested with multiple price ranges - all passing 

---

#### **Bug 2: Product Update Creating Duplicates** 
**The Problem:**
- PUT endpoint was calling `create()` instead of `update()`
- Every product update created a duplicate entry
- Database integrity compromised

**My Solution:**
```java
// BEFORE (Buggy):
public void updateProduct(@PathVariable int id, @RequestBody Product product) {
    productDao.create(product);  // Creates duplicate!
}

// AFTER (Fixed):
public void updateProduct(@PathVariable int id, @RequestBody Product product) {
    productDao.update(id, product);  // Properly updates!
}
```

**Impact:**
- Products now update correctly without duplicates
- Database maintains referential integrity
- Verified with comprehensive testing 

---

###  Phase 3: Shopping Cart 

**What I Built:**
- Full shopping cart functionality for login users
- Add products to cart
- View cart with calculated totals
- User-specific carts (each user has their own cart)

**Technical Implementation:**
- Created `MySqlShoppingCartDao.java` from scratch
- Implemented cart operations with SQL JOIN to get product details
- Used `ON DUPLICATE KEY UPDATE` for smart quantity handling
- Calculated line totals and cart totals automatically

**Endpoints:**
```
GET    /cart                     → Get user's cart
POST   /cart/products/{id}       → Add product to cart
PUT    /cart/products/{id}       → Update product quantity
DELETE /cart                     → Clear cart
```

**Key Features:**
- Adding same product twice increases quantity (no duplicates)
- Multiple products in one cart
- Full product details in cart response

**Known Issue:**
- Cart clearing requires additional work (noted for future enhancement)
- Quantity can not be adjusted 
- Total does not appear

---
##  Challenges & Solutions

### Challenge 1: SQL Query Bug Detection
**Problem:** Product search returned incorrect results for price ranges

**My Approach:**
1. Analyzed the SQL query structure
2. Identified missing `minPrice` condition
3. Corrected PreparedStatement parameter bindings
4. Tested with multiple price ranges

**Learning:** Careful attention to SQL query logic and parameter ordering is critical for accurate data retrieval.

---

### Challenge 2: Understanding DAO vs Controller Responsibilities
**Problem:** Product update was in wrong layer (calling create instead of update)

**My Approach:**
1. Reviewed the separation of concerns
2. Traced through the code flow
3. Identified controller calling wrong DAO method
4. Fixed the method call

**Learning:** Understanding the MVC pattern and DAO pattern prevents logic errors.


---

### Challenge 3: Smart Quantity Management
**Problem:** Prevent duplicate cart entries when adding same product

**My Solution:**
```sql
INSERT INTO shopping_cart (user_id, product_id, quantity) 
VALUES (?, ?, 1) 
ON DUPLICATE KEY UPDATE quantity = quantity + 1
```

