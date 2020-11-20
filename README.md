# MongoDB Case Study

This case study walks you through a set of example operations in MongoDB. The collections can be created using data set files with the corresponding names.

You can consult the following for explanations of the operations:
* https://docs.mongodb.com/manual/crud/
* https://www.tutorialspoint.com/mongodb/index.htm

You can try the examples using the data sets and the following online tools:
* The `mongo` web shell on this page https://docs.mongodb.com/manual/tutorial/insert-documents/
* Mongo Playground at https://mongoplayground.net/

Some data sets come from the "Introduction to Data Mining and Analytics" book.

1. List all employee names.
```
db.Employees.find({}, {"Name":1, "_id":0})
```
Expected output:
```json
{ "Name" : "Janet Clive" }
{ "Name" : "Betty Love" }
{ "Name" : "Mary Smith" }
{ "Name" : "Josh Camper" }
{ "Name" : "Brian Lewis" }
{ "Name" : "Martha Swan" }
{ "Name" : "Veronica Clarence" }
{ "Name" : "Mike Jones" }
{ "Name" : "Mary Hayes" }
```

2. List student names and phone numbers.
```
db.Students.find({}, {"Name":1, "Phone":1, "_id":0})
```
Expected output:
```json
{ "Name" : "Monica Davis", "Phone" : "555-1212" }
{ "Name" : "Jimmy Lewis", "Phone" : "555-1213" }
{ "Name" : "Sara Knight", "Phone" : "555-1214" }
{ "Name" : "Billy Williams", "Phone" : "555-1214" }
```
3. List names of employees who are supervisors (the "Position" field is "Supervisor").
```
db.collection.find({
  "Position": "Supervisor"
},
{
  "Name": 1,
  "_id": 0
})
```
Expected output:
```json
{ "Name" : "Mary Smith" }
{ "Name" : "Martha Swan" }
```

4. List names and states of employees who do not live in Arizona.
```
db.Employees.find({"State": {$ne: "Arizona"}}, {"Name":1, "State":1, "_id":0})
```
Expected output:
```json
{ "Name" : "Janet Clive", "State" : "California" }
{ "Name" : "Betty Love", "State" : "Boston" }
{ "Name" : "Mary Smith", "State" : "Florida" }
{ "Name" : "Josh Camper", "State" : "AZ" }
{ "Name" : "Brian Lewis", "State" : "Washington" }
{ "Name" : "Veronica Clarence", "State" : "AZ" }
{ "Name" : "Mike Jones", "State" : "California" }
```

5. List orders with a total price ("Total") greater than or equal to $39.99.
```
db.Sales.find({"Total": {$gte: 39.99}}).pretty()
```
Expected output:
```json
{
	"_id" : 3,
	"SalesPerson" : 103,
	"Region" : "West",
	"CustomerID" : 403,
	"ProductID" : 3,
	"Total" : 39.99,
	"SaleDate" : "10-01-2020"
}
{
	"_id" : 4,
	"SalesPerson" : 104,
	"Region" : "West",
	"CustomerID" : 401,
	"ProductID" : 4,
	"Total" : 49.99,
	"SaleDate" : "09-01-2020"
}
{
	"_id" : 5,
	"SalesPerson" : 105,
	"Region" : "West",
	"CustomerID" : 401,
	"ProductID" : 5,
	"Total" : 59.99,
	"SaleDate" : "08-01-2020"
}
{
	"_id" : 8,
	"SalesPerson" : 108,
	"Region" : "West",
	"CustomerID" : 402,
	"ProductID" : 3,
	"Total" : 39.99,
	"SaleDate" : "08-01-2020"
}
{
	"_id" : 9,
	"SalesPerson" : 109,
	"Region" : "West",
	"CustomerID" : 402,
	"ProductID" : 4,
	"Total" : 49.99,
	"SaleDate" : "09-01-2020"
}
```

6. List names and states of employees who live in Arizona or AZ.
```
db.Employees.find({
  $or: [
    {
      "State": "Arizona"
    },
    {
      "State": "AZ"
    }
  ]
},
{
  "Name": 1,
  "State": 1,
  "_id": 0
})
```
Expected output:
```json
{ "Name" : "Josh Camper", "State" : "AZ" }
{ "Name" : "Martha Swan", "State" : "Arizona" }
{ "Name" : "Veronica Clarence", "State" : "AZ" }
{ "Name" : "Mary Hayes", "State" : "Arizona" }
```

