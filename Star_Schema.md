
# Star Schema in Data Warehousing: E-Commerce Example

## Overview

This document details the **Star Schema** design, its implementation in an e-commerce context, and how transactional data is processed from **OLTP (Online Transaction Processing)** systems to **OLAP (Online Analytical Processing)** systems. The schema includes detailed table definitions, the role of surrogate keys, and examples of how the system operates.

---

## What is a Star Schema?

A **Star Schema** consists of:
1. **Fact Table**: Stores measurable, quantitative data (facts) and foreign keys referencing dimension tables.
2. **Dimension Tables**: Contain descriptive attributes (dimensions) that provide context for the facts.

### **Surrogate Keys**
Surrogate keys are artificial, unique identifiers assigned to dimension tables. These are used instead of natural keys (e.g., email or customer ID) to:
- Ensure consistency and avoid duplicates.
- Simplify database joins and management.
- Provide independence from changes in source systems.

---

## Schema Definition

### **1. Fact Table: `Sales_Fact`**

Central table storing transactional metrics and references to dimension tables.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| OrderID          | INT       | Unique identifier for each sale.               |
| DateKey          | INT       | Foreign key to `Date_Dimension`.               |
| CustomerKey      | INT       | Foreign key to `Customer_Dimension`.           |
| ProductKey       | INT       | Foreign key to `Product_Dimension`.            |
| DeliveryKey      | INT       | Foreign key to `Delivery_Dimension`.           |
| PaymentKey       | INT       | Foreign key to `Payment_Dimension`.            |
| EmployeeKey      | INT       | Foreign key to `Employee_Dimension`.           |
| SalesAmount      | FLOAT     | Total sale amount before tax and discount.     |
| Quantity         | INT       | Number of products sold.                       |
| Discount         | FLOAT     | Discount applied to the transaction.           |
| Tax              | FLOAT     | Tax applied to the transaction.                |
| NetAmount        | FLOAT     | Final amount after tax and discount.           |

---

### **2. Dimension Tables**

#### **a. `Date_Dimension`**

Describes the date of the transaction.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| DateKey          | INT       | Surrogate key for the date.                    |
| Date             | DATE      | Full date.                                     |
| Year             | INT       | Year of the date.                              |
| Quarter          | VARCHAR   | Quarter of the year (e.g., Q1, Q2).            |
| Month            | VARCHAR   | Month name (e.g., January).                    |
| WeekOfYear       | INT       | Week number within the year.                   |
| DayOfWeek        | VARCHAR   | Day name (e.g., Monday).                       |
| Holiday          | BOOLEAN   | Indicates if the date is a holiday.            |

---

#### **b. `Customer_Dimension`**

Describes customers.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| CustomerKey      | INT       | Surrogate key for the customer.                |
| CustomerName     | VARCHAR   | Full name of the customer.                     |
| Region           | VARCHAR   | Region where the customer resides.             |
| City             | VARCHAR   | City of residence.                             |
| ZipCode          | VARCHAR   | Postal code.                                   |
| Email            | VARCHAR   | Email address (not used as a key).             |
| Age              | INT       | Age of the customer.                           |
| Gender           | VARCHAR   | Gender of the customer.                        |
| LoyaltyPoints    | INT       | Total loyalty points accumulated.              |

---

#### **c. `Product_Dimension`**

Describes products.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| ProductKey       | INT       | Surrogate key for the product.                 |
| ProductName      | VARCHAR   | Name of the product.                           |
| Category         | VARCHAR   | Product category (e.g., Electronics).          |
| SubCategory      | VARCHAR   | Product subcategory (e.g., Audio).             |
| Brand            | VARCHAR   | Brand of the product.                          |
| Price            | FLOAT     | Price of the product.                          |
| Stock            | INT       | Number of units available in inventory.        |

---

#### **d. `Delivery_Dimension`**

Describes delivery details.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| DeliveryKey      | INT       | Surrogate key for the delivery method.          |
| DeliveryType     | VARCHAR   | Type of delivery (e.g., Standard Shipping).     |
| Carrier          | VARCHAR   | Name of the delivery carrier (e.g., FedEx).     |
| DeliveryTime     | VARCHAR   | Expected delivery time (e.g., 5 Days).          |
| Cost             | FLOAT     | Cost of the delivery service.                   |

---

#### **e. `Employee_Dimension`**

Tracks employee details.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| EmployeeKey      | INT       | Surrogate key for the employee.                |
| EmployeeName     | VARCHAR   | Full name of the employee.                     |
| Role             | VARCHAR   | Role of the employee (e.g., Sales Manager).    |
| Region           | VARCHAR   | Region where the employee operates.            |
| ExperienceYears  | INT       | Years of experience.                           |

---

#### **f. `Payment_Dimension`**

Tracks payment details.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| PaymentKey       | INT       | Surrogate key for the payment method.          |
| PaymentType      | VARCHAR   | Type of payment (e.g., Credit Card).           |
| Provider         | VARCHAR   | Payment provider (e.g., Visa).                 |
| ProcessingFee    | FLOAT     | Processing fee for the payment method.         |

---

## How Data Moves: OLTP to OLAP

### **1. OLTP Example: Raw Transaction Data**

**Sales Table in OLTP**:
| OrderID | CustomerID | ProductID | DeliveryID | EmployeeID | PaymentID | SalesAmount | Quantity | Discount | Tax |
|---------|------------|-----------|------------|------------|-----------|-------------|----------|----------|-----|
| 1001    | 1          | 10        | 2          | 5          | 3         | 50.00       | 2        | 5.00     | 2.50 |

**Customer Table in OLTP**:
| CustomerID | CustomerName | Email            |
|------------|--------------|------------------|
| 1          | Alice Johnson| alice@gmail.com  |

---

### **2. ETL Process (Extract, Transform, Load)**

- **Extract**:
  - Pull data from OLTP tables.
  - Example: Map `CustomerID = 1` to `CustomerKey = 101`.

- **Transform**:
  - Clean data and assign surrogate keys.
  - Aggregate facts (e.g., total sales for each customer).

- **Load**:
  - Populate dimension and fact tables in OLAP.

---

## Benefits of Surrogate Keys

1. **Consistency**:
   - Surrogate keys remain stable even if natural keys (e.g., email) change.

2. **Simplified Joins**:
   - Numeric surrogate keys are faster for joins than textual natural keys.

3. **Flexibility**:
   - Enables mapping of multiple source systems to a single schema.

---

## Example Query

**Question**: "What are the total sales by category in December 2023?"

**SQL Query**:
```sql
SELECT P.Category, SUM(F.NetAmount) AS TotalSales
FROM Sales_Fact F
JOIN Product_Dimension P ON F.ProductKey = P.ProductKey
JOIN Date_Dimension D ON F.DateKey = D.DateKey
WHERE D.Month = 'December' AND D.Year = 2023
GROUP BY P.Category;
```

**Result**:
| Category      | TotalSales |
|---------------|------------|
| Electronics   | $42.50     |

---

## Conclusion

The star schema simplifies data analysis for e-commerce stores by organizing facts and dimensions into a structured format. The use of surrogate keys ensures flexibility, consistency, and optimized performance in OLAP systems, making it a robust solution for reporting and analytics.
