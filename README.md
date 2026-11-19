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

### Part B: Filtering & Projections (`find`)
Practice retrieving specific documents and limiting which fields return:

1. **Basic Filter:** Query `students` or `products` using an exact field match.
2. **Comparison Operators:** Query `products` or `sales` using range operators (`$gt`, `$gte`, `$lt`, `$lte`).
3. **Projections:** Return only the 2–3 fields you need while hiding `_id`:

```javascript
// Template: replace <collection>, <field>, and <value> with actual schema fields
printjson(
  db.<collection>.find(
    { <field>: { $gte: <value> } },
    { <field1>: 1, <field2>: 1, _id: 0 }
  ).toArray()
);
```

### Part C: Safe Document Updates & Deletes (`CRUD`)
Use the `students` collection first to practice modifying documents safely:

1. **Insert (`insertOne`):** Add one new student document with custom fields (for example, an array of enrolled courses).
2. **Update (`updateOne` / `$set` / `$push`):** Update a field on your newly added document and append an item to an array field.
   * **Checkpoint:** Print the update result and inspect `matchedCount` and `modifiedCount` to confirm only the intended document changed.
3. **Delete (`deleteOne`):** Remove your test document and verify `countDocuments()` returns to its original value.

```javascript
// Example CRUD workflow
const updateRes = db.students.updateOne(
  { /* unique filter */ },
  { $set: { /* field: newValue */ } }
);
printjson(updateRes);
```

### Part D: Building Aggregation Pipelines in Stages
When answering business questions from `business.txt`, build your `aggregate()` pipeline **one stage at a time** so you can inspect the output at each step:

1. **Stage 1 (`$match`):** Filter down to the relevant subset of documents first.
2. **Stage 2 (`$group`):** Group by a category, employee, place, or customer ID using accumulators (`$sum`, `$avg`, `$count`).
3. **Stage 3 (`$sort` & `$limit`):** Rank the grouped results from highest to lowest.
4. **Stage 4 (`$lookup` — Cross-Collection Join):** Combine related collections (such as `sales` with `products` or `customers`) when data lives across two files.

```javascript
// Incremental Pipeline Template
printjson(
  db.sales.aggregate([
    // Stage 1: Filter
    // { $match: { <statusField>: "<value>" } },

    // Stage 2: Group & Summarize
    // { $group: { _id: "$<groupField>", totalCount: { $sum: 1 } } },

    // Stage 3: Sort Descending
    // { $sort: { totalCount: -1 } }
  ]).toArray()
);
```