7. List names and states of employees who are sales team members ("Position"="Sales") and do not live in California.
```
db.Employees.find({
  $and: [
    {
      "State": {
        $ne: "California"
      }
    },
    {
      "Position": "Sales"
    }
  ]
},
{
  "Name": 1,
  "State": 1,
  "_id": 0
}).pretty()
```
Expected output:
```json
{ "Name" : "Betty Love", "State" : "Boston" }
{ "Name" : "Josh Camper", "State" : "AZ" }
{ "Name" : "Brian Lewis", "State" : "Washington" }
{ "Name" : "Veronica Clarence", "State" : "AZ" }
{ "Name" : "Mary Hayes", "State" : "Arizona" }
```

8. List employee names in descending order.
```
db.Employees.find({}, {"Name":1, "_id":0}).sort({"Name":-1})
```
Expected output:
```json
{ "Name" : "Veronica Clarence" }
{ "Name" : "Mike Jones" }
{ "Name" : "Mary Smith" }
{ "Name" : "Mary Hayes" }
{ "Name" : "Martha Swan" }
{ "Name" : "Josh Camper" }
{ "Name" : "Janet Clive" }
{ "Name" : "Brian Lewis" }
{ "Name" : "Betty Love" }
```

9. Count the number of documents in the SalesTeam collection.
```
db.SalesTeam.aggregate(
  [
    {
      $group: {
        _id: null,
        count: {$sum: 1}
      }
    }
  ]
)
```
Expected output:
```json
{ "_id" : null, "count" : 9 }
```

10. Calculate the average product price using the "Products" collection.
```
db.Products.aggregate(
  [
    {
      $group: {
        _id: null,
        avgPrice: {$avg: "$Price"}
      }
    }
  ]
)
```
Expected output:
```json
{ "_id" : null, "avgPrice" : 39.989999999999995 }
```

11. Calculate total sales by customer using the "Sales" collection.
```
db.Sales.aggregate(
  [
    {
      $group: {
        _id: "$CustomerID",
        total: {
          $sum: "$Total"
        }
      }
    },
    {
      $sort: {total: -1}
    }
  ]
)
```
Expected output:
```json
{ "_id" : 402, "total" : 139.96 }
{ "_id" : 401, "total" : 129.97 }
{ "_id" : 403, "total" : 69.98 }
```

12. Update the employee with ID 12345 to the position of supervisor.
```
db.Employees.update(
  {"_id": 12345},
  {$set: {"Position": "Supervisor"}}
)
```
Before:
```json
>>> db.Employees.find({"_id":12345}).pretty()
{
	"_id" : 12345,
	"Name" : "Betty Love",
	"State" : "Boston",
	"Position" : "Sales",
	"Phone" : "555-1217",
	"Region" : "East",
	"Email" : "BD@MyBusiness.com"
}
```
After:
```json
>>> db.Employees.find({"_id":12345}).pretty()
{
	"_id" : 12345,
	"Name" : "Betty Love",
	"State" : "Boston",
	"Position" : "Supervisor",
	"Phone" : "555-1217",
	"Region" : "East",
	"Email" : "BD@MyBusiness.com"
}
```

13.  Delete the employee with ID 12345.
```
db.Employees.deleteOne({"_id": 12345})
or
db.Employees.remove({"_id": 12345})
```
Expected output:
```json
{ "acknowledged" : true, "deletedCount" : 1 }
```

14. Delete all employees who work in California.
```
db.Employees.deleteMany({"State": "California"})
```
Expected output:
```json
{ "acknowledged" : true, "deletedCount" : 2 }
```
```
db.Employees.remove({"State": "California"})
```
Expected output:
```json
WriteResult({ "nRemoved" : 2 })
```

15. Delete all documents in the Employees collection.
```
db.Employees.deleteMany({});
```
Expected output:
```json
{ "acknowledged" : true, "deletedCount" : 9 }
```
```
db.Employees.remove({})
```
Expected output:
```
WriteResult({ "nRemoved" : 9 })
```

## Geospatial Queries

Use the `$near` operator to return documents that are at least 1000 meters from and at most 5000 meters from the specified GeoJSON point, sorted in order from nearest to farthest.
```json
db.Places.find(
   {
     location:
       { $near:
          {
            $geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },
            $minDistance: 1000,
            $maxDistance: 5000
          }
       }
   }
)
```

Use the `geoNear` aggregation operation to return documents that match the query filter `{ category: "Parks" }`, sorted in order of nearest to farthest to the specified GeoJSON point.
```json
db.Places.aggregate( [
   {
      $geoNear: {
         near: { type: "Point", coordinates: [ -73.9667, 40.78 ] },
         spherical: true,
         query: { category: "Parks" },
         distanceField: "calcDistance"
      }
   }
] )
```

Check out [this tutorial](https://docs.mongodb.com/manual/tutorial/geospatial-tutorial/) for more advanced geospatial queries.
