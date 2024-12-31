
# Snowflake Schema in Data Warehousing: E-Commerce Example

## Overview

This document explains the **Snowflake Schema** design in the context of an e-commerce store. The schema demonstrates how normalized dimensions reduce data redundancy while maintaining consistency and flexibility for analytical purposes. It includes a fact table and three normalized dimension tables for a simplified example.

---

## What is a Snowflake Schema?

A **Snowflake Schema** is an extension of the **Star Schema** where dimension tables are normalized into multiple related tables. This results in a more complex schema resembling a snowflake, which reduces data redundancy but may increase query complexity.

---

## Components of a Snowflake Schema

### 1. **Fact Table**

Central table storing measurable metrics (facts) and foreign keys to normalized dimension tables.

### 2. **Dimension Tables**

Dimension tables are normalized to eliminate redundancy by splitting attributes into related tables.

---

## Schema Definition

### **Fact Table: `Sales_Fact`**

Stores sales transaction data and references to dimension tables.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| OrderID          | INT       | Unique identifier for each sale.               |
| DateKey          | INT       | Foreign key to `Date_Dimension`.               |
| CustomerKey      | INT       | Foreign key to `Customer_Dimension`.           |
| ProductKey       | INT       | Foreign key to `Product_Dimension`.            |
| SalesAmount      | FLOAT     | Total sale amount.                             |
| Quantity         | INT       | Number of products sold.                       |

---

### **Normalized Dimension Tables**

#### **1. `Date_Dimension`**
Provides details about the transaction date.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| DateKey          | INT       | Surrogate key for the date.                    |
| Year             | INT       | Year of the date.                              |
| Quarter          | VARCHAR   | Quarter of the year (e.g., Q1, Q2).            |
| MonthKey         | INT       | Foreign key to `Month_Dimension`.              |

**`Month_Dimension`**:
| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| MonthKey         | INT       | Surrogate key for the month.                   |
| Month            | VARCHAR   | Name of the month (e.g., January).             |

---

#### **2. `Customer_Dimension`**
Stores customer details.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| CustomerKey      | INT       | Surrogate key for the customer.                |
| CustomerName     | VARCHAR   | Full name of the customer.                     |
| RegionKey        | INT       | Foreign key to `Region_Dimension`.             |

**`Region_Dimension`**:
| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| RegionKey        | INT       | Surrogate key for the region.                  |
| Region           | VARCHAR   | Name of the region (e.g., Texas).              |
| City             | VARCHAR   | Name of the city.                              |

---

#### **3. `Product_Dimension`**
Describes products.

| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| ProductKey       | INT       | Surrogate key for the product.                 |
| ProductName      | VARCHAR   | Name of the product.                           |
| CategoryKey      | INT       | Foreign key to `Category_Dimension`.           |

**`Category_Dimension`**:
| Column Name     | Data Type | Description                                     |
|------------------|-----------|-------------------------------------------------|
| CategoryKey      | INT       | Surrogate key for the category.                |
| Category         | VARCHAR   | Product category (e.g., Electronics).          |
| SubCategory      | VARCHAR   | Product subcategory (e.g., Audio).             |

---

## Advantages of Snowflake Schema

1. **Reduced Redundancy**:
   - Normalization eliminates duplicate data in dimension tables.

2. **Data Integrity**:
   - Centralized updates to related tables ensure consistency.

3. **Scalable Design**:
   - New attributes or hierarchies can be added easily.

---

## Disadvantages of Snowflake Schema

1. **Complex Queries**:
   - Requires more joins to retrieve data, which can increase query complexity.

2. **Slower Performance**:
   - Query execution may be slower due to additional joins.

---

## Example Use Case: Sales Analysis in Snowflake Schema

### **Fact Table: Sales_Fact**
| OrderID | DateKey | CustomerKey | ProductKey | SalesAmount | Quantity |
|---------|---------|-------------|------------|-------------|----------|
| 1001    | 20231225| 101         | 201        | $50         | 2        |

---

### **Dimension Tables**

**`Date_Dimension`**:
| DateKey | Year | Quarter | MonthKey |
|---------|------|---------|----------|
| 20231225| 2023 | Q4      | 12       |

**`Month_Dimension`**:
| MonthKey | Month    |
|----------|----------|
| 12       | December |

**`Customer_Dimension`**:
| CustomerKey | CustomerName     | RegionKey |
|-------------|------------------|-----------|
| 101         | Alice Johnson    | 1         |

**`Region_Dimension`**:
| RegionKey | Region | City     |
|-----------|--------|----------|
| 1         | Texas  | Austin   |

**`Product_Dimension`**:
| ProductKey | ProductName        | CategoryKey |
|------------|--------------------|-------------|
| 201        | Bluetooth Headphones | 10        |

**`Category_Dimension`**:
| CategoryKey | Category     | SubCategory |
|-------------|--------------|-------------|
| 10          | Electronics  | Audio       |

---

## Query Example

**Question**: "What are the total sales by category in December 2023?"

**SQL Query**:
```sql
SELECT C.Category, SUM(F.SalesAmount) AS TotalSales
FROM Sales_Fact F
JOIN Product_Dimension P ON F.ProductKey = P.ProductKey
JOIN Category_Dimension C ON P.CategoryKey = C.CategoryKey
JOIN Date_Dimension D ON F.DateKey = D.DateKey
JOIN Month_Dimension M ON D.MonthKey = M.MonthKey
WHERE M.Month = 'December' AND D.Year = 2023
GROUP BY C.Category;
```

**Result**:
| Category      | TotalSales |
|---------------|------------|
| Electronics   | $50        |

---

## Conclusion

The Snowflake Schema organizes data efficiently by normalizing dimensions, reducing redundancy, and enhancing scalability. While it may introduce query complexity, it ensures data integrity and is ideal for systems with hierarchical relationships and large datasets.
