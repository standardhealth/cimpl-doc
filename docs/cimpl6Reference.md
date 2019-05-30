# CIMPL 6.0 Reference Documentation

>**Note:** This documentation is a draft.

_This is a comprehensive guide to CIMPL 6.0 syntax.  If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md). If you're looking for a more in-depth introduction, try the [Tutorial](cimpl6Tutorial_detail.md)._

***

**Table of Contents**

[TOC]

***

# General Overview

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles and implementation guides (IG). Because it is a _language_, written in text statements, CIMPL encourages distributed, team-based development using conventional source-code control tools such as Github. CIMPL provides tooling that enables you to define a model once, and publish that model to multiple versions of FHIR.

### CIMPL Versioning
CIMPL follows the [Semantic Versioning](https://semver.org) convention (MAJOR.MINOR.PATCH).

* MAJOR: A major release has significant new functionality and potentially, grammar changes or other non-backward-compatible changes.
* MINOR: Contains new or modified features, while maintaining backwards compatability within the major version.
* PATCH: Contains minor updates and bug fixes, while maintaining backwards compatability within the major version.

CIMPL is currently on major version 6.

For a full changelog, see the [Release Notes](https://github.com/standardhealth/shr-cli/releases).

<!-- TODO: Find MAJOR release differences -->

### Namespaces
CIMPL organizes information into namespaces. A namespace conventionally denoted by the authoring organization and the broad category of elements the namespace defines, delineated by a period. Namespaces must begin with a lowercase character. For example:
```
Namespace: shr.oncology
```
>**Note:** The organization name is recommended but not required (e.g., `Namespace: oncology` is also allowed).

### File Types
Model information in CIMPL is stored in one of the following files types:
* Data Element files (.txt): contain definitions of model elements, which map to FHIR profiles and extensions.
* Value Set files (.txt): contain definitions of value sets defined in the namespace.
* Mapping files (.txt): contain information on how the CIMPL data element relate to FHIR resources, profiles, and elements.

Each namespace will have one or more data element file, and zero or more value set and mapping files.

### Naming Conventions
#### File names
File names must begin with lower case and typically include the namespace:

* Data Element files: `namespace.txt` or `namespace_something.txt`
* Value Set files: `namespace_vs.txt` or `namespace_something_vs.txt`
* Mapping files: `namespace_map.txt` or `namespace_something_map.txt`

Any periods in the `namespace` should be replaced by underscores. For example, a namespace called `odh.occupation`, the recommended value set file name is `odh_occupation_vs.txt`. The `something` is useful when you want to break up the contents of a single namespace into multiple files, for example:

* `shr_core_action.txt`
* `shr_core_finding.txt`

Additional files for configuring and producing FHIR implementation guides (IGs), and their naming conventions include:
* Configuration files: `ig-igname-config.json`
* Content profile files: `ig-igname-cp.json`
* IG examples: `ignameExampleName.json`
* IG HTML pages: `anyName.html`


#### Elements, Groups, Entries
Elements, Groups and Entries are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase). For example:
```
* Element: DetectionTime
* Group: Dosage
* Entry: AdverseEvent
```
>**Note:** Letters, numbers, and hyphens are allowed. However, it is required that an element name begins with an uppercase letter.

#### Value Sets
Value Sets are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase), with an additional "VS" suffix. For example:
```
ValueSet: HomeEnvironmentRiskVS
```
>**Note:** Value Set names must begin with an uppercase letter.

