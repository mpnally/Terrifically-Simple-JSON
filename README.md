# Terrifically Simple JSON

## Introduction

The goal of
Terrifically Simple JSON is to define the simplest and most regular way possible of using JSON to represent data in Web APIs. 
The syntax of Terrifically Simple JSON is exaclty the same as the syntax of regular JSON—Terrifically Simple JSON concerns 
itself only with how you use JSON.

[Regular JSON](http://www.json.org/) describes an object as "an unordered set of name/value pairs"<a href="#footnote1" id="ref1"><sup>1</sup></a>. 
It does not otherwise say what an object is, what a name is or what a value is (beyond defining basic datatypes).
This gives designers a lot of flexibility, which often leads to complex JSON. 

Terrifically Simple JSON requires that you to use JSON simply and directly. In particular, it makes these 3 constraints:

1. Every Terrifically Simple JSON object must correspond to an entity in the data model of the API. Terrifically Simple JSON does not allow 
   JSON objects that are purely syntactic or are technical features of the representation.
2. The `name` of a name/value pair must refer to a property or relationship in the state of the corresponding entity.
3. The `value` of a name/value pair must be the value of the property referenced by the name.

Together, these constraints basically say,
"use JSON directly—don't use it to construct your own data-representation format on top of JSON". Since Terrifically Simple JSON is just about using JSON directly,
it does not have its own media type.

Terrifically Simple JSON defines a special property, `_id`, that is used to express Rule #1 above. `_id` allows you to declare which data model entity a particular JSON object corresponds to.

Following the 3 constraints above, and using `_id` to do so explicitly, are the only requirements of Terrifically Simple JSON. It really is that simple.

It turns out that there is a slight generalization of the `_id` concept that allows all datatypes to be expressed in JSON in a very regular fashion. 
This generalization is expressed with the optional `_idRef`, `_idRefNotation` properties. Terrifically Simple JSON
does not require you to use them, but they are there if you want an explcit way to handle arbitrary datatypes in JSON.

## Tutorial

The following is a Terrifically Simple JSON document:
```JSON
{
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org#"
}
```
Because of Terrifically Simple JSON's 3 restrictions, we know that this single JSON object with 3 name/value pairs represents a single entity 
in the API data model with 3 property values.

The following JSON document encodes the same information in a different style. It is not Terrifically Simple JSON—it violates all 3 of the rules above. That does not mean it's a bad
design, just that it follows a different set of rules from Terrifically Simple JSON. It is its own media type.
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
Line 9 violates rule 3—`bornIn` is a property (or relationship) name in the data model, not a value.

### _id

The use of the `_id` property is fundamental to Terrifically Simple JSON. Rule #1 of Terrifically Simple JSON is that every JSON object
corresponds to an entity in the API data model. The `_id` property provides a direct way of saying which entity—its value is the URL of the entity. 
Here is an example of its use:

```JSON
{
 "_id": "http://martin-nally.name#",
 "firstName": "Martin"
}
```
This example says simply that the entity whose id is http://martin-nally.name# has the first name Martin.
Standard JSON tells us that the first name is Martin. The `_id` property tells us which API entity we are talking about.

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
This example encodes two separate pieces of information:
* http://martin-nally.name# was born in http://www.scotland.org#
* The Gaelic name for http://www.scotland.org# is Alba

### When `_id` is missing

When the `_id` property of a Terrifically Simple JSON object is missing, as in the previous example, the object still must correspond to an 
entity in the API data model. A JSON object with no `_id` should be read as a noun clause that references an entity. This example
```JSON
{
 "isA": "Person",
 "name": "Martin",
 "eyeColor": 
    {
     "isA": "RGBColor",
     "red": 0,
     "green": 0,
     "blue": 155
    }
}
```

should be read as meaning,
"The eyeColor of that Person whose name is Martin is that RGBColor whose red value is zero, green value is 0 and blue value is 155"

### <a name="explicit-urls"></a>An explicit way of encoding URLs

The following two examples are both Terrifically Simple JSON (you can verify that they follow the rules)
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
Interpreted strictly,
the second says I was born in a country while the first says I was born in a string. Common sense tells us that
the intent of the first is the same as that of the second. Computers are not very good at common sense, but humans are if they have some context.
Whether you use the first or second form (or both) in your API might depend on who the audience is and whether you want to require 
them to have context. This choice involves a trade-off between ease-of-use and precision. 
The first form is easier to understand and code to if you have the required context—the second is more precise and correct 
and therefore can be reliably interpreted without additional context.
To understand what it is like to lack the required context, imagine both examples with all names and values in Chinese characters
(unless you can actually read Chinese, in which case use Cryllic or Arabic).

### <a name="datatypes"></a>Datatypes

Terrifically Simple JSON offers an optional generalization of the `_id` concept for handling any datatype that is not built into JSON. 
One of the challenges of JSON is that it only supports 3 useful datatypes: number, string, and boolean (the null value should probably be 
considered the unique member of a 4th datatype). 
The two most common
datatypes in Web API programming that are not covered by JSON are date and URI. Unless you are willing to invent extensions or
conventions on top of JSON, the best you can do is to encode them as strings. The [example above](#explicit-urls) shows how the `_id` property
can be used in Terrifically Simple JSON to encode URLs more precisely, at some cost to simplicity and ease-of-programming. 
This idea can be extended to other datatypes. The following two Terrifically Simple JSON examples are equivalent.
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": {"_id": "http://www.scotland.org#"}
}
```
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornIn": {
    "_idRef": "http://www.scotland.org#",
    "_idRefNotation": "URI"
    }
}
```
The `_idRefNotation` value tells you what the notation is of the reference in the `_idRef` field.
The `_id` property is a convenient way to express `_idRef` values for references whose _idRefNotation is URI. 
Other values for `_idRefNotation` can be used, so this pattern can be used for other datatypes, e.g. dates, like this:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornOn": {
    "_idRef": "1957-01-05",
    "_idRefNotation": "ISO8601"
    }
}
```

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
    "_idRef": "178",
    "_idRefNotation": "JSONNumber"
    }
}
```
I'm not suggesting that anyone would actually encode a number in JSON in this way—the point is to show
that the concept works for all datatypes. If you need a way
to encode dates that distinguishes them from strings, this is the way you should do it in Terrifically Simple JSON.
If you don't need to distinguish dates from their stringified equivalents—you rely on the client having enough context—
you can represent dates as simple strings.

