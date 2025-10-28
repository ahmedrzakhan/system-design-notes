# MongoDB 50 Practice Questions

_A comprehensive practice set inspired by SQL patterns, adapted for MongoDB_

## Overview

This document contains 50 MongoDB practice questions organized by category and difficulty level. Each question includes sample data structure, the problem statement, and the MongoDB query solution.

---

## 📊 Sample Collections Setup

### Products Collection

```javascript
db.products.insertMany([
  {
    product_id: 1,
    name: "Laptop",
    category: "Electronics",
    price: 999,
    low_fat: "N",
    recyclable: "Y",
    stock: 45,
  },
  {
    product_id: 2,
    name: "Yogurt",
    category: "Food",
    price: 3.99,
    low_fat: "Y",
    recyclable: "Y",
    stock: 100,
  },
  {
    product_id: 3,
    name: "Pizza",
    category: "Food",
    price: 12.99,
    low_fat: "N",
    recyclable: "N",
    stock: 30,
  },
  {
    product_id: 4,
    name: "Salad",
    category: "Food",
    price: 8.99,
    low_fat: "Y",
    recyclable: "Y",
    stock: 25,
  },
]);
```

### Customers Collection

```javascript
db.customers.insertMany([
  {
    customer_id: 1,
    name: "Alice",
    referee_id: null,
    created_date: ISODate("2024-01-15"),
  },
  {
    customer_id: 2,
    name: "Bob",
    referee_id: null,
    created_date: ISODate("2024-01-20"),
  },
  {
    customer_id: 3,
    name: "Charlie",
    referee_id: 2,
    created_date: ISODate("2024-02-01"),
  },
  {
    customer_id: 4,
    name: "David",
    referee_id: 1,
    created_date: ISODate("2024-02-15"),
  },
  {
    customer_id: 5,
    name: "Eve",
    referee_id: 2,
    created_date: ISODate("2024-03-01"),
  },
]);
```

### Orders Collection

```javascript
db.orders.insertMany([
  {
    order_id: 1,
    customer_id: 1,
    product_id: 1,
    quantity: 1,
    order_date: ISODate("2024-03-01"),
    status: "completed",
  },
  {
    order_id: 2,
    customer_id: 2,
    product_id: 2,
    quantity: 5,
    order_date: ISODate("2024-03-02"),
    status: "completed",
  },
  {
    order_id: 3,
    customer_id: 1,
    product_id: 3,
    quantity: 2,
    order_date: ISODate("2024-03-03"),
    status: "pending",
  },
  {
    order_id: 4,
    customer_id: 3,
    product_id: 2,
    quantity: 10,
    order_date: ISODate("2024-03-04"),
    status: "completed",
  },
]);
```

### Employees Collection

```javascript
db.employees.insertMany([
  {
    emp_id: 1,
    name: "John",
    department: "Sales",
    salary: 55000,
    manager_id: null,
    hire_date: ISODate("2020-01-15"),
  },
  {
    emp_id: 2,
    name: "Sarah",
    department: "Engineering",
    salary: 75000,
    manager_id: 1,
    hire_date: ISODate("2021-03-20"),
  },
  {
    emp_id: 3,
    name: "Mike",
    department: "Engineering",
    salary: 68000,
    manager_id: 1,
    hire_date: ISODate("2021-06-01"),
  },
  {
    emp_id: 4,
    name: "Emily",
    department: "Sales",
    salary: 52000,
    manager_id: 1,
    hire_date: ISODate("2022-02-10"),
  },
  {
    emp_id: 5,
    name: "David",
    department: "HR",
    salary: 48000,
    manager_id: null,
    hire_date: ISODate("2019-11-30"),
  },
]);
```

---

## 🟢 Basic Queries (Questions 1-15)

### Question 1: Find Recyclable and Low Fat Products

**Difficulty:** Easy
**Problem:** Find all products that are both low fat (low_fat = "Y") and recyclable (recyclable = "Y").

```javascript
db.products.find(
  {
    low_fat: "Y",
    recyclable: "Y",
  },
  {
    product_id: 1,
    name: 1,
    _id: 0,
  }
);
```

### Question 2: Find Customers Without Specific Referee

