# CIMPL 6.0 Reference Documentation

>**Note:** This documentation is a draft.

_This is a comprehensive guide to CIMPL 6.0 syntax.  If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md). If you're looking for a more in-depth introduction, try the [Tutorial](cimpl6Tutorial_detail.md)._

***

**Table of Contents**

[TOC]

***

# Overview

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles and implementation guides (IG). Because it is a _language_, written in text statements, CIMPL encourages distributed, team-based development using conventional source-code control tools such as Github. CIMPL provides tooling that enables you to define a model once, and publish that model to multiple versions of FHIR.

## CIMPL Versioning
CIMPL follows the [Semantic Versioning](https://semver.org) convention (MAJOR.MINOR.PATCH).

* MAJOR: A major release has significant new functionality and potentially, grammar changes or other non-backward-compatible changes.
* MINOR: Contains new or modified features, while maintaining backwards compatability within the major version.
* PATCH: Contains minor updates and bug fixes, while maintaining backwards compatability within the major version.

CIMPL is currently on major version 6.

For a full changelog, see the [Release Notes](https://github.com/standardhealth/shr-cli/releases).

<!-- TODO: Find MAJOR release differences -->

## Namespaces
CIMPL organizes information into namespaces. A namespace is conventionally denoted by the authoring organization and the broad category of elements the namespace defines, delineated by a period. Namespaces must begin with a lowercase character. For example:
```
Namespace: shr.lab
```
>**Note:** The organization name is recommended but not required (e.g., `Namespace: lab` is also allowed).

## File Types
Model information in CIMPL is stored in the following three file types:
* Class files (.txt): contain definitions of CIMPL classes, which map to FHIR profiles and extensions.
* Value Set files (.txt): contain definitions of value sets defined in the namespace.
* Map files (.txt): contain information on how the CIMPL models relate to FHIR resources, profiles, and elements.

Each namespace will typically have one or more class files, and if needed, a value set and map file.

## Naming Conventions
### File Names
File names must begin with lower case and typically include the namespace:

* Class files: `namespace.txt` or `namespace_something.txt`
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


### Class Names
Elements, Groups and Entries are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase). For example:
```
* Element: DetectionTime
* Group: Dosage
* Entry: AdverseEvent
```
>**Note:** Letters, numbers, and hyphens are allowed. However, it is required that an element name begins with an uppercase letter.

### Fully Qualified Names
CIMPL does not support duplicate class names within a single namespace. However, the same name may occur in different namespaces. In the case of naming collisions due to an included namespace in `Uses`, you have to refer to the class by its fully qualified name (FQN). The FQN is a combination of the element's namespace and its declared name (through concatention delimited by a period). For example, the element `Observation` in the namespace `shr.finding` has the following FQN:
```
shr.finding.Observation
```
FQNs are only required to be used in case of naming collisions.

### Value Set Names
Value Sets are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase), with an additional "VS" suffix. For example:
```
ValueSet: HomeEnvironmentRiskVS
```
>**Note:** Value Set names must begin with an uppercase letter.

