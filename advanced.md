# **Elasticsearch Advanced Guide - Manual Queries in Python** 🚀  

🔗 **Previous:** [Beginner Guide](beginner.md)  

---

## **📌 Overview**
Welcome to the **Advanced Guide** for manual Elasticsearch queries in Python.  
This tutorial builds on the **Beginner Guide** and introduces:  

✅ `bool` queries for **complex filtering**  
✅ `range` queries for **date & number filtering**  
✅ **Aggregations** (counting, grouping, stats)  
✅ **Sorting & Pagination techniques**  
✅ **Optimized raw Elasticsearch queries**  

---

## **1️⃣ Defining Our Log Document (Activity Logs)**
Since the **Beginner Guide** used `Product`, let’s now work with an **Activity Log**.  
Each log tracks:  

- `user_id`: The user performing the action  
- `action`: The type of action (e.g., "created", "updated")  
- `object_type`: What was affected (e.g., "Product", "Order")  
- `object_id`: The ID of the affected item  
- `timestamp`: When the action occurred  

### **📝 ActivityLogDocument (Elasticsearch DSL)**
```python
from django_elasticsearch_dsl import Document, fields

class ActivityLogDocument(Document):
    user_id = fields.IntegerField()
    action = fields.KeywordField()
    object_type = fields.KeywordField()
    object_id = fields.IntegerField()
    timestamp = fields.DateField()

    class Index:
        name = "activity_logs"
```
✔ **This replaces `ProductDocument` from the Beginner Guide.**  

---

## **2️⃣ Advanced Filtering with `bool` Queries**
The `bool` query allows complex filtering using:  

- `must` → **All conditions must match** (AND)  
- `should` → **At least one condition must match** (OR)  
- `must_not` → **Exclude documents** (NOT)  

### **🔹 Example: Find logs where**  

- **User `42` performed an action**  
- **The action was "updated"**  
- **It happened after `"2024-01-01"`**  

```python
from elasticsearch_dsl import Search, Q

connection = ActivityLogDocument._get_connection()
index_name = ActivityLogDocument._index._name

search = Search(using=connection, index=index_name).query(
    Q("bool", must=[
        Q("term", user_id=42),  
        Q("term", action="updated"),  
        Q("range", timestamp={"gte": "2024-01-01"})  
    ])
)

results = search.execute()
```
✔ **Use `bool` queries for complex filtering**.  

---

## **3️⃣ Searching for Multiple Actions (`terms` Query)**
If we want logs where **the action is either `"created"` or `"updated"`**, we use `terms`:  

```python
search = Search(using=connection, index=index_name).query(
    Q("terms", action=["created", "updated"])
)

results = search.execute()
```
✔ **`terms` is used for multiple possible values**.  

---

## **4️⃣ Date & Numeric Filtering (`range` Query)**
### **🔹 Example: Get logs in the last 7 days**  

```python
from datetime import datetime, timedelta

seven_days_ago = (datetime.now() - timedelta(days=7)).strftime("%Y-%m-%d")

search = Search(using=connection, index=index_name).query(
    Q("range", timestamp={"gte": seven_days_ago})
)

results = search.execute()
```
✔ **`range` works with dates and numbers.**  

---

## **5️⃣ Sorting & Pagination**
### **🔹 Sorting Logs by `timestamp` (Newest First)**  

```python
search = Search(using=connection, index=index_name).sort("-timestamp")

results = search.execute()
```
✔ **Use `-fieldname` for descending, `fieldname` for ascending.**  

### **🔹 Pagination (Fetching More Results)**  
By default, Elasticsearch returns only **10 results**.  

Use `[start:end]` for pagination:  

```python
search = Search(using=connection, index=index_name)[0:50]  # Get first 50 logs
results = search.execute()
```
✔ **Adjust `[start:end]` to control pagination.**  

---

## **6️⃣ Aggregations (Counting & Grouping)**
### **🔹 Count logs per action type**  

```python
search = Search(using=connection, index=index_name)
search.aggs.bucket("actions_count", "terms", field="action.keyword")

response = search.execute()

for bucket in response.aggregations.actions_count.buckets:
    print(bucket.key, bucket.doc_count)
```
✔ **Aggregations help analyze data efficiently.**  

---

## **7️⃣ Optimizing Queries**
To improve **query speed**:  

1️⃣ **Use filters (`filter`) instead of queries (`query`)**  
2️⃣ **Retrieve only necessary fields (`_source`)**  
3️⃣ **Paginate results**  

### **🔹 Example: Optimized Query**  

```python
search = Search(using=connection, index=index_name).filter(
    "term", user_id=42
).source(["action", "timestamp"]).extra(size=50)  # Fetch only required fields

results = search.execute()
```
✔ **Optimizations improve Elasticsearch performance.**  

---

## **8️⃣ Executing Raw Queries (Low-Level `elasticsearch-py`)**
If you need **full control**, use `elasticsearch`'s native API.  

### **🔹 Example: Raw Query**  

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(["http://localhost:9200"])

query = {
    "query": {
        "bool": {
            "must": [
                {"match": {"action": "created"}},
                {"range": {"timestamp": {"gte": "2024-01-01"}}}
            ]
        }
    }
}

response = es.search(index="activity_logs", body=query)
```
✔ **Raw queries allow custom Elasticsearch requests**.  

---

# **✅ Summary of Advanced Queries**
| **Query Type** | **Example** |
|--------------|-----------|
| **Bool Query (AND/OR/NOT)** | `Q("bool", must=[...], should=[...], must_not=[...])` |
| **Filter Multiple Actions (`terms`)** | `Q("terms", action=["created", "updated"])` |
| **Date/Number Filtering (`range`)** | `Q("range", timestamp={"gte": "2024-01-01"})` |
| **Sorting Results** | `.sort("-timestamp")` |
| **Aggregations (Stats & Grouping)** | `.aggs.bucket("actions_count", "terms", field="action.keyword")` |
| **Optimized Query (Limit Fields & Paginate)** | `.source(["action", "timestamp"]).extra(size=50)` |
| **Raw Query Using `elasticsearch-py`** | `es.search(index="activity_logs", body=query)` |

---

# **🎯 Next Steps**
✅ **You are now an Elasticsearch expert!** 🎉  

- **Try building a REST API for log searches** 🚀  
- **Experiment with complex aggregations**  
- **Integrate Elasticsearch into a real project**  

---

🔗 **Previous:** [Beginner Guide](beginner.md)  

🔥 Let me know if you need any refinements! 🚀
