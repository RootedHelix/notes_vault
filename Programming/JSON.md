Related to [[programming]]
#json 

## AI Response
JSON has a small set of fundamental building blocks. Understanding the terminology is useful when writing parsers, validators, or schema checkers.

## Example JSON

```json
{
  "user": {
    "name": "Harrison",
    "age": 25,
    "active": true,
    "skills": ["C", "Python", "Embedded"],
    "address": null
  }
}
```

## 1. Object

An **object** is a collection of key-value pairs enclosed in curly braces.

```json
{
  "name": "Harrison",
  "age": 25
}
```

Terms:

- Object
    
- JSON object
    
- Dictionary (common in Python)
    
- Map / Associative array (common in systems programming)
    

---

## 2. Key

A **key** is the name that identifies a value within an object.

```json
{
  "name": "Harrison"
}
```

Here:

```text
"name"
```

is the key.

Requirements:

- Must be a string
    
- Must be enclosed in double quotes
    

---

## 3. Value

A **value** is the data associated with a key.

```json
{
  "name": "Harrison"
}
```

The value is:

```text
"Harrison"
```

Every key maps to exactly one value.

---

## 4. Key-Value Pair (Member)

Together, a key and value form a **member** or **key-value pair**.

```json
"name": "Harrison"
```

This entire line is one member.

---

## 5. Array

An **array** is an ordered collection of values enclosed in square brackets.

```json
[
  "C",
  "Python",
  "Embedded"
]
```

Arrays can contain:

- Strings
    
- Numbers
    
- Booleans
    
- Null
    
- Objects
    
- Other arrays
    

---

## 6. String

Text enclosed in double quotes.

```json
{
  "name": "Harrison"
}
```

Strings may contain:

```json
{
  "path": "C:\\Users\\Sam"
}
```

Escape sequences:

```text
\"   quote
\\   backslash
\n   newline
\t   tab
```

---

## 7. Number

JSON supports numeric values.

```json
{
  "age": 25,
  "price": 19.99
}
```

Examples:

```json
42
-10
3.14
1e6
```

Unlike C, JSON does not distinguish:

- int
    
- long
    
- float
    
- double
    

The parser decides how to store them.

---

## 8. Boolean

Represents true/false.

```json
{
  "active": true,
  "deleted": false
}
```

Valid values:

```json
true
false
```

Only lowercase.

---

## 9. Null

Represents the absence of a value.

```json
{
  "address": null
}
```

Useful when data exists conceptually but has no value.

---

## 10. Nested Object

An object contained inside another object.

```json
{
  "user": {
    "name": "Harrison"
  }
}
```

The object associated with `"user"` is a nested object.

---

## 11. Nested Array

An array inside another structure.

```json
{
  "skills": [
    "C",
    "Python"
  ]
}
```

---

## 12. Root Element

The top-level JSON value.

Examples:

Root object:

```json
{
  "name": "Harrison"
}
```

Root array:

```json
[
  "C",
  "Python"
]
```

The root may be any valid JSON value, though objects and arrays are most common.

---

## Parse Tree View

For a validator or parser, the JSON is often represented as a tree:

```text
Root Object
└── user (Object)
    ├── name (String)
    ├── age (Number)
    ├── active (Boolean)
    ├── skills (Array)
    │   ├── "C"
    │   ├── "Python"
    │   └── "Embedded"
    └── address (Null)
```

This tree representation is how libraries such as yyjson, cJSON, and Jansson internally model parsed JSON.

## Terms You'll See in Production Systems

|Term|Meaning|
|---|---|
|Document|Entire JSON payload|
|Root|Top-level element|
|Object|Collection of key-value pairs|
|Member|Single key-value pair|
|Key|Name of a field|
|Value|Data associated with a key|
|Array|Ordered collection of values|
|Node|Generic tree element|
|Leaf Node|Value with no children|
|Nested Object|Object inside object|
|Nested Array|Array inside another structure|
|Schema|Rules describing valid JSON structure|
|Serialization|Converting memory structures → JSON text|
|Deserialization / Parsing|JSON text → memory structures|

For the JSON validation tool you're building in C, the most important concepts are: **document → root → object → member → key → value → type checking**, because a validator typically walks the tree and verifies that each value matches the expected type and structure.