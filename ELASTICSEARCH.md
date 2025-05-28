# **Elasticsearch** :

# **Step 1: Introduction to Elasticsearch**

## **In this step, we’ll cover the following:**

- ✅ What is Elasticsearch?
- ✅ Why use Elasticsearch?
- ✅ How Elasticsearch compares to traditional databases
- ✅ Basic architecture (Nodes, Clusters, Indices, Shards)

## **🔹 What is Elasticsearch?**

Elasticsearch is a **distributed search and analytics engine** that is built on top of **Apache Lucene**. It is primarily used for **full-text search, log analysis, and real-time data analytics**.

## **🟢 Key Features of Elasticsearch:**

- **Fast Search**: Uses an inverted index to quickly search millions of records.
- **Scalability**: Can handle large-scale data and grow as needed.
- **Distributed**: Data is spread across multiple nodes for fault tolerance and high availability.
- **JSON-Based**: Stores and processes data in JSON format.
- **Powerful Querying**: Supports full-text search, filtering, and aggregations.
- **RESTful API**: Interact with Elasticsearch using simple HTTP requests.

## **🛠️ Where is Elasticsearch Used?**

- **Search engines** (like Google, YouTube, and e-commerce search)
- **Log & monitoring systems** (used by Kibana, ELK Stack)
- **Business intelligence & analytics**
- **Auto-complete & recommendation systems** (e.g., "Did you mean?" feature)
- **Real-time data analysis** (financial markets, social media monitoring)

## **🔹 Why Use Elasticsearch?**

Traditional databases (like MySQL, PostgreSQL) are not optimized for searching text, while Elasticsearch is built for this purpose.

| **Use Case**         | **Why Elasticsearch?**                                 |
| -------------------- | ------------------------------------------------------ |
| Full-text search     | Much faster than SQL LIKE queries                      |
| Logging & monitoring | Easily analyze logs in real-time                       |
| Analytics dashboards | Aggregates large data sets efficiently                 |
| Scalability          | Handles millions of documents without performance loss |

## **🔹 How Elasticsearch Compares to Traditional Databases**

| Feature        | Elasticsearch                  | Relational Databases (MySQL, PostgreSQL) |
| -------------- | ------------------------------ | ---------------------------------------- |
| Data Format    | JSON Documents                 | Tables & Rows                            |
| Query Language | REST API & Query DSL           | SQL                                      |
| Search Speed   | Optimized for fast text search | Slower text searches                     |
| Scalability    | Distributed & scalable         | Scaling is difficult                     |
| Best Use Cases | Search, analytics, logs        | Structured transactional data            |

➡️ **Key Difference:**
Elasticsearch **indexes data like a search engine**, while traditional databases store data in structured tables.

## **🔹 Basic Architecture of Elasticsearch**

Elasticsearch follows a **distributed architecture**, meaning it runs on multiple servers (**nodes**) to ensure high performance and reliability.

📌 **1. Cluster**

- A cluster is a collection of one or more nodes working together.
- If a cluster has multiple nodes, Elasticsearch automatically distributes data across them.

📌 **2. Node**

- A node is a single instance of Elasticsearch.
- Nodes store data and perform indexing and search operations.
- **Types of Nodes:**
  - **Master Node** – Manages the cluster (adding/removing nodes).
  - **Data Node** – Stores data and processes queries.
  - **Client Node** – Routes requests without storing data.

📌 **3. Index**

- An index is like a **table** in relational databases.
- It contains documents that share a similar structure.
- **Example:** If you're storing product data, you can create an index called **"products"**.

📌 **4. Document**

- A document is a single unit of data in Elasticsearch (stored in JSON format).
- **Example:** A product document in an e-commerce store:

```json
{
  "name": "iPhone 15",
  "brand": "Apple",
  "price": 999,
  "category": "smartphone"
}
```

📌 **5. Shards**

- When an index grows large, Elasticsearch splits it into smaller parts called **shards**.
- Each shard can be stored on different nodes for better **performance** and **fault tolerance**.

📌 **6. Replicas**

- A **replica** is a copy of a shard.
- Used for **backup** and **load balancing** to prevent data loss.

📌 **🔹 Visual Representation of Elasticsearch Architecture**

```
Cluster
└── Node 1 (Master + Data)
│    ├── Index: "products"
│    │    ├── Primary Shard 1 → Stores documents
│    │    ├── Primary Shard 2 → Stores documents
│    │    ├── Replica Shard 1 → Backup of Primary Shard 1
│    │    ├── Replica Shard 2 → Backup of Primary Shard 2
│
└── Node 2 (Data Node)
     ├── Holds Replica Shards
     ├── Processes Queries
```

## **🔹 Summary**

- ✔ Elasticsearch is a **distributed search and analytics engine** designed for fast searching.
- ✔ It is **scalable, fault-tolerant, and optimized for full-text search**.
- ✔ Unlike traditional databases, Elasticsearch **stores data as JSON documents** and **uses an inverted index for fast searches**.
- ✔ It follows a **distributed architecture** consisting of **Clusters, Nodes, Indices, Shards, and Replicas**.

---

# **Step 2: Installing and Running Elasticsearch on Windows using Docker**

## **Prerequisites**

