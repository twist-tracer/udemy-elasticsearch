# Связь через связующее поле

~~~
PUT /department/_mapping
{
  "properties": {
    "join_field": { 
      "type": "join",
      "relations": {
        "department": "employee"
      }
    }
  }
}

PUT /department/_doc/1
{
  "name": "Development",
  "join_field": "department"
}

PUT /department/_doc/2?routing=1
{
  "name": "Bo Andersen",
  "age": 28,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
~~~

# Поиск дочерних документов по родителям

## По id

~~~
GET department/_search
{
  "query": {
    "parent_id": {
      "type": "employee",
      "id": 1
    }
  }
}
~~~

## По фильтру

~~~
GET department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      "query": {
        "match": {
          "name": "Development"
        }
      }
    }
  }
}
~~~

# Поиск родителей по дочерним документам

~~~
GET department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "term": {
          "age": 28
        }
      }
    }
  }
}
~~~

# Много уровневые связи

~~~
PUT /company
{
  "mappings": {
    "properties": {
      "join_field": { 
        "type": "join",
        "relations": {
          "company": ["department", "supplier"],
          "department": "employee"
        }
      }
    }
  }
}

PUT /company/_doc/4
{
  "name": "Another Company, Inc.",
  "join_field": "company"
}

PUT /company/_doc/5?routing=4
{
  "name": "Marketing",
  "join_field": {
    "name": "department",
    "parent": 4
  }
}

PUT /company/_doc/6?routing=4
{
  "name": "John Doe",
  "join_field": {
    "name": "employee",
    "parent": 5
  }
}

GET company/_search
{
  "query": {
    "has_child": {
      "type": "department",
      "query": {
        "has_child": {
          "type": "employee",
          "query": {
            "term": {
              "name.keyword": "John Doe"
            }
          }
        }
      }
    }
  }
}
~~~

routing = id родителя. Он нужен для того чтобы elastic положит дочерние документы положить на нужную шарду (к родителю)

**!!! Cвязи через join_field работают плохо (долго) на больших обьемах дочерних или родительских элементах и имеют лимиты.
Если дочерних элементов может быть много лучше использовать nested связи !!!**
