To keep a MongoDB database dedicated to this project, you only need to **pick a database name and use it consistently in your code and in `mongosh`**. MongoDB stores it inside its own data directory, not inside your project files.

### 1\. Create a project\-specific database

In `mongosh`:

```bash
mongosh
use my_project_db          # choose any name, e.g. "clion_mongo_demo"
db.my_collection.insertOne({ init: true })  # creates the DB+collection on first write
```

Once you do a write, `my_project_db` will exist as a separate DB, isolated from others.

### 2\. Use that database from your C project

In your C code, your MongoDB URI should include the same DB name:

```c
const char *uri = "mongodb://localhost:27017/my_project_db";
mongo_status_t status = mongo_connect(uri);
```

Later, when you add real MongoDB C driver calls, you will use that URI to connect to `my_project_db`.

### 3\. Where is the DB stored on disk?

- MongoDB stores all databases in its **data directory**, usually:
  - macOS/Homebrew default: `/usr/local/var/mongodb` or `/opt/homebrew/var/mongodb`
- Inside that directory, each database is represented by internal files managed by `mongod`.

Your CLion project files (e.g. `src/database_practice/mongo_db_utils.c`) **do not contain the database**; they only contain the code that connects to it.