- Install **Docker Desktop for Windows** from [Docker Official Website](https://www.docker.com/products/docker-desktop).
- Enable **WSL 2 Backend** (for Windows 10/11 users):
  - Open **Docker Desktop** → Settings → Enable **WSL 2 based engine**.

### **1️⃣ Pull and Run Elasticsearch using Docker**

#### **Step 1: Pull Elasticsearch Docker Image**

```sh
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.7.0
```

#### **Step 2: Start an Elasticsearch Container**

```sh
docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.7.0
```

📌 **Explanation:**

- `-d` → Run in the background.
- `--name elasticsearch` → Name of the container.
- `-p 9200:9200` → Maps port 9200 (Elasticsearch API).
- `-e "discovery.type=single-node"` → Runs in single-node mode.

#### **Step 3: Check if Elasticsearch is Running**

```sh
docker ps
```

Open your browser and go to: [http://localhost:9200/](http://localhost:9200/)

✔ You should see a JSON response indicating Elasticsearch is running!

### **2️⃣ Running Kibana (Optional, Recommended)**

```sh
docker pull docker.elastic.co/kibana/kibana:8.7.0
docker run -d --name kibana -p 5601:5601 --link elasticsearch docker.elastic.co/kibana/kibana:8.7.0
```

Open: [http://localhost:5601/](http://localhost:5601/) to access Kibana.

## **Testing Elasticsearch with Postman**

### **1️⃣ Check if Elasticsearch is Running**

- **Method:** `GET`
- **URL:** `http://localhost:9200/`

✔ Expected Response:

```json
{
  "name": "your-node-name",
  "cluster_name": "docker-cluster",
  "version": { "number": "8.x.x" },
  "tagline": "You Know, for Search"
}
```

### **2️⃣ Index (Create) a Document**

- **Method:** `POST`
- **URL:** `http://localhost:9200/products/_doc/1`
- **Body (JSON):**

```json
{
  "name": "iPhone 15",
  "brand": "Apple",
  "price": 999
}
```

✔ Expected Response:

```json
{
  "_index": "products",
  "_id": "1",
  "result": "created"
}
```

### **3️⃣ Retrieve a Document**

- **Method:** `GET`
- **URL:** `http://localhost:9200/products/_doc/1`
  ✔ Response:

```json
{
  "_index": "products",
  "_id": "1",
  "_source": {
    "name": "iPhone 15",
    "brand": "Apple",
    "price": 999
  }
}
```

### **4️⃣ Update a Document**

- **Method:** `POST`
- **URL:** `http://localhost:9200/products/_update/1`
- **Body (JSON):**

```json
{
  "doc": {
    "price": 899
  }
}
```

✔ Response:

```json
{
  "_index": "products",
  "_id": "1",
  "result": "updated"
}
```

### **5️⃣ Delete a Document**

- **Method:** `DELETE`
- **URL:** `http://localhost:9200/products/_doc/1`
  ✔ Response:

```json
{
  "result": "deleted"
}
```

### **📌 Summary of Elasticsearch API Requests in Postman**

| Action                            | HTTP Method | URL                                        |
| --------------------------------- | ----------- | ------------------------------------------ |
| Check if Elasticsearch is running | `GET`       | `http://localhost:9200/`                   |
| Index (Create) a document         | `POST`      | `http://localhost:9200/products/_doc/1`    |
| Retrieve (Read) a document        | `GET`       | `http://localhost:9200/products/_doc/1`    |
| Update a document                 | `POST`      | `http://localhost:9200/products/_update/1` |
| Delete a document                 | `DELETE`    | `http://localhost:9200/products/_doc/1`    |

---

# **Step 3: Understanding Data in Elasticsearch**

## Introduction

This step explains how Elasticsearch stores and organizes data using indices, documents, and mappings. It also covers how to define a mapping, index data, and retrieve it using basic HTTP methods.

---

## 1. Basic Concepts in Elasticsearch

Elasticsearch is different from relational databases like MySQL. Instead of tables, rows, and columns, Elasticsearch uses:

| **Concept**  | **Elasticsearch Equivalent**    | **Traditional Database Equivalent** |
| ------------ | ------------------------------- | ----------------------------------- |
| **Cluster**  | Group of nodes working together | Entire database system              |
| **Node**     | Single Elasticsearch instance   | Database server                     |
| **Index**    | Collection of documents         | Database (e.g., MySQL database)     |
| **Document** | JSON object storing data        | Row in a table                      |
| **Field**    | Key-value pairs in a document   | Column in a table                   |
| **Mapping**  | Schema for documents            | Table schema                        |

---

## 2. Documents, Indices, and Types

### 1️⃣ What is a Document?

A document is a **JSON object** that contains actual data. Example:

```json
{
  "name": "iPhone 15",
  "brand": "Apple",
  "price": 999
}
```

✔ This is similar to a **row** in a relational database.

### 2️⃣ What is an Index?

An index is a **collection of documents**. Think of it like a **table** in a database.
Example: If you are storing product data, you can have an index called `products`.

### 3️⃣ What is a Type? (Deprecated)

In older versions, **types** were used like database tables inside an index.
Now, Elasticsearch **does not use types** (each index has just one document type).

---

## 3. How Elasticsearch Stores Data (Lucene & Inverted Index)

Unlike SQL databases, Elasticsearch does not store data in rows and columns. Instead, it uses:

- **Lucene** → The underlying search engine.
- **Inverted Index** → A data structure that allows fast full-text search.

### 📌 What is an Inverted Index?

An inverted index is a structure that **maps words to their document locations**.
Instead of storing documents as-is, Elasticsearch **indexes words separately**.

#### Example:

- **Document 1**: "The quick brown fox"
- **Document 2**: "The quick dog"

👉 Elasticsearch **inverts** the data and stores it like this:

| **Word** | **Document IDs** |
| -------- | ---------------- |
| "The"    | 1, 2             |
| "quick"  | 1, 2             |
| "brown"  | 1                |
| "fox"    | 1                |
| "dog"    | 2                |

### 📌 Benefit?

If you search for **"quick"**, Elasticsearch instantly finds **both documents (1 and 2)**.
This makes search operations **super fast** compared to traditional databases.

---

## 4. What is Mapping?

Mapping in Elasticsearch defines:

- What fields exist in a document.
- The data types for those fields (text, number, date, etc.).
- How Elasticsearch indexes and searches these fields.

### Example: Mapping for a `books` Index

We want to store books with:

- **Title** (text for searching).
- **Author** (exact match search).
- **Price** (numeric value).
- **Published date** (date type).

### **Creating the `books` Index with Mapping**

#### **HTTP Request:**

```http
PUT: http://localhost:9200/books
Content-Type: application/json
```

```json
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "author": { "type": "keyword" },
      "price": { "type": "float" },
      "published_date": { "type": "date" }
    }
  }
}
```

#### **Explanation:**

- **`text`** → Used for full-text search (e.g., search for "Harry Potter" inside titles).
- **`keyword`** → Used for exact matching (e.g., filtering books by author).
- **`float`** → Numeric value with decimals (e.g., price of a book).
- **`date`** → Stores dates for filtering (e.g., find books published after 2020).

---

## 5. Indexing (Adding) a Document

After defining the mapping, we can add a book document to the `books` index.

#### **HTTP Request:**

```http
POST: http://localhost:9200/books/_doc/1
Content-Type: application/json
```

```json
{
  "title": "Harry Potter and the Sorcerer’s Stone",
  "author": "J.K. Rowling",
  "price": 19.99,
  "published_date": "1997-06-26"
}
```

✔ **This stores the book with ID `1` in Elasticsearch.**

---

## 6. Retrieving a Document

Now, let’s retrieve the book we just added.

#### **HTTP Request:**

```http
GET: http://localhost:9200/books/_doc/1
```

#### **Response:**

```json
{
  "_index": "books",
  "_id": "1",
  "_source": {
    "title": "Harry Potter and the Sorcerer’s Stone",
    "author": "J.K. Rowling",
    "price": 19.99,
    "published_date": "1997-06-26"
  }
}
```

✔ **Document successfully retrieved!**

---

## 7. Updating a Document

We can update the price of the book.

#### **HTTP Request:**

```http
POST: http://localhost:9200/books/_update/1
Content-Type: application/json
```

```json
{
  "doc": {
    "price": 17.99
  }
}
```

✔ **The price is now updated from `19.99` to `17.99`.**

---

## 8. Deleting a Document

If we no longer need a document, we can delete it.

#### **HTTP Request:**

```http
DELETE: http://localhost:9200/books/_doc/1
```

✔ **The book document is removed from Elasticsearch.**

---

## Summary

| **Concept**        | **Definition**                                       |
| ------------------ | ---------------------------------------------------- |
| **Index**          | Like a database in SQL (stores documents).           |
| **Document**       | A JSON object representing data (like a row in SQL). |
| **Mapping**        | Defines field types (like a table schema).           |
| **Inverted Index** | A fast search structure used in Elasticsearch.       |

---

# **Step 4: Searching in Elasticsearch**

## **Introduction**

This document provides a comprehensive guide on performing searches in Elasticsearch. You will learn how to execute full-text searches, exact match queries, filtering, wildcard searches, fuzzy searches, sorting, and pagination using Elasticsearch Query DSL.

---

## 1. How Searching Works in Elasticsearch

Elasticsearch provides powerful search capabilities using **Query DSL (Domain Specific Language)**, which is a JSON-based query language.

### 🔹 Types of Searches:

1. **Full-Text Search** → Used for searching inside text fields.
2. **Exact Match Search** → Used for filtering exact values.
3. **Range Search** → Used for filtering numeric and date values.
4. **Wildcard & Fuzzy Search** → Used for handling partial matches and typos.
5. **Sorting & Pagination** → Used for controlling search results efficiently.

---

## 2. Full-Text Search (`match` Query)

Use **match queries** to perform full-text searches on analyzed fields.

### Example: Searching for Books by Title

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "match": {
      "title": "Harry Potter"
    }
  }
}
```

✔ This will return books with **"Harry Potter"** in the title.

---

## 3. Exact Match Search (`term` Query)

Use **term queries** when searching for exact values, especially for `keyword` fields.

### Example: Finding Books by Author

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "term": {
      "author": "J.K. Rowling"
    }
  }
}
```

✔ Returns books where the author is **exactly** "J.K. Rowling".

---

## 4. Filtering Results (`range` Query)

Filters help improve search performance by excluding results that do not match specific conditions.

### Example: Finding Books Cheaper than $20

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "range": {
      "price": { "lt": 20 }
    }
  }
}
```

✔ Returns books where **price < 20**.

### Example: Finding Books Published After 2000

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "range": {
      "published_date": { "gte": "2000-01-01" }
    }
  }
}
```

