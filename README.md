# MongoDB Database Case Study

In this lab, you will load the repository's multi-collection dataset directly into a browser-based MongoDB sandbox, explore how BSON documents differ from relational tables, practice core CRUD queries, and build step-by-step aggregation pipelines—no local database installation required.

## 1. Repository Files & Dataset Overview

Your repository includes **7 JSON data files**. Each JSON file maps to a MongoDB collection:

| JSON File | Target Collection | Role in the Case Study |
| :--- | :--- | :--- |
| `Customers.json` | `customers` | Customer profile records and contact/account details |
| `Employees.json` | `employees` | Employee directory, roles, and organizational attributes |
| `Places.json` | `places` | Locations, regions, or branch/site records |
| `Products.json` | `products` | Product catalog, categories, and pricing fields |
| `Sales.json` | `sales` | Sales transactions linking customers, products, and staff |
| `SalesTeam.json` | `salesTeam` | Sales team assignments, territories, and reporting structure |
| `Students.json` | `students` | Practice collection for basic CRUD and document structure warm-ups |

## 2. Step-by-Step Setup in OneCompiler

Because OneCompiler runs in your browser, you will load data using `insertMany(...)` instead of local command-line tools like `mongoimport`.

### Step 1: Open the Online Sandbox
1. Go to **[https://onecompiler.com/mongodb](https://onecompiler.com/mongodb)** in your browser.
2. Clear any default starter code in the editor pane so you have a clean script.

### Step 2: Load Your JSON Datasets (`insertMany`)
OneCompiler resets its temporary database state on each run, so **keep your `insertMany(...)` setup at the top of your script** and place your queries directly below it:

```javascript
// 1. Paste the JSON array from each repository file inside insertMany(...)
db.students.insertMany([
  // Paste contents of Students.json here
]);

db.customers.insertMany([
  // Paste contents of Customers.json here
]);

db.employees.insertMany([
  // Paste contents of Employees.json here
]);

db.places.insertMany([
  // Paste contents of Places.json here
]);

db.products.insertMany([
  // Paste contents of Products.json here
]);

db.sales.insertMany([
  // Paste contents of Sales.json here
]);

db.salesTeam.insertMany([
  // Paste contents of SalesTeam.json here
]);
```

> **Quick Formatting Check:** If a `.json` file already starts with `[` and ends with `]`, paste the entire array directly inside `insertMany( ... )`. If the file lists one JSON object per line without outer brackets, wrap the pasted lines in `[` and `]` separated by commas.

### Step 3: Verify Your Data Loaded
Add this verification block below your `insertMany` statements and click **Run**:

```javascript
// Check collection names and document counts
db.getCollectionNames().forEach(name => {
  print(name + ": " + db.getCollection(name).countDocuments() + " documents");
});
```

**Checkpoint:** Confirm all 7 collections print a document count greater than `0` in the output pane before starting the exercises.

## 3. Tasks

Keep your `insertMany(...)` block at the top of the OneCompiler editor and add each exercise below it. Use `.toArray()` or `printjson(...)` if a query cursor does not automatically print all documents in the output window.

### Part A: Inspect Document Shapes (Warm-Up)
Unlike SQL tables, MongoDB collections do not enforce a fixed schema upfront. Inspect one document from each collection so you know the exact field names, capitalization, and data types:

```javascript
printjson(db.students.findOne());
printjson(db.customers.findOne());
printjson(db.employees.findOne());
printjson(db.products.findOne());
printjson(db.sales.findOne());
```

* **Self-Check:** Note which fields are strings, numbers, arrays (`[...]`), or nested subdocuments (`{...}`). Remember that MongoDB field names are **case-sensitive**.

## Part B: Filtering & Projections (`find`)

### 1. Basic Exact-Match Filter
*Example: Find all students in a specific major/grade or products in a specific category.*

```javascript
printjson(
  db.products.find(
    { category: "Electronics" } // Replace with an actual field/value from Part A
  ).toArray()
);
```

### 2. Comparison Operators (`$gt`, `$gte`, `$lt`, `$lte`)
*Example: Find products priced at or above $50, or sales transactions with quantity greater than 2.*

```javascript
printjson(
  db.products.find(
    { price: { $gte: 50, $lte: 500 } } // Replace 'price' with your numeric field
  ).toArray()
);
```

### 3. Projections (Limit Returned Fields & Hide `_id`)
*Example: Return only product name and price without `_id`.*

```javascript
printjson(
  db.products.find(
    { price: { $gte: 50 } },
    { name: 1, price: 1, _id: 0 } // 1 = include, 0 = exclude
  ).toArray()
);
```

## Part C: Safe Document Updates & Deletes (`CRUD`)

```javascript
// Record initial count
print("Initial student count: " + db.students.countDocuments());

// 1. Insert one test student document
const insertRes = db.students.insertOne({
  studentId: "TEST-999",
  name: "Alex Rivera",
  major: "Information Systems",
  courses: ["Database Systems", "Data Analytics"]
});
print("=== INSERT RESULT ===");
printjson(insertRes);

// 2. Update field ($set) and append to array ($push)
const updateRes = db.students.updateOne(
  { studentId: "TEST-999" },
  {
    $set: { major: "Computer Science" },
    $push: { courses: "Cloud Computing" }
  }
);
print("=== UPDATE RESULT ===");
printjson(updateRes); // Expected: { acknowledged: true, matchedCount: 1, modifiedCount: 1 }

// Verify updated document
printjson(db.students.findOne({ studentId: "TEST-999" }));

// 3. Delete test document and confirm count returns to baseline
const deleteRes = db.students.deleteOne({ studentId: "TEST-999" });
print("=== DELETE RESULT ===");
printjson(deleteRes); // Expected: { acknowledged: true, deletedCount: 1 }
print("Final student count: " + db.students.countDocuments());
```

## Part D: Building Aggregation Pipelines in Stages

### Stages 1–3: Filter (`$match`), Group (`$group`), and Rank (`$sort` & `$limit`)
*Example: Summarize total sales volume and transaction count per product (or customer/salesperson), sorted highest to lowest.*

```javascript
printjson(
  db.sales.aggregate([
    // Stage 1: Filter out invalid or zero-quantity rows (adjust field names to match Sales.json)
    {
      $match: {
        quantity: { $gt: 0 }
      }
    },
    // Stage 2: Group by productId and calculate totals
    {
      $group: {
        _id: "$productId",
        transactionCount: { $sum: 1 },
        totalUnitsSold: { $sum: "$quantity" }
      }
    },
    // Stage 3: Sort descending by units sold and take top 5
    {
      $sort: { totalUnitsSold: -1 }
    },
    {
      $limit: 5
    }
  ]).toArray()
);
```

### Stage 4: Cross-Collection Join (`$lookup`)
*Example: Join `sales` with `products` so each sale includes product details.*

```javascript
printjson(
  db.sales.aggregate([
    {
      $lookup: {
        from: "products",           // Target collection to join
        localField: "productId",    // Field in 'sales'
        foreignField: "productId",  // Matching field in 'products' (must share same data type)
        as: "productDetails"        // Output array field name
      }
    },
    // Optional: Flatten single-match array into an object
    {
      $unwind: "$productDetails"
    },
    // Project clean output fields
    {
      $project: {
        _id: 0,
        saleId: 1,
        productId: 1,
        productName: "$productDetails.name",
        quantity: 1
      }
    }
  ]).toArray()
);
```
