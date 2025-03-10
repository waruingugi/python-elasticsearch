# **Beginner Guide: Elasticsearch Queries in Python (Manual)**

## **📌 Introduction**
Welcome to the first part of our Elasticsearch tutorial series!


This guide will teach you how to manually query Elasticsearch using Python **without relying on Django ORM**.

By the end of this tutorial, you will:  
✅ Set up Elasticsearch & Python  
✅ Connect to Elasticsearch  
✅ Create & Index Documents  
✅ Perform Simple Queries (`match`, `term`, `terms`)  
✅ Sort & Paginate Results  

**🔗 Next:** [Advanced Guide - Elasticsearch Queries in Python (Manual)](advanced.md)

---

## **1️⃣ Setting Up Elasticsearch & Python**
Before we begin querying, let’s ensure Elasticsearch is installed and running.

### **🔹 Install Elasticsearch & Python Packages**
```sh
pip install elasticsearch-dsl
```

### **🔹 Start Elasticsearch (if not running)**
If you have Elasticsearch installed, start it:
```sh
systemctl start elasticsearch  # For Linux
brew services start elasticsearch  # For macOS
```

### **🔹 Connect to Elasticsearch in Python**
```python
from elasticsearch_dsl import connections

# Create an Elasticsearch connection
connections.create_connection(alias="default", hosts=["http://localhost:9200"])
```

---

## **2️⃣ Creating & Indexing Documents**
Before we can query, we need some data in Elasticsearch.

### **🔹 Sample Index & Data**
```python
from elasticsearch_dsl import Document, Text, Keyword, Integer

class Product(Document):
    name = Text()
    category = Keyword()
    price = Integer()
    
    class Index:
        name = "products"  # Elasticsearch index name

# Create the index
Product.init()

# Add sample documents
Product(name="Laptop", category="Electronics", price=1200).save()
Product(name="Phone", category="Electronics", price=800).save()
Product(name="Shoes", category="Fashion", price=100).save()
```

---

## **3️⃣ Querying Elasticsearch (Basic Queries)**
### 🔹 **Get All Documents**
```python
from elasticsearch_dsl import Search

search = Search(using="default", index="products")
results = search.execute()

for product in results:
    print(product.name, product.category, product.price)
```

### 🔹 **Search by Exact Match (`term`)**
Find all products in the "Electronics" category:
```python
from elasticsearch_dsl import Q

search = Search(using="default", index="products").query(Q("term", category="Electronics"))
results = search.execute()

for product in results:
    print(product.name, product.price)
```

✔ **`term` is used for exact matches (case-sensitive).**

### 🔹 **Search by Partial Match (`match`)**
Find products where `name` contains "phone":
```python
search = Search(using="default", index="products").query(Q("match", name="phone"))
results = search.execute()

for product in results:
    print(product.name, product.price)
```

✔ **`match` is used for full-text search (not case-sensitive).**

### 🔹 **Search Multiple Values (`terms`)**
Find products in **both** "Electronics" and "Fashion":
```python
search = Search(using="default", index="products").query(Q("terms", category=["Electronics", "Fashion"]))
results = search.execute()

for product in results:
    print(product.name, product.category)
```

✔ **`terms` is used for searching multiple exact values.**

---

## **4️⃣ Sorting & Pagination**
### 🔹 **Sort by Price (Ascending)**
```python
search = Search(using="default", index="products").sort("price")
results = search.execute()

for product in results:
    print(product.name, product.price)
```

### 🔹 **Sort by Price (Descending)**
```python
search = Search(using="default", index="products").sort("-price")
results = search.execute()

for product in results:
    print(product.name, product.price)
```

### 🔹 **Pagination (Limit & Offset)**
Get the first **2** results:
```python
search = Search(using="default", index="products")[0:2]
results = search.execute()

for product in results:
    print(product.name, product.price)
```

✔ **Pagination helps optimize queries when dealing with large datasets.**

---

## **🎯 What’s Next?**
Now that you’ve mastered basic Elasticsearch queries, it’s time to go **deeper**! In the next guide, we’ll cover:
- `bool` Queries (`must`, `should`, `must_not`)
- Date & Numeric Filtering (`range`)
- Aggregations (Grouping & Stats)
- Optimizing Queries
- Raw Queries with `elasticsearch-py`

**🔗 Next:** [Advanced Guide - Elasticsearch Queries in Python (Manual)](advanced.md)