✔ Returns books published after the year **2000**.

---

## 5. Wildcard & Fuzzy Search

Wildcard and fuzzy searches allow searching for partial matches or handling spelling errors.

### Example: Wildcard Search (`wildcard` Query)

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "wildcard": {
      "title": "Harry*"
    }
  }
}
```

✔ Returns books where the title **starts with "Harry"**.

### Example: Fuzzy Search (`fuzzy` Query)

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "Hary Poter",
        "fuzziness": 2
      }
    }
  }
}
```

✔ Matches **"Harry Potter"**, even if the user typed **"Hary Poter"**.

---

## 6. Sorting & Pagination

### Example: Sorting by Price (Low to High)

```json
GET http://localhost:9200/books/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "price": "asc" }
  ]
}
```

✔ Returns books **sorted by price in ascending order**.

### Example: Paginating Results

Fetch **page 2** with **5 results per page**:

```json
GET http://localhost:9200/books/_search
{
  "from": 5,
  "size": 5,
  "query": { "match_all": {} }
}
```

✔ Skips **first 5 results** and shows the **next 5**.

---

## 7. Summary Table

| **Query Type** | **Use Case**        | **Example Query**                                      |
| -------------- | ------------------- | ------------------------------------------------------ |
| **match**      | Full-text search    | Search `"Harry Potter"` in titles                      |
| **term**       | Exact match         | Find books by `"J.K. Rowling"`                         |
| **range**      | Number/date filters | Find books published after 2000                        |
| **wildcard**   | Partial match       | Find books starting with `"Harry*"`                    |
| **fuzzy**      | Typo handling       | Search `"Hary Poter"` and still match `"Harry Potter"` |
| **sort**       | Sorting results     | Sort by price (low to high)                            |
| **from/size**  | Pagination          | Skip 5 results, get next 5                             |