### Local Code Names
Within a custom value set, the individual codes (denoted by # symbol) are conventionally written in lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case). For example:
```
ValueSet:          HomeEnvironmentRiskVS
#smoke_detectors   "No Smoke Detectors"
#radiation         "Radon or other radiation source"
#swimming_pool     "Swimming Pool"
```
## Reserved Words
CIMPL has a number of reserved words that are part of CIMPL's grammar (e.g., `boolean`, `or`, `from`, `maps to`). For a complete list of reserved words, refer to the [ANTLR4 grammar](#https://github.com/standardhealth/shr-grammar/tree/master/src/main/antlr).

## Whitespace
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
## Comments
CIMPL follows [JavaScript syntax](https://www.w3schools.com/js/js_comments.asp) for code comments:
```
// Use a double-slash for comments on a single line

/*
Use slash-asterisk and asterisk-slash for larger block comments.
These comments can take up multiple lines.
*/
```
## Primitives
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

CIMPL does not have an explicit "Reference" type (see [References](#references).)

## Concept Codes
CIMPL uses a single primitve type, `concept` to represent coded terms from controlled vocabularies. A concept combines three elements: code system, code, and optional display test. The grammar for specifying concepts is: 

> `SYSTEM#code "Display text"`

For example:
```
SCT#363346000 "Malignant neoplastic disease (disorder)"
ICD10CM#C004  "Malignant neoplasm of lower lip, inner aspect"
```
The SYSTEM is an alias for a canonical URI that represents a controlled vocabulary. Aliases must be declared in the file header using the [CodeSystem keyword](#codesystem). You cannot use the canonical URI directly in the concept grammar.

Unlike FHIR, CIMPL does not differentiate between code, Coding, and CodeableConcept. See [Mapping Concept Codes](#mapping-concept-codes) for information on CIMPL maps `concept` to these FHIR types.

>**Note:** Future versions of CIMPL will expand `concept` to include the code system version.

***
# Keywords
Keywords are used to make various declarations. Keywords always appear at the beginning of a line and are followed by a colon. 

Keyword Summary:
| Keyword | Purpose | File Type(s) |
|----------|---------|---------|
| [Abstract](#abstract) | Declare an non-instantiable class | Class |
|[CodeSystem](#codesystem) | Define a code system alias | Class, Value Set |
| [Concept](#concept) | Express the meaning of a class | Class |
| [Description](#description) | Provide a human-readable description | Class, Value Set |
| [Element](#element) | Declare lowest-level (elemental) model building block  | Class |
| [Entry](#entry) | Declare a stand-alone class | Class |
| [Grammar](#grammar) | Define how a file should be parsed | All |
| [Group](#group) | Declare mid-level, non-stand-alone model building block  | Class |
| [Namespace](#namespace) | Associate a file with a namespace | All |
| [Parent](#parent) | Specify inheritance | Class |
| [Path](#path) | Alias long URLs | Class, Value Set |
| [Property](#property) | Define an attribute in a class | Class |
| [Uses](#uses) | Import namespace(s) | Class |
| [Value](#value) | Define data type(s) of Elements | Class |
| [ValueSet](#valueset) | Define a value set | Value Set |


## Abstract
The `Abstract` keyword is used to declare a new Abstract class. Abstract classes are identical to Entries, except they are not instantiable. For details, see [Building Blocks](#building-blocks).

Example:

`Abstract: ActionStatement`

## CodeSystem
The `CodeSystem` keyword provides an alias or shorthand for the canonical URI of code system. You *must* define a code system alias before you can use codes from that system in constraints or value sets. By convention, the alias should be be UPPER CASE, although the tooling does allow for numbers and hyphens after the first upper case letter.

Examples:

`CodeSystem: LNC = http://loinc.org`

`CodeSystem: SCT = http://snomed.info/sct`

## Concept
The `Concept` keyword (not to be confused with the [concept primitive](#concept-codes)) establishes the meaning of a class in terms of a code (or codes) from controlled vocabularies. Assigning a concept allows the meaning of the class to be understood without inferring it from the class name. If multiple concept codes are used, they should appear as a comma-separated list. This keyword is optional.

Examples:

`Concept: MTH#C3858779 "Security classification"`

## Description
The `Description` keyword is used for a narrative that defines the namespace, class, or value set.
<!-- TO DO: Do we support Markdown? -->
Example:

`Description: "The SHR Environment domain contains definitions related to surroundings experienced by the person of record."`

>**Note:** While the CIMPL tooling will allow for any pattern of [Unicode](http://unicode.org/standard/WhatIsUnicode.html) characters within enclosed double quotation marks (`"`), certain exporters such as the FHIR exporter will object to non-[ASCII](https://en.wikipedia.org/wiki/ASCII) text.)

## Element
The `Element` keyword is used to declare a new class of type Element, the lowest-level building block in CIMPL. For details, see [Building Blocks](#building-blocks).

Example:

`Element: EvidenceType`

## Entry
The `Entry` keyword is used to declare a new stand-alone class, analogous to a Resource or Profile in FHIR. For details, see [Building Blocks](#building-blocks).

Example:

`Entry: GenomicsReport`

## Grammar
The `Grammar` keyword defines how the file is to be parsed. The grammar declaration must be the first line in each CIMPL file.

| File Type | Requried First Line |
|----------|---------|---------|
| Class | `Grammar: DataElement 6.0` |
| Value Set | `Grammar: ValueSet 6.0` |
| Map | `Grammar: Map 6.0` |

The version details the MAJOR and MINOR version of the grammar the file was written for. The grammar version may not be the same as the CIMPL release. The grammar is forward compatible within the MAJOR version number, i.e. the declaration `DataElement 6.0` is correct for any version 6.x.x tooling. For more information, see [Versioning](#versioning).

## Group
The `Group` keyword is used to declare a building block comprised of one or more properties, similar to a complex data type in FHIR. For details, see [Building Blocks](#building-blocks).

Example:

`Group: Dosage`

## Namespace
The `Namespace` keyword defines the association of the file with a namespace, and is required in `DataElement`, `ValueSet` and `Map` files. The namespace can be any number of lowercase period-delimited words. Best practice is to follow the naming convention pattern of `organization`.`domain` (followed by subdomains if necessary).

Examples:

`Namespace: odh`

`Namespace: shr.environment`

>**Note:** While it is best practice to use only lower case letters, the strict requirements only require the word to begin with a lowercase letter. The rest of the namespace allows for mixed casing and hyphens.

## Parent
Through the `Parent` keyword, CIMPL provides a mechanism for basing a class on another class, the child inheriting all properties and constraints from the parent. The child class can then extend information inherited from the parent by adding additional properties and applying additional constraints. Additionally, mappings are inherited alongside properties. At most one parent class can be specified. The Parent declaration is optional

There are restrictions on which building blocks can inherit from other building blocks. The rules can be summarized as _like inherits from like_:

| Building Block| Can Inherit From |
|----------|---------|
| `Element` | `Element` |
| `Group` | `Group` |
| `Entry` | `Abstract` or `Entry` |
| `Abstract` | `Abstract` |

>**Note:** Although the CIMPL 6.0 tooling does not currently enforce these inheritance rules, they will enforced in future releases. You are strongly encouraged to comply to these rules, and refactor any definitions ported from CIMPL 5.x that do not comply.

Example:

`Parent: Observation`

## Path
The `Path` keyword allows for abbreviations of long URLs. This functionality is completely optional, but it is provided to make authoring easier. By convention, the alias should be be an all UPPER CASE word, although the tooling does allow for numbers and hyphens after the first letter.

Example:

`Path: FHIR = http://hl7.org/fhir/ValueSet`

## Property

Classes based on Group, Entry, and Abstract are composed of one or more properties (called _fields_ or _attributes_ in object-oriented programming). A property can be any Element, Group, or Entry, but never a primitive. Each property must have a specified cardinality range, represented as `min..max`, indicating the number of repeats of the property. **The Property keyword cannot be used in Elements.**

In the following example, StudyArm has three properties: `Name`, `Type`, and `Comment`. `Name` is singular and required, `Type` is optional and repeating, and `Comment` is singular and optional:
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

## Uses
The `Uses` keyword appears in the Class file and provides a comma-separated list of the namespaces imported to the current namespace. This keyword allows you to use the classes and value sets defined in other namespaces. The list order has no effect.

Examples:

`Uses: shr.core`

`Uses: shr.core, onco.core`

<!-- TODO: VERIFY THIS: In CIMPL version 6, it is *required* that you import `shr.core` for the tooling to run.-->

## Value

`Value` represents the data type(s) an Element can accept. This keyword can only be when defining an Element. Each Element must have exactly one `Value`, although the value itself can be a choice of several data types. Value choices can be [primitives](#primitives), [Elements](#element), [Groups](#group), or [Entries](#entry). A value may be inherited and constrained in the child class.

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

Although in general, constraints cannot be applied to a property or value at the same time the item is defined, you are allowed to bind a value set to a `concept` value choice at the same time the value is defined, as follows:

```
Element:           IsPrimaryTumor
Description:       "Whether the tumor is the original or first tumor in the body, for a particular cancer."
Value:             concept from YesNoUnknownVS (required)
```
For more information on binding, see [Value Set Binding Constraint](#value-set-binding-constraint).

>**Note:** CIMPL 5.0 Grammar associating a cardinality to a Value is now obsolete.

## ValueSet
Defines the name of a value set defined inside CIMPL (references to external value sets are also allowed). By convention, all value sets should be Pascal case, start with the name of the corresponding concept-type Element that is being bound to the value set, and end with `VS`.

Example:

`ValueSet: ProposedStatusVS`
***

# CIMPL Paths
CIMPL provides grammar that allows you to refer to any part of a CIMPL class, regardless of nesting. This section outlines CIMPL Path Grammar, which is used in [constraints](#constraints) and [mapping](#map-file).

## Dot Notation
Dot notation is used to denote nested properties. The path lists the properties in order, separated by a dot (.) For example, if `Patient` has a `ContactPoint`, and `ContactPoint` has a `Priority`, then the path to `Priority` is `Patient.ContactPoint.Priority`.

## Paths Cannot Contain Entries
Except as the first element, paths cannot contain Entries. Paths describe nested containment within one class, and Entries represent independent objects. In FHIR terms, paths cannot span across References.

For example, even though `Procedure` has a `Patient` property, a path starting in `Procedure` cannot end in `Patient`, because `Patient` is an Entry. If such a path were allowed, it would be possible to do nonsensical things, such as putting a constraint on `Patient` from within the `Procedure` class, or mapping the elements within `Patient` when mapping `Procedure`.

## Bracket Notation for Value Choices
Values can offer a [choice of data types](#value). Bracket notation is the method used to refer to a particular value choice. Bracket notation must always be used when an Element is part of a CIMPL path.

How value choices appear in paths depends on whether the path appears with an Element definition or not. 
Example:
``` 
Element: Priority
Value: integer or concept
```
| Inside Class... | Grammar |
|----------|---------|
| `Element` | `Value[integer]` or `Value[concept]` |
| `Entry` or `Group` | `Priority[integer]` or `Priority[concept]` |

>**Note:** The requirement to use bracket notation _even when the value has only one choice_ is a change from CIMPL 5.

## Path Example

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

A reliable method to create a CIMPL path is to first view it as a dot path, then bracket any part of the path that is a Value, and finally, remove the dot before each left (open) bracket. Here's an example of that process, for the path from Landmark to Units:

1) Find the path: `Landmark` to `Distance`, `Distance` to `Quantity`, `Quantity` to `Units`.
1) Reduce to dot path: `Landmark.Distance.Quantity.Units`
1) Bracket each Value: `Landmark.Distance.[Quantity].Units`
1) Remove dot before left (open) bracket: `Landmark.Distance[Quantity].Units`

***
# Constraints
Constraints are a way to shape a general model for a particular purpose. Properties, values, and value choices can be constrained. Constraint targets can be be top-level or deeply nested (the latter referred to using [CIMPL paths](#cimpl-paths)), inherited or locally-defined.

Unlike keyword declarations, constraint statements do not begin with a keyword. Most constraint statements have three parts:

* The subject of the constraint
* Reserved word indicating the type of constraint
* The constraint value being applied

Here are several examples of this pattern:

| Example | Subject | Reserved word | Value applied |
|----------|---------|---------|---------|
|`IsPrimaryTumor = true`| `IsPrimaryTumor` | `=` | `true` |
|`Specimen substitute TumorSpecimen` | `Specimen` | `substitute` | `TumorSpecimen` |
| `BodyLocation from BodyLocationVS` | `BodyLocation` | `from` | `BodyLocationVS` |
| `LanguageUsed 0..1` | `LanguageUsed` | (implicit) | `0..1`

Here is a summary of constraints:

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
| Fix the units associated with a Quantity value choice  | `Value[Quantity].Units = UCUM#cm` |
| Fix a code to a value  | <del>`Value = SCT#233613009 "Pneumonia"`</del> invalid|

>**Notes:**
1) [Bracket notation](#bracket-notation-for-value-choices) must always be used when applying a constraint to a value choice, _even if the value has only one choice_. This is a change from CIMPL 5.
1) CIMPL allows fixed concept values to be overridden in child classes. Although this seems to violate the notion that constraints should get progressive tighter in subclasses, it is necessary when narrowing the meaning of a class characterized by a concept code. For example, the class `Pneumonia`, with fixed code, `SCT#233604007`, may have a child, `Fungal Pneumonia`, with fixed code `SCT#233613009`, the latter overriding the former. CIMPL will assume, without checking, that the code describing the child class has more constrained semantics than the code it replaces.
1) Fixing numerical types and strings (including URLs) is not yet supported, although this feature is planned.
1) If you only need to fix a value code for one particular mapping (one version of FHIR, for example), do so using the [`fix`](#fix) keyword in the mapping file.

## Append Constraint
The append (`+=`) operator appends a fixed primitive value (boolean, concept, etc.) to an array property. You can use the `+=` operation repeatedly on the same array, limited only be the cardinality of the array. The append operator can only be applied to repeating properties.

For example, you would use `+=` to insert a particular Social Security Number (SSN) or Driver's License number into an array of identifiers (in constrast, [`includes`](#includes-constraint) would be used to require the array contain one or more SSNs, without specifying a particular SSN.)

| Example | Syntax |
|----------|---------|
| Insert "Behavior" into Category array |  `Category += LNC#54511-1 "Behavior"` |
| Add "Social History" to same array | `Category += OBSCAT#social-history "Social History"` |

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
* Constraining may leave the binding strength the same and change the value set instead. However, certain changes permitted in CIMPL may violate [FHIR principles](http://hl7.org/fhir/R4/profiling.html#binding-strength). FHIR will permit a required value set to be replaced by another required value set only if the codes in the new value set are a subset of the codes in the original value set. For extensible bindings, the new value set can contain codes not in the existing value set, but additional codes SHOULD NOT have the same meaning as existing codes in the base value set.

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
Values can have multiple data types. Use the `only` constraint in an Element to narrow the choice of data types. The `only` constraint can only be applied to a Value that has multiple choices. The grammar of `only` constraints varies depending on whether the constraint appears in an Element, or another type of Class. 

Example:
``` 
Element: Priority
Value: integer or concept
```
| Inside Class... | Grammar |
|----------|---------|
| `Element` | `Value only integer` |
| `Entry` or `Group` | `Priority only integer` |

>**Note:** CIMPL currently does not support narrowing a choice to a more limited choice, for example, narrowing a choice of three types to two types. This feature is planned.

***
# Class File
Class files contain definitions of the information model.

## Class File Example

Here is an abbreviated example of a class file:
```
Grammar:           DataElement 6.0
Namespace:         shr.core
Description:       "The SHR Core domain contains definitions for a variety of general-purpose elements that are used widely in SHR."
CodeSystem:        UCUM = http://unitsofmeasure.org
CodeSystem:        LNC = http://loinc.org
CodeSystem:        SCT = http://snomed.info/sct

Group:             ContactPoint
Description:       "An electronic means of contacting an organization or individual."
Property:          TelecomNumberOrAddress 0..1
Property:          Type 0..1
Property:          Purpose 0..1
Property:          PriorityRank 0..1
Property:          EffectiveTimePeriod 0..1
                   Type from http://hl7.org/fhir/ValueSet/contact-point-system (required)
                   Purpose from http://hl7.org/fhir/ValueSet/contact-point-use (required)

Element:           TelecomNumberOrAddress
Description:       "A user name or other identifier on a telecommunication network, such as a telephone number (including country code and extension, if necessary), email address, or SkypeID."
Value:             string

Element:           PriorityRank
Description:       "The priority of the procedure, as a number (1 being the highest priority)"
Value:             positiveInt


```
## Class File Header
`Grammar` and `Namespace` are required keywords in a class file header. A namespace `Description` is recommended. In practice, `Uses` and `CodeSystem` will frequently be needed. For a further description of the header keywords, see [Keywords](#keywords). 

The `Uses` keyword brings in classes defined in other namespaces. In most cases, you can use classes in imported namespaces as if they were locally defined. However, if a class name in an imported namespace collides with a local class name, you must refer to that element by its [fully qualified name](#fully-qualified-names).

## Class Definitions
Following the header, the class file contains a number of class definitions. The order of definitions does not matter. A class definition has a sequence of declarations, although after the first declaration, a strict order is not prescribed. Follow the links for further explanation of each item:

1) [Class Type and Name Declaration](#class-type-and-name-declaration) (required)
1) [Parent Declaration](#parent) (optional)
1) [Concept Declaration](#concept) (optional)
1) [Description](#description) (optional but highly recommended)
1) [Property Declarations](#property) (for Entry, Abstract and Group) or [Value Declaration](#value) (for Element) (optional, but typically present)
1) [Constraint statements](#constraints) (optional)

## Class Type and Name Declaration

The first line of all class definitions is a keyword representing the type of building block, followed by a descriptive name you choose. Class names must be unique within a given namespace. CIMPL provides four types of building blocks:

| Building Block | Description | Has... | Analogous FHIR Type |
|----------|---------|---------|---------|
| `Element` | The lowest-level building block, representing a name-value pair. | Value | Property or simple extension |
| `Group` | A building block comprised of other building blocks, specifically, other Groups, Elements, and Entries. | Properties | Backbone element or complex extension |
| `Entry`  | A building block representing a group of related information, complete enough to support stand-alone interpretation. | Properties | Resource or Profile |
| `Abstract` | A special type of Entry that cannot be instantiated, and will not be present in the target mapping. | Properties | none |

## Properties and Values

When defining classes, bear in mind that an `Element` has exactly one `Value` field and no `Property` fields, whereas, a `Group`, `Entry`, or `Abstract` can have multiple `Property` fields but cannot have a `Value` field. Furthermore, a `Property` cannot have choices, but a `Value` can.

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

| Keyword | Appears in | How Many Times? | Has Cardinality? | Has Choices? | Can be a... |
|----------|---------|---------|---------|---------|---------|
| `Value` | `Element` | Exactly Once | No | Yes (optionally) | primitive, Element, Group, or Entry |
| `Property` | `Group`, `Entry`, or `Abstract` | Zero or more | Yes | No | Element, Group, or Entry |


### Naming Recommendations

All classes in CIMPL, from Elements to Entries, are reusable outside of their original context. To promote reuse, try to choose names that are self-explanatory and context-independent. For example, a property named `Text` is too vague, whereas `DisplayText` and `InstructionText` are more self-explanatory and may be completely understandable when placed into a specific context. However, an overly-specific name like `AddressDisplayText` could inhibit reuse because it affixes the _context_ of an address to the _concept_ of a display text.

***
# Value Set File
The value set files are used to define custom value sets and codes when existing value set sources like [HL7 v3](https://www.hl7.org/fhir/terminologies-v3.html), [FHIR](https://www.hl7.org/fhir/terminologies-systems.html), [VSAC](https://vsac.nlm.nih.gov/), or [PHIN VADS](https://phinvads.cdc.gov/) are insufficient, and a new value set must be defined.

## Value Set File Example

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

`Grammar` and `Namespace` are required keywords in a Value Set file header. One or more `CodeSystem` will be needed. For a further description of the header keywords, see [Keywords](#keywords). 

## Value Set Definition

Following the header, the Value Set file contains a number of value set definitions. The order of definitions does not matter. Each definition follows a set sequence of declarations. Follow the links for further explanation of each item:

1) [Value Set Declaration](#ValueSet) (required)
1) [Description](#description) (optional but highly recommended)
1) [Code Declarations](#code-declaration) (optional)
1) [Includes Codes statements](#includes-codes) (optional)

## Explicit Codes

Explicit code declarations are used to add specific codes to a value set. Codes can either be locally-defined, or selected from external code systems, such as the `CAP#29915` example, in which `29915` is an existing code within the `CAP` codesystem. If the code is defined locally, the namespace serves as the code system. For the syntax of concept codes, see [Concept Codes](#concept-codes).

| Type | Example |
|---------|---------|
| Locally-defined code | `#proposed "The proposal has been proposed, but not accepted or rejected."` |
| Externally-defined code | `CAP#29915 "None/Negative"`|

## Implicit Codes
Implicit code declarations are used to add sets of codes to a value set, in particular, from hierarchically-defined code systems. At present, the only hierarchical system supported is SNOMED-CT. There are three variants involving the `Includes codes` phrase:

| Phrase | Example | Result|
|----------|---------|---------|
| `Includes codes from` | `Includes codes from MDR`| Value set contains all codes from MDR code system |
| `Includes codes descending from` | `Includes codes descending from SCT#105590001` | Value set contains SCT#1055900001 and all codes below it in the SNOMED-CT hierarchy |  
|  `and not descending from` | `Includes codes descending from SCT#363346000 "Malignant neoplastic disease" and not descending from SCT#128462008  "Secondary malignant neoplastic disease"` | Value set contains SCT#363346000 and all codes below it in the SNOMED-CT hierarchy, _except_ code SCT#128462008 and all codes below it  |

# Map File
Map files control how classes defined in CIMPL are expressed as FHIR profiles. Map files are FHIR-version dependent. Currently, you can map to FHIR DSTU 2, STU 3, R4.

## Inheritance of Mapping Rules
Maps are defined in a series of mapping rules. A powerful feature of CIMPL is that mapping rules are inherited. If your class that inherits from a parent class that already is mapped, you do not need to create a new map for your class. However, if you you want specify mappings for new properties, or override the inherited map, you may do so.

The inheritance of mapping rules follows the class hierarchy. If there is no map for a class or attribute, CIMPL tooling automatically looks to the parent class to try and find a mapping. The first mapping rule found will be applied. If no mapping rule is found for a property defined in CIMPL, a FHIR extension will be created. If no mapping is found for an Entry, that Entry will be mapped to the [Basic resource](https://www.hl7.org/fhir/basic.html). In general, mapping to Basic is not recommended because it has no inherent semantic meaning for implementers. Any unmapped properties will appear as FHIR extensions.

## Mapping CIMPL Primitives to FHIR
CIMPL follows somewhat flexible rules on how CIMPL primitives map to FHIR. For example, a unsignedInt in CIMPL can map to an integer type in FHIR (since every unsignedInt is an integer, there is no loss of information). However, mappings that lose information, such as integer in CIMPL to unsignedInt in FHIR, generally will trigger mapping errors. An exception is that CIMPL `concept` is allowed to map to FHIR `code`. The following table shows the acceptable mappings between CIMPL and FHIR:

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

## Map File Example
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

## Map File Header
The header of a map file must include a `Grammar`, `Namespace`, and `Target` keywords. The Target keyword gives the version of FHIR that the CIMPL model is mapped to, either `FHIR_DSTU_2`, `FHIR_STU_3`, or `FHIR_R4`.

For a further description of the header keywords, see [Keywords](#keywords).

## Setting the Mapping Target
Each block in a mapping file begins with statement that establishes a resource, profile, or complex data type that is the target for the mapping. In CIMPL map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in the target output. The mapping target can be a FHIR resource or profile that is part of the target FHIR release version. The initial statement in each mapping block must end with a colon (:).

| CIMPL Building Block | FHIR Target | Example |
|----------|---------|---------|
| Entry | Resource | `SocialHistoryObservation maps to Observation:`|
| Entry | Profile | `LaboratoryObservation maps to http://hl7.org/fhir/us/core/StructureDefinition/us-core-observationresults:` |
| Group | Complex Datatype | `Timing maps to Timing:` |

## Mapping Properties
After the first line, the remainder of the block consists of statements that map CIMPL properties to FHIR elements. Each statment begins with a [CIMPL path](#cimpl-paths), followed by the phrase `maps to`, and ending with the target FHIR path.

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

## Placement of Extensions
As mentioned above, if no mapping rule is found for a property, an extension will be automatically created. By default, the extension will be a top-level element in the FHIR profile. Sometimes you may want the extension to appear in a different place, perhaps inside a nested element. To put the extension precisely where you want it, you can map to FHIR paths that include `extension`.

Here's an example involving AdverseEvent:

`PotentialCause.CauseCategory maps to suspectEntity.extension`

This rule will create an extension named `CauseCategory` under AdverseEvent.suspectEntity, rather than directly under AdverseEvent. Note that rules such as this:

`DetectionTime maps to extension`

leave the default behavior unchanged, and are not required. However, they can be used to document the intent to create an extension.

## Special Mapping Statements

### Fix
The `fix` keyword allows FHIR fields to be assigned a value. Note that it is the FHIR resource that is getting the value, not the CIMPL class. This enables you to fix a FHIR value without having change the class definition, which would affect all potential mappings.

Example:

`fix status to #completed`

### Constrain
The `constrain` keyword allows users to constrain the cardinality of FHIR elements. Note that is the FHIR resource that is getting the constraint, not the CIMPL class. The cardinality is expressed in {min}..{max} form.

| Keyword | Example |
|----------|---------|
| `constrain` | `constrain explanation to 0..1` |

## Slicing

When fields are allowed to repeat on a target model (e.g., have cardinality with an upper bound greater than 1), it is common practice to specify which CIMPL elements are mapped to instances of that repeated field on the target model.  This process is called **slicing**. For example, the [FHIR Task resource](https://www.hl7.org/fhir/task.html) allows for an arbitrary number of `input` fields to be added.  To ensure that each `input` field is uniquely addressable by a parsing engine, those inputs need to be differentiated by a **discriminator**.

See the FHIR documentation for more detail about [slicing](https://www.hl7.org/fhir/profiling.html#slicing) and [discriminators](https://www.hl7.org/fhir/profiling.html#discriminator).

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

### Declaring a Discriminator

To perform any slicing, a discriminator needs to be declared, using `slice on` followed by the discriminator.

CIMPL supports the FHIR processing types for discriminators:

| Discriminator | Definition |
| ------------- | ---------- |
| value	| The slices have different values in the nominated element|
| exists |	The slices are differentiated by the presence or absence of the nominated element|
| pattern |	The slices have different values in the nominated element, as determined by testing them against the applicable ElementDefinition.pattern[x]|
| type |The slices are differentiated by type of the nominated element to a specifed profile|
| profile |	The slices are differentiated by conformance of the nominated element to a specifed profile (not recommended)|

### Moving the Slice Location
If the discriminator used for slicing is not the actual field that is being mapped to, `slice at` can allow you to explicitly define the location of the slice.

| Keyword | Example |
|----------|---------|
| `slice at` | `Members.Observation maps to related.target (slice at = related; slice on type)` |

In the above example, though the mapping is declared such that `Members.Observation maps to related.target`, the slicing is not occuring at `related.target`.  Instead, the `slice at = related` ensures that the slicing is occuring at the `related` element.

***
# Appendix A: Changes from CIMPL 5.x

For those who have created detailed models using CIMPL 5.0, there have been significant grammar changes to CIMPL 6.0.  

The table below summarizes these changes:

| Change Type | Change Description | CIMPL 5.0 Example | CIMPL 6.0 Example | Section |
|:---- |:----------|:---------------------- |:-------------------|:----------------- |
| New | keyword `only` eliminates all value choices except one | None | `FindingResult only concept` | [Only Constraint](#only-constraint) |
| New |keyword `Property` is required to define properties for an Entry or Element. |```0..1 TreatmentIntent```| ```Property:  TreatmentIntent 0..1``` | [Property Declaration](#property-declaration) |
| Replace | `EntryElement` keyword replaced by `Entry` | ```EntryElement: CourseOfTreatmentPerformed```| ```Entry:  CourseOfTreatmentPerformed``` | [Element Name Declaration](#element-name-declaration) |
| Replace | `Based on` keyword replaced by `Parent` | `Based on: Observation` | `Parent:  Observation` | [Inheritance](#inheritance) |
| Syntax change | Cardinality is specified _after_ the property or class name | `0..1 TreatmentIntent` | `Property:  TreatmentIntent 0..1` | [Cardinality](#cardinality) |
| Replace | `is` constraint for fixed values replaced by `=` | `FindingTopicCode is LNC#48676-1` | `FindingTopicCode = LNC#48676-1` | [Field Constraints](#field-constraints) |
| Replace | substitution of a more specific element derived from a parent element using `is type` keyword replaced by `substitute`. | `Specimen is type BreastSpecimen` | `Specimen substitute BreastSpecimen` | [Substitute](#substitute) |
| Replace | `code`, `Coding`, and `CodeableConcept` primitives are replaced by a new primitive `concept` | `Value: CodeableConcept from AttributionCategoryVS` | `Value: concept from AttributionCategoryVS` | [Primitives](#primitives) |
| Replace | `must be`, `should be`, `could be`, and `if covered` value set constraints are obsolete and replaced by `(required)`, `(preferred)`, `(extensible)`, and `(example)` | `Type from BreastSpecimenTypeVS if covered` | `Type from BreastSpecimenTypeVS (required)` | [Value Set](#value-set) |
| Replace | `ref()` is now obsolete. `value is type` replaced by `substitute` and bracket notation denoting value choices| `SourceSpecimen value is type ref(BreastSpecimen)` | `SourceSpecimen[Specimen] substitute BreastSpecimen` | [ References](#references) |


<br />

>**Note:** As of CIMPL 6.0, `must be`, `should be`, `could be`, and `if covered` value set constraints are obsolete and replaced by `(required)`, `(preferred)`, `(extensible)`, and `(example)`.

| CIMPL 5.0 Example | CIMPL 6.0 Example | Binding Strength |
|----------|---------|---------|
| `from`   | `Substance from SubstanceOfAbuseVS` | Required |
| `Priority must be from http://hl7.org/fhir/ValueSet/request-priority` |`Priority from http://hl7.org/fhir/ValueSet/request-priority (required)` | Required
| `1..1 Status should be from http://hl7.org/fhir/ValueSet/event-status` | `Status 1..1 // set cardinality separately`  <br/> `Status from http://hl7.org/fhir/ValueSet/event-status (preferred)`    | Preferred
| `CodeableConcept could be from http://hl7.org/fhir/ValueSet/medication-as-needed-reason` | `concept from http://hl7.org/fhir/ValueSet/medication-as-needed-reason (example)` | Example |
| `Category from http://hl7.org/fhir/ValueSet/goal-category if covered`| `Category from http://hl7.org/fhir/ValueSet/goal-category (extensible)` | Extensible |