**Difficulty:** Easy
**Problem:** Find names of customers that are not referred by customer with customer_id = 2.

```javascript
db.customers.find(
  {
    $or: [{ referee_id: null }, { referee_id: { $ne: 2 } }],
  },
  {
    name: 1,
    _id: 0,
  }
);
```

### Question 3: Find Big Orders

**Difficulty:** Easy
**Problem:** Find orders where quantity is greater than or equal to 5 OR the order status is "pending".

```javascript
db.orders.find({
  $or: [{ quantity: { $gte: 5 } }, { status: "pending" }],
});
```

### Question 4: Select Specific Fields

**Difficulty:** Easy
**Problem:** Return only name and salary fields for all employees in Engineering department.

```javascript
db.employees.find(
  { department: "Engineering" },
  { name: 1, salary: 1, _id: 0 }
);
```

### Question 5: Count Documents

**Difficulty:** Easy
**Problem:** Count the total number of products in the "Food" category.

```javascript
db.products.countDocuments({ category: "Food" });
```

### Question 6: Find with NOT Operator

**Difficulty:** Easy
**Problem:** Find all products that are NOT in the "Electronics" category.

```javascript
db.products.find({
  category: { $ne: "Electronics" },
});
```

### Question 7: Find with IN Operator

**Difficulty:** Easy
**Problem:** Find employees whose emp_id is in [1, 3, 5].

```javascript
db.employees.find({
  emp_id: { $in: [1, 3, 5] },
});
```

### Question 8: String Pattern Matching

**Difficulty:** Easy
**Problem:** Find all customers whose name starts with 'D'.

```javascript
db.customers.find({
  name: { $regex: "^D", $options: "i" },
});
```

### Question 9: Date Comparison

**Difficulty:** Easy
**Problem:** Find all orders placed after March 2, 2024.

```javascript
db.orders.find({
  order_date: { $gt: ISODate("2024-03-02") },
});
```

### Question 10: Exists Check

**Difficulty:** Easy
**Problem:** Find all customers who have a referee_id field (not null).

```javascript
db.customers.find({
  referee_id: { $exists: true, $ne: null },
});
```

### Question 11: Range Query

**Difficulty:** Easy
**Problem:** Find all products with price between 5 and 50.

```javascript
db.products.find({
  price: { $gte: 5, $lte: 50 },
});
```

### Question 12: Sorting

**Difficulty:** Easy
**Problem:** Find all employees sorted by salary in descending order.

```javascript
db.employees.find().sort({ salary: -1 });
```

### Question 13: Limit Results

**Difficulty:** Easy
**Problem:** Get the top 3 most expensive products.

```javascript
db.products.find().sort({ price: -1 }).limit(3);
```

### Question 14: Skip and Limit (Pagination)

**Difficulty:** Easy
**Problem:** Get products 3-5 when sorted by product_id (skip first 2, take next 3).

```javascript
db.products.find().sort({ product_id: 1 }).skip(2).limit(3);
```

### Question 15: Distinct Values

**Difficulty:** Easy
**Problem:** Find all distinct departments in the employees collection.

```javascript
db.employees.distinct("department");
```

---

## 🟡 Aggregation Basics (Questions 16-25)

### Question 16: Simple GROUP BY

**Difficulty:** Medium
**Problem:** Count the number of employees in each department.

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
      count: { $sum: 1 },
    },
  },
]);
```

### Question 17: Average Calculation

**Difficulty:** Medium
**Problem:** Calculate the average salary by department.

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
      avg_salary: { $avg: "$salary" },
    },
  },
]);
```

### Question 18: Sum Aggregation

**Difficulty:** Medium
**Problem:** Calculate total quantity ordered for each product.

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$product_id",
      total_quantity: { $sum: "$quantity" },
    },
  },
]);
```

### Question 19: Min and Max

**Difficulty:** Medium
**Problem:** Find the minimum and maximum salary in each department.

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
      min_salary: { $min: "$salary" },
      max_salary: { $max: "$salary" },
    },
  },
]);
```

### Question 20: Having Clause Equivalent