---

# **Step 5: Advanced Search & Aggregations in Elasticsearch**

## 🔹 Introduction

In this step, we will learn about **advanced searching techniques** and **aggregations** in Elasticsearch. You will understand how to:

- Combine multiple queries using **`bool` queries**
- Use **filters vs. queries** for performance optimization
- Summarize data using **aggregations** (count, average, min/max values)

---

## 🔹 1. Combining Multiple Queries (`bool` Query)

Elasticsearch allows you to combine multiple queries using `bool`. The `bool` query supports:

- **must** → Conditions that **must** match (AND logic).
- **should** → Conditions that **should** match (OR logic).
- **must_not** → Conditions that **must NOT** match (exclusion).
- **filter** → Similar to `must`, but faster (no scoring).

## 1. must (AND logic)

- All conditions in must must match.

### ✅ Example: Books titled "Harry Potter" AND priced below $20

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Harry Potter" } },
        { "range": { "price": { "lt": 20 } } }
      ]
    }
  }
}
```

## 2. should (OR logic)

- At least one of these should match. If you add a minimum_should_match, you can control how many must match.

### ✅ Example: Books that are either "Harry Potter" OR "Lord of the Rings"

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "Harry Potter" } },
        { "match": { "title": "Lord of the Rings" } }
      ],
      "minimum_should_match": 1
    }
  }
}
```

## 3. 🚫 must_not (NOT logic)

- Conditions that must not match.

### ✅ Example: Books titled "Harry Potter" but not written by 'J.K. Rowling'

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Harry Potter" } }
      ],
      "must_not": [
        { "match": { "author": "J.K. Rowling" } }
      ]
    }
  }
}
```

## 4. ⚡ filter (Faster than must, no scoring)

- Used for exact matches or ranges, doesn’t affect the relevance score, so it’s faster.

### ✅ Example: "Harry Potter" books priced under $20 AND published after 2010

```json
GET http://localhost:9200/books/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Harry Potter" } }
      ],
      "filter":
        { "range": { "price": { "lt": 20 } } }
    }
  }
}
```

---

## 🔹 2. Filtering vs. Querying (Performance Optimization)

| Type     | Use Case                      | Performance Impact        |
| -------- | ----------------------------- | ------------------------- |
| `must`   | Full-text search              | Slower (calculates score) |
| `filter` | Exact matches & range queries | Faster (no scoring)       |

✅ **Use `filter` for exact values, numbers, and date queries** to optimize performance.

---

## 🔹 3. Aggregations (Summarizing Data)

Elasticsearch allows you to summarize and analyze data using **aggregations**.

### **Example 1: Count Books by Author (`terms` Aggregation)**

```json
GET http://localhost:9200/books/_search
{
  "size": 0,
  "aggs": {
    "books_by_author": {
      "terms": { "field": "author.keyword" }
    }
  }
}
```

📌 **Example Output:**

```json
"aggregations": {
  "books_by_author": {
    "buckets": [
      { "key": "J.K. Rowling", "doc_count": 10 },
      { "key": "George R.R. Martin", "doc_count": 5 }
    ]
  }
}
```

✔ **J.K. Rowling has 10 books, George R.R. Martin has 5.**

### **Example 2: Find the Average Book Price**

```json
GET http://localhost:9200/books/_search
{
  "size": 0,
  "aggs": {
    "average_price": {
      "avg": { "field": "price" }
    }
  }
}
```

📌 **Example Output:**

```json
"aggregations": {
  "average_price": { "value": 25.45 }
}
```

✔ The **average book price is $25.45**.

### **Example 3: Find the Cheapest and Most Expensive Book**

```json
GET http://localhost:9200/books/_search
{
  "size": 0,
  "aggs": {
    "cheapest_book": { "min": { "field": "price" } },
    "most_expensive_book": { "max": { "field": "price" } }
  }
}
```

📌 **Example Output:**

```json
"aggregations": {
  "cheapest_book": { "value": 9.99 },
  "most_expensive_book": { "value": 49.99 }
}
```

✔ The **cheapest book costs $9.99**, and the **most expensive book costs $49.99**.

---

## 🔹 4. Nested Aggregations (Grouping & Stats Together)

You can **combine multiple aggregations** to get deeper insights.

### **Example 4: Find Average Price of Books Per Author**

```json
GET http://localhost:9200/books/_search
{
  "size": 0,
  "aggs": {
    "books_by_author": {
      "terms": { "field": "author.keyword" },
      "aggs": {
        "average_price": {
          "avg": { "field": "price" }
        }
      }
    }
  }
}
```

📌 **Example Output:**

```json
"aggregations": {
  "books_by_author": {
    "buckets": [
      { "key": "J.K. Rowling", "doc_count": 10, "average_price": { "value": 22.75 } },
      { "key": "George R.R. Martin", "doc_count": 5, "average_price": { "value": 30.50 } }
    ]
  }
}
```

✔ **J.K. Rowling's books average $22.75**, while **George R.R. Martin's books average $30.50**.

---

## 🔹 Summary

| Concept                   | Example Use Case                                                    |
| ------------------------- | ------------------------------------------------------------------- |
| `bool` Query              | Combine multiple conditions (e.g., `"Harry Potter" AND price < 20`) |
| `must`                    | Requires a match (AND logic)                                        |
| `should`                  | Matches one or more conditions (OR logic)                           |
| `filter`                  | Faster for exact matches (no scoring)                               |
| `terms` Aggregation       | Count books by author                                               |
| `avg` Aggregation         | Find average price of books                                         |
| `min` / `max` Aggregation | Find the cheapest/most expensive book                               |

---

# **Step 6 : Managing Data in Elasticsearch**

## **Introduction**

In this step, we will explore how to efficiently manage data in Elasticsearch. This includes indexing, updating, deleting, and handling large data sets effectively.

---

## **1. Adding Data Efficiently (Bulk Indexing)**

Instead of inserting documents one by one, the Bulk API allows multiple documents to be added at once.

### **Example: Bulk Insert Books**

```json
POST http://localhost:9200/books/_bulk

