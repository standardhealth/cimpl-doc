# CIMPL Language Reference Manual

## Table of Contents

[TOC]

***

## Introduction

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles, extensions and implementation guides (IG). Because it is a _language_, written in text statements, CIMPL encourages distributed, team-based development using conventional source code control tools such as Github. CIMPL provides tooling that enables you to define a model once, and publish that model to multiple versions of FHIR.

### Purpose

This document describes the components and syntax of the CIMPL language, Version 6.0 and above, used to specify clinical models, value sets, and maps to FHIR.

### Intended Audience

The CIMPL Language Reference is targeted to the users of CIMPL, involved with creating or reviewing models expressed in the CIMPL language.

### Prerequisite

This guide assumes you have:

* Installed the latest version of the SHR-CLI software as documented in [CIMPL Setup and Installation](cimplInstall.md) (preferably installed in the `~/cimpl/shr-cli` directory).
* A text editor to create your CIMPL files (preferably VSCode with the _vs-code-language-cimpl_ extension, but not required).
* Reviewed the [In-Depth Tutorial](cimpl6Tutorial_detail.md).

***
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

***
## Naming Conventions

### File Names

File names must begin with lower case and typically include the namespace. The conventions for class, value set, and map files are (bold indicates where you substitute your information):

* Class files: _**namespace**.txt_ or _**namespace**-**subtopic**.txt_
* Value Set files: _**namespace**-vs.txt_ or _**namespace**-**subtopic**-vs.txt_
* Map files: _**namespace**-map.txt_ or _**namespace**-**subtopic**-map.txt_

Any periods in the _**namespace**_ should be replaced by dashes. For example, for a namespace called _onco.core_, the recommended value set file name is _onco-core-vs.txt_. The _subtopic_ is useful when you want to break up the contents of a single namespace into multiple files, for example, in the namespace _obf_:

* _obf-action.txt_
* _obf-finding.txt_

>**Note:** _obf_ stands for _Objective FHIR_, a set of classes provided with CIMPL that can help you create FHIR profiles. For more information, see [Objective FHIR Overview](cimpl6ObjectiveFHIR.md)

Additional files for configuring and producing FHIR implementation guides (IGs), and their naming conventions include:

* Configuration files: _ig-**myigname**-config.json_
* Content profile files: _ig-**myigname**-cp.json_
* IG examples: _igname-**myexamplename**.json_
* IG front matter pages: _**anyName**.html_

Detail about these files and how they are used to compile CIMPL models and create FHIR IGs can be found in the [CIMPL Tooling Reference Manual](#cimpl6ToolingReference.md).

### Namespace Names

Best practice is to follow the naming convention pattern _project_ or _project.domain_ (followed by subdomains if necessary).

| Example | Purpose |
|-------|-------|
| `obf.lab` | Laboratory profiles belonging to Objective FHIR |
| `odh` | Occupational Data for Health |
| `onco.core` | Core oncology profiles |

>**Note:** While it is best practice to use only lower case letters, the strict requirements only require the word to begin with a lowercase letter. The rest of the namespace allows for mixed casing and hyphens.

### Class Names

CIMPL classes are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase), for example, `Dosage`, `DetectionTime`, and `Neutrophils100WBCNFrPtBldQnAutoCntLabObs`.

>**Note:** Letters, numbers, and hyphens are allowed. However, class names must begin with an uppercase letter.

### Fully Qualified Names