**Difficulty:** Medium
**Problem:** Find departments with average salary greater than 60000.

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
      avg_salary: { $avg: "$salary" },
    },
  },
  {
    $match: {
      avg_salary: { $gt: 60000 },
    },
  },
]);
```

### Question 21: Count with Condition

**Difficulty:** Medium
**Problem:** Count completed vs pending orders.

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$status",
      count: { $sum: 1 },
    },
  },
]);
```

### Question 22: Multiple Group Fields

**Difficulty:** Medium
**Problem:** Group orders by customer_id and status, count each combination.

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: {
        customer: "$customer_id",
        status: "$status",
      },
      count: { $sum: 1 },
    },
  },
]);
```

### Question 23: Project and Calculate

**Difficulty:** Medium
**Problem:** Calculate total price (quantity \* product price) for each order.

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "product_id",
      as: "product_info",
    },
  },
  {
    $unwind: "$product_info",
  },
  {
    $project: {
      order_id: 1,
      total_price: { $multiply: ["$quantity", "$product_info.price"] },
    },
  },
]);
```

### Question 24: Array Operations

**Difficulty:** Medium
**Problem:** Find products and count how many orders each has.

```javascript
db.products.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "product_id",
      foreignField: "product_id",
      as: "orders",
    },
  },
  {
    $project: {
      name: 1,
      order_count: { $size: "$orders" },
    },
  },
]);
```

### Question 25: Date Aggregation

**Difficulty:** Medium
**Problem:** Count orders by month in 2024.

```javascript
db.orders.aggregate([
  {
    $match: {
      order_date: {
        $gte: ISODate("2024-01-01"),
        $lt: ISODate("2025-01-01"),
      },
    },
  },
  {
    $group: {
      _id: { $month: "$order_date" },
      count: { $sum: 1 },
    },
  },
  {
    $sort: { _id: 1 },
  },
]);
```

---

## 🔴 Advanced Queries (Questions 26-40)

### Question 26: LEFT JOIN Equivalent

**Difficulty:** Hard
**Problem:** Find all customers with their orders (including customers with no orders).

```javascript
db.customers.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "customer_id",
      foreignField: "customer_id",
      as: "orders",
    },
  },
  {
    $project: {
      name: 1,
      order_count: { $size: "$orders" },
      orders: 1,
    },
  },
]);
```

### Question 27: Multiple JOIN

