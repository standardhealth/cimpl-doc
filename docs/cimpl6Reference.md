# CIMPL 6.0 Reference Documentation

>**Note:** This documentation is a draft.

_This is a comprehensive guide to CIMPL 6.0 syntax.  If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md). If you're looking for a more in-depth introduction, try the [Tutorial](cimpl6Tutorial_detail.md)._

***

**Table of Contents**

[TOC]

***

# General Overview

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles and implementation guides (IG). Because it is a _language_, written in text statements, CIMPL encourages distributed, team-based development using conventional source-code control tools such as Github. CIMPL enables you to define a model once, then publish that model to multiple versions of FHIR.

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

* Data Element files: `namespace.txt`
* Value Set files: `namespace_vs.txt`
* Mapping files: `namespace_map.txt`

>**Note:**  The file naming convention (_vs, _map) is useful, but does not define the file type. The `Grammar` keyword (appearing in the first line of each file) is required to define the file type.

>**Note:** Any periods in the `namespace` should be replaced by underscores.

For example:
If there is a namespace called `odh.occupation`, the value set file name could be called `odh_occupation_vs.txt`.

Optionally, you can break up the contents of a single namespace into multiple files, for example:

* Data Element files: `namespace_file1.txt`, `namespace_file2.txt`, etc.
* Value Set files: `namespace_file1_vs.txt`, etc.

Additional files for configuring and producing FHIR implementation guides (IGs), and their naming conventions include:
* Configuration files: `ig-igname-config.json`
* Content profile files: `ig-igname-cp.json`
* IG examples: Typically in form: `ignameExampleName.json`
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
### Keywords
CIMPL has a number of keywords. Keywords in CIMPL are followed by colons (:). Keywords are not reserved words in CIMPL language, but it using keywords for other purposes, e.g., as data element names, is not good practice.

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
CIMPL does not enforce or require elements to appear in any particular order, i.e. an element defined at the bottom of the file can be used as field on an element defined at the top.

