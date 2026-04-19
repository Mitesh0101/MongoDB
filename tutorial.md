# 🍃 MongoDB 101: The Casual Guide

Hey! Ready to master MongoDB? It's a NoSQL document database, which basically means it stores data as JSON-like documents. Flexible, fast, and pretty fun once you get the hang of it.

This guide is designed so you can scan it in one glance and immediately understand what's happening. Let's dive in!

---

## 📂 1. The Basics: Collections & Variables
Before doing anything else, you need a place to store your documents. Think of **Collections** like tables in SQL.

### Creating & Dropping Collections
You can explicitly create a collection, though MongoDB is nice enough to create one automatically if you just insert data into a non-existent one!
```javascript
// Create a collection
db.createCollection("my_cool_collection")

// Delete an entire collection (Be careful!)
db.my_cool_collection.drop()
```

### Shell Variables
If you're using the Mongo shell and want to save yourself some typing, you can assign objects to variables:
```javascript
// Save your query object to a variable
const hyundaiQuery = { maker: "Hyundai" }

// Pass the variable directly into your query!
db.cars.find(hyundaiQuery)
```

---

## 🏗️ 2. Create (Inserting Data)
Time to add some data so we actually have something to query.

### Single Insert
To add a single document:
```javascript
db.students.insertOne({
  name: "Spongebob",
  age: 30,
  gpa: 3.2,
  fullTime: true
})
```

### Batch Insert
To add multiple objects at once (which is much faster than running single inserts in a loop):
```javascript
db.students.insertMany([
  { name: "Patrick", age: 38, gpa: 1.5 },
  { name: "Sandy", age: 27, gpa: 4.0 },
  { name: "Mitesh", age: 18, gpa: 3.0 }
])
```

---

## 🔍 3. Read (Finding Data)
This is where the magic happens. We query data using the `find()` method.

### Listing All Records
```javascript
db.students.find()
```

### Querying Nested Objects and Arrays
JSON allows deep nesting. MongoDB uses **"dot notation"** to dive into those layers.
> [!IMPORTANT]
> When using dot notation for nested fields, you **MUST** enclose the key in quotes!

```javascript
// Example 1: Finding nested object values
// Find cars where the nested "engine" document has a "type" of "Turbocharged"
db.cars.find({ "engine.type": "Turbocharged" })

// Example 2: Finding a value inside an array
// "features" is an array of strings. This will return any car that has "Sunroof" anywhere in its features array!
db.cars.find({ features: "Sunroof" })
```

### Filtering & Paginating Results (Cursor Methods)
When you run `find()`, MongoDB returns a "cursor". You can chain methods to this cursor to format your results.

```javascript
// COUNTING: How many students have a GPA strictly greater than 9?
db.students.find({ gpa: { $gt: 9 } }).count()

// SORTING: Top 3 students by GPA (1 = Ascending, -1 = Descending)
db.students.find().sort({ gpa: -1 }).limit(3)

// SKIPPING: Useful for pagination! Skip the first 3 records, then return the rest.
db.students.find().skip(3)
```

### Projections (Picking Specific Columns)
If your documents are huge and you only want to see specific fields (like `name` and `age`), use a projection object.
```javascript
// Syntax: db.collection.find({ query }, { projection })
// 1 means "show this", 0 means "hide this"
db.students.find({}, { name: 1, age: 1 }) 
```
> [!TIP]
> MongoDB always includes the `_id` by default in projections. To hide it, explicitly set it to 0 or false:
> `db.students.find({}, { name: 1, _id: 0 })`

---

## 🛠️ 4. Query Operators
Standard lookups are great, but operators give you real power to filter data accurately. 