**Difficulty:** Hard
**Problem:** Find customer name, product name, and quantity for all orders.

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customer_id",
      foreignField: "customer_id",
      as: "customer",
    },
  },
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "product_id",
      as: "product",
    },
  },
  {
    $unwind: "$customer",
  },
  {
    $unwind: "$product",
  },
  {
    $project: {
      customer_name: "$customer.name",
      product_name: "$product.name",
      quantity: 1,
    },
  },
]);
```

### Question 28: Self JOIN Equivalent

**Difficulty:** Hard
**Problem:** Find employees and their manager names.

```javascript
db.employees.aggregate([
  {
    $lookup: {
      from: "employees",
      localField: "manager_id",
      foreignField: "emp_id",
      as: "manager",
    },
  },
  {
    $unwind: {
      path: "$manager",
      preserveNullAndEmptyArrays: true,
    },
  },
  {
    $project: {
      employee_name: "$name",
      manager_name: "$manager.name",
    },
  },
]);
```

### Question 29: UNION Equivalent

**Difficulty:** Hard
**Problem:** Combine high-value products (price > 50) and low-stock products (stock < 30).

```javascript
db.products.aggregate([
  {
    $facet: {
      high_value: [
        { $match: { price: { $gt: 50 } } },
        { $project: { name: 1, reason: { $literal: "high_value" } } },
      ],
      low_stock: [
        { $match: { stock: { $lt: 30 } } },
        { $project: { name: 1, reason: { $literal: "low_stock" } } },
      ],
    },
  },
  {
    $project: {
      combined: { $concatArrays: ["$high_value", "$low_stock"] },
    },
  },
  {
    $unwind: "$combined",
  },
  {
    $replaceRoot: { newRoot: "$combined" },
  },
]);
```

### Question 30: Window Functions - Row Number

**Difficulty:** Hard
**Problem:** Rank employees by salary within each department.

```javascript
db.employees.aggregate([
  { $sort: { department: 1, salary: -1 } },
  {
    $group: {
      _id: "$department",
      employees: { $push: { name: "$name", salary: "$salary" } },
    },
  },
  {
    $unwind: {
      path: "$employees",
      includeArrayIndex: "rank",
    },
  },
  {
    $project: {
      department: "$_id",
      name: "$employees.name",
      salary: "$employees.salary",
      rank: { $add: ["$rank", 1] },
    },
  },
]);
```

### Question 31: Running Total

**Difficulty:** Hard
**Problem:** Calculate running total of orders by date.

```javascript
db.orders.aggregate([
  { $sort: { order_date: 1 } },
  {
    $group: {
      _id: { $dateToString: { format: "%Y-%m-%d", date: "$order_date" } },
      daily_orders: { $sum: 1 },
    },
  },
  { $sort: { _id: 1 } },
  {
    $group: {
      _id: null,
      dates: { $push: { date: "$_id", count: "$daily_orders" } },
    },
  },
  {
    $unwind: {
      path: "$dates",
      includeArrayIndex: "index",
    },
  },
  {
    $lookup: {
      from: "orders",
      pipeline: [
        {
          $group: {
            _id: { $dateToString: { format: "%Y-%m-%d", date: "$order_date" } },
            count: { $sum: 1 },
          },
        },
        { $sort: { _id: 1 } },
      ],
      as: "all_dates",
    },
  },
  {
    $project: {
      date: "$dates.date",
      daily_count: "$dates.count",
      running_total: {
        $reduce: {
          input: { $slice: ["$all_dates", { $add: ["$index", 1] }] },
          initialValue: 0,
          in: { $add: ["$$value", "$$this.count"] },
        },
      },
    },
  },
]);
```

### Question 32: Percentile Calculation

**Difficulty:** Hard
**Problem:** Find the median salary in each department.

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
      salaries: { $push: "$salary" },
    },
  },
  {
    $project: {
      department: "$_id",
      median: {
        $let: {
          vars: {
            sorted: { $sortArray: { input: "$salaries", sortBy: 1 } },
            size: { $size: "$salaries" },
          },
          in: {
            $cond: {
              if: { $eq: [{ $mod: ["$$size", 2] }, 0] },
              then: {
                $avg: [
                  { $arrayElemAt: ["$$sorted", { $divide: ["$$size", 2] }] },
                  {
                    $arrayElemAt: [
                      "$$sorted",
                      { $subtract: [{ $divide: ["$$size", 2] }, 1] },
                    ],
                  },
                ],
              },
              else: {
                $arrayElemAt: [
                  "$$sorted",
                  { $floor: { $divide: ["$$size", 2] } },
                ],
              },
            },
          },
        },
      },
    },
  },
]);
```

### Question 33: Pivot Table