{ "index": { "_id": "1" } }
{ "title": "Harry Potter and the Sorcerer’s Stone", "author": "J.K. Rowling", "price": 19.99, "published_date": "1997-06-26" }
{ "index": { "_id": "2" } }
{ "title": "A Game of Thrones", "author": "George R.R. Martin", "price": 25.99, "published_date": "1996-08-06" }
{ "index": { "_id": "3" } }
{ "title": "The Hobbit", "author": "J.R.R. Tolkien", "price": 15.50, "published_date": "1937-09-21" }
```

---

## **2. Updating Existing Data**

To modify existing data, use the Update API.

### **Example: Update the Price of a Book**

```json
POST http://localhost:9200/books/_update/1
{
  "doc": { "price": 17.99 }
}
```

### **Example: Increase All Book Prices by $2**

```json
POST http://localhost:9200/books/_update_by_query
{
  "script": {
    "source": "ctx._source.price += 2",
    "lang": "painless"
  },
  "query": {
    "match_all": {}
  }
}
```

---

## **3. Deleting Documents**

To remove data, delete single or multiple documents.

### **Example: Delete a Book by ID**

```json
DELETE http://localhost:9200/books/_doc/3
```

### **Example: Delete All Books by a Specific Author**

```json
POST http://localhost:9200/books/_delete_by_query
{
  "query": {
    "term": { "author": "J.K. Rowling" }
  }
}
```

---

## **4. Using Index Aliases**

Index aliases allow us to switch between indices without changing the application.

### **Example: Create an Alias for an Index**

```json
POST http://localhost:9200/_aliases
{
  "actions": [
    { "add": { "index": "books_v1", "alias": "books" } }
  ]
}
```

### **Example: Switch Alias to a New Index**

```json
POST http://localhost:9200/_aliases
{
  "actions": [
    { "remove": { "index": "books_v1", "alias": "books" } },
    { "add": { "index": "books_v2", "alias": "books" } }
  ]
}
```

---

## **5. Handling Large Data Sets**

To manage large-scale Elasticsearch data effectively:

- ✅ Use **Bulk API** for efficient indexing.
- ✅ Use **Index Aliases** for smooth transitions.
- ✅ Use **\_update_by_query** instead of deleting/reinserting data.
- ✅ Be cautious with **\_delete_by_query**, as it can be expensive.

---

## **Summary**

| **Action**                | **API Used**       | **Example**                       |
| ------------------------- | ------------------ | --------------------------------- |
| Add multiple documents    | `_bulk`            | Insert multiple books at once     |
| Update a document         | `_update`          | Change price of a book            |
| Update multiple documents | `_update_by_query` | Increase all book prices by $2    |
| Delete a document         | `_doc/{id}`        | Delete a book by ID               |
| Delete multiple documents | `_delete_by_query` | Delete all books by an author     |
| Use an alias for index    | `_aliases`         | Use "books" instead of "books_v1" |

---

# **Step 7: Performance Optimization in Elasticsearch**

Elasticsearch is powerful, but improper configuration can lead to **slow searches, high memory usage, and cluster failures**. This guide covers **best practices for performance optimization**.

---

## **1. Understanding How Elasticsearch Works Internally**

To optimize Elasticsearch, we need to understand how it **stores, indexes, and retrieves data**.

### **Internal Components of Elasticsearch**

| **Component** | **Function**                      |
| ------------- | --------------------------------- |
| **Cluster**   | A group of nodes working together |
| **Node**      | A single Elasticsearch instance   |
| **Index**     | A collection of documents         |
| **Shard**     | A subdivision of an index         |
| **Segment**   | A unit of storage within a shard  |

### **How a Search Query Works Internally**

1. **Query Routing** → The query is sent to the correct shard(s).
2. **Search Execution** → Elasticsearch finds relevant documents using an **inverted index**.
3. **Scoring and Ranking** → Documents are scored using **BM25** (default ranking algorithm).
4. **Aggregation and Sorting** → Elasticsearch applies filters, sorts results, and applies aggregations.
5. **Results Merging** → Data is combined and returned.

---

## **2. Index Settings and Optimizations**

Efficient indexing can **reduce memory usage and improve search speed**.

### **1. Choose the Right Number of Shards**

By default, an index has **5 primary shards**, but this can be **too many or too few**.

**Best Practice:**

- Set **shards = number of data nodes** (for balanced performance).
- Use **1 shard per 50GB of data**.

```json
PUT my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```

### **2. Use Proper Field Types**

Avoid using `text` fields when `keyword` is sufficient.

```json
PUT my_index/_mapping
{
  "properties": {
    "name": { "type": "keyword" },
    "description": { "type": "text" },
    "price": { "type": "float" }
  }
}
```

### **3. Use Index Lifecycle Management (ILM)**

For **time-series data**, **old indices should be archived or deleted**.

```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

