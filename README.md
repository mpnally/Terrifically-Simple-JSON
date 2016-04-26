# Terrifically Simple JSON

## Introduction

The goal of
Terrifically Simple JSON is to define the simplest and most regular way possible of using JSON to represent data in Web APIs<a href="#footnote1" id="ref1"><sup>1</sup></a>. 
The syntax of Terrifically Simple JSON is exactly the same as the syntax of regular JSON—Terrifically Simple JSON concerns 
itself only with how you use JSON.

[Regular JSON](http://www.json.org/) describes an object as "an unordered set of name/value pairs"<a href="#footnote2" id="ref2"><sup>2</sup></a>. 
It does not otherwise say what an object is, what a name is or what a value is (beyond defining basic datatypes).
This gives designers a lot of flexibility, which often leads to complex JSON. 

Terrifically Simple JSON reduces complexity by adding these 3 constraints:

1. Every Terrifically Simple JSON object must correspond to an entity in the data model of the API.
2. The `name` of a name/value pair must refer to a property or relationship in the state of the corresponding entity.
3. The `value` of a name/value pair must be the value of the entity property referenced by the name.

Together, these constraints say,
"Terrifically Simple JSON uses JSON directly—it doesn't define its own data-representation format on top of JSON".

Terrifically Simple JSON defines a special property, `_id`, that allows you to declare which data model entity a particular JSON object corresponds to.

The only requirements of Terrifically Simple JSON are that you follow the 3 constraints above, and use the `_id` property to do so explicitly. It really is that simple.

Although the 3 constraints seem to imply that there is no new media type beyond JSON itself, the use of `_id` 
implies a new media type. We have not yet registered a media type—we propose `application/vnd.terrifically-simple+json`.

There is a slight generalization of the `_id` concept that allows arbitrary datatypes to be expressed in JSON in a consistent fashion. 
This generalization is expressed with the optional `_ref`, `_refNotation` properties. Terrifically Simple JSON
does not require you to use them, but they are there if you want an explcit way to handle arbitrary datatypes in Terrifically Simple JSON.

## Tutorial

The following is a Terrifically Simple JSON document:
```JSON
{
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org#"
}
```
Because of Terrifically Simple JSON's 3 constraints, we know that this single JSON object with 3 name/value pairs represents a single entity 
in the API data model with 3 property values.

The following JSON document encodes the same information in a different style. It is not Terrifically Simple JSON—it violates all 3 of the constraints above. That does not mean it's a bad
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

Lines 3 and 8 violate constraint 1—they introduce JSON objects that have no correspondence in the data model.  
Lines 2, 7, 9 and 10 violate constraint 2—they introduce JSON names that are not properties of an entity in the data model.  
Line 9 violates constraint 3—`bornIn` is a property (or relationship) name in the data model, not a value.

### _id

Constraint #1 of Terrifically Simple JSON is that every JSON object
corresponds to an entity in the API data model. The `_id` property provides a direct way of specifying which entity. 
Its value is always a URL.
Here is an example of its use:

```JSON
{
 "_id": "http://martin-nally.name#",
 "firstName": "Martin"
}
```
This example says simply that the entity whose id is http://martin-nally.name# has the first name Martin.
Standard JSON tells us that the first name is Martin and the `_id` property tells us which API entity we are talking about.

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
"**the Person whose name is Martin** has eyes colored **the RGBColor whose red value is zero, green value is 0 and blue value is 155**".

### <a name="explicit-urls"></a>An explicit way of encoding URLs

The following two examples are both Terrifically Simple JSON (you can verify that they follow the constraints).
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
and therefore can be reliably interpreted without context<a href="#footnote3" id="ref3"><sup>3</sup></a>.

### The end

That is it for the required part of Terrifically Simple JSON. The next section describes an optional extension.

### <a name="datatypes"></a>Datatypes

Terrifically Simple JSON offers an optional generalization of the `_id` concept for handling any datatype that is not built into JSON. 
One of the challenges of JSON is that it only supports 3 major datatypes: number, string, and boolean (the null value is the unique member of a 4th datatype). 
The two most common
datatypes in Web API programming that are not covered by JSON are date/time and URI. Unless you are willing to invent extensions or
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
    "_ref": "http://www.scotland.org#",
    "_refNotation": "URI"
    }
}
```
The `_refNotation` value tells you what the notation is of the reference in the `_ref` field.
The `_id` property is a convenient way to express `_ref` values for references whose `_refNotation` is `URI`. 
Other values for `_refNotation` can be used for other datatypes, e.g. dates, like this:
```JSON
{
 "_id": "http://martin-nally.name#",
 "bornOn": {
    "_ref": "1957-01-05",
    "_refNotation": "http://www.w3.org/2001/XMLSchema#dateTime"
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
    "_ref": "178",
    "_refNotation": "http://www.json.org/#number"
    }
}
```
If this example
seems unintuitive, consider that
when you write `178` in JSON, you are writing a reference to an entity. The number 178 has been around a lot longer 
than your JSON—the notation you are using to write this reference was
[developed over 3 millenia](https://en.wikipedia.org/wiki/History_of_the_Hindu-Arabic_numeral_system).
When you write `true` or `false` in JSON, you are writing a reference using a different notation—one that is specific to booleans. 
Strings can be viewed the same way, although it may take a little more thought to see why this is true (think of different notions for designating the same string). 
Any datatype can be thought of as consisting of a pre-defined set of entities with a notation for writing
references to them [and perhaps some operators on them, but operators are outside of the scope of JSON].
JSON has built-in notations for referencing numbers, booleans and strings—for other datatypes, 
we must state explicitly which notation we're using, or rely on contextual knowledge.
This view of datatypes says that in Terrifically Simple JSON, values as well as objects must correspond to entities in the API model.

I'm not suggesting that anyone would actually encode a number in JSON in this way—the point is to show
that the concept works for all datatypes. If you need a way
to encode dates and other dataypes that distinguishes them from strings, this is the way you should do it in Terrifically Simple JSON.
If you don't need to distinguish dates from their stringified equivalents—that is, you rely on the client having enough context—
you can represent dates as simple strings.

## Prior Art and Acknowledgements

If you know RDF, you will recognize that Terrifically Simple JSON is a representation format for a loose interpretation of the RDF data model.
Adding an `_id` property to a JSON object converts JSON's name/value pairs to RDF triples, with the value of the `_id` property providing the subject.
JSON objects without an `_id` property are blank nodes.
Compared to the real RDF data model, Terrifically Simple JSON's interpretation drops the requirements that predicates and classes be entities 
identified with URLs and gives up the ability to express multi-valued properties,
considering it more important to use JSON's array feature in a natural way to express list-valued properties.
Having done a couple of projects using a strict interpretation of the RDF model, 
I've seen that those features cause significant friction in practical API programming. Those features also contribute significantly to the complexity of standard JSON representations
of RDF, especially JSON-LD, whose complexity is likely to be fatal in my opinion, but also RDF/JSON. Hence the need for Terrifically Simple JSON.

Terrifically Simple JSON has few concepts, and those it has are mostly stolen from elsewhere. The value is in what was taken out, not what was put in.

## _
<a name="footnote1"><sup>1</sup></a> Terrifically Simple JSON could also be used in other contexts where JSON is used. <a href="#ref1">↩</a>

<a name="footnote2"><sup>2</sup></a> Although this is what the JSON site says, it is not what is generally implemented for JSON.
What is implemented is an unordered set of names, each with an associated value. The definition as written
implies that there can be more than one name/value pair with the same name but different values. Most implementations accept input
with more than one pair of the same name, but throw away all but one of them. The [full IETF spec for JSON](http://www.ietf.org/rfc/rfc4627.txt?number=4627)
says that "The names within an object SHOULD be unique". <a href="#ref2">↩</a>

<a name="footnote3"><sup>3</sup></a> To understand what it is like to lack the required context, imagine both examples with all names and values in Chinese characters
(unless you can actually read Chinese, in which case use Cryllic or Arabic). <a href="#ref3">↩</a>
