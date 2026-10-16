
#### List Databases
`show dbs`

#### Access Database
`use database_name`

#### See Current Database
`db`

#### **Show collections in current DB**
`show collections`

#### **Create a collection**
`db.createCollection("users")`

#### **Drop a collection**
`db.users.drop()`

---

# 📑 **Document Operations (CRUD)**

## ➕ **Insert**

#### Insert one:
`db.users.insertOne({ name: "Sam", age: 20 })`

#### Insert many:
`db.users.insertMany([   { name: "Alice", age: 25 },   { name: "Bob", age: 30 } ])`

---

## 🔍 **Find**

#### Find all:
`db.users.find()`

#### Pretty-print:
`db.users.find().pretty()`

#### Find with filter:

`db.users.find({ age: 25 })`

#### Find one:
`db.users.findOne({ name: "Sam" })`

---

## ✏️ **Update**

#### Update one:
`db.users.updateOne(   { name: "Sam" },   { $set: { age: 21 } } )`

#### Update many:
`db.users.updateMany(   { age: { $gt: 20 } },   { $set: { active: true } } )`

---

## ❌ **Delete**

### Delete one:

`db.users.deleteOne({ name: "Sam" })`

### Delete many:

`db.users.deleteMany({ active: false })`

---

# ⚙️ **Admin & Utility Commands**

### **Show server status**

`db.serverStatus()`

### **Show current user**

`db.runCommand({ connectionStatus: 1 })`

### **List users**

`db.getUsers()`

### **List roles**

`db.getRoles({ showBuiltinRoles: true })`

---

# 🧪 **Aggregation**

### Basic aggregation example:

`db.users.aggregate([   { $match: { age: { $gte: 21 } } },   { $group: { _id: "$age", count: { $sum: 1 } } } ])`


#### How to Show Current Connections to mongosh

`db.serverStatus().connections`

#### How to List Details of Current connections to mongosh
`db.getSiblingDB("admin").aggregate([`
  `{ $currentOp: { allUsers: true, idleConnections: true } },`
  `{ $project: { client: 1, appName: 1, desc: 1, connectionId: 1 } }`
`])`

#### Kill Operations(Connections) From Server Side

If you **can’t** close the clients (e.g., a hung process), you can ask MongoDB to drop them. You don’t kill “connections” directly; you kill the ops associated with them.

In `mongosh`:

`db.getSiblingDB("admin").aggregate([`  
  `{ $currentOp: { allUsers: true, idleConnections: true } },`  
  `{ $project: { opid: 1, client: 1, appName: 1, desc: 1 } }`  
`])`

Then for an unwanted op:

`db.adminCommand({ killOp: 1, op: <opid> })`

MongoDB will close that operation, which forces the connection closed.

For normal development, just **exit the extra mongosh sessions**, and the corresponding connections will disappear automatically.