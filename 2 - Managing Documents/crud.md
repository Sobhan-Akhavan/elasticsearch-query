# Elasticsearch CRUD Operations Guide

## Table of Contents

1. [Retrieving Documents by ID](#retrieving-documents-by-id)
2. [Indexing Documents](#indexing-documents)
   - [Indexing with Auto-Generated ID](#indexing-with-auto-generated-id)
   - [Indexing with Custom ID](#indexing-with-custom-id)
3. [Updating Documents](#updating-documents)
   - [Simple Update](#simple-update)
   - [Scripted Updates](#scripted-updates)
     - [Reducing a Field Value](#reducing-a-field-value)
     - [Using Parameters in Scripts](#using-parameters-in-scripts)
     - [Conditional Updates](#conditional-updates)
       - [Conditional Field Update](#conditional-field-update)
       - [Conditional No-Operation (noop)](#conditional-no-operation-noop)
       - [Conditional Document Deletion](#conditional-document-deletion)
   - [Upserts](#upserts)
4. [Deleting Documents](#deleting-documents)

## Retrieving Documents by ID

```EQL
GET /products/_doc/100
```

## Indexing Documents

### Indexing with Auto-Generated ID

```EQL
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}
```

### Indexing with Custom ID

```EQL
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 49,
  "in_stock": 4
}
```

Note: If the document exists with the provided ID, it will be replaced.

## Updating Documents

### Simple Update

```EQL
POST /products/_update/100
{
  "doc": {
    "in_stock": 2,
    "tags": ["electronics"]
  }
}
```

### Scripted Updates

#### Reducing a Field Value

```EQL
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```

#### Using Parameters in Scripts

```EQL
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}
```

### Conditional Updates

#### Conditional Field Update

```EQL
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock > 0) {
        ctx._source.in_stock--;
      }
    """
  }
}
```

#### Conditional No-Operation (noop)

```EQL
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock == 0) {
        ctx.op = 'noop';
      }
      
      ctx._source.in_stock--;
    """
  }
}
```

#### Conditional Document Deletion

```EQL
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock < 0) {
        ctx.op = 'delete';
      }
      
      ctx._source.in_stock--;
    """
  }
}
```

### Upserts

```EQL
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Blender",
    "price": 399,
    "in_stock": 5
  }
}
```

## Deleting Documents

```EQL
DELETE /products/_doc/101
```
