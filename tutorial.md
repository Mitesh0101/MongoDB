# 🍃 MongoDB 101: The Casual Guide

Hey! Ready to master MongoDB? It's a NoSQL document database, which basically means it stores data as JSON-like documents. Flexible, fast, and pretty fun once you get the hang of it.

Let's dive in!

---

## 🏗️ 1. Create (Inserting Data)
Before you can query, you need some data!

### Single Insert
To add one document:
```javascript
db.students.insertOne({
  name: "Spongebob",
  age: 30,
  gpa: 3.2,
  fullTime: true
})
```

### Batch Insert
To add multiple at once (efficient!):
```javascript
db.students.insertMany([
  { name: "Patrick", age: 38, gpa: 1.5 },
  { name: "Sandy", age: 27, gpa: 4.0 },
  { name: "Mitesh", age: 18, gpa: 3.0 }
])
```

---

## 🔍 2. Read (Finding Data)
This is where the magic happens.

### Listing All Records
```javascript
db.students.find()
```

### Sorting and Limiting
Want the top 3 students by GPA? Easy:
```javascript
// 1 = Ascending, -1 = Descending
db.students.find().sort({ gpa: -1 }).limit(3)
```

### Projections (Picking Columns)
If you only want the `name` and `age`, use a projection object.
```javascript
// Syntax: db.collection.find({ query }, { projection })
db.students.find({}, { name: 1, age: 1 }) 
```
> [!TIP]
> MongoDB always includes the `_id` by default. To hide it, explicitly set it to 0 or false:
> `db.students.find({}, { name: 1, _id: 0 })`

---

## 🛠️ 3. Query Operators
Standard queries are cool, but operators are cooler.

### Comparison
*   `$ne`: Not equal to
*   `$gt`: Greater than (`$gte` for equal)
*   `$lt`: Less than (`$lte` for equal)
*   `$in`: In an array
*   `$nin`: Not in an array

**Example: Everyone older than 18 but younger than or equal to 30:**
```javascript
db.students.find({ age: { $gt: 18, $lte: 30 } })
```

**Example: Find specifically Rahul, Pranav, or Ishan:**
```javascript
db.students.find({ name: { $in: ["Rahul", "Pranav", "Ishan"] } })
```

### Logical Operators
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

## 🔄 4. Update (Modifying Records)
Don't overwrite the whole document! Use operators to change specific fields.

### Single Update
```javascript
// Update Rahul's age to 19
db.students.updateOne({ name: "Rahul" }, { $set: { age: 19 } })
```

### Bulk Update
```javascript
// Give everyone a GPA boost!
db.students.updateMany({}, { $inc: { gpa: 0.1 } })
```

### Useful Field Operators
*   `$set`: Updates or adds a field.
*   `$unset`: Removes a field.
*   `$inc`: Increments (or decrements) a number.
*   `$exists`: Find documents that have (or don't have) a field.

**Example: Add `fullTime: false` only if the field is missing:**
```javascript
db.students.updateMany({ fullTime: { $exists: false } }, { $set: { fullTime: false } })
```

---

## 🧹 5. Delete (Removing Data)
Be careful here—there's no "Undo"!

```javascript
// Delete just one
db.students.deleteOne({ name: "Patrick" })

// Delete all students with a failing GPA
db.students.deleteMany({ gpa: { $lt: 2.0 } })
```

---

## 🚀 6. Aggregation Pipeline
Aggregation is like a factory assembly line for your data. You pass documents through "stages."

### Common Stages:
*   `$match`: Filter documents (like `find`).
*   `$group`: Group by a field and calculate totals/averages.
*   `$sort`: Sort the results.
*   `$project`: Reshape the output.

**Example: Get the average GPA of all students:**
```javascript
db.students.aggregate([
  { $group: { _id: null, avgGpa: { $avg: "$gpa" } } }
])
```

---

## ⚡ 7. Performance & Indexes
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
> Indexes take up memory and slow down `insert/update` calls because the database has to update the index too. Use them mainly for fields you query often!

---

## 📦 8. Modern Collections
You can create collections with specific rules, like **Capped Collections** (circular buffers with a fixed size).

```javascript
db.createCollection("logs", { 
  capped: true, 
  size: 5242880, // 5MB in bytes
  max: 5000      // Max 5000 documents
})
```
When a capped collection hits its limit, it starts overwriting the oldest records. Perfect for logs!

---

Happy coding! 👩‍💻👨‍💻