---
layout: post
title: "Learning Postgres"
date: 2025-08-16 11:30:00 -0500
categories: homelab
tags: postgres github homelab

---

# 📘 Basic PostgreSQL Admin Commands

This cheat sheet includes common commands for managing PostgreSQL users, databases, and permissions.

## 🔐 Connect to PostgreSQL

### As the `postgres` user (Linux/macOS):
```bash
sudo -u postgres psql
```

### From inside a container:
```bash
docker exec -it <container_name> psql -U postgres
```

---

## 🛠️ Create a New Database

```sql
CREATE DATABASE my_database;
```

---

## 👤 Create a New User

```sql
CREATE USER my_user WITH PASSWORD 'strong_password_here';
```

> 🔒 Password must be quoted. Avoid weak passwords in production.

---

## 🎯 Grant User Full Access to a Database

```sql
GRANT ALL PRIVILEGES ON DATABASE my_database TO my_user;
```

This gives the user ability to connect, read, write, and manage schemas *within* the database.

---

## 🔄 Switch to a Database

```sql
\c my_database
```

---

## 🧱 Grant Access to Tables After Creation (Optional)

PostgreSQL **does not auto-grant table privileges** to users for future tables. Use this to give full access:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT ALL ON TABLES TO my_user;

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO my_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO my_user;
```

---

## 📋 List All Databases

```sql
\l
```

---

## 📋 List All Users/Roles

```sql
\du
```

---

## 🚪 Exit the PostgreSQL CLI

```sql
\q
```

---

## 🧼 Drop Database or User (Be Careful!)

```sql
DROP DATABASE my_database;
DROP USER my_user;
```

---

## ✅ Quick Example

```sql
-- As admin:
CREATE DATABASE blog;
CREATE USER writer WITH PASSWORD 'pen&paper!';
GRANT ALL PRIVILEGES ON DATABASE blog TO writer;
```

---

**Tip**: You can also manage PostgreSQL via GUI tools like `pgAdmin`, DBeaver, or TablePlus.