However, header information must be at the top of the file, before any definitions.
See: [File Headers](#file-headers) for more information

### Naming Collisions
CIMPL does not support duplicate element names within a namespace. However, you are allowed to reuse an element name across different namespaces. In the case that you come across a collision due to an included namespace in `Uses`, you have to refer to the element by its [Fully Qualified Name](#fqn).

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
Primitives are distinguished by starting with lower case letter. CIMPL defines the following primitive data types, which align with FHIR (with one exception, `concept`):

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
* concept
* [oid](https://www.hl7.org/fhir/datatypes.html#oid)
* [id](https://www.hl7.org/fhir/datatypes.html#id)
* [markdown](https://www.hl7.org/fhir/datatypes.html#markdown)
* [unsignedInt](https://www.hl7.org/fhir/datatypes.html#unsignedInt)
* [positiveInt](https://www.hl7.org/fhir/datatypes.html#positiveInt)
* [xhtml](https://www.hl7.org/fhir/datatypes.html#xhtml)

CIMPL also does not have an explicit "Reference" type (see [References](#references).)

### Concept Codes
Unlike FHIR, CIMPL has a single coded type, `concept`. CIMPL does not differentiate between code, Coding, and CodeableConcept.

The grammar for specifying codes is: `SYSTEM#code "Display text"`, for example:
```
SCT#363346000 "Malignant neoplastic disease (disorder)"
ICD10CM#C004  "Malignant neoplasm of lower lip, inner aspect"
```
The systems (appearing before the # sign) are aliases for canonical URIs that are declared using the [CodeSystem keyword](#codesystem). The display text is optional.

***

## Headers

Each CIMPL file contains header information describing the file type and other information. Each of the three main types of files (Data Element, Value Set, and Maps) have slightly different headers.

### Examples

#### Data Element File Header:
```
Grammar:        DataElement 6.0
Namespace:      shr.environment
Description:    "The SHR Environment namespace contains definitions related to the surroundings experienced by the subject."
Uses:           shr.core, shr.base, shr.allergy, shr.observation, shr.medication
CodeSystem:     LNC = http://loinc.org
```

#### Value Set File Header:
```
Grammar:	ValueSet 5.0
Namespace:	shr.core
CodeSystem:     LNC = http://loinc.org
CodeSystem:     MTH = http://ncimeta.nci.nih.gov
CodeSystem:     NCI = https://evs.nci.nih.gov/ftp1/CDISC/SDTM/
CodeSystem:     UCUM = http://unitsofmeasure.org
```

#### Mapping File Header:
```
Grammar:	Map 6.0
Namespace:	shr.core
Target:		FHIR_STU_3
```
The file target is the version of FHIR that the CIMPL model is mapped to, either `FHIR_DSTU_2`, `FHIR_STU_3`, or `FHIR_R4`.

### Keywords In Headers
Keyword that can appear in Headers include, in order: Grammar, Namespace, Description, Uses, Path, CodeSystem, and (only in Map files) Target. The Grammar keyword must be first.

#### Grammar Keyword
The `Grammar` keyword is used in a header to define how the file is to be parsed. The grammar declaration must be the first line in each CIMPL model files.

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

 _(In the rare case where same element name is used in multiple namespaces, you will have to indicate data elements by their [Fully Qualified Name](#fqn))._

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
A Data Element file contains definitions that comprise the information model. It is comprised of a header (see [Headers](#Headers)) followed by any number of class definitions. The order of definitions does not matter.

## Building Blocks
CIMPL provides four types of building blocks:

* `Element` is the lowest-level building block, representing a name-value pair. An Element is present in the FHIR output as a property or extension, but never as a top-level profile.
* `Group` is a building block comprised of other building blocks, specifically, other Groups, Elements, and Entries. A Group is present in the FHIR output as a backbone element or complex extension, but never as a top-level profile.
* `Entry` is a building block representing a stand-alone piece of information, analogous to a FHIR resource or profile. 
* `Abstract` is a special type of Entry that cannot be instantiated, and will not be present in the target mapping.

All classes defined in CIMPL are based on one of these four building blocks. All classes in CIMPL reusable (defined once and used many times).


### Summary
| Building Block | Has... | Cardinality? | Choices? | Analogous FHIR Type |
|----------|---------|---------|---------|---------|
| `Element` | Value | No | Yes | Property or simple extension |
| `Group` |  Properties | Yes | No | Backbone element or complex extension |
| `Entry`  | Properties | Yes | No | Resource or profile |
| `Abstract` | Properties | Yes | No | none |


## Class Definition

### Class Name
The first line of a class definition is a keyword representing the type of building block, followed by a descriptive name you choose. Class names must be unique within a given namespace.

| Building Block | Example |
|----------|---------|
| `Element` | `Element: ZipCode` |
| `Group` |  `Group: ReferenceRange` |
| `Entry`  | `Entry: FinancialSituation` |
| `Abstract` |`Abstract: DomainResource` |

#### Fully Qualified Names

In addition to its declared name, all elements also have a unique identifying `Fully Qualified Name` (`FQN`).

The FQN is a combination of the element's namespace and its declared name (through concatention delimited by a period). For example, the element `Observation` in the namespace `shr.finding` has the following FQN:
```
shr.finding.Observation
```
>**Note:** Naming collisions between elements across multiple namespaces is possible. In the case of a naming collision (such as when a namespace `Uses` multiple namespaces that define an element with the same declared name), you have to refer to the specific unique FQN instead of the declared name in element, field, and value definitions.

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

TO DO: SUPPORT FOR MARKDOWN?

>**Note:** While the CIMPL tooling will allow for any pattern of [Unicode](http://unicode.org/standard/WhatIsUnicode.html) characters within enclosed double quotation marks (`"`), certain exporters such as the FHIR exporter will object to non-[ASCII](https://en.wikipedia.org/wiki/ASCII) text.)

### Property Keyword (Group, Entry, and Abstract only)

Classes based on Group, Entry, and Abstract are composed of one or more properties (also called _fields_ or _attributes_). A property can be any Element, Group, or Entry, but never a primitive. Each property must have a specified cardinality range, represented as `min..max`, indicating the number of repeats of the property.

In the following example, StudyArm has three properties: Name, Type, and Comment. Name is singular and required, Type is optional and repeating, and Comment is singular and optional:
```
Group:             StudyArm
Description:       "Refers to participant(s) in a clinical trial assigned to receive specific interventions according to a protocol."
Property:          Name 1..1
Property:          Type 0..*
Property:          Comment 0..1
Property:          ResearchStudy 1..1
```
>**Note:** Classes that have Properties cannot have a Value element.

#### Implicit References in Properties

A property that refers to an Entry (such as ResearchStudy, above) is implicitly a reference (pointer) to that Entry, rather than implying the Entry is "in-lined" into the class.

>**Note:** CIMPL 5.0 Grammar use of the keyword `ref()` is now obsolete.

### Value Keyword

The Value represents the data type(s) an Element can accept. The Value keyword can only be used in Elements. Values can be [primitives](#primitives), Elements, Groups, or Entries. Everything eventually "bottoms out" to Elements whose values are primitive types.

Values can be simple, such as:
```
Element:           DetectionTime
Description:       "The date on which the condition was first observed."
Value:             dateTime
```
or offer a choice of data types, represented as a list:
```
Element:           ExpectedPerformanceTime
Description:       "When an action should be performed."
Value:             dateTime or TimePeriod or Timing
```
Value choices can freely mix primitives, Elements, Groups, and Entries:
```
Element:           SubstanceOrCode
Description:       "A code or substance reference identifying the ingredient."
Value:             concept or Substance or Medication
```
>**Note:** Classes that have a Value cannot also have Properties.

>**Note:** CIMPL 5.0 Grammar associating a cardinality to a Value is now obsolete.

#### Grammar to Bind a Value Set to a Value

In general, constraints cannot be applied to a property or value at the same time the item is defined. One exception is that you are allowed to bind a value set to a `concept` at the same time an Element is defined, as follows:

```
Element:           IsPrimaryTumor
Description:       "Whether the tumor is the original or first tumor in the body, for a particular cancer."
Value:             concept from YesNoUnknownVS (required)
```
CIMPL supports four binding strengths, `required`, `extensible`, `preferred`, and `example`, the same as FHIR.

# Constraints
Constraints can be applied properties, values, or value choices. Constraints can be placed on top-level properties or nested properties (see [Constraining Nested Properties](#constraining-nested-properties)). Unlike class definitions, _constraint statements do not begin the line with a keyword followed by a colon_.

Constraints can be applied to locally-defined properties or properties inherited from the parent.

| To do this.. | Use this Keyword or Symbol | Applies to |
|----------|---------|---------|
| [Narrow cardinality](#cardinality-constraint) | `{min}..{max}` | Property |
| [Assign a fixed value](#fixed-value-constraint) | `=`| Property or Value[Choice] |
| [Narrow data type](#substitute-constraint) | `substitute` | Property or Value[Choice] |
| [Bind a value set](#value-set-binding-constraint) | `from` | Property or Value[Choice] |
| [Slice an array](#includes-constraint) | `includes` | Array Property |
| [Populate an array](#array-population-constraint) | `+=` | Array Property |
| [Narrow choices for a value](#only-constraint) | `only` | Value |

### Using Bracket Notation to Denote a Value Choice
As explained in [Value Keyword](#value-keyword), a Value can offer a choice of data types. When applying constraints, you must indicate _which_ choice the constraint applies to. This is expressed in _bracket notation_.

For example, if an element defines `Value: boolean or concept`, then `Value[boolean]` refers to the boolean choice, and `Value[concept]` refers to the `concept` choice. Any constraint that can apply to a boolean type can apply to `Value[boolean]` (such as fixing the value to `true`), and any constraint that can apply to a concept can be applied to `Value[concept]` (such as binding a value set).

>**NOTE:** Bracket notation must always be used when applying a constraint to a value choice, even if the value has only one choice. This is a change from CIMPL 5.

## Cardinality Constraint
Cardinality defines the number of repeats that can exist for a property. Cardinality is specified using FHIR syntax, {min}..{max}where the first integer indicates the minimum repeats (0 implying optional), and the second digit expresses the upper bound. To express no upper bound, a `*` is used. When constraining cardinality, you can only narrow the previously declared cardinality.

Cardinality constraints apply only to properties.

| Previous Cardinality | Example Constraint |
|----------|----------|
| `Property: LanguageUsed 0..*` | `LanguageUsed  0..0` |
| `Property: GovernmentIssuedID  0..*` <br> `GovernmentIssuedID 1..*` | `GovermentIssuedID 1..2` |
| `Property: Code 1..1` |<del>`Code 0..0`</del> invalid|

>**Note:** By constraining a property to 0..0, you effectively remove an inherited field from an element. Use 0..0 with caution, because if an instance _does_ include that property, the instance will fail validation and may be rejected.

## Fixed Value Constraint

Fixed values are designated with an `=` operator. Using `=` limits the field to a specific value (e.g., a certain code to a `concept`, a specific `string`, or a `boolean` value). The assigned data type must be consistent with the definition of the property. Fixed value constraints can only be applied to primitives.

CIMPL _does_ allow a fixed concept to be overridden in child classes. This feature is necessary when inheritance narrows the meaning of a class characterized by a concept code (for example, when Fungal Pneumonia (fixed code SCT#233613009) inherits from Pneumonia (fixed code SCT#233604007)). By allowing the child class to assign a different fixed code, CIMPL _assumes_ without checking that the overriding code in the child class has more constrained semantics than the code it replaces.

Fixed value constraints apply to properties and value choices.
Currently, CIMPL does not yet support fixing all value types. Fixing numerical types and strings (including URLs) is not yet supported.

| Example | Syntax |
|----------|---------|
| Fix a code to a concept property | `ObservationCode = LNC#82810-3` |
| Fix a boolean value to a boolean property | `IsPrimaryTumor = true` |
| Fix a code to a concept value choice | `Value[concept] = SCT#233613009 "Pneumonia"` |
| Fix a code to a value  | <del>`Value = SCT#233613009 "Pneumonia"`</del> invalid|

>**Note:** If you only need to fix a value code for one particular mapping (one version of FHIR, for example), do so using the [`fix`](#fix) keyword in the mapping file.

## Substitute Constraint
The `substitute` keyword constrains the data type of a property. The new data type that is substituted for the original must be a subclass of the original data type. 

| Example | Syntax |
|----------|---------|
| Constrain the property `Specimen` to be a tumor specimen | `Specimen substitute TumorSpecimen` <br/> _note: TumorSpecimen must be a subclass of Specimen_ |
| Constrain a value choice of type `Quantity` to `SimpleQuantity` | `Value[Quantity] substitute SimpleQuantity` <br/> _note: SimpleQuantity is a subclass of Quantity_ |
| Constrain a value of type `Quantity` to `SimpleQuantity` | <del>`Value substitute SimpleQuantity`</del> invalid |


## Value Set Binding Constraint

Binding is the process of associating a concept with a set of possible values. Binding uses the keyword `from`. The object of a binding must be an property that is an Element whose value is a concept. The value set that is being bound can be a named value set, defined in CIMPL, or a canonical URL external to CIMPL.

CIMPL uses the same [binding strengths as FHIR](https://www.hl7.org/fhir/valueset-binding-strength.html), namely:

* `required` indicates that the code must come from the specified value set.
* `extensible` indicates that the code must come from the specified set _if the value set contains a relevant code_; otherwise a code outside the value set may be chosen.
* `preferred` indicates that the code should ideally come from the specified value set, but codes outside the value set may also be chosen.
* `example` indicates that the code may come from anywhere; the specified value set is for example purposes only.

If no binding strength is specified, the binding is assumed to be `required`.

| Example | Syntax |
|----------|---------|
| Required binding of a Property | `ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical (required)` |
| Required binding of a Property (by default) | `ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical` |
| Extensible binding of a Property | `ReasonCode from ReasonCodeVS (extensible)` |
| Preferred binding of a Property | `BodyLocation from BodyLocationVS (preferred)` |
| Required binding of a Value Choice that is a concept | `Value[concept] from http://hl7.org/fhir/ValueSet/condition-clinical (required)` |
| Required binding of a Value Choice that is an Element with whose value is a `ClinicalStatus` | `Value[ClinicalStatus] from http://hl7.org/fhir/ValueSet/condition-clinical (required)` |

### Includes Constraint

The `includes` constraint is used to specify that a repeating property (array) should contain item(s) or a certain class or classes. FHIR refers to this as [slicing](https://www.hl7.org/fhir/profiling.html#slicing). To avoid confusion that usually surrounds slicing, CIMPL just says the array "includes" certain types of elements. The `includes` statement has several requirements:

* The elements included in the array must conform to the data type of the array. For example, if the array property is `Identifier 0..*`, the items included in the array must be Identifiers or be inherit from Identifier.
* Cardinality of the inclusion must be specified, and can be any cardinality that fits the cardinality of the array. For example, if the array has a finite maximum, it cannot include an element with cardinality 0..*.
* The keyword `includes` is repeated for each item the array should contain.
* Includes constraint does not apply to Values.

Here is an example:
```
Measurement 0..*
...
Measurement includes TumorLength 1..1 includes TumorWidth 0..1 includes TumorDepth 0..1
```
For clarity, the second statement can be written on separate lines (since CIMPL is whitespace insensitive), as follows:
```
Measurement
includes TumorLength 1..1
includes TumorWidth 0..1
includes TumorDepth 0..1
```
In this example, TumorLength, TumorWidth, and TumorDepth all must inherit from Measurement, because Measurement is the array property being sliced. Here is another example:
```
Identifier 1..*
...
Identifier
includes NationalProviderIdentifier 0..1
includes TaxIdentificationNumber 0..1
```
In this case, either NationalProviderIdentier or TaxIdentificationNumber must be present, even though both are individually optional, because of the 1..* cardinality on Identifier.

>**Note:** When using `includes`, special mapping syntax is required. See [Slicing](#slicing).

### Array Population Constraint
Use the `+=` operator insert fixed values into value arrays. The `+=` is similar to the fixed value `=` constraint, but it applies to repeating properties (arrays). You can use the `+=` operation repeatedly on the same array, limited only be the cardinality of the array. The `+=` operator cannot be applied to value choices.

For example, you would use `+=` to insert a particular Social Security Number (SSN) or Driver's License number into an array of identifiers. Note that `includes` would be used to require the array contain one or more SSNs, without specifying a particular SSN.

| Example | Syntax |
|----------|---------|
| Insert "Behavior" into Category array |  `Category += LNC#54511-1 "Behavior"` |
| Add "Social History" to same Category array | `Category += OBSCAT#social-history "Social History"` |

### Only Constraint
Values can have multiple data types. Use the `only` constraint to narrow the choice of data types. The `only` constraint can only be applied to Value that have multiple choices. It cannot be applied to properties or value choices. 

| Example | Syntax |
|----------|---------|
| Only allow a `string` for a Value defined as:<br/>`Value: string or boolean or dateTime` |  `Value only string` |
| Only allow a `string`  for a Value defined as:<br/>`Value: string or boolean or dateTime`  | <del>`Value[string] only string`</del> invalid|

>**Note:** CIMPL currently does not support narrowing a choice to a more limited choice, for example, narrowing a choice of three types to two types.

>**Note:** `only` shall never be preceded by bracketed term, because intrinsically, it applies to all value choices.





***

# Value Set File
The value set files are used to define custom value sets and codes when existing sources like [HL7 v3](https://www.hl7.org/fhir/terminologies-v3.html), [FHIR](https://www.hl7.org/fhir/terminologies-systems.html), [VSAC](https://vsac.nlm.nih.gov/), or [PHIN VADS](https://phinvads.cdc.gov/) are insufficient.

An example value set file is shown below:

```
Grammar:        ValueSet 6.0
Namespace:      shr.base

ValueSet:               ValueAbsentReasonVS
Description:            "Reasons that a value associated with a test or other finding is missing."
Includes codes from DAR
#missing_refused        "Human source was asked but declined to respond to the question, or an applicable question was left unanswered."
#missing_noexplanation  "The reason the information is not present is not known."
#missing_nonesuch       "The answer is missing because nothing of a type of thing is known to exists, e.g., the siblings of an only child. Also use this code to represent a 'none of the above' answer"
#missing_collection     "Missing due to a problem collecting, identifying, or locating the specimen, including patient refusal or unable to provide specimen"
#missing_specimen       "Missing due to a problem with the specimen, e.g. contamination, clotting, improper tube type, improper storage, too small, etc."
#missing_malfunction    "Missing due to instrument malfunction."
```

### Value Set Declaration
Defines the name of the value set.

| Keyword | Example |
|----------|---------|
| `ValueSet` | `ValueSet: ProposedStatusVS` |

>**Note:** By convention, all value sets should be Pascal case and end with `VS`.

### Value Set Description
Defines the name of the value set.

| Keyword | Example |
|----------|---------|
| `Description` | `Description: "The status of a proposal."` |

### Code-Value Declaration
Define the code or value.

| Keyword | Example |
|----------|---------|
|| `#proposed "The proposal has been proposed, but not accepted or rejected."` |
|| `CAP#29915		"None/Negative"`|

Each value is prefaced by a hash symbol (`#`) and followed by a description enclosed in quotation marks. By convention, each value should use lowercase [snake_case formatting](https://en.wikipedia.org/wiki/Snake_case).

This can either be a new user defined `code`, or it can be individual codes from separate codesystems, such as the `CAP#29915` example, in which `29915` is an existing code within the `CAP` codesystem.

### Includes

Extends a custom defined `ValueSet` to include codes from other codesystems.

| Keyword | Example |
|----------|---------|
| `Includes` | `Includes codes from MDR` |
| `Includes` and `descending from` | `Includes codes descending from SCT#105590001` |
| `Includes` and `descending from` and `and not descending from` | `Includes codes descending from SCT#105590001 "Substance" and not descending from SCT#410942007 "Drug or medicament"` |

# Map File
By default, new elements will appear as FHIR extensions unless the element is mapped to an existing FHIR element. These mapping allow for simpler slicing and fixed values.

## Sample Map File
```
Grammar:	Map 6.0
Namespace:	shr.allergy
Target:		FHIR_STU_3


Condition maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-condition:
    RelatedEncounter maps to context
    Informant maps to asserter
    Subject maps to subject
    Entry.CreationTime maps to assertedDate
    Value maps to code
    Category maps to category
    ClinicalStatus maps to clinicalStatus
    Onset[x] maps to onset[x]
    Abatement[x] maps to abatement[x]
    BodySiteOrCode[concept] maps to bodySite
    BodySiteOrCode[BodySite] maps to http://hl7.org/fhir/StructureDefinition/body-structure
    Severity maps to severity
    Stage maps to stage
    Stage.Value maps to stage.summary
    Evidence maps to evidence
```
## Element Mapping
Maps a CIMPL element to a target element.

| Keyword | Example |
|----------|---------|
| `maps to` | `FinancialSituation maps to DiagnosticResult:` |


In CIMPL map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in your target output.

If no FHIR mapping is defined for an element and it inherits no mappings from its parents, it defaults to a mapping to the [Basic resource](https://www.hl7.org/fhir/basic.html).  In general, this is not recommended because it has no inherent semantic meaning for implementers.  To promote interoperability and reuse, CIMPL modelers are encouraged to find target models that already exist and use constraints or extensions to modify them according to project needs.

## Field Mapping
Maps a CIMPL element field to a target field.

| Keyword | Example |
|----------|---------|
| `maps to` | `MaritalStatus maps to maritalStatus` |
| `maps to` | `Ethnicity maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity` |
| `maps to` | `MultipleBirth[x] maps to multipleBirth[x].boolean` |
| `maps to` | `Value.ActiveFlag maps to active` |
| `maps to` | `AdverseReaction.AllergenIrritant maps to reaction.substance` |
| `maps to` | `BodySiteOrCode[concept] maps to bodySite` |


In CIMPL map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in your target output.

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