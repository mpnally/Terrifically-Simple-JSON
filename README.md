# TS-JSON

## Introduction

TS-JSON stands for "Terrifically Simple JSON". The goal of
TS-JSON is to define the simplest and most regular way possible of using JSON to represent data in Web APIs. 
The syntax of TS-JSON is exaclty the same as the syntax of JSON—TS-JSON concerns 
itself only with how you use JSON to represent your data.

[Regular JSON](http://www.json.org/) describes an object as "an unordered set of name/value pairs". 
It does not otherwise say what an object is, what a name is or what a value is (beyond defining basic datatypes).
This gives designers a lot of flexibility, which
of course is part of the reason that JSON is popular, but it also allows them to make complex JSON, and they often do. 

TS-JSON places 3 restrictions on the use of JSON for Web APIs:

1. Every TS-JSON object must correspond to an entity in the data model of the API. TS-JSON does not allow 
   objects that are purely syntactic or are technical features of the representation.
2. The `name` of a name/value pair must correspond to a property or relationship in the state of the entity.
3. The `value` of a name/value pair must be the value of the property referenced by the name.

We think these restrictions are simply enforcing the most straightforward and most natural way of using JSON. They basically say,
"use JSON directly—don't use it to construct your own data-representation format". Since TS-JSON is just about using JSON directly,
it does not have its own media type. 
Other formats build new concepts or structures on top of JSON, requiring a mapping that needs to be learned between the
JSON and the data of the API. 
Our ideal is that the mapping from JSON to the data is so direct, obvious and intuitive that it appears not to exist.

TS-JSON also defines three JSON properties, `_id<sup>1</sup>`, `_idType` and `_isA`. 
* `_id` is used to declare the URL of the data model entity corresponding to a JSON object. 
* `_idType` is used for datatypes that are not built in to JSON. 
* `_isA` is used to declare the type(s) of an entity.  

`_id` is fundamental to TD-JSON. `_idType` is an optional-use feature. 
`_isA` needn't really be part of TS-JSON, but it makes it a bit easier to talk about the concepts and it is generally useful.

The 3 JSON restrictions and 3 JSON properties above comprise the complete specification of TS-JSON. There is no more. 

## Tutorial


The following is a valid TS-JSON document:
```JSON
{
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org#"
}
```
Because of TS-JSON's 3 restrictions, we know that this single JSON object with 3 name/value pairs represents a single entity 
in the API data model with 3 property values.

The following JSON document encodes the same information in a different style. It is not valid TS-JSON. It violates all 3 of the rules above. That does not mean it's a bad
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
10.      "href": "http://www.scotland.org#"
11.     } 
12.  ]
13. }
```

Lines 3 and 8 violate rule 1—they introduce JSON objects that have no correspondence in the data model.  
Lines 2, 7, 9 and 10 violate rule 2—they introduce JSON names that are not properties of an entity in the data model.  
Line 9 violates rule 3—`bornIn` is a property name in the data model, not a value.

Humans can easily see that this example is following different rules, because we understand the data model, and we 
can see that the mapping of the data model to the JSON is different from TS-JSON's. Unfortunately, it is not easy to
write a computer program to verify this.

## _id

The value of the `_id` property is the identifier of the data model entity corresponding to the JSON object. 
By default, its value is always a URI (possibly relative).
Here is an example of its use:

```JSON
{
 "_id": "http://martin-nally.name#",
 "firstName": "Martin"
}
```
This example says simply that the entity whose id is http://martin-nally.name# has the first name Martin.

The `_id` property can be used in nested objects too, like this:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": 
    {
    "_id": "http://www.scotland.org#",
    "gaelicName": "Alba"
    }
}
```
This example makes two independent statements:
* http://martin-nally.name# was born in http://www.scotland.org#
* The Gaelic name for http://www.scotland.org# is Alba

## <a name="explicit-urls"></a>An explicit way of encoding URLs

