---
layout: post
title:  Upgrade database backend of “Required-Reading Web Application” 
date:   2024-03-09 08:54:00 -0600
category: project
show: Data Migration  
description: Automated filtering data of accelerator operators' training records to be migrated to a Postgresql database.
skills: [Python, Posgresql]
---

## Objectives 

- Required Reading application is Fermilab's Operations Department records of mandatory training for accelerator operators. 
- In an effort to upgrade the application, training records needs to be transfered to a Postgres database hosted by the Control Department from an outdated database server. 

## What did I do? 

>> Note: Due to export control reasons, information that I show here does not reflect entirely what I had done. I'm doing my best to show the technical aspect that I put in the project.

### **Data cleaning** 

- I used lambda functions to filter out data columns that needed to be kept. 

- Examples: 

This is a short, simpler example of how I filtered out data that needed to be present in a different format: 

```python
check_active=lambda x: 'active' if int(x)==1 else ('archived'if int(x)==0 else 'deprecated')
```

There are a few requirements from the development team that certain data values needs to be changed: 

```python 
check_postion= lambda pos: pos if pos not in ["Xfr'd","Gone"] else 'redacted'
```

In this line of code, `position` will stay the same if it's not either `Xfr'd` or `Gone`.

Or, some data even needed to combine together to form another data field: 

```python
unique_usr=lambda usr,id: usr if usr not in [ops[id]['username'] for id in range(id)] else usr+str(id)
```



### **Data collection**

I used `DictReader` to collect necessary data, and fill in approriate data if it's missing. Below is a part of the code: 

```python
ops=defaultdict(dict)
ops.update({id:{'username':row['Name Last'] if not row['Username'] else row['Username'],
                'position': check_postion(row['Status'])
                }
            })
```


### **Postgres**

Below is an example table schema I created for cross joining operators and all the documents:

```postgres
CREATE TABLE IF NOT EXISTS "example"(
    "id" INTEGER,
    "user_id" INTEGER,
    "doc_id" TEXT,
    "status" TEXT DEFAULT 'non-read' CHECK ("status" IN ('non-read','active','expired')),
    PRIMARY KEY("id"),
    FOREIGN KEY("user_id") REFERENCES "users"("id"),
    FOREIGN KEY("doc_id") REFERENCES "documents"("id")
);
```

I then set up triggers: 

```postgres
CREATE TRIGGER "update record"
AFTER UPDATE ON "records"
WHEN (NEW."status"="active")
BEGIN
    INSERT INTO "histories"("user_id","doc_id","record_id","self approval")
    VALUES (NEW."user_id",NEW."doc_id",NEW."id",datetime('now','localtime'));

END;
```

Finally, I imported values into tables.  

## Summary 

- Using python for data preprocessing gave me flexible options for data structures with its varieties of built-in modules. 
- I was introduced to Relational Database Management Systems attending the live CS50’s Introduction to Databases course, where I learned about SQLite. Applying this knowledge to PostgreSQL further enhanced my database skills and practical experience. 

















