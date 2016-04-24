# VS-JSON

VS-JSON defines an especially simple and regular way of using JSON to represent data in Web APIs. 
The syntax of VS-JSON is exaclty the same as the syntax of JSON—no more and no less.
VS-JSON stands for "Very Simple JSON". I'm not sure yet whether it makes sense to have a separate media type for VS-JSON—it might.

Regular [JSON](http://www.json.org/) describes as object as an unordered set of name/value pairs. It does not otherwise say what an object is.
This gives designers a lot of flexibility, which
of course is part of the reason that JSON is popular. VS-JSON adds 3 restrictions to JSON:

1. VS-JSON objects must be meaningful objects of the conceptual data model of the API. VS-JSON does not allow 
   objects that are purely syntactic, or technical
1. The name of a name/value pair must correspond to a true property or relationship in the conceptual data model
   of the API
1. The value of a name/value pair must be the value of the property.

These restrictions are somewhat subjective—I will try to explain below what they mean.

The following is a valid VS-JSON document:
```JSON
{
 "name": "Martin",
 "birthday": "1957-01-05",
 "birthplace": "http://www.scotland.org"
}
```

The following JSON document is not valid VS-JSON. It violates each of the rules above. Thst does not necessarily mean it's a bad
representstion design, just tht it doesn't follow the rules of "Very Simple JDON"
```JSON
{
 "properties": 
    {
     "name": "Martin",
     "birthday": "1957-01-05"
    }
 "links": [
    {
     "rel": "birthplace",
     "href": "http://www.scotland.org"
    } 
 ]
}
```

VS-JSON defines a special JSON property whose name is `_id`. It is used to define the identity of an object.

