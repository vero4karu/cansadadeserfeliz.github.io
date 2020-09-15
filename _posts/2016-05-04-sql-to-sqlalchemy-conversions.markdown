---
layout: post
title: "SQL to SQLAlchemy conversions"
date: 2016-05-04 21:15:39 -0500
comments: true
featured_image: '/images/demo/demo-square.jpg'
disqus_identifier: 5a5d8828-8c29-48d3-9092-83419aebcc2b
excerpt_separator: <!-- more -->
categories: 
- Python
tags: 
- SQL
- SQLAlchemy
---

Some examples on how to convert raw SQL to SQLAlchemy query:

<!-- more -->

**SELECT**

```sql
SELECT COUNT(*)
```

```python
> from sqlalchemy import text, func
> db.session.query(func.count()).all()
```

**Add labels**

```sql
SELECT COUNT(*) as requiests
```

```python
> rows = db.session.query(func.count().label('requests')).all()
> row = rows[0]
> row.requests
12
```

**SUM function**

```sql
SELECT SUM(status) as rides
```
```python
> rows = db.session.query(func.sum(Book.status).label('books')).all()
> row = rows[0]
> row.books
12
```

**COUNT and DISTINCT**

```sql
SELECT COUNT(DISTINCT(author_id)) as authors
```

```python
from sqlalchemy import distinct
> rows = db.session.query(
    func.count(distinct(Book.author_id)).label('authors')
).all()
```

**Query a date range (BETWEEN)**

```python
> now = datetime.datetime.utcnow()
> db.session.query(Book).filter(
    Book.created_at.between(now - datetime.timedelta(hours=5), now)
).all()
```

**Conditional SUM**

```sql
SELECT SUM(((status IN (4, 7))::int)) as books ...
```
```python
> from sqlalchemy.sql.expression import case
> db.session.query(
    func.sum(case([(Book.status.in_((4, 7)), 1)], else_=0)).label('books'),
).filter(
    # ...
).all()
```

*IN clause*

```python
> Book.query.filter(
    Book.status.in_((BOOK_CONFIRMED, BOOK_FINISHED)),
).count()
```

For `NOT IN` just add `~` symbol:

```python
> Book.query.filter(
    ~Book.status.in_((BOOK_CONFIRMED, BOOK_FINISHED)),
).count()
```

Labels and AS clause

In this example we'll make a fingle table out of `books` and `authors` tables, labeling `books.title` and `authors.last_name` columns as `name`:

```sql
SELECT books.title AS name 
FROM books UNION ALL SELECT authors.last_name AS name 
FROM authors
```

And that's how out SQL converts to SQLAlchemy:

```python
db.session.query(
    label('name', Book.title)
).union_all(db.session.query(
    label('name', Author.last_name)
)).all()
```

Now we can also filter our result table by `ǹame` column:

```python
db.session.query(
    label('name', Book.title)
).union_all(db.session.query(
    label('name', Author.last_name)
)).filter_by(name='some string')
```

**LIKE and ILIKE**

```python
Book.query.filter(Book.title.like('%tale%')).all()
Book.query.filter(Book.title.ilike('%tale%')).all()
```

**GROUP BY**

Group books my month name:

```sql
SELECT to_char(created_at, 'Mon'), count(created_at)
FROM books 
GROUP BY to_char(created_at, 'Mon') 
ORDER BY to_char(created_at, 'Mon') 
```

```
| Month    | Count |
|----------|-------|
| January  | 12    |
| March    | 29    |
| November | 8     |
```


```python
books = db.session.query(
    func.to_char(Book.created_at, 'MM'),
    func.count(Book.created_at)
).group_by(
    func.to_char(Book.created_at, 'MM')
).order_by(func.to_char(Book.created_at, 'MM')).all()
```