If the number example
seems unintuitive, consider that
when you write `178` in JSON, you are really writing a reference to an existing number—the number 178 has been around a lot longer than your JSON. 
The notation we are using to write this reference was
[developed over 3 millenia](https://en.wikipedia.org/wiki/History_of_the_Hindu-Arabic_numeral_system).
When you write `true` or `false` in JSON, you are using a different reference notation—one that is specific to booleans. 
Strings can be viewed the same way, although it may take a little more thought to see why this is true (think of different notions for designating the same string). 
Any datatype can be thought of as consisting of a pre-defined set of entities with a notation for writing
references to them [and perhaps some operators on them, but operators are out of the scope of JSON].
JSON has built-in notations for referencing numbers, booleans and strings.
For other datatypes, we must state explicitly which notation we're using, or rely on out-of-band knowledge.

## Not a media type? Really?

I may be on shaky ground here. I do not think the 3 constraints indicate a new media type—they emphasize using JSON directly instead of using it to build your own media type—
but it may be that the use of `_id` (and optionally `_idRef` and `_idRefNotation`) constitutes the definition of a new media type.

## Full Disclosure

If you know RDF, you will recognize that Terrifically Simple JSON is a proprietary format for a loose interpretation of the RDF data model.
Adding an `_id` property to a JSON object converts JSON's name/value pairs to RDF triples, with the value of the `_id` property providing the subject.
JSON objects without an `_id` property are interpreted as blank nodes.
Compared to the real RDF data model, Terrifically Simple JSON's interpretation drops the requirements that predicates and classes be entities 
identified with URLs and gives up the ability to express multi-valued properties,
considering it more important to use JSON's array feature in a natural way for list-valued properties.
Having done a couple of projects using a strict interpretation of the RDF model, 
I've seen that those features cause significant friction in practical API programming, and that usually neither API providers nor consumers are
willing to accept that friction. Those features may be important for RDF's ambitions as an internet-scale information model but they are not
important for most API programming needs. Those features also contribute significantly to the complexity of standard JSON representations
of RDF, especially JSON-LD, whose complexity is likely to be fatal in my opinion, but also RDF/JSON. Hence the need for Terrifically Simple JSON.

## _

<a name="footnote1"><sup>1</sup></a> Although this is what the JSON site says, it is not what is generally implemented for JSON.
What is implemented is "an unordered set of names, each with an associated value". The definition as written
implies that there can be more than one name/value pair with the same name but different values. Most implementations accept input
with more than one pair of the same name, but throw away all but one of them. The [full IETF spec for JSON](http://www.ietf.org/rfc/rfc4627.txt?number=4627)
says that "The names within an object SHOULD be unique". <a href="#ref1">↩</a>