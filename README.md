# TS-JSON

## Introduction

TS-JSON stands for "Terrifically Simple JSON". 
TS-JSON defines an especially simple and regular way of using JSON to represent data in Web APIs. 
The syntax of TS-JSON is exaclty the same as the syntax of JSON—no more and no less.

Regular [JSON](http://www.json.org/) describes as object as an unordered set of name/value pairs. 
It does not otherwise say what an object is, what a name is or what a value is (beyond defining basic datatypes).
This gives designers a lot of flexibility, which
of course is part of the reason that JSON is popular, but it also allows them to make complex JSON, and they often do. 

TS-JSON places 3 restrictions on the use of JSON for Web APIs:

1. TS-JSON objects must be meaningful objects of the data model of the API. TS-JSON does not allow 
   objects that are purely syntactic, or are technical features of the representation rather than the state of the resource.
2. The name of a name/value pair must correspond to a property or relationship of the state of the object.
3. The value of a name/value pair must be the value of the property referenced by the name.

TS-JSON also defines two JSON properties, `_id` and `_isA`. `_id` is used to express the identity of an object. `_isA` is used to
express the type(s) of an object. `_id` is fairly fundamental to TD-JSON, because it is the construct that allows you
to specify the correspondence between a JSON object and the underlying entity it represents. `_isA` is not so fundamental and needn't
really be built into TS-JSON—I added it because it makes it a bit easier to talk about the concepts and it is generally useful.

## Tutorial

The 3 JSON restrictions and 2 JSON properties above comprise the complete specification of TS-JSON. There is no more. What follows
is descriptive.

The value of adhering to the 3 restrictions is that it creates a direct 1-1 mapping between the data of the API and the JSON.
If you know the data model, you already know what the JSON will look like a vice versa. Other formats also have this property,
but none of them do it as simply and directly as TS-JSON. Other formats build new concepts or structures on top of JSON, 
while TS-JSON just uses the concepts that are already there in the simplest way we could imagine. 
This is why there is no media type for TS-JSON.

The following is a valid TS-JSON document:
```JSON
{
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org#"
}
```
Because of TS-JSON's 3 restrictions, we know that this single JSON object with 3 name/value pairs represents 3 property values of a single entity in the data model.

The following JSON document is not valid TS-JSON. It violates all 3 of the rules above. That does not mean it's a bad
representation design, just that it follows a different set of rules from Terrifically Simple JSON.
```
 1. {
 2.  "properties": 
 3.     {
 4.      "name": "Martin",
 5.      "bornOn": "1957-01-05"
 6.     },
 7.  "links": [
 8.     {
 9.      "rel": "bornIn",
10.     "href": "http://www.scotland.org#"
11.     } 
12.  ]
13. }
```

Lines 3 and 8 violate rule 1—they introduce JSON objects that have no correspondence in the data model.  
Lines 2, 7, 9 and 10 violate rule 2—they introduce JSON names that are not properties of an entity in the data model.  
Line 9 violates rule 3—`bornIn` is a property name, not a value.

This example has 3 JSON objects, an array, and 6 name/value pairs. Nevertheless, it appears to only encode the same 3 properties
of the same single entity from the data model that the TS-JSON example encodes.

Humans can easily see that this example is following different rules, because we understand the data model, and we 
can see that the mapping of the data model to the JSON is different from TS-JSON's. Unfortuntely, it is not easy to
write a computer program to verify this.

## _isA

`_isA` is used to define the "type" or "kind" of an object.
It can have more than one value, so its value can be a JSON array, but can also be a simple value.
```JSON
{
 "_isA": "Person",
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org#"
}
```

JSON allows nested objects, so in TS-JSON it is valid to write this:
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

## _id

The `_id` property is used to define the identity of the data model entity corresponding to a JSON object. Its value is always a URL (possibly relative).
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
This example makes two independent statements:
* http://martin-nally.name# was born in http://www.scotland.org#
* http://www.scotland.org# is a country

The following two examples are equivalent—both are valid TS-JSON (verify that they follow the rules)
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": "http://www.scotland.org#"
}
```
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": {"_id": "http://www.scotland.org#"}
}
```
API designers using TS-JSON can choose which they prefer, or allow both.

In TS-JSON it is possible for the `_id` property of a JSON object to be missing, but the object still must correspond to an 
entity in the API data model. A JSON object with no `_id` should be read as a noun clause that references an entity. For example

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

## datatypes

One of the challenges of JSON is that it only supports 3 datatypes: number, string, and boolean (maybe the null value should be 
considered a member of a 4th datatype). 
The two most common
datatypes in Web API programming that are not covered by JSON are date and URI. Unless you are willing to invent extensions or
conventions on top of JSON, the best you can do is to encode them as strings. The examples above show how the `_id` property
can be used to encode URLs more explicitly. This idea can be extended to other datatypes. The idea is that this JSON:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": {"_id": "http://www.scotland.org#"}
}
```
is really a shorthand for this:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": {
    "_id": "http://www.scotland.org#",
    "_idType": "URI"
    }
}
```
This pattern can be used for dates, like this:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornOn": {
    "_id": "1957-01-05",
    "_idType": "ISO8601"
    }
}
```
In other words, the rule that said that the value of `_id` is a URL is just the default. 

The JSON built-in types can be
handled the same way. In other words, the following are equivalent:
```JSON
{
 "_id": "http://martin-nally.name#",
 "heightInCM": 178
}
```
```JSON
{
 "_id": "http://martin-nally.name#",
 "heightInCM": {
    "_id": "178",
    "_idType": "number"
    }
}
```
I'm not suggesting that anyone would actually encode a number in JSON in this way—I just wanted to illustrate
that the concept can apply to all datatypes—but if you need a way
to encode dates that distinguishes them from strings, this would be a way to do it in TS-JSON. If the number example
seems unintuitive, consider that
when you write `178` in JSON, you are really writing a reference to a number. The language we use to write these references was
[developed over 3 milleania](https://en.wikipedia.org/wiki/History_of_the_Hindu-Arabic_numeral_system).
When you write `true` or `false` in JSON, you are using a different reference language—one that is specific to booleans. 
Strings can be viewed the same way. Any datatype can be viewed as being a set of pre-existing entities with a language for writing
references to them [and perhaps some rules for operators that can be performed on them, but JSON does not have operators].
JSON has built-in support for the reference languages for numbers, booleans and strings.
Lacking a built-in reference language for other datatypes like URLs and dates, 
we're forced to state explicitly which language we're using. [RDF/JSON](https://www.w3.org/TR/rdf-json/) uses a version of this idea.