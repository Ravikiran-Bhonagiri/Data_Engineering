
# Slowly Changing Dimensions (SCDs) in Data Warehousing: E-Commerce Examples

## Overview

This document explains three key types of Slowly Changing Dimensions (SCDs) in the context of an e-commerce platform. SCDs manage changes to descriptive data over time, ensuring accurate and flexible reporting.

---

## Key SCD Types with E-Commerce Examples

### **1. Type 1: Overwrite**

- **Description**:  
  Updates the dimension by overwriting the old value with the new one. No historical data is retained.

- **Best Use Case**:  
  When maintaining historical accuracy is unnecessary, such as fixing typographical errors or updating non-critical fields (e.g., a customer's phone number).

- **Example: Customer’s Phone Number**  
  Alice Johnson updates her phone number in the system.

  **Before Update**:  
  | CustomerID | Name          | City       | Phone      |
  |------------|---------------|------------|------------|
  | 101        | Alice Johnson | Austin     | 555-1234   |

  **After Update**:  
  The new number `555-5678` replaces the old value.

  | CustomerID | Name          | City       | Phone      |
  |------------|---------------|------------|------------|
  | 101        | Alice Johnson | Austin     | 555-5678   |

  **Pros**: Simple and requires less storage.  
  **Cons**: Historical data is lost.

---

### **2. Type 2: Add a New Row**

- **Description**:  
  Inserts a new row in the dimension table whenever an attribute changes. Historical data is fully preserved with metadata columns like `EffectiveDate`, `EndDate`, and `IsCurrent`.

- **Best Use Case**:  
  When tracking historical changes is crucial, such as customer address changes or product price updates.

- **Example: Customer’s Address**  
  Alice Johnson moves from Austin to Dallas.

  **Before Update**:  
  | CustomerID | Name          | City       | EffectiveDate | EndDate    | IsCurrent |
  |------------|---------------|------------|---------------|------------|-----------|
  | 101        | Alice Johnson | Austin     | 2020-01-01    | NULL       | Yes       |

  **After Update**:  
  A new row is added for Dallas, and the old record is updated with an `EndDate`.

  | CustomerID | Name          | City       | EffectiveDate | EndDate    | IsCurrent |
  |------------|---------------|------------|---------------|------------|-----------|
  | 101        | Alice Johnson | Austin     | 2020-01-01    | 2023-06-30 | No        |
  | 101        | Alice Johnson | Dallas     | 2023-07-01    | NULL       | Yes       |

  **Pros**: Full historical tracking; highly flexible for audits and reporting.  
  **Cons**: Increased storage and query complexity due to duplicate rows.

---

### **3. Type 3: Add a New Column**

- **Description**:  
  Adds a new column to store the previous value of the changing attribute. Limited historical tracking as only the immediate past value is retained.

- **Best Use Case**:  
  When only recent changes are relevant, such as tracking the last city a customer lived in.

- **Example: Customer’s City**  
  Alice Johnson moves from Austin to Dallas.

  **Before Update**:  
  | CustomerID | Name          | CurrentCity | PreviousCity |
  |------------|---------------|-------------|--------------|
  | 101        | Alice Johnson | Austin      | NULL         |

  **After Update**:  
  The new city is stored in `CurrentCity`, and the old city moves to `PreviousCity`.

  | CustomerID | Name          | CurrentCity | PreviousCity |
  |------------|---------------|-------------|--------------|
  | 101        | Alice Johnson | Dallas      | Austin       |

  **Pros**: Simple implementation with minimal storage overhead.  
  **Cons**: Limited to one level of historical tracking.

---

## Comparison of SCD Types

| **Type**  | **Tracks History?**    | **Storage Impact** | **Query Complexity** | **Best For**                           |
|-----------|-------------------------|--------------------|-----------------------|-----------------------------------------|
| **Type 1**| No                      | Minimal            | Low                   | Non-critical updates (e.g., typos).    |
| **Type 2**| Yes (Full History)      | High               | High                  | Critical changes (e.g., address).      |
| **Type 3**| Yes (Limited History)   | Moderate           | Moderate              | Recent changes (e.g., last city).      |

---

## Choosing the Right Type

- **Type 1**: Use for non-critical fields where historical data is irrelevant.  
- **Type 2**: Use for critical fields where tracking all historical changes is necessary.  
- **Type 3**: Use for fields where limited historical tracking is sufficient and storage constraints exist.  