The following two examples are both valid TS-JSON (you can verify that they follow the rules)
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
Viewed pedantically,
the second says I was born in a country while the first says I was born in a string. Common sense tells us that
the intent of the first is the same as the second. Computers are not very good at common sense, but humans are if they have some context.
Whether you use the first or second form for your API might depend on who the audience is and whether you want to require 
them to have context.
The first form is easier to code to for programmers who do have the required context, so there is a trade-off.

## _isA

`_isA` is used to define the "type" or "kind" of an object.
It can have more than one value, so its value can be a JSON array, but can also be a simple value.
```JSON
{
 "_isA": "Person",
 "name": "Martin"
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

## When `_id` is missing

When the `_id` property of a TS-JSON object is missing, as in the previous example, the object must still must correspond to an 
entity in the API data model. A JSON object with no `_id` should be read as a noun clause that references an entity. The previous example
should be read as meaning
"The eyeColor of that Person whose name is Martin is that RGBColor whose red value is zero, green value is 0 and blue value is 155"

## <a name="datatypes"></a>Datatypes

One of the challenges of JSON is that it only supports 3 datatypes: number, string, and boolean (possibly the null value should be 
considered the unique member of a 4th datatype). 
The two most common
datatypes in Web API programming that are not covered by JSON are date and URI. Unless you are willing to invent extensions or
conventions on top of JSON, the best you can do is to encode them as strings. The [example above](#explicit-urls) shows how the `_id` property
can be used in TS-JSON to encode URLs more explicitly, at some cost to simplicity and ease-of-programming. 
This idea can be extended to other datatypes. The idea is that this JSON:
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
The `_idType` value tells you what the notation is of the reference in the `_id` field.
This longer pattern can be used for dates, like this:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornOn": {
    "_id": "1957-01-05",
    "_idType": "ISO8601"
    }
}
```
The default value for `_idType` is `URL`, but it can be overridden. 

In principle, the JSON built-in types can be
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
I'm not suggesting that anyone would actually encode a number in JSON in this way—the point is to show
that the concept works for all datatypes. If you need a way
to encode dates that distinguishes them from strings, this is the right way to do it in TS-JSON. 

If the number example
seems unintuitive, consider that
when you write `178` in JSON, you are really writing a reference to a number. The notation we use to write this reference was
[developed over 3 millenia](https://en.wikipedia.org/wiki/History_of_the_Hindu-Arabic_numeral_system).
When you write `true` or `false` in JSON, you are using a different reference notation—one that is specific to booleans. 
Strings can be viewed the same way. Any datatype can be thought of as consisting of a set of pre-existing entities with a notation for writing
references to them [and perhaps some rules for operators that can be performed on them, but operators are out of the scope of JSON].
JSON has built-in support for the reference notations for numbers, booleans and strings.
Lacking a built-in reference notation for other datatypes, 
we must state explicitly which notation we're using, or rely on out-of-band knowledge. 
[RDF/JSON](https://www.w3.org/TR/rdf-json/) uses a version of this idea—that is where I learned it.

## Full Disclosure

If you know RDF, you will recognize that TS-JSON is a proprietary format for a loose interpretation of the RDF data model.
Compared to the real RDF data model, TS-JSON's interpretation drops the requirements that predicates and classes be entities 
identified with URLs. It also gives up the ability to express multi-valued properties,
considering it more important to exploit JSON's array feature naturally for list-valued properties.
Having done a couple of projects using a strict interpretation of the RDF model, 
I've seen that those particular features cause significant friction in practical API programming, and that neither API providers nor consumers are
willing to accept that friction. I believe those features may be important for RDF's ambitions as an internet-scale information model but are not
important for most API programming needs. Those features also contribute significantly to the complexity of standard JSON representations
of RDF, especially JSON-LD, whose complexity is likely to be fatal in my opinion, but also RDF/JSON. Hence my need for Terrifically Simple JSON.