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

The following JSON document is not valid VS-JSON. It violates each of the rules above. That does not mean it's a bad
representation design, just that it follows a different set of rules from "Very Simple JSON"
```JSON
{
 "properties": 
    {
     "name": "Martin",
     "birthday": "1957-01-05"
    },
 "links": [
    {
     "rel": "birthplace",
     "href": "http://www.scotland.org"
    } 
 ]
}
```

The 3rd and 7th lines violate rule 1—they introduce JSON objects that have no meaning in the conceptual model  
The 2nd and 6th lines violate rule 2—they introduce JSON names that are not properties of the entity on the data model  
Line 9 violates rule 3—`birthplace` is a property name, not a value.

VS-JSON defines a special JSON property whose name is `isA`. It is used to define the "type" or "kind" of an object.
It can have more than one value, so its value can be a JSON array, but can also be a simple value.
```JSON
{
 "isA": "Person",
 "name": "Martin",
 "birthday": "1957-01-05",
 "birthplace": "http://www.scotland.org"
}
```

VS-JSON allows nested objects, so it is valid to write this:
```JSON
{
 "isA": "Person",
 "name": "Martin",
 "birthday": "1957-01-05",
 "birthplace": "http://www.scotland.org",
 "eyeColor": 
    {
     "isA": "RGBColor",
     "red": 0,
     "green": 0,
     "blue": 155
    }
}
```

VS-JSON defines a special JSON property whose name is `_id`. It is used to define the identity of an object.