## **3. Caching for Faster Searches**

Elasticsearch has **three main types of caching**:

| **Cache Type**          | **Function**                                           |
| ----------------------- | ------------------------------------------------------ |
| **Query Cache**         | Stores **frequent queries** to avoid re-executing them |
| **Field Data Cache**    | Speeds up **sorting & aggregations**                   |
| **Shard Request Cache** | Caches **entire search requests**                      |

### **Enable Caching for Frequent Queries**

```json
GET my_index/_search
{
  "query": {
    "match": { "status": "active" }
  },
  "request_cache": true
}
```

---

## **4. Scaling Elasticsearch**

Elasticsearch scales **horizontally** (by adding nodes) and **vertically** (by increasing resources).

### **Horizontal Scaling (Adding Nodes)**

✅ Add more nodes to distribute the load.

```bash
# Start additional Elasticsearch nodes
docker run -d --name es-node2 -e "discovery.seed_hosts=es-node1" elasticsearch:8.0.0
```

### **Vertical Scaling (Optimizing Resources)**

#### **1. Optimize JVM Heap Size**

- Set heap size to **50% of available RAM** (Max **32GB**).
- Edit `jvm.options`:

```bash
-Xms8g
-Xmx8g
```

#### **2. Use Coordinating Nodes for Large Queries**

A **coordinating node** handles requests **without storing data**.

```bash
elasticsearch -E node.roles=[]
```

---

## **5. Handling Large-Scale Searches Efficiently**

### **1. Use `_source` Filtering**

Reduce the size of the returned data by selecting only needed fields.

```json
GET my_index/_search
{
  "_source": ["name", "price"],
  "query": {
    "match": { "category": "electronics" }
  }
}
```

### **2. Use `pre_filter_shard_size` for Distributed Searches**

Avoid searching all shards unnecessarily.

```json
GET my_index/_search
{
  "pre_filter_shard_size": 3,
  "query": {
    "match": { "description": "laptop" }
  }
}
```

### **3. Use `search_after` Instead of `from + size`**

For **deep pagination**, use `search_after` instead of `from + size`.

```json
GET my_index/_search
{
  "size": 10,
  "search_after": [100000],
  "sort": ["_id"]
}
```

---

## **Summary**

| **Optimization**                      | **Benefit**                          |
| ------------------------------------- | ------------------------------------ |
| **Tune Shard Settings**               | Avoids too many/too few shards       |
| **Use Proper Data Types**             | Saves memory and improves filtering  |
| **Enable Caching**                    | Speeds up repeated queries           |
| **Scale with More Nodes**             | Distributes search load              |
| **Optimize JVM Heap**                 | Prevents out-of-memory crashes       |
| **Use `_source` Filtering**           | Reduces response size                |
| **Use `search_after` for Pagination** | Improves deep pagination performance |

---

# **Step 8: Securing Elasticsearch** 🔐

Elasticsearch does not have built-in security enabled by default, making it vulnerable to unauthorized access, data leaks, and cyberattacks. This guide covers best practices for securing Elasticsearch.

---

## **1️⃣ User Authentication & Role-Based Access Control (RBAC)**

By default, anyone can access your Elasticsearch cluster without authentication. We must enable security features and define user roles.

### **📌 Step 1: Enable Elasticsearch Security**

Edit the `elasticsearch.yml` file:

```yaml
xpack.security.enabled: true
xpack.security.authc.api_key.enabled: true
```

Restart Elasticsearch:

```bash
systemctl restart elasticsearch
```

### **📌 Step 2: Set Up Users & Passwords**

Run:

```bash
elasticsearch-setup-passwords interactive
```

This will ask for passwords for built-in users like `elastic` (superuser).

### **📌 Step 3: Create Role-Based Access Control (RBAC)**

#### **Create a Custom Role (Read-Only)**

```json
POST _security/role/read_only_user
{
  "indices": [
    {
      "names": ["orders"],
      "privileges": ["read"]
    }
  ]
}
```

#### **Assign a Role to a User**

```json
POST _security/user/john_doe
{
  "password": "secure_password",
  "roles": ["read_only_user"]
}
```

---

## **2️⃣ Securing Elasticsearch APIs**

Elasticsearch APIs can be publicly accessible, posing a security risk.