**Difficulty:** Hard
**Problem:** Create a pivot table showing product sales by status (columns: product_name, completed, pending).

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "product_id",
      as: "product",
    },
  },
  { $unwind: "$product" },
  {
    $group: {
      _id: { product: "$product.name", status: "$status" },
      count: { $sum: "$quantity" },
    },
  },
  {
    $group: {
      _id: "$_id.product",
      statuses: {
        $push: {
          k: "$_id.status",
          v: "$count",
        },
      },
    },
  },
  {
    $project: {
      product_name: "$_id",
      completed: {
        $ifNull: [
          {
            $arrayElemAt: [
              {
                $filter: {
                  input: "$statuses",
                  cond: { $eq: ["$$this.k", "completed"] },
                },
              },
              0,
            ],
          },
          { v: 0 },
        ],
      },
      pending: {
        $ifNull: [
          {
            $arrayElemAt: [
              {
                $filter: {
                  input: "$statuses",
                  cond: { $eq: ["$$this.k", "pending"] },
                },
              },
              0,
            ],
          },
          { v: 0 },
        ],
      },
    },
  },
  {
    $project: {
      product_name: 1,
      completed: "$completed.v",
      pending: "$pending.v",
    },
  },
]);
```

### Question 34: Recursive Query Equivalent

**Difficulty:** Hard
**Problem:** Find all employees reporting to manager with emp_id = 1 (direct and indirect).

```javascript
db.employees.aggregate([
  { $match: { emp_id: 1 } },
  {
    $graphLookup: {
      from: "employees",
      startWith: "$emp_id",
      connectFromField: "emp_id",
      connectToField: "manager_id",
      as: "subordinates",
    },
  },
  {
    $project: {
      manager: "$name",
      subordinates: "$subordinates.name",
    },
  },
]);
```

### Question 35: Complex Conditional Aggregation

**Difficulty:** Hard
**Problem:** Categorize employees as Junior (<50k), Mid (50k-70k), or Senior (>70k) and count each.

```javascript
db.employees.aggregate([
  {
    $project: {
      category: {
        $switch: {
          branches: [
            { case: { $lt: ["$salary", 50000] }, then: "Junior" },
            { case: { $lte: ["$salary", 70000] }, then: "Mid" },
          ],
          default: "Senior",
        },
      },
    },
  },
  {
    $group: {
      _id: "$category",
      count: { $sum: 1 },
    },
  },
]);
```

### Question 36: Find Duplicates

**Difficulty:** Hard
**Problem:** Find products with duplicate names.

```javascript
db.products.aggregate([
  {
    $group: {
      _id: "$name",
      count: { $sum: 1 },
      ids: { $push: "$product_id" },
    },
  },
  {
    $match: {
      count: { $gt: 1 },
    },
  },
]);
```

### Question 37: Consecutive Records

**Difficulty:** Hard
**Problem:** Find customers who placed orders on consecutive days.

```javascript
db.orders.aggregate([
  { $sort: { customer_id: 1, order_date: 1 } },
  {
    $group: {
      _id: "$customer_id",
      dates: { $push: "$order_date" },
    },
  },
  {
    $project: {
      customer_id: "$_id",
      has_consecutive: {
        $anyElementTrue: {
          $map: {
            input: { $range: [0, { $subtract: [{ $size: "$dates" }, 1] }] },
            as: "i",
            in: {
              $lte: [
                {
                  $divide: [
                    {
                      $subtract: [
                        { $arrayElemAt: ["$dates", { $add: ["$$i", 1] }] },
                        { $arrayElemAt: ["$dates", "$$i"] },
                      ],
                    },
                    86400000,
                  ],
                },
                1,
              ],
            },
          },
        },
      },
    },
  },
  {
    $match: { has_consecutive: true },
  },
]);
```

### Question 38: Top N per Group

**Difficulty:** Hard
**Problem:** Find top 2 highest paid employees in each department.

```javascript
db.employees.aggregate([
  { $sort: { salary: -1 } },
  {
    $group: {
      _id: "$department",
      employees: {
        $push: {
          name: "$name",
          salary: "$salary",
        },
      },
    },
  },
  {
    $project: {
      department: "$_id",
      top_employees: { $slice: ["$employees", 2] },
    },
  },
]);
```

### Question 39: Year-over-Year Growth

**Difficulty:** Hard
**Problem:** Calculate month-over-month order growth rate.

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: {
        year: { $year: "$order_date" },
        month: { $month: "$order_date" },
      },
      count: { $sum: 1 },
    },
  },
  { $sort: { "_id.year": 1, "_id.month": 1 } },
  {
    $group: {
      _id: null,
      months: {
        $push: {
          date: "$_id",
          count: "$count",
        },
      },
    },
  },
  {
    $project: {
      growth: {
        $map: {
          input: { $range: [1, { $size: "$months" }] },
          as: "i",
          in: {
            month: { $arrayElemAt: ["$months.date", "$$i"] },
            current_count: { $arrayElemAt: ["$months.count", "$$i"] },
            previous_count: {
              $arrayElemAt: ["$months.count", { $subtract: ["$$i", 1] }],
            },
            growth_rate: {
              $multiply: [
                {
                  $divide: [
                    {
                      $subtract: [
                        { $arrayElemAt: ["$months.count", "$$i"] },
                        {
                          $arrayElemAt: [
                            "$months.count",
                            { $subtract: ["$$i", 1] },
                          ],
                        },
                      ],
                    },
                    {
                      $arrayElemAt: [
                        "$months.count",
                        { $subtract: ["$$i", 1] },
                      ],
                    },
                  ],
                },
                100,
              ],
            },
          },
        },
      },
    },
  },
  { $unwind: "$growth" },
  { $replaceRoot: { newRoot: "$growth" } },
]);
```