CIMPL does not support duplicate class names within a single namespace. However, the same name may occur in different namespaces. In the case of naming collisions due to an namespace included by a [`Uses`](#Uses) statement, you have to refer to the class by its fully qualified name (FQN). The FQN is a concatenation of the namespace name and the class name (delimited by a period). For example, the class `CancerCondition` in the namespace `onco.core` has the FQN `onco.core.CancerCondition`.

FQNs are only required in case of naming collisions between namespaces.

### Value Set Names

Value Set names are by convention written in [PascalCase](http://wiki.c2.com/?PascalCase), with a suffix of "VS", for example, `HomeEnvironmentRiskVS` and `LandmarkTypeVS`.

>**Note:** Value Set names must begin with an uppercase letter.

### Local Code Names

Within a locally defined value set, the individual codes (denoted by `#`) are by convention written in lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case) with an underscore between words, for example, `#no_smoke_detectors`, `#radiation`, and `#swimming_pool`.

***

## Language Basics

### Whitespace

Repeated whitespace is not meaningful within CIMPL files. This:

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

Primitives are data types, distinguished by starting with a lower case letter. CIMPL defines the following primitive data types. With the exception of `concept` (a simplified representation of **code**, **Coding**, and **CodeableConcept**), the primitive types in CIMPL align with FHIR:

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
* [`xhtml`](https://www.hl7.org/fhir/narrative.html#xhtml)

>**Note:** Unlike FHIR, CIMPL does not have an explicit [**Reference**](https://www.hl7.org/fhir/references.html) data type. References are determined during the [mapping process](#mapping-to-references).

### Concept Codes

CIMPL uses a single primitive type, `concept` to represent coded terms from controlled vocabularies. A concept is comprised of code system, code, and optional display text. The grammar for specifying concepts follows the pattern:

<code><i>SYSTEM</i>#<i>code</i> "<i>Display text</i>"</code>

Examples:

`SCT#363346000 "Malignant neoplastic disease (disorder)"`

`ICD10CM#C004  "Malignant neoplasm of lower lip, inner aspect"`

_SYSTEM_ (`SCT` and `ICD10CM` in the examples) is an alias for a canonical URI that represents a code system. Aliases must be declared in the file header using the keyword [`CodeSystem`](#codesystem). You cannot use the canonical URI directly in the concept grammar.

Unlike FHIR, CIMPL does not differentiate between **code**, **Coding**, and **CodeableConcept**. See [Mapping Concept Codes](#mapping-concept-codes) for information on how CIMPL maps `concept` to these FHIR data types.

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
| [`Element`](#element) | Declare lowest-level model building block  | Class |
| [`Entry`](#entry) | Declare a stand-alone class | Class |
| [`Grammar`](#grammar) | Define how a file should be parsed | All |
| [`Group`](#group) | Declare mid-level, non-stand-alone model building block  | Class |
| [`Namespace`](#namespace) | Associate a file with a namespace | All |
| [`Parent`](#parent) | Specify inheritance | Class |
| [`Path`](#path) | Alias long URLs | Class, Value Set |
| [`Property`](#property) | Define an attribute in a class | Class |
| [`Uses`](#uses) | Import namespace(s) | Class |
| [`Value`](#value) | Define data type(s) of an `Element` | Class |
| [`ValueSet`](#valueset) | Define a value set | Value Set |

### Abstract
<!-- MK note - there aren't any details about `abstract` anywhere - thoughts? -->

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

Example:

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

The `Entry` keyword is used to declare a new class, analogous to a resource or profile in FHIR. For details, see [Class File](#class-file).

Example:

`Entry: GenomicsReport`

### Grammar

The `Grammar` keyword defines how the file is to be parsed. The grammar declaration must be the first line in each CIMPL file.

| File Type | Required First Line | Current Version # |
|----------|---------|---------|
| Class | <code>Grammar: DataElement <i>#.#</i></code> | 6.0 |
| Value Set | <code>Grammar: ValueSet <i>#.#</i></code> | 5.1 |
| Map | <code>Grammar: Map <i>#.#</i></code> | 5.1 |

>**Note:** no space in `ValueSet`

The version details the _major_ and _minor_ version of the grammar the file conforms to. Generally, the major version of the grammar matches the major version number of CIMPL, however, the ValueSet and Map grammar remain at 5.1.

### Group

The `Group` keyword is used to declare a reusable collection of one or more properties, similar to the [complex data types](https://www.hl7.org/fhir/datatypes.html#complex) in FHIR. For details on using the `Group` keyword, see [Class File](#class-file).

Example:

`Group: Dosage`

### Namespace

The `Namespace` keyword defines the association of the file with a namespace, and is required in `Class`, `ValueSet` and `Map` files. The namespace name should follow [naming conventions](#namespace-names) discussed earlier.

Examples:

`Namespace: odh`

`Namespace: onco.core`

### Parent

Through the `Parent` keyword, CIMPL provides a mechanism for deriving a class from another class. The child class inherits all properties and constraints from the parent. The child class can then add additional properties and constraints. Additionally, maps are inherited alongside properties. At most one parent class can be specified. The `Parent` declaration is optional.

There are restrictions on which classes can inherit from other classes. The rules can be summarized as _like inherits from like_:

| Building Block| Can Inherit From |
|----------|---------|
| `Element` | `Element` |
| `Group` | `Group` |
| `Entry` | `Abstract` or `Entry` |
| `Abstract` | `Abstract` |

Note that `Abstract` is _like_ an `Entry`, except that `Abstract` cannot be instantiated.

Examples:

```
Entry:     BodyWeight
Parent:    VitalSign
```

```
Group:     Age
Parent:    Quantity
```

```
Element:   EncounterParticipant
Parent:    Participant
```

### Path

The `Path` keyword allows for abbreviations of long URLs. This functionality is optional; it is provided to make authoring easier. By convention, the alias should be UPPERCASE, although the tooling allows for numbers and hyphens after the first letter.

Example:

`Path: FHIRVS = http://hl7.org/fhir/ValueSet`

### Property

Classes based on `Group`, `Entry`, and `Abstract` are composed of one or more properties (called _fields_ or _attributes_ in object-oriented programming). A property can be any `Element`, `Group`, or `Entry`, but never a primitive data type. Each `Property` must have a specified cardinality range, represented as `min..max`, indicating the number of repeats of the `Property`. The `Property` keyword cannot be used in the definition of an `Element`.

In the following example, `StudyArm` has four properties: `Name`, `Type`, `Comment` and `ResearchStudy`. `Name` and `ResearchStudy` are singular and required, `Type` is optional and repeating, and `Comment` is singular and optional:

```
Group:             StudyArm
Description:       "Refers to participant(s) in a clinical trial assigned to receive specific interventions according to a protocol."
Property:          Name 1..1
Property:          Type 0..*
Property:          Comment 0..1
Property:          ResearchStudy 1..1
```

A property that refers to an `Entry` (such as `ResearchStudy`, above) is implicitly a reference (pointer) to an instance of that `Entry`, rather than implying the `Entry` is _in-lined_ into the class.

### Uses

The `Uses` keyword appears in the Class file and provides a comma-separated list of the namespaces imported to the current namespace. This keyword allows you to use the classes and value sets defined in other namespaces. The list order has no effect.

Examples:

`Uses: obf.datatype`

`Uses: obf.datatype, obf, onco.core`

<!-- Carmela: TODO: VERIFY THIS: In CIMPL version 6, it is *required* that you import `shr.core` for the tooling to run.-->

### Value

`Value` represents the data type(s) an `Element` can accept. This keyword can only be used when defining an `Element`. Each `Element` must have exactly one `Value`, although the `Value` itself can be a choice of several data types. A `Value` choice can be a [primitive](#primitives), an [Element](#element), a [Group](#group), or an [Entry](#entry). A `Value` may be inherited and constrained in the child class.

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

Most types of constraints cannot be applied _in line_ in a `Value:` statement, but you are allowed to bind a value set to a `concept` value choice at the same time the `Value` is defined, as follows:

````
Element:           IsPrimaryTumor
Description:       "Whether the tumor is the original or first tumor in the body, for a particular cancer."
Value:             concept from YesNoUnknownVS (required)
````
For more information on binding, see [Value Set Binding Constraint](#value-set-binding-constraint).

>**Note:** You cannot assign a cardinality to a `Value`. Cardinalities can only be introduced in `Property` declarations.

### ValueSet

Defines the name of a value set inside CIMPL value set files. For details on the use of this statement, see [Value Set File](#value-set-file).

Example:

`ValueSet: ProposedStatusVS`
***

## CIMPL Paths

CIMPL provides grammar that allows you to refer to any part of a CIMPL class, regardless of nesting. This section outlines CIMPL path grammar, which is used in [constraints](#constraints) and [mapping](#map-file).

### Dot Notation

Dot notation is used to denote nested properties. The path lists the properties in order, separated by a dot (.) For example, if `Patient` has a `ContactPoint` property, and `ContactPoint` has a `Priority` property, then the path is `Patient.ContactPoint.Priority`.

### Entry Terminates Path

An `Entry` can appear in a path only as its _final component_. Paths represent nested containment within one class, and `Entry` is an independent object. Paths cannot hop from one `Entry` to another.

For example, if `Patient` is an `Entry` and `Procedure` has a `Patient` property, and `Patient` has a `Gender` property, then `Procedure.Patient` is a valid path, but `Procedure.Patient.Gender` is not.  The path `Procedure.Patient` can be used to set the cardinality of `Patient` within the `Procedure` class, but `Procedure.Patieht.Gender` would (illegally) constrain an attribute within `Patient` from inside `Procedure`.

### Bracket Notation for Value Choices

The `Value` in an `Element` can offer a [choice of data types](#value). Bracket notation is used to refer to a particular value choice. Bracket notation must always be used when an `Element` is part of a CIMPL path. How `Value` choices appear in paths depends on whether the path appears with an `Element` definition or not.

| Inside an: | Refer to the data type choices as: |
|----------|---------|
| `Element` | <code>Value[<i>datatype</i>]</code> |
| `Entry` or `Group` | <code><i>ElementName</i>[<i>datatype</i>]</code> |

Example:

```
Element: Priority
Value: integer or concept
Value[concept] from PriorityVS (required)  // not Priority[concept]

Entry: Procedure
Property: Priority 0..1
Priority[concept] = SCT#394849002 "High Priority" // not Value[concept]
```

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

This example assumes all paths begin at `Procedure`, which is an `Entry`, but the same patterns would apply if the path started elsewhere, for example, at `Landmark`.

| Path to | Path |
|---------|---------|
| `Direction` | `Procedure.Landmark.Direction` |
| `Laterality` | `Procedure.Landmark.Location.Laterality` |
| `ClockFaceDirection` | `Procedure.Landmark.Direction[ClockFaceDirection]` |
| `Angle` | `Procedure.Landmark.Direction[ClockFaceDirection][Angle]` |
| `Units` | `Procedure.Landmark.Distance[Quantity].Units` |

A reliable method to create a CIMPL path is to first view it as a _dot path_, then bracket any part of the path that is a `Value`, and finally, remove the dot before each left (open) bracket. Here's an example of that process, for the path from `Landmark` to `Units`:

1. Find the path: `Landmark` to `Distance`, `Distance` to `Quantity`, `Quantity` to `Units`.
1. Reduce to dot path: `Landmark.Distance.Quantity.Units`
1. Bracket each Value: `Landmark.Distance.[Quantity].Units`
1. Remove dot before left (open) bracket: `Landmark.Distance[Quantity].Units`

***
## Constraints

Constraints are a way to shape a general model for a particular purpose. `Property`, `Value` can be constrained. Constraint targets can be be top-level or deeply nested (the latter referred to using [CIMPL paths](#cimpl-paths)), inherited or locally-defined.

Unlike keyword declarations, constraint statements do not begin with a keyword. Most constraint statements have three parts:

* The subject of the constraint
* A reserved word or phrase indicating the type of constraint
* The constraint value being applied

Here are several examples of this pattern:

| Example | Subject | Reserved word | Value applied |
|----------|---------|---------|---------|
|`IsPrimaryTumor = true`| `IsPrimaryTumor` | `=` | `true` |
|`Specimen substitute TumorSpecimen` | `Specimen` | `substitute` | `TumorSpecimen` |
| `BodyLocation from BodyLocationVS` | `BodyLocation` | `from` | `BodyLocationVS` |
| `LanguageUsed 0..1` | `LanguageUsed` | _none_ | `0..1`

Here is a summary of supported constraints:

| To do this.. | Use this Keyword or Symbol | Applies to |
|----------|---------|---------|
| [Narrow cardinality](#cardinality-constraint) | `{min}..{max}` | `Property` |
| [Assign a fixed value](#fixed-value-constraint) | `=`| `Property` or <code>Value[<i>choice</i>]</code> |
| [Append a fixed value to an array](#append-constraint) | `+=` | Array `Property` |
| [Narrow a data type](#substitute-constraint) | `substitute` | `Property` or <code>Value[<i>choice</i>]</code> |
| [Bind a value set](#value-set-binding-constraint) | `from` | `Property` or <code>Value[<i>choice</i>]</code> |
| [Slice an array](#includes-constraint) | `includes` | Array `Property` |
| [Narrow choices for a value](#only-constraint) | `only` | `Value` |

### Cardinality Constraint

Cardinality defines the number of repeats that can exist for a `Property`. Cardinality is specified using FHIR syntax, {min}..{max}, where the first integer indicates the minimum repeats (0 implying optional), and the second digit expresses the upper bound on the number of repeats. To express no upper bound, an asterisk _*_ is used. When constraining cardinality, you can only narrow the previously declared cardinality. The cardinality constraint only applies to `Property`.

| Declaration | Constraint | Subsequent Constraint |
|----------|----------|----------|
| `Property: LanguageUsed 0..*` | `LanguageUsed  0..1` | `LanguageUsed  0..0` |
| `Property: GovernmentIssuedID  0..*` | `GovernmentIssuedID 1..*` | `GovernmentIssuedID 1..1` |
| `Property: Status 1..1`| <del>`Status 0..1`</del> | _invalid_|

>**Note:** By constraining a `Property` to 0..0, you effectively remove the `Property`. Use 0..0 with caution, because if an instance _does_ include that `Property`, the instance will fail validation and may be rejected.

### Fixed Value Constraint

The `=` operator fixes an `Element`'s value to a specific concept, boolean, string, integer, decimal, or URL. The fixed value must be consistent with the defined data type, and is always a primitive.

| To do this... | Example |
|----------|---------|
| Fix a code to an `Element` | `ObservationCode = LNC#82810-3` |
| Fix a boolean value to an `Element` | `IsPrimaryTumor = true` |
| Fix a string to an `Element`'s `Value` | `Value: string = "hello world"` |
| Fix a code to an `Element`'s `Value` where `concept` is one of the available data types | `Value[concept] = SCT#233613009 "Pneumonia"` |
| Fix the units associated with a `Quantity` data type choice within an Element | `Value[Quantity].Units = UCUM#cm` |
| Fix the units associated with a `Quantity` data type choice within an Entry or Group | `MyPropertyName[Quantity].Units = UCUM#cm` |

>**Notes:**
>
>1. [Bracket notation](#bracket-notation-for-value-choices) must always be used when applying a constraint to a `Value`, if that `Value` has a choice of data types. This makes it clear which of the choices is being constrained.
>1. CIMPL allows fixed concept values to be overridden in child classes. Although this seems to violate the notion that constraints should get progressively tighter in subclasses, it is necessary when narrowing the meaning of a class characterized by a concept code. For example, the class `Pneumonia`, with fixed code, `SCT#233604007`, may have a child, `Fungal Pneumonia`, with fixed code `SCT#233613009`, the latter overriding the former. CIMPL will assume, without checking, that the code describing the child class has more constrained semantics than the code it replaces.
>1. If you only need to fix a code in one particular mapping (one version of FHIR, for example), do so using the [`fix`](#fix) keyword in the Map file and not as a constraint in the Class file.

### Append Constraint

The append `+=` operator is similar to the fixed value (`=`) operator, but it applies to a repeating `Property` (upper cardinality >1). The `Property` must be an `Element` whose data type is `concept`. The operator appends a fixed `concept` to the array `Property`.  You can use the `+=` operation repeatedly on the same `Property`, only limited by the upper cardinality.

Example: Insert two values into an array of categories:

```
Property:  Category 0..*
...

Category += LNC#54511-1 "Behavior"
Category += OBSCAT#social-history "Social History"
```

>**Note:** Appending other data types is not supported.

### Substitute Constraint

The `substitute` keyword constrains the data type of an `Element`. The new data type that is substituted for the original _must_ be a subclass of the original data type.

| Example | Interpretation |
|----------|---------|
| `Specimen substitute TumorSpecimen` | Constrain `Specimen` to be a `TumorSpecimen` |
| `Value[Quantity] substitute SimpleQuantity`| Constrain a `Value`'s `Quantity` data type to `SimpleQuantity` |

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

<pre>
Value from <i>valueset</i>

Value from <i>valueset</i> (<i>binding strength</i>)

Value[<i>data type</i>] from <i>valueset</i> (<i>binding strength</i>)
</pre>

For example:

```
Value from TerminationReasonVS

Value from http://hl7.org/fhir/ValueSet/condition-clinical (preferred)

Value[AgeGroup] from AgeGroupVS (extensible)
```

>**Note:** You _must_ specify which data type the binding applies to if the `Value` has multiple choices (the third pattern, above).

#### Value Set Binding - Entry, Abstract, or Group

The following syntax is allowable when binding a value set to Entry, Abstract, or Group:

<pre>
<i>Path</i> from <i>valueset</i>

<i>Path</i> from <i>valueset</i> (<i>binding strength</i>)
</pre>

where the _Path_ must end at an `Element` whose `Value` includes a `concept` data type.
For example:

```
ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical

ReasonCode from ReasonCodeVS (extensible)

BodyLocation.Code from BloodPressureBodyLocationVS (extensible)
```

### Includes Constraint

The `includes` constraint is used to specify that a repeating `Property` (array) should contain a certain class or classes. FHIR refers to this as [**slicing**](https://www.hl7.org/fhir/profiling.html#slicing). In CIMPL, [slicing](#slicing) is accomplished by declaring that an array `includes` certain types of components.

The `includes` statement has several requirements:
<!--  Carmela - something not quite right in 2nd bullet -->
* The classes included in the array must conform to the data type of the array. For example, if the array is `Identifier 0..*`, the classes included in the array must inherit from `Identifier`.
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

>**Note:** `TumorLength`, `TumorWidth`, and `TumorDepth` all must inherit from `Measurement`, because the members of the `Measurement` array must be `Measurements`.

>**Note:** When using `includes`, special mapping syntax is required. See [Slicing](#slicing).

### Only Constraint

`Element Value` can have multiple data types. Use the `only` constraint in an `Element` to narrow the choice of data types. The `only` constraint can be applied to a `Value` that has multiple choices. The grammar of `only` constraints varies depending on whether the constraint appears in an `Element`, or another class type.
<!-- Carmela - figure out what happened to the <datatype> info -->
| In the class... | Use the grammar... |
|----------|---------|
| `Element` | <code>Value only <i>datatype</i></code>  |
| `Element` | <code>Value only <i>datatype1</i> or <i>datatype2</i> or <i>datatype3</i></code> etc.  |
| `Entry` or `Group` | <code><i>Property</i> only <i>datatype</i></code> |

>**Note**: The <code><i>Property</i> only <i>datatype</i></code> syntax is allowed only for a single data type; to constrain to multiple choices, use the `substitute` constraint (see Example 2, below)

**Example 1:** Multiple data types narrowed to a single data type

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

**Example 2:** Multiple data types narrowed to a smaller number of choices
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
// Do not use one of these patterns: "Answer only date or time" or "Answer only TemporalAnswer"
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

1. [Class Type and Name](#class-type-and-name-declaration) (required, must be first)
1. [`Parent`](#parent) declaration (optional)
1. [`Concept`](#concept) declaration (optional)
1. [`Description`](#description) (optional but highly recommended)
1. [`Property`](#property) declarations (for Entry, Abstract and Group), or
1. [`Value`](#value) declaration (for Element) (optional, but typically present)
1. [Constraint statements](#constraints) (optional)

### Class Type and Name Declaration

The first line of all class definitions is a keyword representing the type of class, followed by a descriptive name you choose. Class names must be unique within a given namespace. CIMPL provides four class types:

| Class | Description | Has... | Analogous FHIR Type |
|----------|---------|---------|---------|
| `Element` | The lowest-level class, representing a property-value pair. | `Value` | element or simple (not nested) **Extension** |
| `Group` | A class comprised of other classes, specifically, other `Group`, `Element`, and `Entry`. | `Property` | **BackboneElement**, non-primitive data type (such as **Address**), or complex (nested) **Extension** |
| `Entry`  | A class representing a group of related information, complete enough to support stand-alone interpretation. | `Property` | **Resource** or profiled **Resource** |
| `Abstract` | A special type of `Entry` that cannot be instantiated, and will not be present in the target mapping. | `Property` | **Resource** or profiled **Resource**, if **StructureDefinition.abstract** = true |

### Properties, Elements and Values

When defining classes, bear in mind that `Property` defines a model data item and its cardinality. A `Property` can be mapped to a FHIR element or extension. 

<!-- MK - the only "connection" between a property and an element is the name - true? I think that needs to be incorporated somewhere in this guide. Thoughts? -->

* A `Property` can be an `Element`, `Group` or `Entry`.
* An `Element` _must_ have one `Value` and cannot have a `Property`
* `Value` may have more than one choice (using `or`).
* `Group`, `Entry`, or `Abstract` may have more than one `Property`, but cannot have a `Value`.

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

| Keyword | Appears with | How Many? | Has Cardinality? | Has Choices? | Can be a... |
|----------|---------|---------|---------|---------|---------|
| `Value` | `Element` | exactly 1 | no | yes (optional) | primitive, `Element`, `Group`, or `Entry` |
| `Property` | `Group`, `Entry`, or `Abstract` | 0 or more | yes (required) | no | `Element`, `Group`, or `Entry` |

### Naming Recommendations

All classes in CIMPL, from `Element` to `Entry`, are reusable outside of their original context. To promote reuse, choose names that are self-explanatory and context-independent. For example, a `Property` named `Text` is vague, whereas `DisplayText` and `InstructionText` are more self-explanatory and may be understandable when placed into a specific context. However, an overly-specific name like `AddressDisplayText` could inhibit reuse because it affixes the `context` of an address to the `concept` of a display text.

***

## Value Set File

Value Set files are used to define custom value sets and codes when existing value set sources like [HL7 v3](https://www.hl7.org/fhir/terminologies-v3.html), [FHIR](https://www.hl7.org/fhir/terminologies-systems.html), [VSAC](https://vsac.nlm.nih.gov/), or [PHIN VADS](https://phinvads.cdc.gov/) are insufficient, and a new value set must be defined.

### Value Set File Example

An example Value Set file is shown below:

```
Grammar:           ValueSet 5.1
Namespace:         onco.core
CodeSystem:        SCT = http://snomed.info/sct
CodeSystem:        ICD10CM = http://hl7.org/fhir/sid/icd-10-cm

ValueSet:          CancerDiseaseStatusEvidenceTypeVS
Description:       "The type of evidence backing up the clinical determination of cancer progression."
SCT#363679005      "Imaging (procedure)"
SCT#252416005      "Histopathology test (procedure)"
SCT#711015009      "Assessment of symptom control (procedure)"
SCT#5880005        "Physical examination procedure (procedure)"
SCT#250724005      "Tumor marker measurement (procedure)"
SCT#386344002      "Laboratory data interpretation (procedure)"

ValueSet:           SecondaryCancerDisorderVS
Description:       "Types of secondary malignant neoplastic disease"
Includes codes descending from SCT#128462008  "Secondary malignant neoplastic disease (disorder)"
ICD10CM#C7B00       "Secondary carcinoid tumors, unspecified site"
ICD10CM#C7B01       "Secondary carcinoid tumors of distant lymph nodes"
ICD10CM#C7B02       "Secondary carcinoid tumors of liver"
ICD10CM#C7B03       "Secondary carcinoid tumors of bone"
ICD10CM#C7B04       "Secondary carcinoid tumors of peritoneum"

ValueSet:           HomeEnvironmentRiskVS
Description         "Risks present in the home environment"
#no_smoke_detectors "No Smoke Detectors"
#radiation          "Radon or Other Radiation Source"
#swimming_pool      "Swimming Pool"
```

### Value Set File Header

`Grammar` and `Namespace` are required keywords in a Value Set file header. A `CodeSystem` declaration is required for every Code System referenced in a value set defined in the Value Set file. For a further description, see [Code System Keyword](#CodeSystem).

### Value Set Definition

Following the header, the Value Set file contains value set definitions. The order of definitions does not matter. Each definition follows a required sequence of declarations. Follow the links for further explanation of each item:

1. [`ValueSet`](#ValueSet) declaration (required)
1. [`Description`](#description) (optional but highly recommended)
1. [Explicit code declarations](#explicit-code-declarations) (optional)
1. [Implicit code declarations ](#implicit-code-declarations) (optional)

### Explicit Code Declarations

Explicit code declarations are used to add specific codes to a value set. This method of defining a value set is sometimes called _extensional_. Codes can either be locally-defined, or selected from external code systems, such as `ICD10CM#C7B00`, where `C7B00` is a code in the `ICD10CM` code system. If the code is defined locally, the value set name serves as the code system. For the syntax of concept codes, see [Concept Codes](#concept-codes).

| Type | Example |
|---------|---------|
| Locally-defined code | `#proposed "The proposal has been proposed, but not accepted or rejected."`|
| Externally-defined code | `ICD10CM#C7B00 "Secondary carcinoid tumors, unspecified site"`|

### Implicit Code Declarations

 Implicit code declarations add codes to a value set using an expression rather than a list. This allows you to, for example, add all codes from a code system, add all codes descending from a point in a hierarchy (currently only supported for SNOMED-CT). This method of defining a value set is sometimes called _intensional_. There are three variants involving the `Includes codes` phrase:

| Phrase | Example | Result|
|----------|---------|---------|
| `Includes codes from` | `Includes codes from MDR`| Value set contains all codes from MDR code system (the MDR alias must be defined by the `CodeSystem` keyword) |
| `Includes codes descending from` | `Includes codes descending from SCT#105590001` | Value set contains SCT#1055900001 and all codes below it in the SNOMED-CT hierarchy |
|  `and not descending from` | `Includes codes descending from SCT#363346000 "Malignant neoplastic disease" and not descending from SCT#128462008 "Secondary malignant neoplastic disease"` | Value set contains SCT#363346000 and all codes below it in the SNOMED-CT hierarchy, _except_ code SCT#128462008 and all codes below it  |

***

## Map File

Map files control how classes defined in CIMPL are expressed as FHIR profile. Map files are FHIR-version dependent. You can map to FHIR DSTU 2, STU 3, and R4. If you use the [Objective FHIR base classes](cimpl6ObjectiveFHIR.md), you many not need to create a map file.

### Map File Example

The following example demonstrates how the class _Dosage_, defined in the OBF namespace, maps to the FHIR 4.0 data type with the same name. The second example demonstrates mapping between the OBF _MedicationStatement_ class and the FHIR 4.0 **US Core MedicationStatement** profile.

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

### Inheritance of Mapping Rules

Maps are defined in a series of mapping rules. A powerful feature of CIMPL is that mapping rules are inherited. If your class inherits from a parent class that is already mapped, you do not need to create a new map for your class. However, if you want to specify mappings for a new `Property`, or override the inherited map, you may do so.

The inheritance of mapping rules follows the class hierarchy. If there is no map for a class or attribute, CIMPL tooling automatically looks to the parent class to try and find a map. The first mapping rule found will be applied. If no mapping rule is found for a `Property` defined in CIMPL, a FHIR extension will be created. If a map is not found for an `Entry`, that `Entry` will be mapped to the [**Basic**](https://www.hl7.org/fhir/basic.html) resource. In general, mapping to **Basic** is not recommended because it has no inherent semantic meaning for implementers.

### Mapping CIMPL Primitives to FHIR

CIMPL follows somewhat flexible rules on how CIMPL primitive data types map to FHIR. For example, an `unsignedInt` in CIMPL can map to an **integer** data type in FHIR (since every `unsignedInt` is an **integer**, there is no loss of information). However, maps that lose information, such as `integer` in CIMPL mapped to **unsignedInt** in FHIR, generally will trigger mapping errors. An exception is that CIMPL `concept` is allowed to map to FHIR **code**. The following table shows the acceptable mappings between CIMPL and FHIR:

| CIMPL Primitive | Can map to FHIR... |
|----------|---------|
| `concept` | **code, Coding, CodableConcept** |
| `boolean` |**boolean**, **code** |
| `integer` | **integer**, **Quantity** |
| `string` | **string** |
| `decimal` | **decimal** |
| `uri` | **uri** |
| `base64Binary` | **base64Binary** |
| `instant` | **instant** |
| `date` | **date** |
| `dateTime`| **dateTime, date, instant** |
| `time` | **time** |
| `oid` | **oid** |
| `id` | **id** |
| `markdown` | **markdown, string** |
| `unsignedInt` | **unsignedInt, integer, Quantity** |
| `positiveInt` | **positiveInt, unsignedInt, integer, Quantity** |
| `xtml` | **xtml** |

### Map File Header

The header of a map file _must_ include the `Grammar`, `Namespace`, and `Target` keywords. The `Grammar` keyword declares the CIMPL mapping grammar version. The version is currently 5.1. The `Namespace` keyword relates the maps defined in this file to the Class file in the same namespace. The `Target` keyword provides the version of FHIR that the CIMPL model is mapped to, either `FHIR_DSTU_2`, `FHIR_STU_3`, or `FHIR_R4`.

For more information about the header keywords, see [Keywords](#keywords).

### Setting the Mapping Target

Each section in a Map file begins with a statement that establishes a **resource, profile**, or **complex data type** that is the target for the map. In CIMPL Map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in the target output. The mapping target can be a FHIR **resource, profile**, or **complex data type** that is part of the target FHIR release version. The initial statement in each mapping block must end with a colon (:).

| CIMPL Class | FHIR Target | Example |
|----------|---------|---------|
| `Entry` | resource | `SocialHistoryObservation maps to Observation:` |
| `Entry` | profile | `LaboratoryObservation maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-observationresults:` |
| `Group` | complex data type | `Timing maps to Timing:` |

### Mapping Properties
<!-- plural property -- thinking -->
After the first line, which establishes the class to be mapped, the remainder of the section consists of statements that map CIMPL `Property` to FHIR elements. Each statement begins with a [CIMPL path](#cimpl-paths), followed by the phrase `maps to`, and ending with the target FHIR path.

With FHIR as the target output, the right-hand side of the map statement can point to any element of a FHIR resource, including:

* FHIR elements that are at the root of the resource (e.g., **maritalStatus**)
* FHIR elements that are child elements of root elements on the resource (e.g., **reaction.substance**)
* FHIR elements that have a [choice of data types](https://www.hl7.org/fhir/formats.html#choice) (e.g., **multipleBirth[x].boolean**)
* FHIR elements that have been defined in existing profile or other **StructureDefinition** resources (e.g., `http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity`)
* FHIR elements that support FHIR **Reference()** data types.

| CIMPL source | FHIR target | Example |
|----------|---------|---------|
| `Property` | element | `MaritalStatus maps to maritalStatus` |
| nested `Property` | element |`Value.ActiveFlag maps to active` |
| nested `Property` | nested element| `AdverseReaction.AllergenIrritant maps to reaction.substance`|
| `Value` choice | element | `BodySiteOrCode.concept maps to bodySite` _(see note)_ |
| `Property` | **Extension** | `Ethnicity maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity` |
| any | nested **Extension** | _not supported_|


>**Note:** The mapping grammar version is 5.1; the bracketed value syntax cannot be used in paths.  This is why the above example uses `BodySiteOrCode.concept` rather than `BodySiteOrCode[concept]`.

### Mapping to References

FHIR resources and profile require an explicit [**Reference()** indicator](https://www.hl7.org/fhir/references.html) when referring to another resource or profile. In CIMPL, whenever a `Property` is an `Entry`, that `Property` is mapped as a reference in FHIR. There is no need to create an explicit **Reference()**.

### Placement of Extensions

As mentioned above, if no mapping rule is found for a `Property`, an extension to the FHIR resource or profile will be automatically created. By default, the extension will be a top-level element in the FHIR profile. Sometimes you may want the extension to appear in a different place, perhaps inside a nested element. To put the extension precisely where you want it, you can map to FHIR paths that include `extension`.

Here's an example involving the FHIR resource [AdverseEvent](https://www.hl7.org/fhir/adverseevent.html):

`PotentialCause.CauseCategory maps to suspectEntity.extension`

This mapping rule will create an extension named **CauseCategory** for **AdverseEvent.suspectEntity**. Rules such as:

`DetectionTime maps to extension`

leave the default behavior unchanged, and are not required. However, they can be used to document the _intent_ to create an extension.

### Special Mapping Statements

There are a limited number of mapping statements that allow actions to be taken directly on FHIR profiles, without a corresponding property defined in CIMPL.

#### fix

`fix` allows FHIR elements to be assigned a value. This constrains the FHIR resource, not the CIMPL class. This enables you to fix a value to an element in a FHIR resource without having to change the class definition, which would affect all potential maps. This also allows you to fix values to FHIR elements that have no corresponding `Property` in the CIMPL definition.

Example:

`fix status to #completed`

#### constrain

`constrain` allows users to constrain the cardinality of FHIR elements. This constrains the FHIR resource, not the CIMPL class. The cardinality is expressed in {min}..{max} form.

Example:

`constrain explanation to 0..1`

### Slicing

> **Note:** Mapping statements for slicing arrays is an advanced topic.

_Slicing_ involves specifying the items that can be contained in an array. An array is an element whose upper cardinality is greater than one. An example of this is the [**FHIR Blood Pressure profile**](https://www.hl7.org/fhir/profiling.html#slicing), which slices **Observation.components** into diastolic and systolic pressure, identical in structure other than their code. Another example is **DiagnosticReport.result**, an array of Observations. Slicing can be used to specify the particular type of Observations in a given type of DiagnosticReport.

See the FHIR documentation for more detail about [slicing](https://www.hl7.org/fhir/profiling.html#slicing).

In CIMPL, the [`includes`](#includes-constraint) constraint creates slices. This section discusses how slices defined in the Class file are mapped to FHIR.

The following parameters are used to specify the mapping CIMPL classes to FHIR slices:

* `slice on`
* `slice on type`
* `slice strategy`
* `slice at`
* `slice #`

The values for these parameters refer to the FHIR structure that is being sliced.

<!-- Carmela - add text to relate to the class file slicing definition - they are related -->

#### Set the Discriminator Path with `slice on`

FHIR requires [discriminators](https://www.hl7.org/fhir/profiling.html#discriminator) to uniquely identify the slice to which an element or resource belongs. The path to the discriminator is specified by the CIMPL `slice on` parameter. The value is a [FHIRPath expression](https://www.hl7.org/fhir/fhirpath.html) that indicates what distinguishes the slices.

The discriminator path is not necessarily the array where the slicing takes place. For example, in slicing **Observation.component**, if `slice on = code.coding.code`, then **component.code.coding.code** will be examined to determine which slice the component belongs to. Each defined slice must have unique, fixed value at **component.code.coding.code**.

When the sliced array contains references to other FHIR resources (e.g., **DiagnosticReport.result**), the discriminator must involve referenced resource. In this case, the `slice on` will include `reference.resolve()`, a FHIRPath notation that refers to a referenced resource. There are two options:

1. The discriminator is referenced resource itself, and the mapping to a slice will be determined by conformance to a profile.
1. The discriminator is a property in the referenced resource, e.g., **reference.resolve().code.coding.code**

>**Note:** The `slice on` parameter is required except when the `slice #` parameter is present.

#### Set the Discriminator Type with `slice on type`

The discriminator type defines how the discriminator is used to differentiate instances from one another. By default, CIMPL uses the FHIR value discriminator type. However, a different discriminator type can be declared using `slice on type` followed by one of the following FHIR-defined discriminator types:

| `slice on type` | Interpretation |
| ------------- | ---------- |
| `value`| The slices have different values in the discriminator (default) |
| `exists` | The slices are differentiated by the presence or absence of the discriminator |
| `pattern` |The slices have different values in the discriminator, as determined by testing them against the applicable **ElementDefinition.pattern[x]** |
| `type` |The slices are differentiated by the data type of the discriminator |
| `profile` | The slices are differentiated by conformance of the discriminator to a specified profile. This can only be used when the discriminator is a reference to a resource. |

#### Slice Strategy
<!-- Carmela - confirm underscore with slice strategy check the code -->

The `slice strategy` indicates how the CIMPL to create slices. To map a CIMPL `includes` constraint, always specify `slice strategy = includes`. Each included CIMPL data type becomes a separate slice in the target FHIR array.

If `slice strategy` is omitted, CIMPL adds a single slice, corresponding to the CIMPL data type, in the target FHIR array.

#### Set the Slice Location using `slice at`

If the CIMPL element in the mapping statement is mapped to a sub-element of the array being sliced,`slice at` explicitly defines the location of the slice. The location must be an array element, such as a **BackboneElement** that contains the mapping target.

#### Set the Slice Number using `slice #`

When mapping to an array that is part of an existing profile that has already been sliced, you can indicate which existing slice is the target of the mapping.

> **Note**: This feature is available in SHR-CLI 6.5 and above.

### Slicing Examples

#### Example 1

`Components.ObservationComponent maps to component (slice on = code.coding.code; slice strategy = includes)`

In the above example, each `ObservationComponent` mentioned in the CIMPL `includes` constraint will create a slice in the FHIR **component** array. The slices are distinguished by the value at the FHIR path **component.code.coding.code**.

#### Example 2

`PanelMembers.Observation maps to related.target (slice at = related; slice on = target.reference.resolve(); slice on type = profile; slice strategy = includes)`

> **Note:** This example is based on the FHIR STU 3 Observation resource.

Each `Entry` included in the `PanelMembers.Observation` CIMPL model will create a slice in the **Observation.related** array because of the `slice at` parameter, and appear in the **related.target** element because of the `maps to` statement. The `slice on` parameter indicates the discriminator is the resource in the **related.target** array. The `target.reference.resolve()` is the FHIRPath that points to the referenced resource. The `slice on type = profile` indicates the slices are differentiated through the conformance of the referenced resource to the profile that define the slices.

#### Example 3

```
BloodPressure maps to http://hl7.org/fhir/StructureDefinition/bp:
  Components.SystolicPressure maps to component (slice # = 1)
  Components.DiastolicPressure maps to component (slice # = 2)
```

This example shows how to map to existing slices in the mapping target. The numbers refer to the order of the slices as they are defined in http://hl7.org/fhir/StructureDefinition/bp. This type of mapping allows new constraints and extensions in the CIMPL model to be added to an existing profile. When mapping to an existing profile, the discriminator and other slicing properties cannot be changed.

<!--
#### Example 4

`Laterality maps to qualifier (slice on = concept)`
`

Multiple elements in CIMPL map to a single target (which has to be an array)


	Observation maps to result (slice on = $this.resolve().code.coding.code; slice strategy = includes)
-->
***

# Appendix - Document Conventions

| Style | Explanation | Example |
|:----------|:---------|:---------|
| **Bold**  | A FHIR name | **Observation** |
| `Code` | A CIMPL term, phrase, example, or command | `CodeSystem: LNC = http://loinc.org` |
| <code><i>Italics</i> appearing in a code block | Indicates an item that should be substituted | <code>Value only <i>datatype</i></code> |
| _Italics_ | A file name (also used for general emphasis in text) | _obf-action.txt_ |
| _Italics with **bold** highlight_ | Indicates a substring in the file name that should be substituted | _ig-**myigname**-config.json_ |
| Leading Capitalization | CIMPL keywords or references that are capitalized; specific instances of FHIR artifacts | The `Grammar` keyword |
| **Note:** | Something to keep in mind about the current topic | **Note:** Value Set names must begin with an uppercase letter. |