## **🔹 Best Practices to Secure APIs**

### **📌 1. Disable Public Access to Elasticsearch**

Change `elasticsearch.yml`:

```yaml
network.host: 127.0.0.1
```

### **📌 2. Enable HTTPS for Secure API Communication**

Generate a self-signed certificate:

```bash
bin/elasticsearch-certutil cert --pem
```

Enable SSL in `elasticsearch.yml`:

```yaml
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/elastic-cert.pem
```

### **📌 3. Secure API Access Using API Keys**

#### **Generate an API Key**

```json
POST _security/api_key
{
  "name": "readonly-key",
  "role_descriptors": {
    "readonly_role": {
      "indices": [
        {
          "names": ["orders"],
          "privileges": ["read"]
        }
      ]
    }
  }
}
```

#### **Use API Key for Requests**

```bash
curl -H "Authorization: ApiKey YOUR_GENERATED_KEY" -X GET "https://localhost:9200/orders/_search"
```

---

## **3️⃣ Preventing Unauthorized Access & Common Security Threats**

| **Threat**              | **Solution**                                           |
| ----------------------- | ------------------------------------------------------ |
| **Unauthorized Access** | Enable authentication (`xpack.security.enabled: true`) |
| **DDoS Attacks**        | Rate-limit API requests                                |
| **Data Leaks**          | Restrict public access (`network.host: 127.0.0.1`)     |
| **Unencrypted Traffic** | Use TLS/SSL encryption                                 |
| **Weak Passwords**      | Enforce strong passwords & use API keys                |
| **Open Index Deletion** | Disable `action.auto_create_index`                     |

### **📌 1. Disable Automatic Index Creation**

Edit `elasticsearch.yml`:

```yaml
action.auto_create_index: false
```

### **📌 2. Implement Rate-Limiting for API Requests**

#### **Nginx Rate-Limiting Configuration**

```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
server {
  location / {
    limit_req zone=api_limit burst=20;
  }
}
```

### **📌 3. Restrict Access with IP Whitelisting**

#### **Example (Nginx)**

```nginx
location / {
  allow 192.168.1.100;
  deny all;
}
```

---

## **✅ Summary**

| **Security Measure**        | **Benefit**                         |
| --------------------------- | ----------------------------------- |
| **Enable Authentication**   | Prevents unauthorized access        |
| **Use RBAC**                | Restricts users to specific indices |
| **Secure APIs with HTTPS**  | Encrypts sensitive data             |
| **Use API Keys**            | Securely authenticate API requests  |
| **Disable Public Access**   | Protects against external attacks   |
| **Rate-Limit API Requests** | Prevents DDoS attacks               |
| **Restrict IP Access**      | Blocks unauthorized traffic         |

---

# **Step 9: Using Elasticsearch with Node.js**

Elasticsearch provides a powerful search engine that can be integrated with Node.js for **fast and efficient searches**. This guide covers:

## **1️⃣ Setting up Elasticsearch with Node.js**

To interact with Elasticsearch in Node.js, we use the **official Elasticsearch client (`@elastic/elasticsearch`)**.

### **🔹 Step 1: Install Dependencies**

Create a new Node.js project and install required dependencies:

```bash
mkdir elasticsearch-nodejs
cd elasticsearch-nodejs
npm init -y
npm install @elastic/elasticsearch express body-parser dotenv
```

### **🔹 Step 2: Connect to Elasticsearch**

Create a `.env` file:

```ini
ELASTICSEARCH_NODE=http://localhost:9200
ELASTICSEARCH_USERNAME=elastic
ELASTICSEARCH_PASSWORD=yourpassword
```

Then, create `server.js` and connect to Elasticsearch:

```javascript
require("dotenv").config();
const { Client } = require("@elastic/elasticsearch");

const client = new Client({
  node: process.env.ELASTICSEARCH_NODE,
  auth: {
    username: process.env.ELASTICSEARCH_USERNAME,
    password: process.env.ELASTICSEARCH_PASSWORD,
  },
});

async function checkConnection() {
  try {
    const health = await client.cluster.health();
    console.log("Elasticsearch cluster health:", health);
  } catch (error) {
    console.error("Elasticsearch connection failed:", error);
  }
}

checkConnection();
```

Run the file:

```bash
node server.js
```

---

## **2️⃣ CRUD Operations in Elasticsearch with Node.js**

### **🔹 Create an Index**

```javascript
async function createIndex(indexName) {
  const exists = await client.indices.exists({ index: indexName });

  if (!exists) {
    await client.indices.create({ index: indexName });
    console.log(`Index '${indexName}' created.`);
  } else {
    console.log(`Index '${indexName}' already exists.`);
  }
}

createIndex("products");
```

### **🔹 Insert Data (Create Document)**

```javascript
async function addDocument(indexName, id, data) {
  await client.index({ index: indexName, id: id, body: data });
  console.log(`Document added: ${JSON.stringify(data)}`);
}

addDocument("products", "1", {
  name: "iPhone 15",
  category: "Electronics",
  price: 999,
  inStock: true,
});
```

### **🔹 Retrieve Data (Search by ID)**

```javascript
async function getDocument(indexName, id) {
  const response = await client.get({ index: indexName, id: id });
  console.log("Document:", response.body);
}

getDocument("products", "1");
```

### **🔹 Search for Products**

```javascript
async function searchProducts(category) {
  const response = await client.search({
    index: "products",
    body: {
      query: { match: { category: category } },
    },
  });
  console.log("Search Results:", response.body.hits.hits);
}

searchProducts("Electronics");
```

