# CIMPL Language Reference Manual

## Table of Contents

[TOC]

***

## Introduction

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles, extensions and implementation guides (IG). Because it is a _language_, written in text statements, CIMPL encourages distributed, team-based development using conventional source code control tools such as Github. CIMPL provides tooling that enables you to define a model once, and publish that model to multiple versions of FHIR.

### Purpose

This document is a comprehensive guide to the CIMPL language, Version 6.0 and above, covering the syntax used to specify clinical models, value sets, and mappings to FHIR.

### Intended Audience

The CIMPL Language Reference is targeted to the users of CIMPL, involved with creating or reviewing models expressed in the CIMPL language.

### Prerequisite

This guide assumes you have:

* Installed the latest version of the SHR-CLI software as documented in [CIMPL Setup and Installation](cimplInstall.md) (preferably installed in the `~/cimpl/shr-cli` directory).
* A text editor to create your CIMPL files (preferably VSCode with the _vs-code-language-cimpl_ extension, but not required).
* Reviewed the [In-Depth Tutorial](cimpl6Tutorial_detail.md).


## Fundamentals

### Versioning

CIMPL follows the [semantic versioning](https://semver.org) convention (MAJOR.MINOR.PATCH).

* MAJOR: A major release has significant new functionality and potentially, grammar changes or other non-backward-compatible changes.
* MINOR: Contains new or modified features, while maintaining backwards compatibility within the major version.
* PATCH: Contains minor updates and bug fixes, while maintaining backwards compatibility within the major version.

CIMPL is currently on major version 6.

For a full change log, see the [Release Notes](https://github.com/standardhealth/shr-cli/releases).

### File Types

Model information in CIMPL is stored in the following three file types:

* Class files (.txt): contain definitions of CIMPL classes.
* Value Set files (.txt): contain definitions of value sets defined in the namespace.
* Map files (.txt): contain information on how the CIMPL classes relate to FHIR resources, profiles, and elements.

Each namespace will typically have one or more class files, and if needed, a value set and map file.

### Classes

CIMPL models consist of classes. Classes define clinical information at some granularity. Classes can be combined in various ways (composition and specialization) to create meaningful structures.

### Namespaces

CIMPL organizes information into namespaces. [Namespaces](https://en.wikipedia.org/wiki/Namespace) are used to organize code and classes into logical collections and to prevent name collisions.

## Naming Conventions

### File Names

File names must begin with lower case and typically include the namespace:

* Class files: _namespace.txt_ or _namespace-something.txt_
* Value Set files: _namespace-vs.txt_ or _namespace-something-vs.txt_
* Mapping files: _namespace-map.txt_ or _namespace-something-map.txt_

Any periods in the `namespace` should be replaced by dashes. For example, for a namespace called `odh.occupation`, the recommended value set file name is _odh-occupation-vs.txt_. The _something_ is useful when you want to break up the contents of a single namespace into multiple files, for example:

* _obf-action.txt_
* _obf-finding.txt_

>**Note:** `obf` stands for _Objective FHIR_, a set of classes provided with CIMPL that can help you create FHIR profiles. For more information, see [Objective FHIR Overview](cimpl6ObjectiveFHIR.md)

Additional files for configuring and producing FHIR implementation guides (IGs), and their naming conventions include:

* Configuration files: _ig-igname-config.json_
* Content profile files: _ig-igname-cp.json_
* IG examples: _ignameExampleName.json_
* IG HTML pages: _anyName.html_

### Namespace Names

Best practice is to follow the naming convention pattern of `organization.domain` or `project.domain` (followed by subdomains if necessary).

| Example | Purpose |
|-------|-------|
| `obf.lab` | Laboratory profiles belonging to Objective FHIR |
| `odh` | Occupational Data for Health |
| `onco.core` | Core namespace for oncology-related profiles |

### Class Names

CIMPL classes are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase). For example:

```
Dosage
DetectionTime
AdverseEvent
Neutrophils100WBCNFrPtBldQnAutoCntLabObs
```

>**Note:** Letters, numbers, and hyphens are allowed. However, class names must begin with an uppercase letter.

### Fully Qualified Names

CIMPL does not support duplicate class names within a single namespace. However, the same name may occur in different namespaces. In the case of naming collisions due to an included namespace in [`Uses`](#Uses), you have to refer to the class by its fully qualified name (FQN). The FQN is a combination of the element's namespace and its declared name (through concatenation delimited by a period). For example, the element `CancerCondition` in the namespace `onco.core` has the following FQN:

```
onco.core.CancerCondition
```

FQNs are only required in case of naming collisions between namespaces.

### Value Set Names

Value Sets are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase), with a suffix of "VS". For example:

```
ValueSet: HomeEnvironmentRiskVS
```
>**Note:** Value Set names must begin with an uppercase letter.

### Local Code Names

Within a custom value set, the individual codes (denoted by # symbol) are conventionally written in lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case). For example:
```
ValueSet:          HomeEnvironmentRiskVS
#smoke_detectors   "No Smoke Detectors"
#radiation         "Radon or Other Radiation Source"
#swimming_pool     "Swimming Pool"
```

## Language Basics

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

### Comments

CIMPL follows [JavaScript syntax](https://www.w3schools.com/js/js_comments.asp) for code comments:

```
// Use a double-slash for comments on a single line

/*
Use slash-asterisk and asterisk-slash for larger block comments.
These comments can take up multiple lines.
*/
```

### Reserved Words

CIMPL has a number of reserved words and phrases that are part of CIMPL's grammar (e.g., `boolean`, `or`, `from`, `maps to`). For a complete list of reserved words, refer to the [ANTLR4 grammar](https://github.com/standardhealth/shr-grammar/tree/master/src/main/antlr).

### Primitives

Primitives are data types, distinguished by starting with a lower case letter. CIMPL defines the following primitive data types. With the exception of `concept` (a simplified representation of Code, Coding, and CodeableConcept), the primitive types in CIMPL align with FHIR:

* [`concept`](#concept-codes)
* [`boolean`](https://www.hl7.org/fhir/datatypes.html#boolean)
* [`integer`](https://www.hl7.org/fhir/datatypes.html#integer)
* [`string`](https://www.hl7.org/fhir/datatypes.html#string)
* [`decimal`](https://www.hl7.org/fhir/datatypes.html#decimal)
* [`uri`](https://www.hl7.org/fhir/datatypes.html#uri)
* [`base64Binary`](https://www.hl7.org/fhir/datatypes.html#base64Binary)
* [`instant`](https://www.hl7.org/fhir/datatypes.html#instant)
* [`date`](https://www.hl7.org/fhir/datatypes.html#date)
* [`dateTime`](https://www.hl7.org/fhir/datatypes.html#dateTime)
* [`time`](https://www.hl7.org/fhir/datatypes.html#time)
* [`oid`](https://www.hl7.org/fhir/datatypes.html#oid)
* [`id`](https://www.hl7.org/fhir/datatypes.html#id)
* [`markdown`](https://www.hl7.org/fhir/datatypes.html#markdown)
* [`unsignedInt`](https://www.hl7.org/fhir/datatypes.html#unsignedInt)
* [`positiveInt`](https://www.hl7.org/fhir/datatypes.html#positiveInt)
* [`xhtml`](https://www.hl7.org/fhir/datatypes.html#xhtml)

>**Note:** CIMPL does not have an explicit _Reference_ data type. References are determined during the [mapping process](#mapping-to-references).

### Concept Codes

CIMPL uses a single primitive type, `concept` to represent coded terms from controlled vocabularies. A concept is comprised of code system, code, and optional display text. The grammar for specifying concepts is:

> `SYSTEM#code "Display text"`

For example:

```
SCT#363346000 "Malignant neoplastic disease (disorder)"
ICD10CM#C004  "Malignant neoplasm of lower lip, inner aspect"
```
`SYSTEM` is an alias for a canonical URI that represents a code system. Aliases must be declared in the file header using the [CodeSystem keyword](#codesystem). You cannot use the canonical URI directly in the concept grammar.

Unlike FHIR, CIMPL does not differentiate between **Code**, **Coding**, and **CodeableConcept**. See [Mapping Concept Codes](#mapping-concept-codes) for information on how CIMPL maps `concept` to these FHIR data types.

***

## Keywords

Keywords are used to make various declarations. Keywords always appear at the beginning of a line and are followed by a colon.

Keyword Summary:

| Keyword | Purpose | File Type(s) |
|----------|---------|---------|
| [`Abstract`](#abstract) | Declare an non-instantiable class | Class |
|[`CodeSystem`](#codesystem) | Define a code system alias | Class, Value Set |
| [`Concept`](#concept) | Express the meaning of a class | Class |
| [`Description`](#description) | Provide a human-readable description | Class, Value Set |
| [`Element`](#element) | Declare lowest-level (elemental) model building block  | Class |
| [`Entry`](#entry) | Declare a stand-alone class | Class |
| [`Grammar`](#grammar) | Define how a file should be parsed | All |
| [`Group`](#group) | Declare mid-level, non-stand-alone model building block  | Class |
| [`Namespace`](#namespace) | Associate a file with a namespace | All |
| [`Parent`](#parent) | Specify inheritance | Class |
| [`Path`](#path) | Alias long URLs | Class, Value Set |
| [`Property`](#property) | Define an attribute in a class | Class |
| [`Uses`](#uses) | Import namespace(s) | Class |
| [`Value`](#value) | Define data type(s) of Elements | Class |
| [`ValueSet`](#valueset) | Define a value set | Value Set |

### Abstract

The `Abstract` keyword is used to declare a new `Abstract` class. `Abstract` classes are identical to `Entry`, except they are not instantiable. For details, see [Class File](#class-file).

Example:

`Abstract: ActionStatement`

### CodeSystem

The `CodeSystem` keyword provides an alias or shorthand for the canonical URI of a code system. You _must_ define a code system alias before you can use codes from that code system in constraints or value sets. By convention, the alias should be be UPPER CASE, although the tooling allows for numbers and hyphens after the first upper case letter.

Examples:

`CodeSystem: LNC = http://loinc.org`

`CodeSystem: SCT = http://snomed.info/sct`

### Concept

The `Concept` keyword (not to be confused with the [concept primitive](#concept-codes)) establishes the meaning of a class in terms of a code (or codes) from controlled vocabularies. Assigning a concept allows the meaning of the class to be understood without inferring it from the class name. If multiple concept codes are used, they should appear as a comma-separated list. This keyword is optional.

Examples:

`Concept: MTH#C3858779 "Security classification"`

### Description

The `Description` keyword is used for a narrative that defines the namespace, class, or value set.
<!-- Carmela - TO DO: Do we support Markdown? -->
Example:

`Description: "Oncology data elements that broadly apply to most cancer cases."`

>**Note:** While the CIMPL tooling allows for any pattern of [Unicode](http://unicode.org/standard/WhatIsUnicode.html) characters within enclosed double quotation marks (`"`), certain exporters such as the FHIR exporter will issue a warning when non-[ASCII](https://en.wikipedia.org/wiki/ASCII) text is encountered.

### Element

The `Element` keyword is used to declare a new class of type `Element`, the lowest-level building block in CIMPL. For details, see [Class File](#class-file).

Example:

`Element: EvidenceType`

### Entry

The `Entry` keyword is used to declare a new class, analogous to a **Resource** or **Profile** in FHIR. For details, see [Class File](#class-file).

Example:

`Entry: GenomicsReport`

### Grammar

The `Grammar` keyword defines how the file is to be parsed. The grammar declaration must be the first line in each CIMPL file.

| File Type | Required First Line | Current Version # |
|----------|---------|---------|
| Class | `Grammar: DataElement #.#` | 6.0 |
| Value Set | `Grammar: ValueSet #.#` | 5.1 |
| Map | `Grammar: Map #.#` | 5.1 |

>**Note:** no space in `ValueSet`

The version details the _major_ and _minor_ version of the grammar the file conforms to. Generally, the major version of the grammar matches the major version number of CIMPL, however, the ValueSet and Map grammar remain at 5.1.

### Group

The `Group` keyword is used to declare a referenceable collection of one or more properties, similar to a **complex data type** in FHIR. For details, see [Class File](#class-file).

Example:

`Group: Dosage`

### Namespace

The `Namespace` keyword defines the association of the file with a namespace, and is required in `Class`, `ValueSet` and `Map` files. The [namespace value](#namespace-names) can be any number of lowercase period-delimited words.

Examples:

`Namespace: odh`

`Namespace: onco.core`

>**Note:** While it is best practice to use only lower case letters, the strict requirements only require the word to begin with a lowercase letter. The rest of the namespace allows for mixed casing and hyphens.

### Parent

Through the `Parent` keyword, CIMPL provides a mechanism for basing a class on another class, the child inheriting all properties and constraints from the parent. The child class can then extend information inherited from the parent by adding additional properties and applying additional constraints. Additionally, maps are inherited alongside properties. At most one parent class can be specified. The `Parent` declaration is optional.

There are restrictions on which building blocks can inherit from other building blocks. The rules can be summarized as _like inherits from like_:

| Building Block| Can Inherit From |
|----------|---------|
| `Element` | `Element` |
| `Group` | `Group` |
| `Entry` | `Abstract` or `Entry` |
| `Abstract` | `Abstract` |

>**Note:** Although the CIMPL 6.0 tooling does not currently enforce these inheritance rules, they will be enforced in future releases. You are strongly encouraged to comply with these rules, and refactor any definitions ported from CIMPL 5.x that do not comply.

Example:

 `Entry:  WhiteBloodCellCountObservation`

 `Parent: LaboratoryObservation`

### Path

The `Path` keyword allows for abbreviations of long URLs. This functionality is optional; it is provided to make authoring easier. By convention, the alias should be be an all UPPER CASE word, although the tooling allows for numbers and hyphens after the first letter.

Example:

`Path: FHIR = http://hl7.org/fhir/ValueSet`

### Property

Classes based on `Group`, `Entry`, and `Abstract` are composed of one or more properties (called _fields_ or _attributes_ in object-oriented programming). A property can be any `Element`, `Group`, or `Entry`, but never a primitive data type. Each `Property` must have a specified cardinality range, represented as `min..max`, indicating the number of repeats of the `Property`. The `Property` keyword cannot be used in `Element`.**

In the following example, _StudyArm_ has four properties: _Name_, _Type_, _Comment_ and _ResearchStudy_. _Name_ is singular and required, _Type_ is optional and repeating, and _Comment_ is singular and optional:

```
Group:             StudyArm
Description:       "Refers to participant(s) in a clinical trial assigned to receive specific interventions according to a protocol."
Property:          Name 1..1
Property:          Type 0..*
Property:          Comment 0..1
Property:          ResearchStudy 1..1
```

A property that refers to an `Entry` (such as _ResearchStudy_, above) is implicitly a reference (pointer) to an instance of that `Entry`, rather than implying the `Entry` is _in-lined_ into the class.

>**Note:** CIMPL 5.0 Grammar use of the keyword `ref()` is obsolete.

### Uses

The `Uses` keyword appears in the Class file and provides a comma-separated list of the namespaces imported to the current namespace. This keyword allows you to use the classes and value sets defined in other namespaces. The list order has no effect.

Examples:

`Uses: obf.datatype`

`Uses: obf.datatype, obf, onco.core`

<!-- Carmela: TODO: VERIFY THIS: In CIMPL version 6, it is *required* that you import `shr.core` for the tooling to run.-->

### Value

`Value` represents the data type(s) an `Element` can accept. This keyword can only be used when defining an `Element`. Each `Element` must have exactly one `Value`, although the `Value` itself can be a choice of several data types. `Value` choices can be [Primitives](#primitives), [Element](#element), [Group](#group), or [Entry](#entry). A `Value` may be inherited and constrained in the child class.

Examples:
```
Element:           DetectionTime
Description:       "The date the condition was first observed."
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

Although in general, constraints cannot be applied to a `Property` or `Value` at the same time the item is defined, you are allowed to bind a value set to a `concept` value choice at the same time the `Value` is defined, as follows:

````
Element:           IsPrimaryTumor
Description:       "Whether the tumor is the original or first tumor in the body, for a particular cancer."
Value:             concept from YesNoUnknownVS (required)
````
For more information on binding, see [Value Set Binding Constraint](#value-set-binding-constraint).

>**Note:** CIMPL 5.0 Grammar associating a cardinality to a `Value` is obsolete.

### ValueSet

Defines the name of a value set defined inside CIMPL (references to external value sets are also allowed). By convention, all value set names should be Pascal case, starting with the name of the corresponding concept data type Element that is being bound to the value set, and ending with _VS_.

Example:

`ValueSet: ProposedStatusVS`
***

## CIMPL Paths

CIMPL provides grammar that allows you to refer to any part of a CIMPL class, regardless of nesting. This section outlines CIMPL path grammar, which is used in [constraints](#constraints) and [mapping](#map-file).

### Dot Notation

Dot notation is used to denote nested properties. The path lists the properties in order, separated by a dot (.) For example, if _Patient_ has a _ContactPoint_, and _ContactPoint_ has a _Priority_, then the path to _Priority_ is _Patient.ContactPoint.Priority_.

### Entry Terminates Path

An `Entry` can appear in a path only as its _final component_. Paths represent nested containment within one class, and `Entry` is an independent object. Paths cannot hop from one `Entry` to another.

For example,  _Procedure_ has a _Patient_ property, and while _Procedure.Patient_ is a valid path, _Procedure.Patient.gender_ is not. The former path can be used to set the cardinality of _Patient_ within the _Procedure_ class, but the latter would (illegally) constrain an attribute within _Patient_ from inside  _Procedure_.

### Bracket Notation for Value Choices

`Value` can offer a [choice of data types](#value). Bracket notation is used to refer to a particular value choice. Bracket notation must always be used when an `Element` is part of a CIMPL path. How `Value` choices appear in paths depends on whether the path appears with an `Element` definition or not.

| Inside an: | Refer to the `Value` choices as: |
|----------|---------|
| `Element` | `Value[choice]` |
| `Entry` or `Group` | `ElementName[choice]` |

Example:

```
Entry: Procedure
Property: Priority 0..1
Priority[concept] = SCT#394849002 "High Priority" // not Value[concept]

Element: Priority
Value: integer or concept
Value[concept] from PriorityVS (required)  // not Priority[concept]


```

>**Note:** The requirement in CIMPL 6.0 to use bracket notation _even when the value has only one choice_ is a change from CIMPL 5.

### Path Example

```
Entry:             Procedure
Property:          Landmark

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

This example assumes all paths begin at _Procedure_, which is an `Entry`, but the same patterns would apply if the path started elsewhere, for example, at _Landmark_.

| Path to | Path |
|---------|---------|
| _Direction_ | _Procedure.Landmark.Direction_ |
| _Laterality_ | _Procedure.Landmark.Location.Laterality_ |
| _ClockFaceDirection_ | _Procedure.Landmark.Direction_[ClockFaceDirection] |
| _Angle_ | _Procedure.Landmark.Direction_[ClockFaceDirection][Angle] |
| _Units_ | _Procedure.Landmark.Distance[Quantity].Units_ |

A reliable method to create a CIMPL path is to first view it as a _dot path_, then bracket any part of the path that is a `Value`, and finally, remove the dot before each left (open) bracket. Here's an example of that process, for the path from _Landmark_ to _Units_:

1. Find the path: _Landmark_ to _Distance_, _Distance_ to _Quantity_, _Quantity_ to _Units_.
1. Reduce to dot path: _Landmark.Distance.Quantity.Units_
1. Bracket each Value: _Landmark.Distance.[Quantity].Units_
1. Remove dot before left (open) bracket: _Landmark.Distance[Quantity].Units_

***
## Constraints

Constraints are a way to shape a general model for a particular purpose. `Property`, `Value`, and `Value Choice` can be constrained. Constraint targets can be be top-level or deeply nested (the latter referred to using [CIMPL paths](#cimpl-paths)), inherited or locally-defined.

Unlike keyword declarations, constraint statements do not begin with a keyword. Most constraint statements have three parts:

* The subject of the constraint
* A reserved word or phrase indicating the type of constraint
* The constraint value being applied

Here are several examples of this pattern:

| Example | Subject | Reserved word | Value applied |
|----------|---------|---------|---------|
|_IsPrimaryTumor = true_| _IsPrimaryTumor_ | `=` | _true_ |
|_Specimen substitute TumorSpecimen_ | _Specimen_ | `substitute` | _TumorSpecimen_ |
| _BodyLocation from BodyLocationVS_ | _BodyLocation_ | `from` | _BodyLocationVS_ |
| _LanguageUsed 0..1_ | _LanguageUsed_ | _(implicit)_ | _0..1_

Here is a summary of supported constraints:

| To do this.. | Use this Keyword or Symbol | Applies to |
|----------|---------|---------|
| [Narrow cardinality](#cardinality-constraint) | `{min}..{max}` | `Property` |
| [Assign a fixed value](#fixed-value-constraint) | `=`| `Property`` or Value[Choice]` |
| [Append a fixed value to an array](#append-constraint) | `+=` | Array `Property` |
| [Narrow a data type](#substitute-constraint) | `substitute` | `Property` or `Value[Choice]` |
| [Bind a value set](#value-set-binding-constraint) | `from` | `Property` or `Value[Choice]` |
| [Slice an array](#includes-constraint) | `includes` | Array `Property` |
| [Narrow choices for a value](#only-constraint) | `only` | `Value` |

### Cardinality Constraint

Cardinality defines the number of repeats that can exist for a `Property`. Cardinality is specified using FHIR syntax, **{min}..{max}**, where the first integer indicates the minimum repeats (0 implying optional), and the second digit expresses the upper bound on the number of repeats. To express no upper bound, an asterisk _*_ is used. When constraining cardinality, you can only narrow the previously declared cardinality. The cardinality constraint only applies to `Property`.

| Declaration | Constraint | Subsequent Constraint |
|----------|----------|----------|
| `Property`: _LanguageUsed 0..*_ | _LanguageUsed  0..1_ | _LanguageUsed  0..0_ |
| `Property`: _GovernmentIssuedID  0..*_ | _GovernmentIssuedID 1..*`|`GovernmentIssuedID 1..1_ |
| `Property`: _Status 1..1_|<del>_Status 0..1_</del> invalid|

>**Note:** By constraining a `Property` to 0..0, you effectively remove the `Property`. Use 0..0 with caution, because if an instance _does_ include that `Property`, the instance will fail validation and may be rejected.

### Fixed Value Constraint

The `=` operator fixes an `Element`'s value to a specific concept, boolean, string, integer, decimal, or URL. The fixed value must be consistent with the defined data type, and is always a Primitive.

| Example | Syntax |
|----------|---------|
| Fix a code to an `Element` | _ObservationCode = LNC#82810-3_ |
| Fix a boolean value to an `Element` | _IsPrimaryTumor = true_ |
| Fix a code to an `Element`'s `Value` | `Value[concept]` = _SCT#233613009 "Pneumonia"_ |
| Fix the units associated with a Quantity Value Choice  | `Value`_[Quantity].Units = UCUM#cm_ |

>**Notes:**
>
>1. [Bracket notation](#bracket-notation-for-value-choices) must always be used when applying a constraint to a `Value[Choice]` _even if the value has only one choice_.
>1. CIMPL allows fixed concept values to be overridden in child classes. Although this seems to violate the notion that constraints should get progressively tighter in subclasses, it is necessary when narrowing the meaning of a class characterized by a concept code. For example, the class _Pneumonia_, with fixed code, _SCT#233604007_, may have a child, _Fungal Pneumonia_, with fixed code _SCT#233613009_, the latter overriding the former. CIMPL will assume, without checking, that the code describing the child class has more constrained semantics than the code it replaces.
>1. If you only need to fix a code in one particular mapping (one version of FHIR, for example), do so using the [`fix`](#fix) keyword in the Map file and not as a constraint in the Class file.

### Append Constraint

The append `+=` operator appends a fixed concept to a `Property` array with an `Element Value` of `concept`. You can use the `+=` operation repeatedly on the same array, limited only by the cardinality of the array. The append operator can only be applied to a repeating `Property`.

For example, use `+=` to insert a specific _Category_ value into an array of categories.

| Example | Syntax |
|----------|---------|
| Insert _"Behavior"_ into _Category_ array |  _Category_ `+=` _LNC#54511-1 "Behavior"_ |
| Add _"Social History"_ to same array | _Category_ `+=` _OBSCAT#social-history "Social History"_ |

>**Note:** Appending other data types is not supported, although this feature is planned.

### Substitute Constraint

The `substitute` keyword constrains the data type of an `Element`. The new data type that is substituted for the original _must_ be a subclass of the original data type.

| Example | Syntax |
|----------|---------|
| Constrain the `Element` _Specimen_ to be a _TumorSpecimen_ | _Specimen substitute TumorSpecimen_ <br/> |
| Constrain a `Value[Choice]` of data type `Quantity` to `SimpleQuantity` | `Value`_[Quantity]_ `substitute` _SimpleQuantity_ <br/>|

### Value Set Binding Constraint

Binding is the process of associating an `Element` with a set of possible values.

Binding uses the keyword `from`. The object of a binding must be an `Element` whose `Value` is `concept`. The value set that is being bound can be a value set defined in a CIMPL Value Set file, or a canonical URL external to CIMPL.

CIMPL uses the [binding strengths defined in FHIR](https://www.hl7.org/fhir/valueset-binding-strength.html), namely:
<!-- note to reviewers - context = FHIR, reserved words are bolded not ticked -->

* **required** (strongest) indicates that the code must come from the specified value set.
* **extensible** indicates that the code must come from the specified set _if the value set contains a relevant code_; otherwise a code outside the value set may be chosen.
* **preferred** indicates that the code should ideally come from the specified value set, but codes outside the value set may also be chosen.
* **example** (weakest) indicates that the code may come from anywhere; the specified value set is for example purposes only.

The following rules apply to binding in CIMPL:
<!-- note to reviewers - context = CIMPL, reserved words are ticked not bolded  -->
* If no binding strength is specified, the binding is assumed to be `required`.
* When further constraining an existing binding, the binding strength can stay the same or be made tighter (e.g., replacing a `preferred` binding with an `extensible` or `required` binding), but never loosened.
* Constraining may leave the binding strength the same and change the value set instead. However, certain changes permitted in CIMPL may violate [FHIR profiling principles](http://hl7.org/fhir/R4/profiling.html#binding-strength). In particular, FHIR will permit a `required` value set to be replaced by another `required` value set only if the codes in the new value set are a subset of the codes in the original value set. For `extensible` bindings, the new value set can contain codes not in the existing value set, but additional codes _should not_ have the same meaning as existing codes in the base value set.

The syntax for binding is different depending on whether the binding is in an `Element` or another class (`Abstract`, `Entry`, or `Group`).

#### Value Set Binding - Element

The following syntax is allowable when binding an `Element` to a value set:
<!--Carmela - need to figure out how to italicize (value set reference & binding strength) inside code block -->

```
Value from <value set reference>

Value from <value set reference> (binding strength)

Value[choice] from <value set reference> (binding strength)
```
 
For example:
```
Value from TerminationReasonVS

Value from http://hl7.org/fhir/ValueSet/condition-clinical (preferred)

Value[AgeGroup] from AgeGroupVS (extensible)
```

>**Note:** You _must_ specify which `Value Choice` the binding applies to if the `Value` has multiple choices (the third pattern, above).

#### Value Set Binding - Entry, Abstract, or Group

The following syntax is allowable when binding a value set to Entry, Abstract, or Group:
<!-- Carmela to do Italicize substitutable text  -->
```
<Concept data type Element> from <value set reference>

<Concept data type Element> from <value set reference> (binding strength)
```

For example:

```
ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical

ReasonCode from ReasonCodeVS (extensible)

BodyLocation from BodyLocationVS (preferred)
```

### Includes Constraint

The `includes` constraint is used to specify that a repeating `Property` (array) should contain a certain class or classes. FHIR refers to this as [**slicing**](https://www.hl7.org/fhir/profiling.html#slicing). In CIMPL, [slicing](#slicing) is accomplished by declaring that an array `includes` certain types of components.

The `includes` statement has several requirements:
<!--  Carmela - something not quite right in 2nd bullet -->
* The classes included in the array must conform to the data type of the array. For example, if the array is _Identifier 0..*_, the classes included in the array must inherit from _Identifier_.
* Cardinality of each inclusion must be specified, and must fit the cardinality of the array. For example, if the array `Property` has a finite maximum cardinality, you cannot include an `Element` with cardinality 0..*.
* The keyword `includes` is repeated for each item the array should contain.
* The `includes` constraint does not apply to `Element Value`.

Here is an example:

```
Property: Measurement 0..*  // the array property
...

Measurement
includes TumorLength 1..1
includes TumorWidth 0..1
includes TumorDepth 0..1
```

>**Note:** _TumorLength_, _TumorWidth_, and _TumorDepth_ all must inherit from _Measurement_, because the members of the _Measurement_ array must be _Measurements_.

>**Note:** When using `includes`, special mapping syntax is required. See [Slicing](#slicing).

### Only Constraint

`Element Value` can have multiple data types. Use the `only` constraint in an `Element` to narrow the choice of data types. The `only` constraint can be applied to a `Value` that has multiple choices. The grammar of `only` constraints varies depending on whether the constraint appears in an `Element`, or another class type.
<!-- Carmela - figure out what happened to the <datatype> info -->
| In the class... | Use the grammar... |
|----------|---------|
| `Element` | `Value only` <datatype>  |
| `Element` | `Value only` <datatype1> or <datatype2> or <datatype3> etc.  |
| `Entry` or `Group` | `<Property> only <datatype>` <br> >**Note**: this is allowed only for a single data type; to constrain to multiple choices, use the `substitute` constraint (see Example 2, below)|

**Example 1:** Multiple `Value Choice` narrowed to a single choice

```
Element: Priority
Value: integer or concept

Element: IntegerPriority
Parent: Priority
Value only integer

// Both of the following are legal and have the same effect:

Group:  Alarm-One
Property: Priority
Priority only integer

Group:  Alarm-Two
Property: Priority
Priority substitute IntegerPriority
```

**Example 2:** Multiple value choices narrowed to a smaller number of choices
<!-- Carmela fix the last comment -->
```
Element:  Answer
Description:  "A potential answer with many different allowed data types."
Value:  integer or date or time or string or concept // example of many choices

Element: TemporalAnswer
Parent:  Answer
Description: "An answer that's either a date or time"
Value only date or time   // valid example

Group: QuestionAndAnswer
Property: Question // definition not included in example
Property: Answer

Group: QuestionWithTemporalAnswer
Parent: QuestionAndAnswer
Description: "A question whose answer is a date or a time"
Answer substitute TemporalAnswer  
// _Do not_ use one of these patterns "Answer only date or time" or "Answer only TemporalAnswer"
```
***

## Class File

Class files contain definitions of the information model. Every model defined in CIMPL must have at least one Class file.

### Class File Example

Here is an abbreviated example of a class file:

```
Grammar:           DataElement 6.0
Namespace:         obf.lab
Description:       "Profiles of laboratory tests and panels."
Uses:              datatype, obf
CodeSystem:        LNC = http://loinc.org
CodeSystem:        UCUM = http://unitsofmeasure.org

Abstract:          SimpleQuantLabWithRequiredUnits
Parent:            SimpleQuantitativeLaboratoryObservation
Description:       "Laboratory tests that must be reported with units."
                   DataValue 1..1
                   DataValue only QuantityWithRequiredUnits
                   DataAbsentReason 0..0

Group:             QuantityWithRequiredUnits
Parent:            Quantity
Description:       "A quantity with that requires a coded unit of measure."
                   Units 1..1

Entry:             GlobulinMCncPtSerQnCalculatedLabObs
Parent:            SimpleQuantLabWithRequiredUnits
Description:       "Globulin [Mass/​volume] in Serum by calculation"
                   Code = LNC#10834-0 "Globulin [Mass/​volume] in Serum by calculation"
                   DataValue[Quantity].Units from EquivalentUnitsVS-g-per-L (required)

```

<!--
    CM: Note that the value set above (EquivalentUnitsVS-g-per-L) violates the naming convention this reference guide encourages.  I left it as-is since this is how the
    current CIMPL source really is, but we should consider changing to EquivalentUnits-g-per-L-VS (or something like that).
-->

### Class File Header

The elements of the Class file header are (in order):

1. `Grammar` (required)
1. `Namespace` (required)
1. `Description` (recommended; one per namespace)
1. `Uses` (optional; as needed)
1. `CodeSystem` (optional; as needed)
1. `Path` (optional)

For a further description, see [Keywords](#keywords).

>**Note:** [`Uses`](#Uses) imports namespaces whose classes can be used as if they were locally defined. However, if a class name in an imported namespace collides with a local class name, you must refer to that class by its [fully qualified name](#fully-qualified-names).

### Class Definitions

Following the header, the Class file contains class definitions. The order of the class definitions does not matter. Each class definition consists of declarations. Follow the links for further explanation of each item:

1. [Class Type and Name Declaration](#class-type-and-name-declaration) (required, must be first)
1. [Parent Declaration](#parent) (optional)
1. [Concept Declaration](#concept) (optional)
1. [Description](#description) (optional but highly recommended)
1. [Property Declarations](#property) (for Entry, Abstract and Group), or
1. [Value Declaration](#value) (for Element) (optional, but typically present)
1. [Constraint statements](#constraints) (optional)

### Class Type and Name Declaration

The first line of all class definitions is a keyword representing the type of class, followed by a descriptive name you choose. Class names must be unique within a given namespace. CIMPL provides four class types:

| Class | Description | Has... | Analogous FHIR Type |
|----------|---------|---------|---------|
| `Element` | The lowest-level class, representing a property-value pair. | `Value` | **Element** or **simple extension** |
| `Group` | A class comprised of other classes, specifically, other `Group`, `Element`, and `Entry`. | `Property` | **Backbone element** or **complex extension** |
| `Entry`  | A class representing a group of related information, complete enough to support stand-alone interpretation. | `Property` | **Resource** or **Profile** |
| `Abstract` | A special type of `Entry` that cannot be instantiated, and will not be present in the target mapping. | `Property` | none |

### Properties and Values

When defining classes, bear in mind that:

* An `Element` has exactly one `Value` field and no `Property` fields.
* The `Value` field may have more than one `Choice` (using `or`).
* A `Property` cannot have `Choice`.
* `Group`, `Entry`, or `Abstract` may have multiple `Property` fields, but no `Value` fields.

These differences are illustrated in this example:

```
Group:             Landmark
Description:       "A point on the body that helps determine a body location."
Property:          Location 0..1
Property:          Direction 0..1
Property:          Distance 0..1
```

```
Element:           Direction
Description:       "The direction from the landmark to the body location of interest."
Value:             concept from AnatomicalDirectionVS (preferred)  
```

Summary:

| Keyword | Appears with | How Many Times? | Has Cardinality? | Has Choices? | Can be a... |
|----------|---------|---------|---------|---------|---------|
| `Value` | `Element` | exactly once | no | yes (optionally) | `primitive`, `Element`, `Group`, or `Entry` |
| `Property` | `Group`, `Entry`, or `Abstract` | zero or more | yes | no | `Element`, `Group`, or `Entry` |

### Naming Recommendations

All classes in CIMPL, from `Element` to `Entry`, are reusable outside of their original context. To promote reuse, try to choose names that are self-explanatory and context-independent. For example, a `Property` named _Text_ is vague, whereas _DisplayText_ and _InstructionText_ are more self-explanatory and may be completely understandable when placed into a specific context. However, an overly-specific name like _AddressDisplayText_ could inhibit reuse because it affixes the _context_ of an address to the _concept_ of a display text.
<!-- Carmela start here -->
***
## Value Set File

Value set files are used to define custom value sets and codes when existing value set sources like [HL7 v3](https://www.hl7.org/fhir/terminologies-v3.html), [FHIR](https://www.hl7.org/fhir/terminologies-systems.html), [VSAC](https://vsac.nlm.nih.gov/), or [PHIN VADS](https://phinvads.cdc.gov/) are insufficient, and a new value set must be defined.

### Value Set File Example

A (truncated) example Value Set file is shown below:

```
Grammar:      ValueSet 5.1
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

### Value Set File Header

`Grammar` and `Namespace` are required keywords in a Value Set file header. A `CodeSystem` declaration is required for every Code System referenced in a value set defined in the Value Set file. For a further description, see [Code System Keyword](#CodeSystem).


### Value Set Definition

Following the header, the Value Set file contains value set definitions. The order of definitions does not matter. Each definition follows a set sequence of declarations. Follow the links for further explanation of each item:

1. [Value Set Declaration](#ValueSet) (required)
1. [Description](#description) (optional but highly recommended)
1. [Code Declarations](#code-declaration) (optional)
1. [Includes Codes statements](#includes-codes) (optional)

### Explicit Codes

Explicit code declarations are used to add specific codes to a value set. This method of defining a value set is sometimes called *extensional*. Codes can either be locally-defined, or selected from external code systems, such as `ICD10CM#C7B00`, in which `C7B00` is an existing code within the `ICD10CM` code system. If the code is defined locally, the value set name serves as the code system. For the syntax of concept codes, see [Concept Codes](#concept-codes). 

| Type | Example |
|---------|---------|
| Locally-defined code | `#proposed "The proposal has been proposed, but not accepted or rejected."` |
| Externally-defined code | `ICD10CM#C7B00 "Secondary carcinoid tumors, unspecified site"`|

#### Locally Defined Value Set 

Within a custom value set (one specific to an IG), the individual codes (denoted by # symbol) are conventionally written in lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case). For example:
```
ValueSet:          HomeEnvironmentRiskVS
#smoke_detectors   "No Smoke Detectors"
#radiation         "Radon or Other Radiation Source"
#swimming_pool     "Swimming Pool"
```

### Implicit Codes

 Implicit code declarations add codes to a value set using an expression rather than a list. This allows you to, for example, add all codes from a code system, add all codes descending from a point in a hierarchy (currently only supported in SNOMED-CT). This method of defining a value set is sometimes called *intensional*. There are three variants involving the `Includes codes` phrase:

| Phrase | Example | Result|
|----------|---------|---------|
| `Includes codes from` | `Includes codes from MDR`| Value set contains all codes from MDR code system |
| `Includes codes descending from` | `Includes codes descending from SCT#105590001` | Value set contains SCT#1055900001 and all codes below it in the SNOMED-CT hierarchy |
|  `and not descending from` | `Includes codes descending from SCT#363346000 "Malignant neoplastic disease" and not descending from SCT#128462008  "Secondary malignant neoplastic disease"` | Value set contains SCT#363346000 and all codes below it in the SNOMED-CT hierarchy, _except_ code SCT#128462008 and all codes below it  |

## Map File

Map files control how classes defined in CIMPL are expressed as FHIR profiles. Map files are FHIR-version dependent. You can map to FHIR DSTU 2, STU 3, R4. If you use the supplied [Objective FHIR base classes](cimpl6ObjectiveFHIR.md), you many not need to create a map file.

### Inheritance of Mapping Rules

Maps are defined in a series of mapping rules. A powerful feature of CIMPL is that mapping rules are inherited. If your class inherits from a parent class that already is mapped, you do not need to create a new map for your class. However, if you want to specify mappings for a new `Property`, or override the inherited map, you may do so.

The inheritance of mapping rules follows the class hierarchy. If there is no map for a class or attribute, CIMPL tooling automatically looks to the parent class to try and find a map. The first mapping rule found will be applied. If no mapping rule is found for a `Property` defined in CIMPL, a FHIR extension will be created. If a map is not found for an `Entry`, that `Entry` will be mapped to the [Basic resource](https://www.hl7.org/fhir/basic.html). In general, mapping to Basic is not recommended because it has no inherent semantic meaning for implementers.

### Mapping CIMPL Primitives to FHIR

CIMPL follows somewhat flexible rules on how CIMPL primitives (data types) map to FHIR. For example, a unsignedInt in CIMPL can map to an integer data type in FHIR (since every unsignedInt is an integer, there is no loss of information). However, maps that lose information, such as integer in CIMPL to unsignedInt in FHIR, generally will trigger mapping errors. An exception is that CIMPL `concept` is allowed to map to FHIR `code`. The following table shows the acceptable mappings between CIMPL and FHIR:

| CIMPL Primitive | Can map to FHIR... |
|----------|---------|
| concept | code, Coding, CodableConcept |
| boolean |boolean, code |
| integer | integer, Quantity |
| string | string |
| decimal | decimal |
| uri | uri |
| base64Binary | base64Binary |
| instant | instant |
| date | date |
| dateTime | dateTime, date, instant |
| time | time |
| oid | oid |
| id | id |
| markdown | markdown, string |
| unsignedInt | unsignedInt, integer, Quantity |
| positiveInt | positiveInt, unsignedInt, integer, Quantity |
| xtml | xtml |

### Map File Example

The following example demonstrates how the class Dosage, defined in the OBF namespace, maps to the FHIR 4.0 data type with the same name. The second example demonstrates mapping between the OBF MedicationStatement class and the FHIR 4.0 US Core MedicationStatement profile.

```
Grammar:    Map 5.1
Namespace:  obf
Target:     FHIR_R4

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
    MaxDosePerPeriod maps to maxDosePerPeriod
    MaxDosePerAdministration maps to maxDosePerAdministration
    MaxDosePerLifetime maps to maxDosePerLifetime
    constrain doseAndRate to 0..1
    DoseAndRate.DoseAmount maps to doseAndRate.dose[x]
    DoseAndRate.DoseRate maps to doseAndRate.rate[x]

MedicationStatement maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-medicationstatement:
    Identifier maps to identifier
    MedicationBasedOn maps to basedOn
    MedicationStatementPartOf maps to partOf
    Status maps to status
    StatusReason maps to statusReason
    Category maps to category
    MedicationCodeOrReference maps to medication[x]
    SubjectOfRecord maps to subject
    CareContext maps to context
    OccurrenceTimeOrPeriod maps to effective[x]
    StatementDateTime maps to dateAsserted
    MedicationInformationSource maps to informationSource
    SupportingInformation maps to derivedFrom
    ReasonCode maps to reasonCode
    MedicationReasonReference maps to reasonReference
    Annotation maps to note
    Dosage maps to dosage
```

### Map File Header

The header of a map file _must_ include the `Grammar`, `Namespace`, and `Target` keywords. The `Grammar` keyword declares the CIMPL Mapping Grammar Version. The version is currently 5.1. The `Namespace` keyword relates the maps defined in this file to the Class file in the same namespace. The `Target` keyword provides the version of FHIR that the CIMPL model is mapped to, either `FHIR_DSTU_2`, `FHIR_STU_3`, or `FHIR_R4`.

For more information about the header keywords, see [Keywords](#keywords).

### Setting the Mapping Target
<!-- Carmela - fix the capitalization in the table -->
Each section in a Map file begins with a statement that establishes a resource, profile, or complex data type that is the target for the mapping. In CIMPL Map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in the target output. The mapping target can be a FHIR resource, profile, or complex data type that is part of the target FHIR release version. The initial statement in each mapping block must end with a colon (:).

| CIMPL Class | FHIR Target | Example |
|----------|---------|---------|
| Entry | Resource | `SocialHistoryObservation maps to Observation:`|
| Entry | Profile | `LaboratoryObservation maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-observationresults:` |
| Group | Complex Datatype | `Timing maps to Timing:` |

### Mapping Properties

After the first line, which establishes the class to be mapped, the remainder of the section consists of statements that map CIMPL properties to FHIR elements. Each statement begins with a [CIMPL path](#cimpl-paths), followed by the phrase `maps to`, and ending with the target FHIR path.

With FHIR as the target output, the right-hand side of the map statement can point to any element of a FHIR resource, including:
<!-- Carmela - need help - how is Structure Definition a code block FHIR stuff - could be bolded - lets try that - reaction.substance, -->
* Elements that are at the root of the resource (e.g., `maritalStatus`)
* Elements that are child elements of root elements on the resource (e.g., `reaction.substance`)
* Elements that have a [choice of data types](https://www.hl7.org/fhir/formats.html#choice) (e.g., `multipleBirth[x].boolean`)
* Elements that have been defined in existing profiles or other `StructureDefinition` resources (e.g., `http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity`)
* Elements that support FHIR Reference() types.
<!-- Carmela, MK suggestion, but what is a simple element?  not a path - think about it  -->
| CIMPL type | FHIR type | Example |
|----------|---------|---------|
| simple property | simple element | `MaritalStatus maps to maritalStatus` |
| simple property | extension | `Ethnicity maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity` |
| dotted path | simple element |`Value.ActiveFlag maps to active` |
| dotted path | dotted path | `AdverseReaction.AllergenIrritant maps to reaction.substance` |
| value choice path | simple element | `BodySiteOrCode.concept maps to bodySite` |

>**Note:** The mapping grammar is still at version 5.1, so the bracketed value syntax cannot be used in paths.  This is why the above example uses `BodySiteOrCode.concept` rather than `BodySiteOrCode[concept]`.

### Mapping to References

FHIR resources and profiles require an explicit `Reference()` indicator when referring to another resource or profile. In CIMPL, whenever a property is an Entry, that property is mapped as a reference in FHIR. There is no need to create an explicit `Reference()`.

Example:

| CIMPL Definition | Corresponding FHIR |
|----------|---------|
| Property:  Encounter 0..1 | encounter:    0..1  Reference(Encounter) |

### Placement of Extensions

As mentioned above, if no mapping rule is found for a property, an extension to the FHIR resource or profile will be automatically created. By default, the extension will be a top-level element in the FHIR profile. Sometimes you may want the extension to appear in a different place, perhaps inside a nested element. To put the extension precisely where you want it, you can map to FHIR paths that include `extension`.

Here's an example involving the FHIR resource [AdverseEvent](https://www.hl7.org/fhir/adverseevent.html):

`PotentialCause.CauseCategory maps to suspectEntity.extension`

This mapping rule will create an extension named `CauseCategory` for AdverseEvent.suspectEntity. Rules such as:

`DetectionTime maps to extension`

leave the default behavior unchanged, and are not required. However, they can be used to document the _intent_ to create an extension.

### Special Mapping Statements

#### fix

`fix` allows FHIR fields to be assigned a value. This constrains the FHIR resource, not the CIMPL class. This enables you to fix a value to an element in a FHIR resource without having to change the class definition, which would affect all potential mappings. This also allows you to fix values to elements in a FHIR resource  that have no corresponding property in the CIMPL definition.
<!-- Carmela - find out if status is the name of an element in a resource, or what - we need context here element in a resource yes this will become bolded -->

Example:

`fix status to #completed`

#### constrain

`constrain` allows users to constrain the cardinality of FHIR elements. This constrains the FHIR resource, not the CIMPL class. The cardinality is expressed in {min}..{max} form.

| Keyword | Example |
|----------|---------|
| `constrain` | `constrain explanation to 0..1` |

### Slicing

_Slicing_ involves specifying the data type of Elements that can be contained in an array. An array is a Property whose upper cardinality is greater than one. In CIMPL, the [includes constraint](#includes-constraint) creates slices. This section discusses how slices defined in the Class file are mapped to FHIR.

See the FHIR documentation for more detail about [slicing](https://www.hl7.org/fhir/profiling.html#slicing).

#### Slicing Rules
<!-- Carmela - capital E for element on 2nd line becomes a tick and stay capitalized -->
Slicing is used to differentiate occurrences of an Element in an instance, modeling it as a sub-list. An example of this is the [FHIR Blood Pressure profile](https://www.hl7.org/fhir/profiling.html#slicing), which is an Observation with two Components, identical in structure other than their represented code.

In CIMPL, slicing rules are specified in the Map file.  Four reserved phrases are used in conjunction to specify the slice:

* `slice on` (required)
* `slice on type` (optional)
* `slice strategy` (optional)
* `slice at` (optional)

#### Declare a Discriminator

Discriminators are used to indicate how each slice is distinguished from the others. FHIR requires discriminators to uniquely identify the slice to which an instance of data belongs.

##### Slice On

Declare a discriminator path to perform slicing, using `slice on` followed by the discriminator path. This is the path to the property whose data identifies the `slice on` instance belongs to. For example, if `slice on = code.coding.code`, the code in each slice must be a unique, fixed value, and every instance must have a code that matches one of the defined slices.

For more explanation of how discriminators are defined in FHIR, see [discriminators](https://www.hl7.org/fhir/profiling.html#discriminator).

##### Discriminator Type
<!-- Carmela - we slice a property not an element check the text  -->
The discriminator type defines how a slice is differentiated from another slice for the same Element. By default, CIMPL uses the `value` discriminator type. However, a different discriminator type can be declared using `slice on type` followed by one of the following FHIR-defined discriminator types:
<!-- Carmela is the first row (and others) a reference to the CIMPL Element class? Should the second row be sliced on Element? We slice a property, but we SLICE ON something about the property - discriminator is both a FHIR word and a CIMPL - so bold it Column 1 all ticks and slice on type is a tick word -->
| Slice on type | Definition |
| ------------- | ---------- |
| value| The slices have different values in the discriminator |
| exists | The slices are differentiated by the presence or absence of the _sliced on_|
| pattern |The slices have different values in the discriminator, as determined by testing them against the applicable ElementDefinition.pattern[x]|
| type |The slices are differentiated by data type of the discriminator to a specified profile|
| profile |Not recommended. The slices are differentiated by conformance of the discriminator to a specified profile. This can only be used when the discriminator is an `Entry`.|

#### Slice Strategy
<!-- confirm underscore with slice strategy check the code -->

The `slice strategy` indicates how the CIMPL Property should map to slices in the target FHIR property. The default `slice strategy` adds a single slice, corresponding to the CIMPL type, in the target FHIR array.  To map a CIMPL property with `includes` constraints such that each included CIMPL data type becomes a slice in the target FHIR array, specify `slice strategy = includes`.

| Reserved Phrase | Example |
|----------|---------|
| `slice strategy` | `Components.ObservationComponent maps to component (slice on = code.coding.code; slice strategy = includes)` |

In the above example, each included `ObservationComponent` subtype will create a new slice in the FHIR `component` array.  Each slice is distinguished by the value at the path `component.code.coding.code`.

#### Moving the Slice Location (slice at)

If the discriminator used for slicing is not the actual field that is being mapped to, `slice at` can allow you to explicitly define the location of the slice.

| Reserved Phrase | Example |
|----------|---------|
| `slice at` | `Members.Observation maps to related.target (slice at = related; slice on = type)` |

In the above example, though the mapping is declared such that `Members.Observation maps to related.target`, the slicing is not occurring at `related.target`.  Instead, the `slice at = related` ensures that the slicing is occurring at the `related` element.

#### Slicing Examples

| Phrase | Example |
|:----------|:---------|
| `slice on` / `slice strategy` | `Components.ObservationComponent maps to component (slice on = code.coding.code; slice strategy = includes)` | 
| `slice on` / `slice strategy` | `Members.Observation maps to related.target (slice at = related; slice on = target.reference.resolve(); slice on type = profile; slice strategy = includes)` |
| `slice on` | `Laterality maps to qualifier (slice on = concept)` |
