
You usually do **not** create many tiny MongoDB servers for each little thing unless you have strong reasons. Most setups:

- One MongoDB cluster (or single instance in dev)
- Many databases inside it
- Many collections per database

But you **can** run multiple servers when you really need hard isolation.

### 1\. Differentiate databases on the *same* server

Same host/IP and port, different DB names:

- URI for service A:  
  `mongodb://localhost:27017/serviceA_db`
- URI for service B:  
  `mongodb://localhost:27017/serviceB_db`

In C (conceptually, matching your `mongo_connect`):

```c
// Service A
const char *uri_a = "mongodb://localhost:27017/serviceA_db";
mongo_status_t status_a = mongo_connect(uri_a);

// Service B
const char *uri_b = "mongodb://localhost:27017/serviceB_db";
mongo_status_t status_b = mongo_connect(uri_b);
```

This is the **usual** microservice pattern: one cluster, many logical DBs.

### 2\. Different servers (different IP/port)

If you want two **physically/logically separate** servers:

- Server 1: `192.168.1.10:27017`, DB `serviceA_db`
- Server 2: `192.168.1.11:27017`, DB `serviceB_db`

URIs:

```c
const char *uri_a = "mongodb://192.168.1.10:27017/serviceA_db";
const char *uri_b = "mongodb://192.168.1.11:27017/serviceB_db";
```

Or same host, different ports (two `mongod` processes):

```bash
# Server for service A
mongod --port 27017 --dbpath /data/serviceA

# Server for service B
mongod --port 27018 --dbpath /data/serviceB
```

URIs:

```c
const char *uri_a = "mongodb://localhost:27017/serviceA_db";
const char *uri_b = "mongodb://localhost:27018/serviceB_db";
```

### 3\. When multiple servers make sense

Use **separate MongoDB servers** when you need:

- Strong resource isolation (CPU, disk, memory)
- Different security domains / compliance boundaries
- Different lifecycle (one can be restarted, upgraded, or wiped without affecting the other)

Otherwise, for most projects and dev setups, it is simpler and good architecture to:

- Run **one** MongoDB instance
- Give each service its **own database name**
- Use different URIs (only the final DB segment changes) in each service.