### **🔹 Update a Document**

```javascript
async function updateDocument(indexName, id, updates) {
  await client.update({
    index: indexName,
    id: id,
    body: { doc: updates },
  });
  console.log(`Document ${id} updated.`);
}

updateDocument("products", "1", { price: 899 });
```

### **🔹 Delete a Document**

```javascript
async function deleteDocument(indexName, id) {
  await client.delete({ index: indexName, id: id });
  console.log(`Document ${id} deleted.`);
}

deleteDocument("products", "1");
```

---

## **3️⃣ Building a Real-World Search API**

### **🔹 Step 1: Set Up Express**

Modify `server.js` to include Express:

```javascript
const express = require("express");
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());

app.get("/", (req, res) => {
  res.send("Elasticsearch Node.js API is running!");
});
```

### **🔹 Step 2: Create Search API Endpoint**

```javascript
app.get("/search", async (req, res) => {
  const { query } = req.query;

  const response = await client.search({
    index: "products",
    body: { query: { match: { name: query } } },
  });

  res.json(response.body.hits.hits);
});
```

### **🔹 Step 3: Start the Server**

```javascript
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

Run the API:

```bash
node server.js
```

### **🔹 Step 4: Test the API with Postman**

Send a **GET request** to:

```
http://localhost:3000/search?query=iPhone
```

✔ **You should see search results for products matching “iPhone.”**

---

## **✅ Summary**

| **Feature**                            | **Implementation**                                         |
| -------------------------------------- | ---------------------------------------------------------- |
| **Connect Node.js with Elasticsearch** | Installed `@elastic/elasticsearch` and set up a connection |
| **Perform CRUD Operations**            | Insert, Search, Update, Delete documents                   |
| **Build a Search API**                 | Used Express to create an API for searching products       |
| **Test API with Postman**              | Retrieved product data dynamically                         |

---

# Step 10: Advanced Topics in Elasticsearch

## 1. Time-Series Data with Elasticsearch

### What is Time-Series Data?

Time-series data is data collected over time. Examples include:

- **Stock prices** (e.g., price changes every second)
- **Server logs** (e.g., requests per second)
- **IoT sensor data** (e.g., temperature readings every minute)
- **User activity logs** (e.g., clicks, page views, purchases)

### Setting Up a Time-Series Index

```json
PUT logs-2025-03-01
{
  "mappings": {
    "properties": {
      "timestamp": { "type": "date" },
      "log_level": { "type": "keyword" },
      "message": { "type": "text" },
      "user_id": { "type": "keyword" }
    }
  }
}
```

### Inserting Sample Log Data

```json
POST logs-2025-03-01/_doc/1
{
  "timestamp": "2025-03-01T12:34:56",
  "log_level": "ERROR",
  "message": "Database connection failed",
  "user_id": "12345"
}
```

### Searching Logs for a Specific Date Range

```json
GET logs-2025-03-01/_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "2025-03-01T00:00:00",
        "lte": "2025-03-01T23:59:59"
      }
    }
  }
}
```

### Best Practices

- Use separate indices per time period (daily/monthly).
- Use ILM (Index Lifecycle Management) to delete old data automatically.
- Optimize storage using `doc_values` for analytics fields.

---

## 2. Using Elasticsearch with Logstash & Kibana (ELK Stack)

### ELK Stack Components

| Component         | Purpose                                                               |
| ----------------- | --------------------------------------------------------------------- |
| **Elasticsearch** | Stores & indexes data                                                 |
| **Logstash**      | Collects, processes & transforms data before sending to Elasticsearch |
| **Kibana**        | Visualizes data with dashboards and graphs                            |

### Setting Up ELK Stack with Docker

Create a `docker-compose.yml` file:

```yaml
version: "3"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.6.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:8.6.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"

  logstash:
    image: docker.elastic.co/logstash/logstash:8.6.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5044:5044"
```

Run ELK Stack:

```bash
docker-compose up -d
```

### Example Logstash Pipeline Configuration (`logstash.conf`)

```plaintext
input {
  file {
    path => "/var/log/syslog"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:log_level} %{GREEDYDATA:message}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### Access Kibana

Visit **http://localhost:5601** and create dashboards for logs.

---

## 3. Real-World Use Cases

### 1. E-Commerce Search

Example Query: Search for Laptops Under $1000

```json
GET products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "category": "laptop" } },
        { "range": { "price": { "lte": 1000 } } }
      ]
    }
  }
}
```

### 2. Logging & Monitoring

Example Query: Find Error Logs from the Last 24 Hours

```json
GET logs-2025-03-01/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "log_level": "ERROR" } },
        { "range": { "timestamp": { "gte": "now-1d/d", "lte": "now/d" } } }
      ]
    }
  }
}
```

### 3. Analytics & Business Intelligence

Example Query: Count Users by Country

```json
GET users/_search
{
  "size": 0,
  "aggs": {
    "users_by_country": {
      "terms": { "field": "country.keyword" }
    }
  }
}
```

---

## Summary of Step 10

| Topic                    | Key Learnings                                                                         |
| ------------------------ | ------------------------------------------------------------------------------------- |
| **Time-Series Data**     | Storing & querying log data with timestamps 📆                                        |
| **ELK Stack**            | Collecting logs with Logstash, storing in Elasticsearch, and visualizing in Kibana 📊 |
| **Real-World Use Cases** | E-commerce search, log analysis, and business intelligence 📈                         |

---
