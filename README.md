# VS-JSON

VS-JSON stands for "Very Simple JSON". 
VS-JSON defines an especially simple and regular way of using JSON to represent data in Web APIs. 
The syntax of VS-JSON is exaclty the same as the syntax of JSON—no more and no less.

Regular [JSON](http://www.json.org/) describes as object as an unordered set of name/value pairs. 
It does not otherwise say what an object is, what a name is or what a value is (beyond basic datatypes).
This gives designers a lot of flexibility, which
of course is part of the reason that JSON is popular. 

VS-JSON adds 3 restrictions to JSON:

1. VS-JSON objects must be meaningful objects of the data model of the API. VS-JSON does not allow 
   objects that are purely syntactic, or technical
2. The name of a name/value pair must correspond to a property or relationship of the object in the data model
   of the API
3. The value of a name/value pair must be the value of the property.

The value of adhering to these restrictions is that there is a simple 1-1 mapping between the data of he API and the JSON.
If you know the data model, you already know what the JSON will look like a vice versa. Some other formats also have this property,
but none of them do it as simply and directly as VS-JSON. Other formats build new concepts and structures on top of JSON, 
while VS-JSON just tells you how to use the concepts thst are already there.

VS-JSON defines two JSON properties, `_id` and `_isA`. `_id` is used to express the identity of an object. `_isA` is used to
express the type(s) of an object.

The 3 JSON restrictions and 2 JSON properties above comprise the complete specification of VS-JSON. There is no more. What follows
is descriptive.

The following is a valid VS-JSON document:
```JSON
{
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org"
}
```

The following JSON document is not valid VS-JSON. It violates all 3 of the rules above. That does not mean it's a bad
representation design, just that it follows a different set of rules from "Very Simple JSON"
```JSON
{
 "properties": 
    {
     "name": "Martin",
     "bornOn": "1957-01-05"
    },
 "links": [
    {
     "rel": "bornIn",
     "href": "http://www.scotland.org"
    } 
 ]
}
```

The 3rd and 7th lines violate rule 1—they introduce JSON objects that have no correspondence in the data model  
The 2nd and 6th lines violate rule 2—they introduce JSON names that are not properties of the entity in the data model  
Line 9 violates rule 3—`bornIn` is a property name, not a value.

`_isA` is used to define the "type" or "kind" of an object.
It can have more than one value, so its value can be a JSON array, but can also be a simple value.
```JSON
{
 "_isA": "Person",
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org"
}
```

VS-JSON allows nested objects, so it is valid to write this:
```JSON
{
 "_isA": "Person",
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org",
 "eyeColor": 
    {
     "_isA": "RGBColor",
     "red": 0,
     "green": 0,
     "blue": 155
    }
}
```

The `_id` property is used to define the identity of an object.
Here is an example of its use:

```JSON
{
 "_id": "http://martin-nally.name#",
 "_isA": "Person"
}
```

The `_id` property can be used in nested objects too, like this:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": 
    {
    "_id": "http://www.scotland.org#",
    "_isA": "Country"
    }
}
```
This example makes two statements:
* http://martin-nally.name# was born in http://www.scotland.org#
* http://www.scotland.org# is a country

The following two examples are equivalent—both are valid VS-JSON
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": 
    {
    "_id": "http://www.scotland.org#",
    }
}
```
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": "http://www.scotland.org#"
}
```
API designers using VS-JSON can choose which they prefer, or allow both.

When an object is missing an `_id` property, it should be read as a noun clause. For example

```JSON
{
 "_isA": "Person",
 "name": "Martin",
 "eyeColor": 
    {
     "_isA": "RGBColor",
     "red": 0,
     "green": 0,
     "blue": 155
    }
}
```
should be read as meaning
"The eyeColor of that Person whose name is Martin is that RGBColor whose red value is zero, green value is 0 and blue value is 155"