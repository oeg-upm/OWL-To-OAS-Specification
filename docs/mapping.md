# Mapping-OWL to OpenAPI Specification (OAS)

This document describes how to generate an [OpenAPI Specification (OAS)](http://spec.openapis.org/oas/v3.0.3) from [OWL](https://www.w3.org/TR/2004/REC-owl-guide-20040210/).

**Authors:**

  * **Paola Espinoza-Arias** (Ontology Engineering Group, Universidad Politecnica de Madrid)
  * **Daniel Garijo** (Information Sciences Institute, University of Southern California)

**Version:** 1.0.0

**Release date:** 23/05/2020

**License:** [License (Apache-2.0)](https://github.com/paoespinozarias/Mapping-OWLtoOAS/blob/master/LICENSE)

**Want to contribute?:** Check out our [GitHub repository](https://github.com/oeg-upm/OWL-To-OAS-Specification)

## Introduction

This is a document defining how to translate [OWL](https://www.w3.org/TR/owl2-overview/) to [OAS](https://github.com/OAI/OpenAPI-Specification) to allow developers to determine how ontologies may be represented as an interface descriptions for REST APIs. In this document you will find the similiarities between the two specifications a well as simple examples of how an ontology code is represented in the OAS.



### <a id="definitions"></a>Definitions from OAS

The following definitions from the [OAS version 3.0.3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md) are a condensed version of the definitions of the specification. Only those definitions that will be used in the mapping section are briefly presented.

It is worth noting that the remaining definitions provided in OAS should be followed according to that specification.

#### <a id="componentsObject"></a>Components Object

Holds a set of reusable objects for different aspects of the OAS.
All objects defined within the [Components Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#componentsObject) will have no effect on the API unless they are explicitly referenced from properties outside the components object.


#### <a id="pathsObject"></a>Paths Object

Holds the relative paths to the individual endpoints and their operations.
The [path](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#pathsObject) is appended to the URL from the [Server Object](#https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#serverObject) in order to construct the full URL.  

#### <a id="pathItemObject"></a>Path Item Object

Describes the operations available on a single path (get, put, post, among others). The path itself is still exposed to the documentation viewer but they will not know which operations and parameters are available.
To check more details provided in the OAS refer to [Path Item Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#paths-object.
).

#### <a id="referenceObject"></a>Reference Object

A simple object to allow referencing other components in the specification, internally and externally. The [Reference Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#referenceObject) is defined by [JSON Reference](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) and follows the same structure, behavior and rules. For OAS, reference resolution is accomplished as defined by the JSON Reference specification and not by the JSON Schema specification.

#### <a id="schemaObject"></a>Schema Object

The [Schema Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#schemaObject) allows the definition of input and output data types. These types can be objects, but also primitives and arrays. This object has several properties taken directly from the JSON shema definition, such as title, required, maximum, among others. In addition, this object manages other properies taken from the JSON Schema definition but their definitions were adjusted to the OpenAPI Specification, such as type, allOf, oneOf, etc.

Alternatively, any time a Schema Object can be used, a [Reference Object](#referenceObject) can be used in its place. This allows referencing definitions instead of defining them inline. Other than the JSON Schema subset fields, the following fields MAY be used for further schema documentation: nullable, deprecated, readOnly, etc.

#### <a id="dataTypes"></a>Data Types

Primitive [data types](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#dataTypes) in the OAS are based on the types supported by the [JSON Schema Specification Wright Draft 00](https://tools.ietf.org/html/draft-wright-json-schema-00#section-4.2). In addition, primitives have an optional modifier property: `format`. OAS uses several known formats to define in fine detail the data type being used. However, to support documentation needs, the `format` property is an open `string`-valued property, and can have any value. Formats such as "email", "uuid", and so on, MAY be used even though undefined by this specification. Types that are not accompanied by a format property follow the type definition in the JSON Schema. Tools that do not recognize a specific format MAY default back to the type alone, as if the format is not specified.


## <a id="mapping"></a>Mapping between OWL and OAS

In this section the similiarities between OWL and OAS are provided. All mappings include examples provided in [Turtle](https://www.w3.org/TR/turtle/) and [YAML](https://yaml.org/) serializations for human-friendly readability. The code of both serializations as well as an ontology diagram are provided in [examples](https://github.com/paoespinozarias/Mapping-OWLtoOAS/tree/master/examples). Also, the full example in the YAML serialization may be opened in the [Swagger editor](http://editor.swagger.io) to be viewed as API documentation. A list with all the OWL constructs covered and those not covered by this mapping specification is provided in the [OWLtoOAS Profile](#OWLtoOAS).

The prefixes that will be used in this section are:

```prefix
@prefix : <https://w3id.org/example#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xml: <http://www.w3.org/XML/1998/namespace> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
```

#### <a id="class"></a>Class

A class will be treated as an object in OAS according to the [Schema Object](#schemaObject) definition as it is explained in this section. However, it is worth noting that a class also must be included in API paths and operations, which will be explained [later]((#paths-and-operations) ) in this document.

OWL| OAS | Comments
------ | -------- | --------
`owl:Class` | `Schema Object` | It should be defined as a [Schema Object](#schemaObject) in the [Component Object](#componentsObject) definition. The `Schema Object` must be a`type: object`. The schema name should be the `rdfs:label` value defined in the OWL Class. It is worth noting that the name should not contain blank spaces. [See example](#classexample)
**Class axioms** |
`rdfs:subClassOf` | `allOf`| It should be defined as a model composition of the common property set and class-specific properties. Thus, a subclass should be defined as a [Schema Object](#schemaObject) in the [Component Object](#componentsObject) definition including the field `allOf`. Such field must include a `type: object` and the corresponding [Reference Object](#referenceObject) to the parent class (`$ref`:'reference to the parent class'). Finally, the specific properties of the subclass should be defined in the `properties` field. [See example](#subclassexample)|

##### <a id="classexample"></a>Class definition example

This example shows the definition of the Person class. However, its properties have been omitted because the details on how to define Data and Object properties will be provided in the next section.

TTL
```ttl
:Person rdf:type owl:Class ;
rdfs:label "Person"@en.
```
YAML
```yaml
components:
  schemas:
    Person:
      type: object
```
[Back to the Class mapping](#class)

##### <a id="subclassexample"></a>Subclass definition example

This example shows the definition of the Professor and Student subclasses. However, the specific properties of each subclass have been omitted because details on how to define Data and Object properties will be provided in the next section.

TTL
```ttl
:Professor rdf:type owl:Class ;
    rdfs:subClassOf :Person ;
    owl:disjointWith :Student ;
    rdfs:label "Professor"@en .

:Student rdf:type owl:Class ;
    rdfs:subClassOf :Person ;
    owl:disjointWith :Professor ;
    rdfs:label "Student"@en .

```
YAML
```yaml
components:
  schemas:
    Person:
      type: object
    Professor:
      allOf:
      - $ref: '#/components/schemas/Person'
      - type: object
    Student:
      allOf:
      - $ref: '#/components/schemas/Person'
      - type: object
```
[Back to the Class mapping](#class)

#### <a id="dataTypes"></a>Data Types

The similarities between data types are shown below.

OWL |OAS | Comments
------ | -------- | --------
`xsd:integer`| `integer` |
 `xsd:int`| `integer` | The `format` must be defined as `int32` (signed 32 bits)
 `xsd:long`| `integer` | The `format` must be defined as `int64` (signed 64 bits)
 `xsd:float`| `number` | The `format` must be defined as `float` |
 `xsd:double`| `number` |The `format` must be defined as `double` |
 `xsd:string`|`string` |
 `xsd:date`|`string` | The `format` must be defined as `date` (As defined by `full-date` - [RFC3339](https://xml2rfc.ietf.org/public/rfc/html/rfc3339.html#anchor14))
 `xsd:dateTime`|`string` | The `format` must be defined as `date-time` (As defined by `date-time` - [RFC3339](https://xml2rfc.ietf.org/public/rfc/html/rfc3339.html#anchor14))
 `xsd:dateTimeStamp`|`string` | The `format` must be defined as `date-time` (As defined by `date-time` - [RFC3339](https://xml2rfc.ietf.org/public/rfc/html/rfc3339.html#anchor14))
 `xsd:byte`|`string` | The `format` must be defined as `byte` (base64 encoded characters)
 `xsd:anyURI`|`string` | The `format` must be defined as `uri`
  `xsd:boolean`|`boolean` |


#### <a id="properties"></a>Properties

A property will be treated as an object property in OAS according to the [Schema Object](#schemaObject) definition. In this section, details on how to define data and object properties are presented.


OWL | OAS | Comments
------ | -------- | --------
`owl:DatatypeProperty` | `properties` | It should be defined as a property of the corresponding [Schema Object](#schemaObject) instance, as aforementioned in the `owl:Class` mapping. The property name will be the `rdfs:label` value defined in the data property. It is worth noting that the name should not contain blank spaces. [See example](#datatypePropertyExample)
`owl:ObjectProperty` | `properties` | It should be defined as a property of the corresponding [Schema Object](#schemaObject) instance, as aforementioned in the `owl:Class` mapping. The property name will be the `rdfs:label` value defined in the object property. It is worth noting that the name should not contain blank spaces. [See example](#objectPropertyExample)
**Property axioms** |
`rdfs:domain` | `Schema Object`| The domain of a property corresponds to the [Schema Object](#schemaObject) that must include that property. Thus, it should be defined as a [Schema Object](#schemaObject) inside of the [Component Object](#componentsObject) definition, as explained in the [`owl:class`](#class) mapping.|
`rdfs:range` | `type`| **_a)_** when the range corresponds to an `owl:DatatypeProperty`: if the range is single-valued the `type` value should be the [data type](dataTypes) of the data property; else if the range is multi-valued the `type` value should be the [data type](dataTypes) of the data property and the possible values defined as an enumeration (`enum`). [See example](#datatypePropertyExample). **_b)_** when the range corresponds to an `owl:ObjectProperty`: if the range cardinality >=1 it should be defined as an array (`type: array`) and the `items` as a [Reference Object](#referenceObject) (`$ref:`'reference to the Range Class'); else if the range cardinality <1 it could be defined as an array (`type: array`) and `items` as a [Reference Object](#referenceObject) (`$ref:`'reference to the single Range Class') or only as an object (`type: object`) and a [Reference Object](#referenceObject) (`$ref:`'reference to the single Range Class'). This last decision will depend on the developer's decision. [See example](#objectPropertyExample)|
**Property characteristics** |
`owl:FunctionalProperty` | `maxItems`| The property with this characteristic should be defined as an array (`type: array`) with 1 as the maximum number of items (`maxItems:` 1) and **_a)_** if it corresponds to an `owl:DatatypeProperty` the type of array items must be the [data type](dataTypes) of the property; **_b)_** if it corresponds to an `owl:ObjectProperty` the type of array items must contain a [Reference Object](#referenceObject) to the range Class (`$ref:`'reference to the range Class'). [See example](#functionalPropertyExample)|


##### <a id="datatypePropertyExample"></a>DatatypeProperty example

The following example presents how to define a Data Property of the Person class. A single-value and a multi-value ranges has been included to show how they look in OAS.

TTL
```ttl
:identifier rdf:type owl:DatatypeProperty ;
  rdfs:domain :Person ;
  rdfs:range xsd:string ;
  rdfs:label "name"@en .

:gender rdf:type owl:DatatypeProperty ;
  rdfs:domain :Person ;
  rdfs:range [ rdf:type rdfs:Datatype ;
    owl:oneOf [ rdf:type rdf:List ;
      rdf:first "female" ;
      rdf:rest [ rdf:type rdf:List ;
        rdf:first "male" ;
        rdf:rest rdf:nil
        ]
      ]
    ] ;
  rdfs:label "gender"@en .
```

YAML
```yaml
components:
  schemas:
    Person:
      type: object
      properties:
        name:
          type: string
        gender:
          type: string
          enum:
            - male
            - female
          nullable: true
```

In addition, the following example shows how to define a Data Property of the Professor subclass.

TTL
```ttl
:researchField rdf:type owl:DatatypeProperty ;
    rdfs:domain :Professor ;
    rdfs:range xsd:string ;
    rdfs:label "research field"@en .
```
YAML
```yaml
components:
  schemas:
    Person:
      type: object
      properties:
        name:
          type: string
        gender:
          type: string
          enum:
            - male
            - female
    Professor:
      allOf:
        - $ref: '#/components/schemas/Person'
        - type: object
      properties:
        researchField:
          type: string
```
[Back to the Properties mapping](#properties)

##### <a id="objectPropertyExample"></a>ObjectProperty example

The following example shows how to define an Object Property of the Professor class:

TTL

```ttl
:teachesTo rdf:type owl:ObjectProperty ;
    rdfs:domain :Professor ;
    rdfs:range :Student ;
    rdfs:label "teaches to"@en .
```
YAML
```yaml
components:
  schemas:
    Person: {...} #Rest of the schema omitted
    Professor:
      allOf:
        - $ref: '#/components/schemas/Person'
        - type: object
      properties:
        researchField:
          type: string
        teachesTo:
          items:
            $ref: '#/components/schemas/Student'
          type: array
```
[Back to the Properties mapping](#properties)

##### <a id="functionalPropertyExample"></a>FunctionalProperty examples

This example shows how to define a Functional Data Property of the Person class.

TTL
```ttl
:identifier rdf:type owl:DatatypeProperty , owl:FunctionalProperty ;
  rdfs:domain :Person ;
  rdfs:range xsd:string ;
  rdfs:label "identifier"@en .
```
YAML
```yaml
components:
  schemas:
    Person:
      type: object
      properties:
        identifier:
          items:
            type: string
          type: array
          maxItems: 1
```
The next example shows how to define a Functional Object Property of the Student.

TTL
```ttl
:hasRecord rdf:type owl:ObjectProperty , owl:FunctionalProperty ;
    rdfs:domain :Student ;
    rdfs:range :StudentRecord ;
    rdfs:label "has record"@en .
```

YAML
```yaml
components:
  schemas:
    Person: {...} #Rest of the schema omitted
    Student:
      allOf:
        - $ref: '#/components/schemas/Person'
        - type: object
      properties:
        hasRecord:
          items:
            $ref: '#/components/schemas/StudentRecord'
          type: array
          maxItems: 1
```
[Back to the Properties mapping](#properties)

#### <a id="restrictions"></a>Restrictions

Depending on the restriction, it will be treated according to the details provided below.

OWL | OAS | Comments
------ | -------- | --------
`owl:onProperty` |  `properties` | The restriction should refers to the property name where it is applied.
`owl:onClass` | `Schema Object` | The restriction should refers to the schema name where it is applied.
`owl:someValuesFrom`| `type` and `nullable`| The restricted property should be defined as a not null (`nullable: false`) array (`type: array`) and depending on the restriction **_a)_**  when it is on an `owl:DatatypeProperty`the value of the items type should be the restricted data type; **_b)_** when it is on an `owl:ObjectProperty` the `items` should be a [Reference Object](#referenceObject) to the restricted class (`$ref:`'reference to the restricted Class'). [See example](#someValuesFromExample) |  
`owl:allValuesFrom` | `type` and `nullable` | The restricted property should be defined as a nullable (`nullable: true`) array (`type: array`) and depending on the restriction **_a)_** when it is on an `owl:DatatypeProperty` the value of the items type should be the restricted data type; **_b)_** when it is on an `owl:ObjectProperty` the `items` should be a [Reference Object](#referenceObject) to the restricted class (`$ref`:'reference to the restricted Class'). [See example](#allValuesFromExample)|
`owl:hasValue`| `default` | It should be defined as a `default` value of the property. Depending on the restriction  **_a)_** when it is on an `owl:DatatypeProperty` the property `type` should be the corresponding data type and the default value should be the specific literal value, **_b)_** when it is on an `owl:ObjectProperty` the property type should be a string (`type: string`) with `format: uri` and the default value should be the individual URI value. [See example](#hasValueExample)|
`owl:minCardinality` | `minItems` | It should be defined as the minimum number of the array items which contains as [Reference Object](#referenceObject) a scheme of the `owl:Thing` (`$ref:`'reference to the owl:Thing'). The value of `minItems` must be a non-negative number. |
`owl:maxCardinality` | `maxItems`| It should be defined as the maximum number of the array items which contains as [Reference Object](#referenceObject) a scheme of the `owl:Thing` (`$ref:`'reference to the owl:Thing'). The value of `maxItems` must be a non-negative number.|
`owl:cardinality` | `minItems` and `maxItems` | It should be defined as the same minimum and maximum number of the array items which contains as [Reference Object](#referenceObject) a scheme of the `owl:Thing` (`$ref:`'reference to the owl:Thing'). The value of `minItems` and `maxItems` must be a non-negative number. |
`owl:minQualifiedCardinality` | `minItems`| It should be defined as the minimum number of array items and **_a)_** if it corresponds to an `owl:DatatypeProperty` the value of the items type must be the restricted [data type](dataTypes); **_b)_** if it corresponds to an `owl:ObjectProperty` the array items must contain the [Reference Object](#referenceObject) to the restricted Class (`$ref:`'reference to the restricted Class'). The value of `minItems` must be a non-negative number. [See example](#minQualifiedCardinalityExample)|
`owl:maxQualifiedCardinality` |`maxItems`| It should be defined as the maximum number of array items and **_a)_** if it corresponds to an `owl:DatatypeProperty` the value of the items type must be the restricted [data type](dataTypes); **_b)_** if it corresponds to an `owl:ObjectProperty` the array items must contain the [Reference Object](#referenceObject) to the restricted Class (`$ref:`'reference to the restricted Class'). The value of `maxItems` must be a non-negative number. [See example](#maxQualifiedCardinalityExample)|
`owl:qualifiedCardinality`|`minItems` and `maxItems` |  It should be defined as the same minimum and maximum number of array items and **_a)_** if it corresponds to an `owl:DatatypeProperty` the value of the items type must be the restricted [data type](dataTypes); **_b)_** if it corresponds to an `owl:ObjectProperty` the array items must contain the [Reference Object](#referenceObject) to the restricted Class (`$ref:`'reference to the restricted Class'). The value of `minItems` and `maxItems` must be a non-negative number. [See example](#qualifiedCardinalityExample)|


##### <a id="someValuesFromExample"></a>someValuesFrom example

This example shows how to represent that Professor teaches some Courses.

TTL
```ttl
:Professor rdf:type owl:Class ;
  rdfs:subClassOf :Person ,
    [ rdf:type owl:Restriction ;
      owl:onProperty :teachesCourse ;
      owl:someValuesFrom :Course
    ] .
```
YAML
```yaml
components:
  schemas:
    Person: {...} #Rest of the schema omitted
    Professor:
      allOf:
        - $ref: '#/components/schemas/Person'
        - type: object
      properties:
        teachesCourse:
          items:
            $ref: '#/components/schemas/Course'
          type: array
        nullable: false
```
[Back to the Restrictions mapping](#restrictions)

##### <a id="allValuesFromExample"></a>allValuesFrom example

This example shows how to represent that a Professor teaches to only Students.

TTL
```ttl
:Professor rdf:type owl:Class ;
  rdfs:subClassOf :Person ,
    [ rdf:type owl:Restriction ;
      owl:onProperty :teachesTo ;
      owl:allValuesFrom :Student
    ] .
```
YAML
```yaml
components:
  schemas:
    Person: {...} #Rest of the schema omitted
    Professor:
      allOf:
        - $ref: '#/components/schemas/Person'
        - type: object
      properties:
        teachesTo:
          items:
            $ref: '#/components/schemas/Student'
          type: array
        nullable: true
```
[Back to the Restrictions mapping](#restrictions)

##### <a id="hasValueExample"></a>hasValue example

This example presents how to represent that an American Student has an American nationality.

TTL
```ttl
:AmericanStudent rdf:type owl:Class ;
  rdfs:subClassOf :Student ,
    [ rdf:type owl:Restriction ;
      owl:onProperty :nationality ;
      owl:hasValue "American"
    ] .
```
YAML
```yaml
components:
  schemas:
    Student: {...} #Rest of the schema omitted
    AmericanStudent:
      allOf:
        - $ref: '#/components/schemas/Student'
        - type: object
      properties:
        nationality:
          type: string
          default: "American"
```
In addition, imagine that the nationality property would have been defined as an `owl:ObjectProperty` therefore the American Student's nationality property should have been described with a default value corresponding to the individual URI as follows:

TTL
```ttl
:AmericanStudent rdf:type owl:Class ;
  rdfs:subClassOf :Student ,
    [ rdf:type owl:Restriction ;
      owl:onProperty :nationality ;
      owl:hasValue <http://example.org/resource/nationality/American>
    ] .
```
YAML
```yaml
components:
  schemas:
    Student: {...} #Rest of the schema omitted
    AmericanStudent:
      allOf:
        - $ref: '#/components/schemas/Student'
        - type: object
      properties:
        nationality:
          type: string
          format: uri
          default: http://example.org/resource/nationality/American
```

[Back to the Restrictions mapping](#restrictions)

##### <a id="minQualifiedCardinalityExample"></a>minQualifiedCardinality example

This example shows how to represent that a Student should take minimum 2 Courses.

TTL
```ttl

:Student rdf:type owl:Class ;
  rdfs:subClassOf :Person ,
    [ rdf:type owl:Restriction ;
      owl:onProperty :takesCourse ;
      owl:minQualifiedCardinality "2"^^xsd:nonNegativeInteger ;
      owl:onClass :Course
    ]  .
```

YAML
```yaml
components:
  schemas:
    Person: {...} #Rest of the schema omitted
    Student:
    allOf:
    - $ref: '#/components/schemas/Person'
    - type: object
      properties:
        takesCourse:
          type: array
          items:
            $ref: '#/components/schemas/Course'
          minItems: 2
```
[Back to the Restrictions mapping](#restrictions)

##### <a id="maxQualifiedCardinalityExample"></a>maxQualifiedCardinality example

This example shows how to represent that a Course should have enrolled maximum 20 Students.

TTL
```ttl

:Course rdf:type owl:Class ;
  rdfs:subClassOf [ rdf:type owl:Restriction ;
    [ rdf:type owl:Restriction ;
      owl:onProperty :hasStudentEnrolled ;
      owl:maxQualifiedCardinality "20"^^xsd:nonNegativeInteger ;
      owl:onClass :Student
    ] .
```

YAML
```yaml
components:
  schemas:    
    Course:
      properties:
        hasStudentEnrolled:
          items:
            $ref: '#/components/schemas/Student'
          type: array
          maxItems: 20
```
[Back to the Restrictions mapping](#restrictions)

##### <a id="qualifiedCardinalityExample"></a>qualifiedCardinality example

This example shows that a Student has exactly 1 Student Record.

TTL
```ttl

:Student rdf:type owl:Class ;
  rdfs:subClassOf :Person ,
  [ rdf:type owl:Restriction ;
    owl:onProperty :hasRecord ;
    owl:qualifiedCardinality "1"^^xsd:nonNegativeInteger ;
    owl:onClass :StudentRecord
  ]  .
```

YAML
```yaml
components:
  schemas:
    Person: {...} #Rest of the schema omitted
    Student:
      allOf:
        - $ref: '#/components/schemas/Person'
        - type: object
      properties:
        hasRecord:
          type: array
          items:
            $ref: '#/components/schemas/StudentRecord'
          minItems: 1
          maxItems: 1
```
[Back to the Restrictions mapping](#restrictions)

#### <a id="booleancombinations"></a>Boolean combinations

In this section some complex combinations allowed in OWL are presented in the OAS.

OWL |OAS| Comments
------ | -------- | --------
`owl:intersectionOf` | `allOf` | It should be defined as `allOf` which validates the value against all the subschemas. [See example](#intersectionOfExample)|
`owl:unionOf` | `anyOf` | It should be defined as `anyOf` which validates the value against any (one or more) the subschemas. [See example](#unionOfExample) |
`owl:complementOf` | `not` | It should be defined as `not` which validates the value is not valid against the specified schema [See example](#complementOfExample)|
`owl:oneOf` | `enum` | It should be defined as an `enum` and the values included in this enumeration list. [See example](#complementOfExample) |

##### <a id="intersectionOfExample"></a>IntersectionOf example

This example shows how to represent that a Professor in Artificial Intelligence must be a Professor and belongs to the Artificial Intelligence Department.

TTL
```ttl
:ProfessorInArtificalIntelligence rdf:type owl:Class ;
  rdfs:subClassOf [ owl:intersectionOf ( :Professor
    [ rdf:type owl:Restriction ;
      owl:onProperty :belongsTo ;
      owl:hasValue <https://w3id.org/example/resource/Department/ArtificialIntelligenceDepartment> ]
    ) ;
    rdf:type owl:Class
  ] .
```
YAML

```yaml
components:
  schemas:
    Professor: {...} #Rest of the schema omitted
    ProfessorInArtificalIntelligence:
      allOf:
        - $ref: '#/components/schemas/Professor'
      properties:
        belongsTo:
          type: string
          format: uri
          default: https://w3id.org/example/resource/Department/ArtificialIntelligenceDepartment
```

[Back to the Boolean combinations mapping](#booleancombinations)

##### <a id="unionOfExample"></a>UnionOf example

This example shows how to represent that a Student will be enrolled in any of the Student Programs listed. Note that in this example is also represented the `owl:someValuesFrom` restriction on the enrrolledIn property.

TTL
```ttl
:Student rdf:type owl:Class ;
  rdfs:subClassOf :Person ,
    [ rdf:type owl:Restriction ;
      owl:onProperty :enrolledIn ;
      owl:someValuesFrom [ rdf:type owl:Class ;
        owl:unionOf ( :BachelorProgram
          :MasterProgram
          :PhDProgram
        )
      ]
    ] .
```
YAML
```yaml
components:
  schemas:
    Person: {...} #Rest of the schema omitted
    Student:
      allOf:
      - $ref: '#/components/schemas/Person'
      - type: object
      properties:
        enrolledIn:
          type: array
          items:
            type: object
            anyOf:
            - $ref: '#/components/schemas/MasterProgram'
            - $ref: '#/components/schemas/PhDProgram'  
            - $ref: '#/components/schemas/BachelorProgram'
      required:
        - enrolledIn
```
[Back to the Boolean combinations mapping](#booleancombinations)

##### <a id="complementOfExample"></a>ComplementOf example

This example shows how a Professor in Other Department is not a Professor In Artifical Intelligence.

TTL
```ttl
:ProfessorInOtherDepartment rdf:type owl:Class ;
  rdfs:subClassOf
    [ rdf:type owl:Class ;
      owl:complementOf :ProfessorInArtificalIntelligence
  ] ;
```
YAML
```yaml
components:
  schemas:
    ProfessorInArtificalIntelligence: {...} #Rest of the schema omitted
    ProfessorInOtherDepartment:
      not:
        type: object
        $ref: '#/components/schemas/ProfessorInArtificalIntelligence'
```
[Back to the Boolean combinations mapping](#booleancombinations)

##### <a id="oneOfExample"></a>OneOf example

This example shows how to represent the gender data property as a enumeration.

TTL
```ttl
:gender rdf:type owl:DatatypeProperty ;
  rdfs:domain :Person ;
  rdfs:range [ rdf:type rdfs:Datatype ;
    owl:oneOf [ rdf:type rdf:List ;
      rdf:first "female" ;
      rdf:rest [ rdf:type rdf:List ;
        rdf:first "male" ;
        rdf:rest rdf:nil
        ]
      ]
    ] .
```
YAML
```yaml
components:
  schemas:
    Person:
      type: object
      properties:
        gender:
          type: string
          enum:
            - male
            - female
          nullable: true
```
[Back to the Boolean combinations mapping](#booleancombinations)


<!-- subclass of the intersection between classes: ClassA `rdfs:subClassOf` (ClassB `owl:intersectionOf` ClassC), e.g. Father is a Man and Parent.
- subclass of the union between classes: ClassA `rdfs:subClassOf` (ClassB `owl:unionOf` ClassC), e.g. Parent is equivalent to the union of Mother and Father.-->

#### <a id="pathsandoperations"></a>Paths and Operations

##### <a id="paths"></a>Paths

Paths in OAS are resources that the API exposes therefore its similiarity with OWL are classes represented in the following manner:

OWL | OAS | Comments
---|:---:|---
`owl:Class`  | `/{path}`  | A class should be defined as a path (`/{path}`); following the [Path Item Object](#pathItemObject) format. The `{path}` value should be defined according to the [Path Naming Strategy](resourcenaming). <!-- corresponds to the `rdfs:label` value defined in the OWL Class. Note that the label value needs to be defined in plural and as lowercase according to the naming practices in REST API. In addition if the label value is composed by several words they should be provided with a hyphen delimiter.-->

**Path example**

The following example corresponds to the path definition of the Professor class. In this case we use the label value (`rdfs:label`) to name the path. This value has been defined as plural and lowercase.

TTL
```ttl
:Professor rdf:type owl:Class ;
rdfs:label "Professor"@en.
```
YAML
```YAML
paths:
  /professors:
```
The next example shows the path definition of the StudyProgram class. In this case we use the label value (`rdfs:label`) to name the path. This value has been defined as plural and lowercase. We also use a hyphen delimiter because the label value is composed by several words.

TTL
```ttl
:StudyProgram rdf:type owl:Class ;
rdfs:label "Study Program"@en.
```
YAML
```YAML
paths:
  /study-programs:
```
The last part regarding to Paths is the [Path Templating](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#path-templating) which allows using template expressions in curly braces `{}` to mark a section of a URL path as replaceable using path parameters.

##### <a id="operations"></a>Operations

Operations in OAS are the HTTP methods used to manipulate these paths. Such operations are defined in [Path Item Objects](#pathItemObject) as [Operation Objects](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#operationObject). It is worth mentioning that a single path can support multiple operations but only one instance of them, for example it is allowed to define only one GET an only one POST for the same path.

Each operation should deal with a response or request that contains the returned or requested resources from executing the operation according to the details described below.

OWL | OAS | Comments
---|:---:|---
`owl:Class`  | `schema`  | If the operation is a GET, the class should be defined as part of the [Response Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#response-object) and as a [Reference Object](#referenceObject) of the `schema`. In addition, if the operation is a POST or PUT it must be included as part of the [Request Body Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#request-body-object) and as a [Reference Object](#referenceObject) of the `schema`.

Next, some basic examples about `get` and `put` operations are provided. For simplicity the response of the `200` HTTP status code is show together with the content defined as `application/json` media type. In OAS further details about all supported [HTTP codes](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#http-status-codes) and [Media Types](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#media-types) are provided.

**Operation examples**

This example shows the `get` operation to retrieved all instances of a class. Note that only the Response Object is included; however, other fields (e.g. `parameters`) not presented in this example may be added such as described in the [Operation Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#operation-object) definition.  

```YAML
/classlabel:
  get:
    responses:
      '200'
      description: A message describing the result of the operation.
      content:
        application/json:
          schema:
            items:
              $ref: ‘#/components/schemas/ClassLabel’
            type: array
```
The next example shows the `get` operation to retrieve an instance of a class by id. Note that in this case the response corresponds to a single instance of the class instead of an array of items of the class as was presented in the previous example.

```YAML
/classlabel/{id}:
  get:
    parameters:
     - description: Filter by resource id
       in: path
       name: id
       required: true
       schema:
         type: string
    responses:
      '200'
      description: A message describing the result of the operation.
      content:
        application/json:      
          schema:
            $ref: ‘#/components/schemas/ClassLabel’
```
In addition, the `put` operation to create a new instance of a class is presented.

```YAML
/classlabel:
  put:
    requestBody:
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ClassLabel'
    responses:
      '200':
        description: A message describing the result of the operation.
```
Finally, the following examples show the aforementioned operations for the Professor class.

```YAML
/professors:
  get:
    responses:
      '200':
        description: The list of Professor has been retrieved successfully
        content:
          application/json:
            schema:
              items:
                $ref: ‘#/components/schemas/Professor’
              type: array
  put:
    description: Create a new instance of Professor
    requestBody:
      description: A new Professor to be created
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Professor'
    responses:
      200:
        description: Professor created successfully

/professors/{id}:
  get:
    parameters:
     - description: Filter by resource id
       in: path
       name: id
       required: true
       schema:
         type: string
    responses:
      '200':
        description: The Professor has been retrieved successfully
        content:
          application/json:
            schema:
              $ref: ‘#/components/schemas/Professor’  
```

#### <a id="ontologyMetadata"></a>Ontology metadata

In this section the mapping between the most common annotation properties describing OWL ontologies and the OAS specification is presented.

 OWL | OAS | Comments
 ------ | -------- | --------
`dcterms:title` |`title`| The annotation value should used in the `title` field of [Info Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#infoObject).   
`rdfs:label`, `skos:label` |  `Schema Object`'s _name_  | The annotation value should be applied in the [Schema Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#schemaObject) as the name of the corresponding class or property. Note that  _name_ is not an OAS keyword for the `Schema Object`, but it is provided as a way to make sense to that similiarity. Please, refer to the [Schema Objects naming strategy](schemanaming) to see further details on how to do it. In addition, the label value may be used in the [Operation Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#operation-object) as the value of the `tags` field. Tags can be used for logical grouping of operations by resources or any other qualifier.|
`rdfs:comment`, `dcterms:description`, `prov:definition`, `skos:definition` |  `description` | These annotations should be applied in the [Schema Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#schemaObject) as a string description of the corresponding class or property. |

Note that due to the aforementioned annotation properties are usually included in several languages (e.g. `xml:lang="en"`)  it will be necessary to specify what language will be used in the OAS. This decision will depend on the developer.

#### <a id="resourcenaming"></a>Path naming strategy

In REST APIs a clear strategy on how to define resource URIs is defined. Therefore, in the OAS this strategy should be adopted in order to be consistent with the REST API paradigm. Summarizing, the main rules on how resource URIs should be structured are:
- Use a slash separator to indicate a hierarchical relationship.
- Use hyphens instead of underscores to delimit words.
- Use lowercase letters in URI paths.
- Use plural for naming collections.

Regarding this mapping, when the label annotation (`rdfs:label`, `skos:label`) of the ontology class is available the aforementioned rules should be adopted when translating the label value to the corresponding Path Item name.

However, in those cases where label annotations are not included in the ontology it will be necessary to adopt other naming strategy for Paths Items. The alternative strategy may be taking the term name from the ontology term path, e.g. http://w3id.org/example/def/#term_name. In this solution it is important to consider that the resource naming strategy in ontologies is not made in a standardized way and the only the rule is to keep it consistent, whichever is chosen. As a result, two cases may happen:

1. _Meaningful URIs_, where ontology classes and properties have natural language names. For example, following a UpperCamelCase format.
2. _Opaque URIs_, where ontology classes and properties have obfuscated names. For exmaple, using specific codes.

In the first case, the REST API naming strategy may be followed in the same manner as the label translation to the corresponding Path Item. In the second case, this strategy may not be useful because a human being looking at the schema or path can not figure out what is on the other end. Thus, developers should name the Path Item is such a manner that conveys an understandable REST API’s resource to its potential client developers instead of the opaque design.

#### <a id="schemanaming"></a>Schema Objects naming strategy

The Schema Objects naming  may include objects and properties and, as the case of Path Items, when the label annotation (`rdfs:label`, `skos:label`) of the ontology class/property is available the label value must be used to name them. To this end, blank spaces must be removed and depending on the developer's decision the names may follow, for example, a UpperCamelCase structure to name objects and a lowerCamelCase to name properties. Remember that these names must be in singular.

However, in those cases where label annotations are not included in the ontology it will be necessary to adopt other naming strategy for Schema Objects. The alternative strategy may be taking the term name from the ontology term path, e.g. http://w3id.org/example/def/#term_name. This strategy might involve two cases that may be solved according as follows:
1. _Meaningful URIs_, in this case the naming strategy may be followed in the same manner as the label value translation to the corresponding Schema Object.
2. _Opaque URIs_, in this case developers should name the Path Item is such a manner that conveys an understandable REST API’s resource to its potential client developers instead of the opaque design.

#### <a id="OWLtoOAS"></a>OWLtoOAS Profile

The subset of the constructs that can be used in a conforming ontology is:

- Existential quantification to a class expression (ObjectSomeValuesFrom) or a data range (DataSomeValuesFrom)
- Universal quantification to a class expression (ObjectAllValuesFrom) or a data range (DataAllValuesFrom)
- Existential quantification to an individual (ObjectHasValue) or a literal (DataHasValue)
- Enumerations involving a single individual  (ObjectOneOf) or a single literal (DataOneOf)
- Intersection of classes (ObjectIntersectionOf)
- Intersection of data ranges (DataIntersectionOf)
- Union of classes (ObjectUnionOf)
- Class negation (ObjectComplementOf)
- Data range negation (DataComplementOf)
- Class inclusion (SubClassOf)
- Domain restrictions (ObjectPropertyDomain and DataPropertyDomain) and range restrictions (ObjectPropertyRange and DataPropertyRange)
- Functional data properties (FunctionalDataProperty) and object properties (FunctionalObjectProperty)
- Cardinality restrictions (ObjectMaxCardinality, ObjectMinCardinality, ObjectExactCardinality, DataMaxCardinality, DataMinCardinality, and DataExactCardinality)

In addition, the following constructs are not supported in the OWLtoOAS specification:

- Self-restriction (ObjectHasSelf)
- Class equivalence (EquivalentClasses)
- Class disjointness (DisjointClasses)
- Disjoint properties (DisjointObjectProperties and DisjointDataProperties)
- Object property inclusion (SubObjectPropertyOf) with or without property chains, and data property inclusion (SubDataPropertyOf)
- Property equivalence (EquivalentObjectProperties and EquivalentDataProperties
- Transitive object properties (TransitiveObjectProperty)
- Reflexive object properties (ReflexiveObjectProperty)
- Irreflexive object properties (IrreflexiveObjectProperty)
- Inverse object properties (InverseObjectProperties)
- Inverse-functional object properties (InverseFunctionalObjectProperty)
- Symmetric object properties (SymmetricObjectProperty)
- Asymmetric object properties (AsymmetricObjectProperty)
- Assertions (SameIndividual, DifferentIndividuals, ClassAssertion, ObjectPropertyAssertion, DataPropertyAssertion, NegativeObjectPropertyAssertion, and NegativeDataPropertyAssertion)
- Keys (HasKey)


#### <a id="limitations"></a>Limitations

It is worth noting that complex boolean combinations are not supported by the mapping. Some of these cases are:

- subclass of the intersection between a class and the negation of other class: ClassA `rdfs:subClassOf` (ClassB `owl:intersectionOf` (`owl:complementOf` ClassC)), e.g. Childless Person is a Person who is not a Parent.
- Complex range restrictions on a property such as the union of two intersections: (ClassA `owl:intersectionOf` ClassB) `owl:unionOf` (ClassC `owl:intersectionOf` ClassD)