#### Code Sets
Within a custom value set, the individual codes (denoted by # symbol) are conventionally written in lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case). For example:
```
ValueSet:          HomeEnvironmentRiskVS
#smoke_detectors   "No Smoke Detectors"
#radiation         "Radon or other radiation source"
#swimming_pool     "Swimming Pool"
```
### Keywords and Reserved Words
CIMPL has a number of keywords and reserved words. Keywords are followed by colons, and appear in declarations. Examples are `Namespace` and `Entry`. Reserved words (e.g., `boolean`, `or`, `from` ) are part of the grammar. Keywords are not reserved, but using keywords for other purposes (for example, as data element names), is not good practice. For a complete list, see [ANTLR4 grammar](#https://github.com/standardhealth/shr-grammar/tree/master/src/main/antlr).

### Whitespace
Repeated whitespace is generally not meaningful within CIMPL files. So this:
```
Grammar:    DataElement 6.0
Namespace:  myExampleNamespace
```
is equivalent to this:
```
Grammar:        DataElement 6.0

  Namespace:    myExampleNamespace
```
### Order of Model Elements
CIMPL does not require elements to appear in any particular order, i.e. an element defined at the bottom of the file can be used as field on an element defined at the top. However, header information must be at the top of the file, before any definitions. 

### Fully Qualified Names
CIMPL does not support duplicate class names within a single namespace. However, the same name may occur in different namespaces. In the case of naming collisions due to an included namespace in `Uses`, you have to refer to the class by its fully qualified name (FQN). The FQN is a combination of the element's namespace and its declared name (through concatention delimited by a period). For example, the element `Observation` in the namespace `shr.finding` has the following FQN:
```
shr.finding.Observation
```
FQNs are only required to be used in case of naming collisions.

### Comments
CIMPL follows [JavaScript syntax](https://www.w3schools.com/js/js_comments.asp) for code comments:
```
// Use a double-slash for comments on a single line

/*
Use slash-asterisk and asterisk-slash for larger block comments.
These comments can take up multiple lines.
*/
```
### Primitives
Primitives are distinguished by starting with lower case letter. CIMPL defines the following primitive data types. With the exception of `concept` (a simplified representation of code, Coding, and CodeableConcept), these align with FHIR:

* [concept](#concept-codes)
* [boolean](https://www.hl7.org/fhir/datatypes.html#boolean)
* [integer](https://www.hl7.org/fhir/datatypes.html#integer)
* [string](https://www.hl7.org/fhir/datatypes.html#string)
* [decimal](https://www.hl7.org/fhir/datatypes.html#decimal)
* [uri](https://www.hl7.org/fhir/datatypes.html#uri)
* [base64Binary](https://www.hl7.org/fhir/datatypes.html#base64Binary)
* [instant](https://www.hl7.org/fhir/datatypes.html#instant)
* [date](https://www.hl7.org/fhir/datatypes.html#date)
* [dateTime](https://www.hl7.org/fhir/datatypes.html#dateTime)
* [time](https://www.hl7.org/fhir/datatypes.html#time)
* [oid](https://www.hl7.org/fhir/datatypes.html#oid)
* [id](https://www.hl7.org/fhir/datatypes.html#id)
* [markdown](https://www.hl7.org/fhir/datatypes.html#markdown)
* [unsignedInt](https://www.hl7.org/fhir/datatypes.html#unsignedInt)
* [positiveInt](https://www.hl7.org/fhir/datatypes.html#positiveInt)
* [xhtml](https://www.hl7.org/fhir/datatypes.html#xhtml)

CIMPL also does not have an explicit "Reference" type (see [References](#references).)

### Concept Codes
CIMPL has a single type, `concept` to represent coded terms from controlled vocabularies. A concept combines three elements: code system, code, and optional display test. The grammar for specifying concepts is: 

> `SYSTEM#code "Display text"`

For example:
```
SCT#363346000 "Malignant neoplastic disease (disorder)"
ICD10CM#C004  "Malignant neoplasm of lower lip, inner aspect"
```
The SYSTEM is an alias for a canonical URI that represents a controlled vocabulary. Aliases must be declared in the file header using the [CodeSystem keyword](#codesystem-keyword). You cannot use the canonical URI directly in the concept grammar.

Unlike FHIR, CIMPL does not differentiate between code, Coding, and CodeableConcept. See [Mapping Concept Codes](#mapping-concept-codes) for information on CIMPL maps `concept` to these FHIR types.

>**Note:** Future versions of CIMPL will expand `concept` to include the code system version.

***

### Header Keywords
Keyword that can appear in Headers include, `Grammar`, `Namespace`, `Description`, `Uses`, `Path`, `CodeSystem`, and `Target`. The Grammar keyword must be first. The 

#### Grammar Keyword
The `Grammar` keyword is used in a header to define how the file is to be parsed. The grammar declaration must be the first line in each CIMPL model files. Each file type has different header requirements, described the corresponding sections.

| Keyword | File Type | Example |
|----------|---------|---------|
| `Grammar` | Data Element | `Grammar: DataElement 6.0` |
| `Grammar` | Value Set | `Grammar: ValueSet 6.0` |
| `Grammar` | Mapping | `Grammar: Map 6.0` |

The version details the MAJOR and MINOR version of the grammar the file was written for. The grammar version is not the same as the CIMPL release. The grammar is backwards compatible within the MAJOR version number, i.e. the declaration `DataElement 6.0` is correct for any version 6.x.x tooling. For more information, see [Versioning](#versioning).

#### Namespace Keyword
The `Namespace` keyword defines the association of the file with a namespace, and is required in `DataElement`, `ValueSet` and `Map` files.

| Keyword | Example |
|----------|---------|
| `Namespace` | `Namespace: shr.environment` |
| `Namespace` | `Namespace: odh` |

The namespace can be any number of lowercase period delimited words.

Best practice is to follow the naming convention pattern of `organization`.`domain` (followed by subdomains if necessary).

The tooling will not allow for duplicate namespaces.

>**Note:** While it is best practice to use only lower case letters, the strict requirements only require the word to begin with a lowercase letter. The rest of the namespace allows for mixed casing and hyphens.

#### Description Keyword
The `Description` keyword provides the user the ability to define the purpose of the file, within the confines of a project.

| Keyword | Example |
|----------|---------|
| `Description` | `Description: "The SHR Environment domain contains definitions related to surroundings experienced by the person of record."` |

The namespace description appears in the Data Element file, and should be defined once in each namespace. It is optional, but recommended.

Best practice is to write human readable text using the [ASCII](https://en.wikipedia.org/wiki/ASCII) standard.

There is no strict requirement for unique descriptions, and as such you will not run into description collisions.

>**Note:** While the CIMPL tooling will allow for any pattern of [Unicode](http://unicode.org/standard/WhatIsUnicode.html) characters within enclosed double quotation marks (`"`), certain exporters such as the FHIR exporter will object to non-[ASCII](https://en.wikipedia.org/wiki/ASCII) text.)

#### Uses Keyword
The `Uses` keyword appears in the Data Element file and provides a comma-separated list of the namespaces used within the current namespace. This keyword allows you to use data elements and value sets defined in other namespaces in the current namespace. The list order has no effect.

| Keyword | Example |
|----------|---------|
| `Uses` | `Uses: shr.core, shr.base, shr.allergy, shr.observation, shr.medication` |

VERIFY THIS: In CIMPL version 6, it is *required* that you import `shr.core` for the tooling to run.

#### Path Keyword
The `Path` keyword allows for abbreviations of long urls.

| Keyword | Example |
|----------|---------|
| `Path` | `Path: FHIR = http://hl7.org/fhir/ValueSet` |

This functionality is completely optional, but it is provided to make authoring easier.

The abbreviation by convention should be be an all UPPER CASE word. _(Although, the tooling does allow for numbers and hyphens after the first letter)._

#### CodeSystem Keyword
The `CodeSystem` keyword provides shorthand for the defining URL of code system. To define and use codes, it is *required* to first define them in the file header.

| Keyword | Example |
|----------|---------|
| `CodeSystem` | `CodeSystem: LNC = http://loinc.org` |

Each code system you use must appear on a separate line, e.g.

```
CodeSystem: LNC = http://loinc.org
CodeSystem: SCT = http://snomed.info/sct
```

The abbreviation by convention should be be an UPPER CASE word. _(Although the tooling does allow for numbers and hyphens after the first letter)_.

# Data Element File
The Data Element file contains definitions of the information model. 

## Building Blocks
CIMPL provides four types of building blocks:

* `Element` is the lowest-level building block, representing a name-value pair. An Element is present in the FHIR output as a property or extension, but never as a top-level profile.
* `Group` is a building block comprised of other building blocks, specifically, other Groups, Elements, and Entries. A Group is present in the FHIR output as a backbone element or complex extension, but never as a top-level profile.
* `Entry` is a building block representing a stand-alone piece of information, analogous to a FHIR resource or profile. 
* `Abstract` is a special type of Entry that cannot be instantiated, and will not be present in the target mapping.

All classes defined in CIMPL are based on one of these four building blocks. All classes in CIMPL reusable (defined once and potentially used in multiple definitions).

| Building Block | Has... | Analogous FHIR Type |
|----------|---------|---------|---------|---------|
| `Element` | Value | Property or simple extension |
| `Group` |  Properties | Backbone element or complex extension |
| `Entry`  | Properties | Resource or profile |
| `Abstract` | Properties | none |

## Properties and Values
An Element has exactly one Value field, and no Property fields. A Group, Entry, or Abstract element has no Value field, and multiple Property fields. A Property cannot have choices, but a Value can.

These differences are illustrated in this example:
```
Element:           ExpectedPerformanceTime
Description:       "When an action should be performed."
Value:             dateTime or TimePeriod or Timing
```
```
Group:             Landmark
Description:       "A point on the body that helps determine a body location."
Property:          Location 0..1
Property:          Direction 0..1
Property:          Distance 0..1
```

Summary:

| Type | Appears in | How Many Times? | Has Cardinality? | Has Choices? |
|----------|---------|---------|---------|---------|
| `Value` | `Element` | Exactly Once | No | Yes (optionally) |
| `Property` | `Group`, `Entry`, or `Abstract` | Zero or more | Yes | No |

## Data Element File Header

Here is an example of a Data Element file header. 
```
Grammar:           DataElement 6.0
Namespace:         shr.lab
Description:       "Profiles of laboratory tests and panels."
Uses:              shr.core
CodeSystem:        LNC = http://loinc.org
CodeSystem:        UCUM = http://unitsofmeasure.org
```
The first line must contain the `Grammar` keyword. For a further description of the header keywords, see [Header Keywords](#header-keywords)

## Class Definitions
Following the header, the data element file is contains any number of class definitions. The order of definitions does not matter.  In most cases, you can reference classes in imported namespaces (through the `Uses` keyword) as if they were locally defined. However, if a class in an imported namespace collides with a local class name, you must refer to that element by its [fully qualified name](#fully-qualified-names).

### Building Block Type and Class Name
The first line of all class definitions is a keyword representing the type of building block, followed by a descriptive name you choose. Class names must be unique within a given namespace.

| Building Block | Example |
|----------|---------|
| `Element` | `Element: ZipCode` |
| `Group` |  `Group: ReferenceRange` |
| `Entry`  | `Entry: FinancialSituation` |
| `Abstract` |`Abstract: DomainResource` |

>**Tip:** Since all classes in CIMPL are reusable outside of their original context, class names should be self-explanatory. For example, a property named `Text` is too vague, whereas `DisplayText` and `InstructionText` are more self-explanatory.

### Concept Keyword
The `Concept` keyword establishes the meaning of the class in terms of a code (or synonymous codes) from controlled vocabularies. Assigning a concept allows the class to be identified without relying on the class name. This keyword is optional.

| Keyword | Example |
|----------|---------|
| `Concept` | `Concept: MTH#C3858779 "Security classification"` |
| `Concept` | `Concept:  MTH#C0006826 "Malignant Neoplasm", ICD10CM#C80 "Malignant neoplasm without specification of site"` |

### Parent Keyword
The Parent keyword controls inheritance. A class that declares a parent gets all properties and constraints from the parent, similar to class inheritance in object-oriented programming. Additionally, mappings are inherited alongside properties. This keyword is optional.

| Keyword | Example |
|----------|---------|
| `Parent` | `Parent: Observation` |
| `Parent` | `Parent: Foo` |

Only one parent class can be specified. Multiple inheritance is not supported.

#### Inheritance Rules
There are restrictions on inheritance that can be summarized as "like inherits from like": 

| Building Block| Can Only Inherit From |
|----------|---------|
| `Element` | `Element` |
| `Group` | `Group` |
| `Entry` | `Abstract` or `Entry` |
| `Abstract` | `Abstract` or `Entry` |

>**Note:** Although the CIMPL 6.0 tooling does not currently enforce these inheritance rules, they will enforced in future releases. You are strongly encouraged to comply to these rules, and furthermore, refactor any definitions ported from CIMPL 5.x that do not comply.

### Description Keyword
The description is a textual description of the element. The element description is optional, but recommended.

| Keyword | Example |
|----------|---------|
 `Description` | `Description: "Measures of the ability of the..."` |

<!-- TO DO: DOES DESCRIPTION SUPPORT MARKDOWN?-->

>**Note:** While the CIMPL tooling will allow for any pattern of [Unicode](http://unicode.org/standard/WhatIsUnicode.html) characters within enclosed double quotation marks (`"`), certain exporters such as the FHIR exporter will object to non-[ASCII](https://en.wikipedia.org/wiki/ASCII) text.)

### Property Keyword (Group, Entry, and Abstract only)

Classes based on Group, Entry, and Abstract are composed of one or more properties (also called _fields_ or _attributes_). A property can be any Element, Group, or Entry, but never a primitive. Each property must have a specified cardinality range, represented as `min..max`, indicating the number of repeats of the property. The Property keyword cannot be used in Elements.

In the following example, StudyArm has three properties: Name, Type, and Comment. Name is singular and required, Type is optional and repeating, and Comment is singular and optional:
```
Group:             StudyArm
Description:       "Refers to participant(s) in a clinical trial assigned to receive specific interventions according to a protocol."
Property:          Name 1..1
Property:          Type 0..*
Property:          Comment 0..1
Property:          ResearchStudy 1..1
```

A property that refers to an Entry (such as ResearchStudy, above) is implicitly a reference (pointer) to an instance of that Entry, rather than implying the Entry is "in-lined" into the class.

>**Note:** CIMPL 5.0 Grammar use of the keyword `ref()` is now obsolete.

### Value Keyword (Elements only)

The Value represents the data type(s) an Element can accept. The Value keyword can only be used in Elements. Each Element must have exactly one Value (although the value can be a choice of multiple data types). Value choices can be [primitives](#primitives), Elements, Groups, or Entries. The Value may be inherited and constrained in the child class.

Examples:
```
Element:           DetectionTime
Description:       "The date on which the condition was first observed."
Value:             dateTime
```

```
Element:           ExpectedPerformanceTime
Description:       "When an action should be performed."
Value:             dateTime or TimePeriod or Timing
```

```
Element:           SubstanceOrCode
Description:       "A code or substance reference identifying the ingredient."
Value:             concept or Substance or Medication
```

>**Note:** CIMPL 5.0 Grammar associating a cardinality to a Value is now obsolete.

#### Special Grammar to Bind a Value Set to a Value

In general, constraints cannot be applied to a property or value at the same time the item is defined. One exception is that you are allowed to bind a value set to a `concept` at the same time an Element is defined, as follows:

```
Element:           IsPrimaryTumor
Description:       "Whether the tumor is the original or first tumor in the body, for a particular cancer."
Value:             concept from YesNoUnknownVS (required)
```
For more information in binding, see [Value Set Binding Constraint](#value-set-binding-constraint).

# CIMPL Paths
CIMPL provides grammar that allows you to refer to any element in a CIMPL model, regardless of nesting. This section outlines CIMPL Path Grammar, which is used in [constraints](#constraints) and [mapping](#map-file).

## Dot Notation
Dot notation is used to denote a path consisting of nested properties. The path lists the properties in order, separated by a dot (.) For example, if `Patient` has a `ContactPoint`, and `ContactPoint` has a `Priority`, then the path to `Priority` is `Patient.ContactPoint.Priority`. 
## Bracket Notation
Elements have Values, and Values can offer a [choice of data types](#value-keyword). When an Element appears in a path, bracket notation is used to indicate which value choice is part of the path. Bracket notation must always be used when an Element is part of a CIMPL path.


| Definition | Value Choices Inside the Element | Value Choices from Outside of Element |
|----------|---------|---------|
| `Element: Priority` <br/>  `Value: integer or concept`| Value[integer] <br/>Value[concept]| Priority[integer] <br/>Priority[concept] |

>**Note:** The requirement to use bracket notation _even when the value has only one choice_ is a change from CIMPL 5.

## Extended Example

```
Group:             Landmark
Description:       "A point on the body that helps determine a body location."
Property:          Location 0..1
Property:          Direction 0..1
Property:          Distance 0..1

Group:             Location
Description:       "Location of the landmark"
Property:          Code 0..1
Property:          Laterality 0..1
Property:          Orientation 0..1

Element:           Distance
Description:       "How far the body location of interest is from the given landmark."
Value:             Quantity

Element:           Direction
Description:       "The direction from the landmark to the body location of interest."
Value:             ClockFaceDirection or AnatomicalDirection

Element:           ClockFaceDirection
Description:       "A direction indicated by o'clock position or degrees relative to 12 o'clock."
Value:             concept or Angle

Group:             Quantity
Description:       "A quantity with units."
Property:          Number 0..1
Property:          Comparator 0..1
Property:          Units 0..1

```

This example assumes all paths begin at Landmark.

| Path to | Path |
|---------|---------|
| Direction | Landmark.Direction |
| Laterality | Landmark.Location.Laterality |
| ClockFaceDirection | Landmark.Direction[ClockFaceDirection] |
| Angle | Landmark.Direction[ClockFaceDirection][Angle] |
| Units | Landmark.Distance[Quantity].Units |

A reliable method to create a CIMPL path is to first view it as a dot path, then put brackets around any part of the path that is a Value, and finally, remove the dot before each left (open) bracket. Here's an example of that process, for the path from Landmark to Units:

1) Find the path: `Landmark` to `Distance`, `Distance` to `Quantity`, `Quantity` to `Units`.
1) Reduce to dot path: `Landmark.Distance.Quantity.Units`
1) Bracket each Value: `Landmark.Distance.[Quantity].Units`
1) Remove dot before left (open) bracket: `Landmark.Distance[Quantity].Units`

# Constraints
Constraints can be applied properties, values, or value choices. Constraints can be placed on top-level properties or nested properties by using [CIMPL paths](#cimpl-paths). Value choices are represented using [bracket notation](#bracket-notation).  Unlike class definitions, constraint statements do not begin with a keyword followed by a colon.

Constraints can be applied to locally-defined properties or properties inherited from the parent.

| To do this.. | Use this Keyword or Symbol | Applies to |
|----------|---------|---------|
| [Narrow cardinality](#cardinality-constraint) | `{min}..{max}` | Property |
| [Assign a fixed value](#fixed-value-constraint) | `=`| Property or Value[Choice] |
| [Append a fixed value to an array](#append-constraint) | `+=` | Array Property |
| [Narrow a data type](#substitute-constraint) | `substitute` | Property or Value[Choice] |
| [Bind a value set](#value-set-binding-constraint) | `from` | Property or Value[Choice] |
| [Slice an array](#includes-constraint) | `includes` | Array Property |
| [Narrow choices for a value](#only-constraint) | `only` | Value |
                                               
## Cardinality Constraint
Cardinality defines the number of repeats that can exist for a property. Cardinality is specified using FHIR syntax, {min}..{max}, where the first integer indicates the minimum repeats (0 implying optional), and the second digit expresses the upper bound on the number of repeats. To express no upper bound, a `*` is used. When constraining cardinality, you can only narrow the previously declared cardinality. The cardinality constraint only applies to properties.

| Declaration | Constraint | Subsequent Constraint |
|----------|----------|----------|
| `Property: LanguageUsed 0..*` | `LanguageUsed  0..1` | `LanguageUsed  0..0` |
| `Property: GovernmentIssuedID  0..*` | `GovernmentIssuedID 1..*` | `GovermentIssuedID 1..1` |
| `Property: Status 1..1` |<del>`Status 0..1`</del> invalid|

>**Note:** By constraining a property to 0..0, you effectively remove the property. Use 0..0 with caution, because if an instance _does_ include that property, the instance will fail validation and may be rejected.

## Fixed Value Constraint

The `=` operator fixes a property to a specific value (e.g., a specific code, boolean value, etc.). The assigned value must be consistent with the defined data type, and is always a primitive. Fixed value constraints apply to properties and value choices.

| Example | Syntax |
|----------|---------|
| Fix a code to a property | `ObservationCode = LNC#82810-3` |
| Fix a boolean value to a property | `IsPrimaryTumor = true` |
| Fix a code to a value choice | `Value[concept] = SCT#233613009 "Pneumonia"` |
| Fix a code to a value  | <del>`Value = SCT#233613009 "Pneumonia"`</del> invalid|

>**Notes:** 
1) [Bracket notation](#bracket-notation) must always be used when applying a constraint to a value choice, _even if the value has only one choice_. This is a change from CIMPL 5.
1) Fixing numerical types and strings (including URLs) is not yet supported, although this feature is planned.
1) If you only need to fix a value code for one particular mapping (one version of FHIR, for example), do so using the [`fix`](#fix) keyword in the mapping file.

### Overriding a Fixed Value
CIMPL allows fixed concept values to be overridden in child classes. Although this seems to violate the notion that constraints should only get progressive tighter in subclasses, it is necessary when narrowing the meaning of a class characterized by a concept code. For example, the class `Pneumonia`, with fixed code, `SCT#233604007`, may have a child `Fungal Pneumonia`, with fixed code `SCT#233613009`, the latter overriding the former fixed value. CIMPL will assume, without checking, that the code describing the child class has more constrained semantics than the code it replaces.

## Append Constraint
The append (`+=`) operator appends a fixed primitive value (boolean, concept, etc.) to an array property. You can use the `+=` operation repeatedly on the same array, limited only be the cardinality of the array. The append operator can only be applied to repeating properties.

For example, you would use `+=` to insert a particular Social Security Number (SSN) or Driver's License number into an array of identifiers (in constrast, [`includes`](#includes-constraint) would be used to require the array contain one or more SSNs, without specifying a particular SSN.)

| Example | Syntax |
|----------|---------|
| Insert "Behavior" into Category array |  `Category += LNC#54511-1 "Behavior"` |
| Add "Social History" to same Category array | `Category += OBSCAT#social-history "Social History"` |

## Substitute Constraint
The `substitute` keyword constrains the data type of a property. The new data type that is substituted for the original must be a subclass of the original data type. 

| Example | Syntax |
|----------|---------|
| Constrain the property `Specimen` to be a `TumorSpecimen` | `Specimen substitute TumorSpecimen` <br/> _note: TumorSpecimen must be a subclass of Specimen_ |
| Constrain a value choice of type `Quantity` to `SimpleQuantity` | `Value[Quantity] substitute SimpleQuantity` <br/> _note: SimpleQuantity is a subclass of Quantity_ |

## Value Set Binding Constraint

Binding is the process of associating a concept with a set of possible values. Binding uses the keyword `from`. The object of a binding must be an Element whose value is a concept. The value set that is being bound can be a value set defined in CIMPL or a canonical URL external to CIMPL.

CIMPL uses the same [binding strengths as FHIR](https://www.hl7.org/fhir/valueset-binding-strength.html), namely:

* `required` (strongest) indicates that the code must come from the specified value set.
* `extensible` indicates that the code must come from the specified set _if the value set contains a relevant code_; otherwise a code outside the value set may be chosen.
* `preferred` indicates that the code should ideally come from the specified value set, but codes outside the value set may also be chosen.
* `example` (weakest) indicates that the code may come from anywhere; the specified value set is for example purposes only.

The following rules apply to binding in CIMPL 6:

* If no binding strength is specified, the binding is assumed to be `required`. 
* When further constraining an existing binding, the binding strength can stay the same or be made tighter (e.g., replacing an `preferred` binding with an `extensible` or `required` binding), but never loosened.
* Constraining may leave the binding strength the same and change the value set instead. However, certain changes may violate [FHIR principles](http://hl7.org/fhir/R4/profiling.html#binding-strength) (see following Note).

>**Note:** When mapping to FHIR, FHIR will permit a required value set to be replaced by another required value set only if the codes in the new value set are a subset of the codes in the original value set. For extensible bindings, the new value set can contain codes not in the existing value set, but additional codes SHOULD NOT have the same meaning as existing codes in the base value set.

| Example | Syntax |
|----------|---------|
| Required binding of a Property (property is an Element with `concept` data type) | `ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical (required)` |
| Required binding of a Property (by default) | `ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical` |
| Extensible binding of a Property | `ReasonCode from ReasonCodeVS (extensible)` |
| Preferred binding of a Property | `BodyLocation from BodyLocationVS (preferred)` |
| Required binding of a Value Choice that is a concept | `Value[concept] from http://hl7.org/fhir/ValueSet/condition-clinical (required)` |
| Required binding of a Value Choice that is an Element with whose value is a `ClinicalStatus` | `Value[ClinicalStatus] from http://hl7.org/fhir/ValueSet/condition-clinical (required)` |

### Includes Constraint

The `includes` constraint is used to specify that a repeating property (array) should contain a certain class or classes. FHIR refers to this as [slicing](https://www.hl7.org/fhir/profiling.html#slicing). In CIMPL, it is a simple matter of saying an array `includes` certain types of elements. 

The `includes` statement has several requirements:

* The elements included in the array must conform to the data type of the array. For example, if the array is `Identifier 0..*`, the items included in the array must be Identifiers or be inherit from Identifier.
* Cardinality of each inclusion must be specified, and must fit the cardinality of the array. For example, if the array property has a finite maximum cardinality, you cannot include an element with cardinality 0..*.
* The keyword `includes` is repeated for each item the array should contain.
* The `includes` constraint does not apply to Values.

Here is an example:
```
Measurement 0..*  // the array property
...
Measurement includes TumorLength 1..1 includes TumorWidth 0..1 includes TumorDepth 0..1
```
TumorLength, TumorWidth, and TumorDepth all must inherit from Measurement, because Measurement is the array property being sliced. Optionally, the second statement can be written on separate lines (since CIMPL is whitespace insensitive):
```
Measurement
includes TumorLength 1..1
includes TumorWidth 0..1
includes TumorDepth 0..1
```

Here is another example:
```
Identifier 1..*  // the array property
...
Identifier
includes NationalProviderIdentifier 0..1
includes TaxIdentificationNumber 0..1
```
In this case, either NationalProviderIdentier or TaxIdentificationNumber must be present, even though both are individually optional, because of the 1..* cardinality on Identifier.

>**Note:** When using `includes`, special mapping syntax is required. See [Slicing](#slicing).

### Only Constraint
Values can have multiple data types. Use the `only` constraint in an Element to narrow the choice of data types. The `only` constraint can only be applied to a Value that has multiple choices. It cannot be applied to properties or individual value choices.

| Example | Syntax |
|----------|---------|
| Only allow a `string` for a Value defined as:<br/>`Value: string or boolean or dateTime` |  `Value only string` |
| Only allow a `concept` for Medication, where Value defined as:<br/>`Value: Medication or Substance`  | <del>`Value[Medication] only concept`</del> invalid (`only` should never be applied to a value choice, because intrinsically, it applies to all value choices) |

>**Note:** CIMPL currently does not support narrowing a choice to a more limited choice, for example, narrowing a choice of three types to two types. This feature is planned.


***

# Value Set File
The value set files are used to define custom value sets and codes when existing sources like [HL7 v3](https://www.hl7.org/fhir/terminologies-v3.html), [FHIR](https://www.hl7.org/fhir/terminologies-systems.html), [VSAC](https://vsac.nlm.nih.gov/), or [PHIN VADS](https://phinvads.cdc.gov/) are insufficient.

A (truncated) example value set file is shown below:

```
Grammar:      ValueSet 6.0
Namespace:    onco.core
CodeSystem:   SCT = http://snomed.info/sct
CodeSystem:   ICD10CM = http://hl7.org/fhir/sid/icd-10-cm

ValueSet:     CancerDiseaseStatusEvidenceTypeVS
Description:  "The type of evidence backing up the clinical determination of cancer progression."
SCT#363679005 "Imaging (procedure)"
SCT#252416005 "Histopathology test (procedure)"
SCT#711015009 "Assessment of symptom control (procedure)"
SCT#5880005   "Physical examination procedure (procedure)"
SCT#250724005 "Tumor marker measurement (procedure)"
SCT#386344002 "Laboratory data interpretation (procedure)"

ValueSet:      SecondaryCancerDisorderVS
Description:  "Types of secondary malignant neoplastic disease"
Includes codes descending from SCT#128462008  "Secondary malignant neoplastic disease (disorder)"
ICD10CM#C7B00  "Secondary carcinoid tumors, unspecified site"
ICD10CM#C7B01  "Secondary carcinoid tumors of distant lymph nodes"
ICD10CM#C7B02  "Secondary carcinoid tumors of liver"
ICD10CM#C7B03  "Secondary carcinoid tumors of bone"
ICD10CM#C7B04  "Secondary carcinoid tumors of peritoneum"
```

## Value Set File Header

Does not use `uses` 

For a further description of the header keywords, see [Header Keywords](#header-keywords)

### Value Set Declaration
Defines the name of the value set.

| Keyword | Example |
|----------|---------|
| `ValueSet` | `ValueSet: ProposedStatusVS` |

>**Note:** By convention, all value sets should be Pascal case and end with `VS`.

### Value Set Description
The description is a free text containing any information about the value set.

| Keyword | Example |
|----------|---------|
| `Description` | `Description: "The type of evidence backing up the clinical determination of cancer progression."` |

### Code-Definition Declaration
Defines each code and its meaning (one code per line). For the syntax of concept codes, see [Concept Codes](#concept-codes). Codes can either be locally-defined, or selected from external code systems, such as the `CAP#29915` example, in which `29915` is an existing code within the `CAP` codesystem. If the code is defined locally, the namespace serves as the code system.

| Type | Example |
|---------|---------|
| Locally-defined code | `#proposed "The proposal has been proposed, but not accepted or rejected."` |
| Externally-defined code | `CAP#29915 "None/Negative"`|

### Includes
The `Includes` keyword phrases can be used to control entire vocabularies and sets of values from hierarchically-defined code systems. At present, the only hierarchical system supported is SNOMED-CT.

| Keyword | Example | Result|
|----------|---------|---------|
| `Includes codes from` | `Includes codes from MDR`| Value set contains all codes from MDR code system |
| `Includes codes descending from` | `Includes codes descending from SCT#105590001` | Value set contains SCT#1055900001 and all codes below it in the SNOMED-CT hierarchy |  
|  `and not descending from` | `Includes codes descending from SCT#363346000 "Malignant neoplastic disease" and not descending from SCT#128462008  "Secondary malignant neoplastic disease"` | Value set contains SCT#363346000 and all codes below it in the SNOMED-CT hierarchy, _except_ code SCT#128462008 and all codes below it  |

# Map File
Map files control how classes defined in CIMPL are expressed as FHIR profiles.  Map files are FHIR-version dependent. The target FHIR version is given in the [mapping file header](#mapping-file-header)

For a further description of the header keywords, see [Header Keywords](#header-keywords)

## Inheritance of Maps
Maps are inherited. If there is no mapping for a class or attribute, CIMPL tooling automatically looks to the parent class to try and find a mapping. For this reason, when you create a new class inheriting from a parent class that already has a mapping, you may not need to create a mapping for your class, unless you want specific mapping for new properties, or override the inherited map (for example, mapping to a different target).

If no mapping is found for an Entry, that Entry will be mapped to the [Basic resource](https://www.hl7.org/fhir/basic.html). In general, mapping to Basic is not recommended because it has no inherent semantic meaning for implementers. Any unmapped properties will appear as FHIR extensions.

## Sample Map File
```
Grammar:    Map 6.0
Namespace:  shr.core
Target:     FHIR_STU_3

Dosage maps to Dosage:
	DoseSequenceNumber maps to sequence
	InstructionText maps to text
	InstructionCode maps to additionalInstruction
	PatientInstruction maps to patientInstruction
	Timing maps to timing
	AsNeeded maps to asNeeded[x]
	AdministrationSite maps to site
	RouteIntoBody maps to route
	Method maps to method
	DoseAndRate.DoseAmount maps to dose[x]
	DoseAndRate.DoseRate maps to rate[x]
	MaxDosePerPeriod maps to maxDosePerPeriod
	MaxDosePerAdministration maps to maxDosePerAdministration
	MaxDosePerLifetime maps to maxDosePerLifetime

MedicationStatement maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-medicationstatement:
	Identifier maps to identifier
	MedicationBasedOn maps to basedOn 
	MedicationStatementPartOf maps to partOf
	CareContext maps to context
	Status maps to status
	StatusReason maps to extension
	Category maps to category
	MedicationCodeOrReference maps to medication[x]
	OccurrenceTimeOrPeriod maps to effective[x]
	StatementDateTime maps to dateAsserted
	MedicationInformationSource maps to informationSource
	SubjectOfRecord maps to subject
	SupportingInformation maps to derivedFrom
	fix taken to #y
	constrain reasonNotTaken to 0..0
	constrain dosage to 0..1
	ReasonCode maps to reasonCode
	MedicationReasonReference maps to reasonReference
	Annotation maps to note
	Dosage maps to dosage
```
## Mapping File Header
```
Grammar:	Map 6.0
Namespace:	shr.core
Target:		FHIR_STU_3
```
The file target is the version of FHIR that the CIMPL model is mapped to, either `FHIR_DSTU_2`, `FHIR_STU_3`, or `FHIR_R4`.

For a further description of the header keywords, see [Header Keywords](#header-keywords)

## FHIR Mapping Target
Each block in a mapping file begins with statement that establishes a resource, profile, or complex data type that is the target for the mapping. In CIMPL map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in your target output. The initial statement in each mapping block must end with a colon (:).

| CIMPL Building Block | FHIR Target | Example |
|----------|---------|---------|
| Entry | Resource | `SocialHistoryObservation maps to Observation:`|
| Entry | Profile | `LaboratoryObservation maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-observationresults:` |
| Group | Complex Datatype | `Timing maps to Timing:` |

## Mapping Properties
After the first line, the remainder of the block consists of statements that map CIMPL properties to FHIR elements. Each statment involves the `maps to` phrase, where the left is a CIMPL path, and the right is the target FHIR path. 

| Keyword | Example |
|----------|---------|
| `maps to` | `MaritalStatus maps to maritalStatus` |
| `maps to` | `Ethnicity maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity` |
| `maps to` | `MultipleBirth[x] maps to multipleBirth[x].boolean` |
| `maps to` | `Value.ActiveFlag maps to active` |
| `maps to` | `AdverseReaction.AllergenIrritant maps to reaction.substance` |
| `maps to` | `BodySiteOrCode[concept] maps to bodySite` |




With FHIR as the target output, the right-hand side of the map statement can point to any field on a FHIR resource, including:
* Fields that are at the root of the resource (e.g., `maritalStatus`)
* Fields that are child elements of root elements on the resource (e.g., `reaction.substance`)
* Fields that have a [choice of data types](https://www.hl7.org/fhir/formats.html#choice) (e.g., `multipleBirth[x].boolean`)
* Fields that have been defined in existing profiles or other `StructureDefinition` resources (e.g., `http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity`)

## Fix
Similar to the [`is`](#fixed-value) keyword, the `fix` keyword allows for FHIR fields to be locked to a certain value.  It is helpful for locking field values in specific mappings (instead of in the `DataElement` definition, which would be locked for all potential mappings).

| Keyword | Example |
|----------|---------|
| `fix` | `fix status to #completed` |

_Note about syntax: The element being fixed on left hand side of this syntax is a FHIR element, not a CIMPL element. The right hand syntax allows for any code or primitive value to be fixed._

## Constrain
The `constrain` keyword allows users to constrain the cardinality of elements in output files.

| Keyword | Example |
|----------|---------|
| `constrain` | `constrain explanation to 0..1` |

_Note about syntax: The element being fixed on the left hand side of this syntax is a FHIR element, not a CIMPL element. The right hand syntax allows for any [cardinality](#cardinality)._

## Slicing

When fields are allowed to repeat on a target model (e.g., have cardinality with an upper bound greater than 1), it is common practice to specify which CIMPL elements are mapped to instances of that repeated field on the target model.  This process is called **slicing**. For example, the [FHIR Task resource](https://www.hl7.org/fhir/task.html) allows for an arbitrary number of `input` fields to be added.  To ensure that all each of the `input` fields is uniquely addressable by a parsing engine, those inputs need to be differentiated by a **discriminator**.

See the FHIR documentation for more detail about [slicing](https://www.hl7.org/fhir/profiling.html#slicing) and [discriminators](https://www.hl7.org/fhir/profiling.html#discriminator)

### Slicing Strategies

Slicing is used when there is a variation of how an individual element is represented and a need to model it as a sub-list.  An example of this is the [FHIR Blood Pressure profile](https://www.hl7.org/fhir/profiling.html#slicing), which can be considered an observation with two components which are identical in structure other than their represented code.

In CIMPL, slicing strategies are specified in the map file.  Two keywords are used in conjunction to specify the slide:

* `slice on`
* `slice strategy`

Examples of its usage in a statement are shown below:

| Keyword | Example |
|:----------|:---------|
| `slice on` / `slice strategy` | Components.ObservationComponent maps to component `(slice on = code.coding.code; slice strategy = includes)` |
| `slice on` / `slice strategy` | Members.Observation maps to related.target `(slice at = related; slice on = target.reference.resolve(); slice on type = profile; slice strategy = includes)` |
| `slice on` | `Laterality maps to qualifier (slice on = concept)` |

### Declaring a Discriminator**

To perform any slicing, a discriminator needs to be declared, using `slice on` followed by the discriminator.

CIMPL supports the FHIR processing types for discriminators:

| Discriminator | Definition |
| ------------- | ---------- |
| value	| The slices have different values in the nominated element|
| exists |	The slices are differentiated by the presence or absence of the nominated element|
| pattern |	The slices have different values in the nominated element, as determined by testing them against the applicable ElementDefinition.pattern[x]|
| type |The slices are differentiated by type of the nominated element to a specifed profile|
| profile |	The slices are differentiated by conformance of the nominated element to a specifed profile|

### Moving the Slice Location
If the discriminator used for slicing is not the actual field that is being mapped to, `slice at` can allow you to explicitly define the location of the slice.

| Keyword | Example |
|----------|---------|
| `slice at` | `Members.Observation maps to related.target (slice at = related; slice on type)` |

In the above example, though the mapping is declared such that `Members.Observation maps to related.target`, the slicing is not occuring at `related.target`.  Instead, the `slice at = related` ensures that the slicing is occuring at the `related` element.

***

# Appendix A: Constraints Summary

## Cardinality Constraints

```
ReferenceRange 0..0 
```

Narrows the cardinality of a field inherited from a base class.  Common examples are making an optional field required (`0..1` to `1..1`) or prohibiting an optional field (`0..1` to `0..0`).

## Subclass Constraints

```
Specimen substitute BreastSpecimen
```

Indicates that the data type of an inherited field should be restricted to a certain subclass of the originally-declared data type.

## Value Type Constraints

```
Stage.StageDetail[Observation] only CancerStage
```

Indicates that an inherited field's _value_ should be substituted with a more specific child type.

## Choice Selection Constraints

```
FindingResult only IntegerQuantity
```

Indicates that an inherited field's _value_, which is a choice of types, should be a specific type selected from the choices. Note that it is also possible to select a child type of one of the choice types.

## Includes Constraints (Slicing)

```
BloodPressure.ObservationComponent 0..*
    includes SystolicPressure 0..1
    includes DiastolicPressure 0..1
```

Indicates that a field with multiple cardinality should be populated with specific child types. Each child type can be assigned its own cardinality as long as the combined cardinalities still fit within the field's total cardinality.

## Fixed Boolean Constraints

```
WasNotGiven = true
```

Indicates that a boolean field must be a specific value (either `true` or `false`).


## Fixed Code Constraints

```
Vaccine = CVX#36 "VZIG"
Value[Quantity].Units = UCUM#cm
```

Indicates the named property must be given as the specified code.


You are allowed to include from either a `codesystem` or a specific `code` from a codesystem.

<br/>

# Appendix B: Summary of Changes in CIMPL 6.0

For those who have created detailed models using CIMPL 5.0, there have been significant grammar changes to CIMPL 6.0.  

The table below summarizes these changes:

| Change Type | Change Description | CIMPL 5.0 Example | CIMPL 6.0 Example | Section |
|:---- |:----------|:---------------------- |:-------------------|:----------------- |
| New | keyword field constraint `only` binds only one data type to a property | None | `FindingResult only concept` | [Only](#only) |
| New |keyword `Property` is required to define properties for an Entry or Element. |```0..1 TreatmentIntent```| ```Property:  TreatmentIntent 0..1``` | [Property Declaration](#property-declaration) |
| Replace | `EntryElement` keyword replaced by `Entry` | ```EntryElement: CourseOfTreatmentPerformed```| ```Entry:  CourseOfTreatmentPerformed``` | [Element Name Declaration](#element-name-declaration) |
| Replace | `Based on` keyword replaced by `Parent` | `Based on: Observation` | `Parent:  Observation` | [Inheritance](#inheritance) |
| Syntax change | Cardinality is specified _after_ the property or class name | ```0..1 TreatmentIntent``` | ```Property:  TreatmentIntent 0..1``` | [Cardinality](#cardinality) |
| Replace | `is` constraint for fixed values replaced by `=` | ``` FindingTopicCode is LNC#48676-1``` | ``` FindingTopicCode is LNC#48676-1``` | [Field Constraints](#field-constraints) |
| Replace | substitution of a more specific element derived from a parent element using `is type` keyword replaced by `substitute`. | None | ```Specimen substitute BreastSpecimen``` | [Substitute](#substitute) |
| Replace | `code`, `Coding`, and `CodeableConcept` primitives are replaced by a new primitive `concept` | `Value: CodeableConcept from AttributionCategoryVS` | `Value: concept from AttributionCategoryVS` | [Primitives](#primitives) |
| Replace | `must be`, `should be`, `could be`, and `if covered` value set constraints are obsolete and replaced by `(required)`, `(preferred)`, `(extensible)`, and `(example)` | `Type from BreastSpecimenTypeVS if covered` | `Type from BreastSpecimenTypeVS (required)` | [Value Set](#value-set) |
| Replace | `ref()` is now obsolete and replaced with `[]`. | `SourceSpecimen value is type ref(BreastSpecimen)` | `SourceSpecimen[Specimen] substitute BreastSpecimen` | [ References](#references) |
| | | | | |

<br />

>**Note:** As of CIMPL 6.0, `must be`, `should be`, `could be`, and `if covered` value set constraints are obsolete and replaced by `(required)`, `(preferred)`, `(extensible)`, and `(example)`.

| CIMPL 5.0 Example | CIMPL 6.0 Example | Binding Strength |
|----------|---------|---------|
| `from`   | `Substance from SubstanceOfAbuseVS` | Required |
| `Priority must be from http://hl7.org/fhir/ValueSet/request-priority` |`Priority from http://hl7.org/fhir/ValueSet/request-priority (required)` | Required
| `1..1 Status should be from http://hl7.org/fhir/ValueSet/event-status` | `1..1 Status from http://hl7.org/fhir/ValueSet/event-status (preferred)` | Preferred
| `CodeableConcept could be from http://hl7.org/fhir/ValueSet/medication-as-needed-reason` | `concept from http://hl7.org/fhir/ValueSet/medication-as-needed-reason (example)` | Example |
| `Category from http://hl7.org/fhir/ValueSet/goal-category if covered`| `Category from http://hl7.org/fhir/ValueSet/goal-category (extensible)` | Extensible |


# Appendix C: Future Architectural Considerations

* To what extent is the `shr-cli` tool specific to SHR versus a general purpose modeling tool (e.g., for [Death Reporting](https://github.com/nightingaleproject/fhir-death-record)
* Are there any plans to support output of [Open API Specification](https://github.com/OAI/OpenAPI-Specification) files?
* Inclusive output generation, for people that only care about generating FHIR profiles and don't want to wait for other docs to generate