### Question 40: Complex Business Logic

**Difficulty:** Hard
**Problem:** Find customers who ordered all products in a category.

```javascript
db.products.aggregate([
  { $match: { category: "Food" } },
  {
    $group: {
      _id: null,
      food_products: { $addToSet: "$product_id" },
    },
  },
  {
    $lookup: {
      from: "orders",
      pipeline: [
        {
          $lookup: {
            from: "products",
            localField: "product_id",
            foreignField: "product_id",
            as: "product",
          },
        },
        { $unwind: "$product" },
        { $match: { "product.category": "Food" } },
        {
          $group: {
            _id: "$customer_id",
            ordered_products: { $addToSet: "$product_id" },
          },
        },
      ],
      as: "customer_orders",
    },
  },
  { $unwind: "$customer_orders" },
  {
    $project: {
      customer_id: "$customer_orders._id",
      has_all_products: {
        $setEquals: ["$food_products", "$customer_orders.ordered_products"],
      },
    },
  },
  { $match: { has_all_products: true } },
]);
```

---

## 🔧 Data Manipulation (Questions 41-50)

### Question 41: Insert with Validation

**Difficulty:** Medium
**Problem:** Insert a new product only if a product with the same name doesn't exist.

```javascript
db.products.updateOne(
  { name: "Tablet" },
  {
    $setOnInsert: {
      product_id: 5,
      name: "Tablet",
      category: "Electronics",
      price: 399,
      low_fat: "N",
      recyclable: "Y",
      stock: 20,
    },
  },
  { upsert: true }
);
```

### Question 42: Update with Increment

**Difficulty:** Medium
**Problem:** Increase the salary of all Engineering employees by 10%.

```javascript
db.employees.updateMany({ department: "Engineering" }, [
  {
    $set: {
      salary: { $multiply: ["$salary", 1.1] },
    },
  },
]);
```

### Question 43: Conditional Update

**Difficulty:** Medium
**Problem:** Set a discount field: 20% for Electronics, 10% for Food, 0% for others.

```javascript
db.products.updateMany({}, [
  {
    $set: {
      discount: {
        $switch: {
          branches: [
            { case: { $eq: ["$category", "Electronics"] }, then: 0.2 },
            { case: { $eq: ["$category", "Food"] }, then: 0.1 },
          ],
          default: 0,
        },
      },
    },
  },
]);
```

### Question 44: Array Update

**Difficulty:** Medium
**Problem:** Add tags array to products based on their properties.

```javascript
db.products.updateMany({}, [
  {
    $set: {
      tags: {
        $concatArrays: [
          { $cond: [{ $eq: ["$low_fat", "Y"] }, ["low-fat"], []] },
          { $cond: [{ $eq: ["$recyclable", "Y"] }, ["eco-friendly"], []] },
          { $cond: [{ $lt: ["$price", 10] }, ["budget"], []] },
        ],
      },
    },
  },
]);
```

### Question 45: Delete with Join

**Difficulty:** Hard
**Problem:** Delete orders for products that are out of stock (stock = 0).

```javascript
// First, find product_ids with stock = 0
const outOfStock = db.products
  .find({ stock: 0 }, { product_id: 1, _id: 0 })
  .toArray()
  .map((p) => p.product_id);

// Then delete orders for those products
db.orders.deleteMany({
  product_id: { $in: outOfStock },
});
```

### Question 46: Bulk Operations

**Difficulty:** Hard
**Problem:** Perform bulk updates: increase stock by 10 for Food, decrease by 5 for Electronics.

```javascript
db.products.bulkWrite([
  {
    updateMany: {
      filter: { category: "Food" },
      update: { $inc: { stock: 10 } },
    },
  },
  {
    updateMany: {
      filter: { category: "Electronics" },
      update: { $inc: { stock: -5 } },
    },
  },
]);
```

### Question 47: Transaction Example

**Difficulty:** Hard
**Problem:** Transfer order from one customer to another (atomic operation).