### Comparison & Array Operators
*   `$eq` / `$ne`: Equal to / Not equal to
*   `$gt` / `$gte`: Greater than / Greater than or equal
*   `$lt` / `$lte`: Less than / Less than or equal
*   `$in` / `$nin`: Exists inside the specified array / Doesn't exist inside it
*   `$size`: Checks if an array has exactly this length.
*   `$all`: Checks if an array includes **all** the specified elements (order doesn't matter).

**Examples:**
```javascript
// Everyone older than 18 but younger than or equal to 30
db.students.find({ age: { $gt: 18, $lte: 30 } })

// Match any document where the name is specifically Rahul, Pranav, or Ishan
db.students.find({ name: { $in: ["Rahul", "Pranav", "Ishan"] } })

// Find cars that specifically have exactly 2 features in their array
db.cars.find({ features: { $size: 2 } })

// Find cars that possess BOTH Sunroof and Leather Seats in their features array
db.cars.find({ features: { $all: ["Sunroof", "Leather Seats"] } })
```

### Logical Operators
Combine multiple query conditions together.
*   `$and`, `$or`, `$nor`, `$not`

**Example: Students who are part-time OR have a GPA above 3.5:**
```javascript
db.students.find({
  $or: [
    { fullTime: false },
    { gpa: { $gt: 3.5 } }
  ]
})
```

---

## 🔄 5. Update (Modifying Records)
Don't overwrite the whole document by accident! Use operators to target specific fields.

### Single Updates & Upserts
```javascript
// Regular update: Update Rahul's age to 19
db.students.updateOne({ name: "Rahul" }, { $set: { age: 19 } })

// Upsert (Update + Insert): Update the model X7 to BMW. 
// If X7 doesn't exist yet, insert a brand new document!
db.cars.updateOne(
  { model: "X7" }, 
  { $set: { maker: "BMW" } }, 
  { upsert: true }
)
```

### Bulk Updates
```javascript
// Give EVERYONE (empty query object {}) a GPA boost of 0.1
db.students.updateMany({}, { $inc: { gpa: 0.1 } })

// Add fullTime: false to all records that don't currently have a fullTime field!
db.students.updateMany(
  { fullTime: { $exists: false } }, 
  { $set: { fullTime: false } }
)
```

### Working with Nested Objects & Arrays
Modifiers let you dive into arrays and objects easily.

*   `$set` / `$unset`: Adds/Updates a field OR removes it entirely.
*   `$push`: Appends an item to an array. (Use with `$each` to append multiple items).
*   `$pull`: Removes a specific item from an array.

**Examples:**
```javascript
// Add or Remove nested properties (Dot Notation)
db.cars.updateOne({ maker: "Honda", model: "City" }, { $set: { "engine.name": "V8" } })
db.cars.updateOne({ maker: "Honda", model: "City" }, { $unset: { "engine.name": "" } })

// Arrays: Push a single item
db.cars.updateOne({ maker: "Tata", model: "Nexon" }, { $push: { features: "Reverse Camera" } })

// Arrays: Push multiple items using $each!
db.cars.updateOne(
  { maker: "Tata", model: "Nexon" }, 
  { $push: { features: { $each: ["Reverse Camera", "LCD Display"] } } }
)

// Arrays: Pull (Remove) an item
db.cars.updateOne({ maker: "Tata", model: "Nexon" }, { $pull: { features: "Reverse Camera" } })
```

---

## 🧹 6. Delete (Removing Data)
Be careful here—there's no "Undo"!

```javascript
// Delete just one specific record
db.students.deleteOne({ name: "Patrick" })

// Delete all students matching the condition (e.g., failing GPA)
db.students.deleteMany({ gpa: { $lt: 2.0 } })
```

---

## 🗺️ 7. Data Modeling & Relationships
MongoDB provides two main ways to structure your connected data: **Embedded** and **Referenced**.

### Strategy 1: Embedded Documents (Denormalized)
You shove related data directly inside the main document. Great for data you frequently read together.
```javascript
// Notice how the 'engine' object and 'owners' array
// are fully contained within this single "Car" document.
{
  _id: ObjectId('...'),
  maker: 'Hyundai',
  model: 'Creta',
  engine: { type: 'Naturally Aspirated', cc: 1493 }, // Nested Object
  owners: [ { name: 'Raju', location: 'Mumbai' } ] // Nested Array of Objects
}
```

### Strategy 2: Referenced Documents (Normalized)
The "SQL" way. Data lives in separate collections, and you connect them using an ID hook.
```javascript
// 1. users collection
{ _id: 'user1', name: 'Amit Sharma' }

// 2. orders collection
{ _id: 'order1', user_id: 'user1', product: 'Laptop' }
```

### Joining Data with `$lookup`
If you used "Referenced Documents", you can manually join collections together using `$lookup` in an aggregation.
```javascript
// This joins the 'users' collection with their respective 'orders'.
db.users.aggregate([
  {
    $lookup: {
      from: "orders",          // The target collection to grab data from
      localField: "_id",       // The field in the CURRENT (users) collection
      foreignField: "user_id", // The corresponding field in the TARGET (orders) collection
      as: "orders_array"       // The name of the new array field to output
    }
  }
])
```

---

## 🚀 8. Aggregation Pipeline
Aggregation is like a factory assembly line. Documents go in, pass through various "Stages" (matching, sorting, formatting), and come out deeply analyzed on the other side. 

*You pass an array of stages to the `aggregate()` method.*

### 🛠 Core Stages (`$match`, `$group`, `$project`)
```javascript
// 1. $match & $count
// Find all Hyundai cars > 1400cc, then count how many there are.
db.cars.aggregate([
  { $match: { maker: "Hyundai", "engine.cc": { $gt: 1400 } } },
  { $count: "Total_cars" } 
])

// 2. $group
// Group ALL cars by brand (_id:"$maker") and count them using $sum
db.cars.aggregate([
  { $group: { _id: "$maker", TotalCars: { $sum: 1 } } }
])

// Advanced $group (Averages, Arrays, Max/Min)
db.sales.aggregate([
  { $group: {
      _id: "$category",
      totalAmount: { $sum: "$amount" },
      averageAmount: { $avg: "$amount" },
      maxAmount: { $max: "$amount" },
      uniqueAmounts: { $addToSet: "$amount" } // Keeps unique values only!
  }}
])

// 3. $project (Like Projections, but you can alter data on the fly)
// Show only Maker, Model, Fuel Type. Hide ID.
db.cars.aggregate([
  { $match: { maker: "Hyundai" } },
  { $project: { maker: 1, model: 1, fuel_type: 1, _id: 0 } }
])
```

### 🧹 Formatting Stages (`$sort`, `$limit`, `$skip`, `$unwind`, `$out`)
```javascript
// Cursor-like stages in the pipeline
db.cars.aggregate([
  { $match: { maker:"Hyundai" } },
  { $sort: { model: 1 } },  // Sort by model ascending
  { $skip: 3 },             // Skip the first 3
  { $limit: 3 }             // Limit return to 3
])

// $sortByCount: Groups by a field and automatically sorts by highest count!
db.cars.aggregate([{ $sortByCount: "$maker" }])

// $unwind: Flattens an array. If a car has 2 owners in its array, 
// $unwind will output 2 separate car documents (one for each owner).
db.cars.aggregate([{ $unwind: "$owners" }])

// $out: Want to save your results permanently into a brand new collection? 
// Use $out as the VERY LAST stage.
db.cars.aggregate([
  { $match: { maker: "Hyundai" } },
  { $out: "hyundai_cars" } // Creates a new collection or overwrites existing!
])
```

### 🧮 Computation & String Magic in `$project`
```javascript
// ARITHMETIC: Total cost of a car's service history
db.cars.aggregate([
  { $project: { 
      maker: 1, model: 1, 
      total_service_cost: { $sum: "$service_history.cost" }, // Sum up a nested array
      new_price_hike: { $add: ["$price", 55000] },           // Add values
      price_in_lakhs: { $divide: ["$price", 100000] },       // Divide values (Use $subtract to subtract)
      // PRO TIP: Use $toString to convert numbers if you want to add suffixes
      formatted_price: { $concat: [{ $toString: { $divide: ["$price", 100000] } }, " lakhs"] }
  }}
])

// STRINGS ($concat, $toUpper, $regexMatch)
db.cars.aggregate([
  { $project: { 
      _id: 0, 
      fullName: { $concat: ["$maker", " ", "$model"] }, // Combine strings
      maker: { $toUpper: "$maker" },                    // Uppercase
      model: { $toLower: "$model" },                    // Lowercase
      is_diesel: { $regexMatch: { input: "$fuel_type", regex: "Diesel" } } // Checks for a regex match (returns boolean)
  }}
])
```

### 🔀 Logic Gates (`$cond` and `$switch`)
Need `IF/ELSE` statements in your pipeline? You got it.
```javascript
// $cond (Ternary / Basic IF-ELSE)
db.cars.aggregate([
  { $project: { 
      fuel_category: {
        $cond: {
          if: { $eq: ["$fuel_type", "Petrol"] },
          then: "Petrol Car",
          else: "Non-Petrol Car"
        }
      }
  }}
])

// $switch (Complex IF-ELSE IF-ELSE chains)
db.cars.aggregate([
  { $project: { 
      budget_category: {
        $switch: {
          branches: [
            { case: { $lt: ["$price", 500000] }, then: "Budget" },
            { case: { $and: [ { $gte: ["$price", 500000] }, { $lte: ["$price", 1000000] } ] }, then: "MidRange" },
            { case: { $gt: ["$price", 1000000] }, then: "Premium" }
          ],
          default: "Unknown" // Fallback if no cases are met
        }
      }
  }}
])
```

---

## 🛡️ 9. Schema Validation
MongoDB is "schemaless" by default, which is flexible but can lead to messy data. You can lock it down using `$jsonSchema` to enforce rules!

```javascript
// You can apply this while creating a collection...
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "age"], // Fields that MUST exist
      properties: {
        name: {
          bsonType: "string",
          description: "Name should be string"
        },
        age: {
          bsonType: "int", // Enforce data type
          minimum: 18,     // Enforce value constraints
          description: "Age must be an integer and >= 18"
        }
      }
    }
  },
  validationLevel: "strict", // Will reject bad data instantly
  validationAction: "error"
})

// OR apply validation to an existing collection using the "collMod" command!
db.runCommand({
  collMod: "users",
  validator: { /* ... same $jsonSchema as above ... */ }
})
```

---

## ⚡ 10. Performance & Indexes
Indexes make queries lightning fast by creating a lookup table (B-Tree).

```javascript
// Create an index on 'name' in ascending order
db.students.createIndex({ name: 1 })

// See all your indexes
db.students.getIndexes()

// Remove an index (use the name shown in getIndexes)
db.students.dropIndex("name_1")
```
> [!WARNING]
> Indexes take up memory and slow down `insert/update` calls because the database has to update the index too. Use them strategically on fields you query constantly!

---

## 📦 11. Modern Collections
You can create collections with specific rules, like **Capped Collections** (circular buffers with a fixed size).

```javascript
db.createCollection("logs", { 
  capped: true, 
  size: 5242880, // Size in bytes (5MB)
  max: 5000      // Max 5000 documents
})
```
When a capped collection hits its limit, it just deletes the oldest records automatically to make room for new ones. Perfect for system logs!

---

Happy learning! 👩‍💻👨‍💻