```javascript
const session = db.getMongo().startSession();
session.startTransaction();

try {
  db.orders.updateOne(
    { order_id: 1 },
    { $set: { customer_id: 3 } },
    { session }
  );

  // Log the transfer
  db.order_transfers.insertOne(
    {
      order_id: 1,
      from_customer: 1,
      to_customer: 3,
      transfer_date: new Date(),
    },
    { session }
  );

  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

### Question 48: Index Creation

**Difficulty:** Medium
**Problem:** Create optimal indexes for common query patterns.

```javascript
// Single field index
db.products.createIndex({ category: 1 });

// Compound index for sorting
db.employees.createIndex({ department: 1, salary: -1 });

// Text index for search
db.products.createIndex({ name: "text", category: "text" });

// Unique index
db.customers.createIndex({ email: 1 }, { unique: true });
```

### Question 49: Data Validation Schema

**Difficulty:** Hard
**Problem:** Create a validation schema for the orders collection.

```javascript
db.runCommand({
  collMod: "orders",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: [
        "order_id",
        "customer_id",
        "product_id",
        "quantity",
        "order_date",
      ],
      properties: {
        order_id: {
          bsonType: "int",
          description: "must be an integer and is required",
        },
        customer_id: {
          bsonType: "int",
          minimum: 1,
          description: "must be a positive integer",
        },
        quantity: {
          bsonType: "int",
          minimum: 1,
          maximum: 100,
          description: "must be between 1 and 100",
        },
        status: {
          enum: ["pending", "completed", "cancelled"],
          description: "must be one of the allowed values",
        },
        order_date: {
          bsonType: "date",
          description: "must be a valid date",
        },
      },
    },
  },
  validationLevel: "moderate",
  validationAction: "warn",
});
```

### Question 50: Performance Analysis

**Difficulty:** Hard
**Problem:** Analyze query performance and create explain plan.

```javascript
// Analyze a complex aggregation
db.orders.explain("executionStats").aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customer_id",
      foreignField: "customer_id",
      as: "customer",
    },
  },
  {
    $match: {
      order_date: { $gte: ISODate("2024-03-01") },
    },
  },
  {
    $group: {
      _id: "$customer_id",
      total_orders: { $sum: 1 },
    },
  },
]);

// Get collection stats
db.orders.stats();

// Get index usage stats
db.orders.aggregate([{ $indexStats: {} }]);

// Profile slow queries
db.setProfilingLevel(1, { slowms: 100 });
db.system.profile.find().limit(5).sort({ ts: -1 }).pretty();
```

---

## 📚 Additional Practice Resources

### Key MongoDB Concepts to Master:

1. **Document Model** - Understanding embedded vs referenced documents
2. **Aggregation Pipeline** - Stages, expressions, and operators
3. **Indexes** - Types, strategies, and performance implications
4. **Transactions** - ACID properties in MongoDB
5. **Sharding** - Horizontal scaling strategies
6. **Replication** - High availability setup
7. **Change Streams** - Real-time data changes
8. **Schema Design Patterns** - Bucket, subset, computed, etc.

### Common Interview Topics:

- Normalization vs Denormalization in MongoDB
- When to use embedded documents vs references
- Aggregation pipeline optimization
- Index selection and compound index ordering
- Write concern and read preference
- MongoDB vs SQL database use cases

### Performance Tips:

1. Use indexes appropriately (but not excessively)
2. Limit returned fields with projection
3. Use `$match` early in aggregation pipelines
4. Consider using `allowDiskUse` for large aggregations
5. Monitor with `explain()` and profiler
6. Use covered queries when possible
7. Batch operations with `bulkWrite()`

---

## 🎯 Practice Recommendations

### Week 1: Basics (Questions 1-15)

- Focus on CRUD operations
- Master query selectors
- Practice projections and sorting

### Week 2: Aggregation Fundamentals (Questions 16-25)

- Understand pipeline stages
- Practice `$group`, `$match`, `$project`
- Learn date and array operations

### Week 3: Advanced Queries (Questions 26-40)

- Master `$lookup` for joins
- Practice complex aggregations
- Understand window functions equivalents

### Week 4: Real-world Scenarios (Questions 41-50)

- Practice data manipulation
- Learn transaction handling
- Focus on performance optimization

---

_Remember: MongoDB's document model requires different thinking than relational databases. Focus on understanding when to embed vs reference, and how to leverage the aggregation framework effectively._

**Happy Learning! 🚀**
