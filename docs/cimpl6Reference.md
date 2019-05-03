# CIMPL 6.0 Reference Documentation

>**Note:** This documentation is in the process of being further expanded and is an actively updating document.

_This is a comprehensive guide to CIMPL 6.0 syntax.  If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md). If you're looking for a more in-depth introduction, try the [Tutorial](cimpl6Tutorial_detail.md)._

***

**Table of Contents**

[TOC]

***

# General Overview

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a domain specific language for defining clinical information models. It was designed to be simple and compact, while coming with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles and implementation guides (IG).

# CIMPL File Types

### Whitespace
Repeated whitespace is generally not meaningful within CIMPL files. So this:
```
Grammar:    DataElement 6.0
Namespace:  myExampleNamespace
```
...is equivalent to this:
```
Grammar:        DataElement 6.0

  Namespace:    myExampleNamespace
```
### Naming Conventions
#### File names
* Data Element files: `namespace.txt`
* Value Set files: `namespace_vs.txt`
* Mapping files: `namespace_map.txt`

>**Note:** "Any periods in the `namespace` should be replaced by underscores"

For example:
If there is a namespace called `odh.occupation`, the value set file name could be called `odh_occupation_vs.txt`.

#### Namespaces
A namespace conventionally is denominated by the authoring organization and the broad category of elements the namespace defines, delineated by a period. For example:
```
Namespace: shr.oncology
```
>**Note:** "The organization name is recommended but not required (e.g., `Namespace: oncology` is also allowed)."

#### Elements
Elements are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase). For example:
```
Element: GeologicalLocation
```
>**Note:** "Letters, numbers, and hyphens are allowed. However, it is required that an element name begins with an uppercase letter."

#### Value Sets
Value Sets are conventionally defined in [PascalCase](http://wiki.c2.com/?PascalCase), with an additional "VS" suffix. For example:
```
ValueSet: HomeEnvironmentRiskVS
```
>**Note:** "It is required that an Value Set name begins with an uppercase letter."

#### Code Sets
Within a custom value set, the individual codes are conventionally written in lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case). For example:
```
ValueSet:          HomeEnvironmentRiskVS
#smoke_detectors   "No Smoke Detectors"
#radiation         "Radon or other radiation source"
#swimming_pool     "Swimming Pool"
```

### Ordinality
CIMPL does not enforce or require ordinality in any of its definitions, i.e. an element defined at the bottom of the file can be used as field on an element defined at the top.

There is, however, a requirement for a file's header information to be at the top of the file, before any definitions.
See: [File Headers](#file-headers) for more information

### Naming Collisions
CIMPL does not support duplicate element names within a namespace. However, you are allowed to reuse an element name across different namespaces. In the case that you come across a collision due to an included namespace in `Uses`, you have to refer to the element by its [Fully Qualified Name](#fqn).

### Versioning
CIMPL follows the [Semantic Versioning](https://semver.org) convention (MAJOR.MINOR.PATCH).

* MAJOR: A breaking change. This is reserved for drastic grammar overhauls. Does not support backwards compatibility to other major version releases.
* MINOR: Feature updates. Allows for backwards compatability within the major version.
* PATCH: Minor updates and bug fixes. Allows for backwards compatability within the major version.

CIMPL is currently on major version 6.

For a full changelog, see the [Release Notes](https://github.com/standardhealth/shr-cli/releases).

<!-- TODO: Find MAJOR release differences -->

### Comments
CIMPL follows [JavaScript syntax](https://www.w3schools.com/js/js_comments.asp) for code comments:
```
// Use a double-slash for comments on a single line

/*
Use slash-asterisk and asterisk-slash for larger block comments.
These comments can take up multiple lines.
*/
```

***

## Header

Each CIMPL file contains header information that describes the file type, its scope and inheritance, and its purpose.  It also allows developers to create aliases for urls and code systems.

An example file type header for a CIMPL Data Element File is shown below:
```
Grammar:        DataElement 6.0
Namespace:      shr.environment
Description:    "The SHR Environment domain contains definitions related to surroundings experienced by the person of record."
Uses:           shr.core, shr.base, shr.allergy, shr.observation, shr.medication
Codesystem:     LNC = http://loinc.org
```

## Grammar
The `Grammar` keyword is used to define the file type and the version of CIMPL used in development/parsing.  This is required for all CIMPL files.

| Keywords | Example |
|----------|---------|
| `Grammar` | `Grammar: DataElement 6.0` |
| `Grammar` | `Grammar: ValueSet 6.0` |
| `Grammar` | `Grammar: Map 6.0` |

There are three possible options for file type:

1. `DataElement`
2. `ValueSet`
3. `Mapping`

The version details the MAJOR and MINOR version the file was written for. The language is backwards compatible within the MAJOR version number, i.e. `DataElement 6.0` files will compile without problems on the `5.9.0` tooling. For more information, see [Versioning](#versioning).

## Namespace Declaration
The `Namespace` keyword defines the scope of the file and serves as a broad grouping of `DataElement`, `ValueSet` and `Map` files within a domain.

| Keywords | Example |
|----------|---------|
| `Namespace` | `Namespace: shr.environment` |

The namespace can be any number of lowercase period delimited words.

Best practice is to follow the naming convention pattern of `organization`.`domain` (followed by subdomains if necessary).

The tooling will not allow for duplicate namespaces.

>**Note:** "While it is best practice to use only lower case letters, the strict requirements only require the word to begin with a lowercase letter. The rest of the namespace allows for mixed casing and hyphens"

### File Description
The `Description` keyword provides the user the ability to define the purpose of the file, within the confines of a project.

| Keywords | Example |
|----------|---------|
| `Description` | `Description: "The SHR Environment domain contains definitions related to surroundings experienced by the person of record."` |

The namespace description is optional, but recommended.

Best practice is to write human readable text using the [ASCII](https://en.wikipedia.org/wiki/ASCII) standard.

There is no strict requirement for unique descriptions, and as such you will not run into description collisions.

>**Note:** "While the CIMPL tooling will allow for any pattern of [Unicode](http://unicode.org/standard/WhatIsUnicode.html) characters within enclosed double quotation marks (`"`), certain exporters such as the FHIR exporter will object to non-[ASCII](https://en.wikipedia.org/wiki/ASCII) text.)"

## Uses
The `Uses` keyword provides a list of the namespaces used within the current namespace. Namespace inclusion allows you to use `DataElement`s and `ValueSet`s defined in other namespaces.

| Keywords | Example |
|----------|---------|
| `Uses` | `Uses: shr.core, shr.base, shr.allergy, shr.observation, shr.medication` |

In CIMPL version 6 and below, it is *required* that you import `shr.core` and `shr.base` for the tooling to run.

 _(In the case of name collisions, you will have to indicidate `DataElement`s by their [Fully Qualified Name](#fqn))._

The order of inclusion has no effect.

## Url Shorthands
The `Path` keyword allows for abbreviations of long urls.

| Keywords | Example |
|----------|---------|
| `Path` | `Path: FHIR = http://hl7.org/fhir/ValueSet` |

This functionality is completely optional, but it is provided to make authoring easier.

The abbreviation by convention should be be an all UPPER CASE word. _(Although, the tooling does allow for numbers and hyphens after the first letter)._

## Codesystems
The `Codesystem` keyword allows for abbrevations of codesystems. In order to use code systems in a DataElement definition or constraint, it is necessary to first define them in the file header (this is largely due to the complexity of codesystems' direct code references).

| Keywords | Example |
|----------|---------|
| `Codesystem` | `Codesystem: LNC = http://loinc.org` |

You must define each codesystem you use in separate definitions across multiple lines, e.g.

```
Codesystem: LNC = http://loinc.org
Codesystem: SCT = http://sct.org
```

The abbreviation by convention should be be an all UPPER CASE word. _(Although, the tooling does allow for numbers and hyphens after the first letter)_.

# Data Element File

## Sample Data Element File
```
Grammar:        DataElement 6.0
Namespace:      shr.environment
Description:    "The SHR Environment domain contains definitions related to surroundings experienced by the person of record."
Uses:           shr.core, shr.base, shr.allergy, shr.observation, shr.medication
Codesystem:     LNC = http://loinc.org

Entry:          FinancialSituation
Concept:        MTH#C0516918
Parent:         Panel
Description:    "Measures of the ability of the subject to obtain and pay for the necessities of life."
Value:          concept is MTH#C0516918
Property:       AnnualIncome    0..1
Property:       IncomeSource    0..*
                PanelCode is LNC#83335-0
```

## Element Name Declaration
The `Element` keyword allows users to define the name of the element and the lines following the `Element` keyword comprise the definition of that element.

| Keywords | Example |
|----------|---------|
| `Element` | `Element: Observation` |
| `Entry`  | `Entry: FinancialSituation` |
| `Abstract Element` | `Abstract Element: Any` |

_Note the spacing discrepancy between `Entry` and `Abstract Element`.  This has no semantic meaning but is inherent to the syntax._

These keywords are used to define data elements with three different levels of expression in the target output:
* `Entry` is used to define elements that will be used directly by the target output, i.e., data will be _entered_ directly into this item by and end user once it has been mapped.  For FHIR exports, these items will be created as 'Primary Profiles'
* `Element` is used to define supporting data elements for `Entry`s.  These data elements will be present in the target output as, but they are not the top-level items that are exported. For FHIR exports, these items will be created as 'Supporting Profiles'.
* `Abstract Element` is used to define elements that are used only within the CIMPL mapping to create basic objects that will be leveraged by other data elements.  They will not be present in the target mapping.

### Property Declaration

The keyword `Property` is required to define properties for an Entry or Element.
```
Entry:          FinancialSituation
Concept:        MTH#C0516918
Parent:         Panel
Description:    "Measures of the ability of the subject to obtain and pay for the necessities of life."
Value:          concept is MTH#C0516918
Property:       AnnualIncome    0..1
Property:       IncomeSource    0..*
                PanelCode is LNC#83335-0
```

### Fully Qualified Names

In addition to its declared name, all elements also have a unique identifying `Fully Qualified Name` (`FQN`).

The FQN is a combination of the element's namespace and its declared name (through concatention delimited by a period). For example, the element `Observation` in the namespace `shr.finding` has the following FQN:
```
shr.finding.Observation
```

>**Note:** "While rare and considered bad practice, naming collisions between elements across multiple namespaces is possible. In the case of a naming collision (such as when a namespace `Uses` multiple namespaces that define an element with the same declared name), you would have to refer to the specific unique FQN instead of the declared name in element, field, and value definitions."

## Concept Code
The `Concept` keyword is used to define the concept code for the element.  These are numerical codes that identify clinical terms, primitive or defined, organized in hierarchies.

| Keywords | Example |
|----------|---------|
| `Concept` | `Concept:  MTH#C0516918` |

Concept codes for the meaning of the defined structure (SNOMED CT and LOINC codes, as an example). While optional, it is recommended you define the concept of an element.

To see how it relates to FHIR output, see [StructureDefinition.keyword](http://hl7.org/fhir/structuredefinition-definitions.html#StructureDefinition.keyword).

## Inheritance
Inheritance provides the parent element from which the properties are inherited. This is designated by the keyword `Parent` and declares that the class inherits the properties and constraints from another class. This keyword is optional. This works similar to class inheritance in object orientated programming. Additionally, mappings are inherited alongside properties.

| Keywords | Example |
|----------|---------|
| `Parent` | `Parent: Observation` |
| `Parent` | `Parent: Foo` |

Multiple inheritance is currently removed from CIMPL support.

Selective inheritance is achievable through 'zeroing out' inherited properties. See [Cardinality](#cardinality)

## Description
The textual description of the element.

| Keywords | Example |
|----------|---------|
 `Description` | `Description: "Measures of the ability of the..."` |

The element description is optional, but recommended.

Best practice is to write human readable text using the [ASCII](https://en.wikipedia.org/wiki/ASCII) standard.

There is no strict requirement for unique descriptions, and as such you will not run into description collisions.

To see how it relates to FHIR output, see [StructureDefinition.description](http://hl7.org/fhir/structuredefinition-definitions.html#StructureDefinition.description).

>**Note:** "While the CIMPL tooling will allow for any pattern of [Unicode](http://unicode.org/standard/WhatIsUnicode.html) characters within enclosed double quotation marks (`"`), certain exporters such as the FHIR exporter will object to non-[ASCII](https://en.wikipedia.org/wiki/ASCII) text.)"

## Fields

### Value

Defines what the element is. Values can be declared as either a DataElement (e.g., CodeableConcept) or a [primitive](#primitives) (e.g., string), as shown here.

| Keywords | Example |
|----------|---------|
| `Value` | `Value: concept` |
|         | `Value: string` |

Value is an _optional_ special field on an element that correspond more heavily to what the element _is_.

You are also allowed to [constrain the value](#field-constraints) similar to other fields.

On the mapping side, you are able to map the value like any other field.

Value defaults to a [cardinality](#cardinality) of `1..1`, but this can be changed, e.g.:
```
Value: Quantity 0..1 
```

### Primitives
CIMPL allows users to define fields as the following primitive data types:

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
* [code](https://www.hl7.org/fhir/datatypes.html#code)
* [oid](https://www.hl7.org/fhir/datatypes.html#oid)
* [id](https://www.hl7.org/fhir/datatypes.html#id)
* [markdown](https://www.hl7.org/fhir/datatypes.html#markdown)
* [unsignedInt](https://www.hl7.org/fhir/datatypes.html#unsignedInt)
* [positiveInt](https://www.hl7.org/fhir/datatypes.html#positiveInt)
* [xhtml](https://www.hl7.org/fhir/datatypes.html#xhtml)

<br />

### Field
Object properties on the element.

| Keywords | Example |
|----------|---------|
|| `AnnualIncome  0..1` |

In most cases, you refer to an element by its name. In the rare case of a naming collision, you must refer to it by its [Fully Qualified Name](#fully-qualified-name).

### Inherited Field
Object properties on the element inherited from the parent or other elements.

| Keywords | Example |
|----------|---------|
|| `PanelCode = LNC#83335-0` |

### Inherited Value
Object value on the element inherited from the parent or other elements.

In CIMPL 6.0, `Value` shall only be used when referring to the value element inside an Element or Entry where that value is defined, or in mapping the value to a target. 

| Keywords | Example |
|----------|---------|
| `Value` | `Value only concept` |
|  | `Value from RadiationProtocolVS` |


### References

**CIMPL 6.0 "References" Grammar** 

In order to refer to an element as a class (i.e. a pointer as opposed to an instance), you can encapsulate the element name using brackets, `[]`.  This is helpful for defining complex objects with child elements that are already defined as parts of existing data elements.

The `[]` syntax should only encapsulate the element name. All other fields should be applied normally to the group.

>**Note:** "CIMPL 5.0 Grammar use of the keyword `ref()` is now obsolete and replaced with `[]`."

| CIMPL 5.0 Grammar | CIMPL 6.0 Grammar | Remarks |
| ----------------- | ----------------- | -------- |
| `SourceSpecimen value is type ref(BreastSpecimen)` | `SourceSpecimen[Specimen] substitute BreastSpecimen` |  |
| `DistanceFromLandmark.Distance.Units is UCUM#cm` | `DistanceFromLandmark[Distance].Units = UCUM#cm` | In this example, `Distance` is designated as the `Value` of `DistanceFromLandmark`. `Units` is a property of `Distance`.  |

**MLT_TODO:** _How do we translate the following from CIMPL 5.0 to CIMPL 6.0_

| Keywords | Example |
|----------|---------|
| `ref()` | `1..1  ref(Role)` |
| `ref()` | `0..1  PartOf value is type ref(Organization)` |
| `ref()` | `Value: ref(Finding)` |
| `ref()` | `Value: 0..* ref(Observation)` |

`ref()` can be used to specify that a reference is either the child element of a data element (e.g., `1..1  ref(Role)`) or the `Value` of an existing child element (e.g., `0..1  PartOf value is type ref(Organization)`).

<br />

## Field Constraints

Fields can be constrained to further specify their acceptable values using [cardinalities](#cardinality) and keywords, e.g.:
* [`=`](#fixed-value)
* [`substitute`](#substitute)
* [`from`](#value-set)
* [`includes`](#includes-code)
* [`only`](#only)

<br />

### No Constraint
Fields can exist without any constraints.  They can also be constrained to a data type (or set of data types) without further constraining the possible values within that data type.

| Keywords | Example |
|----------|---------|
|| `postalCode 0..1` |
|| `Value: concept` |
|| `Value: unsignedInt or positiveInt` |

<br />

### Cardinality
Cardinality defines the number of instances should exist for a value.

The first digit indicates the minimum quantity, and additionally by setting it to `0` or to `1`, has the effect of making a field optional or required.

The second digit expresses the upper bound. To express no upper bound, a `*` can be used in place of a number.

When constraining an inherited field, you can only retain the previous cardinality, or constrain it to be narrower.

_**Note**: By constraining it to 0..0, you can remove an inherited field from an element._

| Keywords | Example |
|----------|---------|
|| `LanguageUsed  0..*` |
|| `GovernmentIssuedID  1..2` |

### Fixed Value

Fixed values are designated with an `=` operator.

The `is` keyword limits the field to a specifc value (e.g., a `code`, a specific `string`, or a `boolean` value).

| Keywords | Example |
|----------|---------|
|  | `ObservationCode = LNC#82810-3` |

>**Note:** "The `=` operator locks the code for all instances of the data element.  If you only need to lock the code for one particular mapping, do so using the [`fix`](#fix) keyword in the mapping file."

### Substitute
The `substitute` keywords constrains to a subclass of an element or the choices in a given value set.

| Keywords | Example |
|----------|---------|
| `substitute` | `SourceSpecimen[Specimen] substitute BreastSpecimen` |

**Use case:** If we have an abstract element, and we are making a more specific version of that element, we also want to make its fields more specific. For instance, if we have a DataElement `Person` which has a field `Pet`, and we making a more specific person `DogOwner` and we want to further specify its pet. In this case, we could constrain by using the `substitute` keyword. For example, `Pet substitute Dog`.

### Only

The keyword `only` binds only one logical choice to a property or value.

| Keywords | Example |
|----------|---------|
| `only` | `PlannedProtocol only Protocol` |

_*>**Note:** "`only` shall never be preceded by bracketed term, because intrinsically, it applies to all value choices."

### Value Set
The `from` keyword limits the value of a field to be from a specific value set, with different levels of requirement.

## Value Set Binding

### Required

```
ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical (required)
```

Indicates that the code _must_ come from the specified value set.  No other codes are allowed.

### Extensible

```
ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical (extensible)
```

Indicates that the code must come from the specified value set _if the value set contains a relevant code_ to represent the concept; otherwise a code outside the value set may be chosen.

### Preferred

```
ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical (preferred)
```

Indicates that the code should ideally come from the specified value set, but codes outside the value set may also be chosen.

### Example

```
ClinicalStatus from http://hl7.org/fhir/ValueSet/condition-clinical (example)
```

Indicates that the code may come from anywhere; the specified value set is for example purposes only.


The binding strengths correspond to [FHIR binding strengths](https://www.hl7.org/fhir/valueset-binding-strength.html) and have the same meaning:

* **Required**: To be conformant, the concept in this element SHALL be from the specified value set
* **Preferred**: Instances are encouraged to draw from the specified codes for interoperability purposes but are not required to do so to be considered conformant.
* **Example**: Instances are not expected or even encouraged to draw from the specified value set. The value set merely provides examples of the types of concepts intended to be included.
* **Extensible**: To be conformant, the concept in this element SHALL be from the specified value set if any of the codes within the value set can apply to the concept being communicated. If the value set does not cover the concept (based on human review), alternate codings (or, data type allowing, text) may be included instead.

### Includes concept allowable entries

For fields that are already defined with a data type of `concept`, the `includes` keyword expands the allowable entries for that field.

_**TODO:** Modify the examples below for CIMPL 6.0_

| Keywords | Example |
|: ----------|: ---------|
| `includes` | `Category includes LNC#54511-1 "Behavior"` |
| `includes` | `ProblemCategory includes #adverse_reaction` |

<!-- * TODO: confirm the functionality of this constraint. Chris? -->

The first example above expands `Category` to include `54511-1` from the LOINC codesystem. Additionally, it is given a brief textual description of `"Behavior"`.

The second example above expands the allowable codes in `ProblemCategory` to include `#adverse_reaction`. Unlike the first example, the value set does not precede the code, because it has defined on `ProblemCategory`'s parent using a [`from`](#value-set) statement. Additionally, there is no textual description, as that is optional.

### Includes Type

Specifies the quantity of particular types of objects within the value.

| Keywords | Example |
|----------|---------|
| `includes` | `Foo includes 	PhysicalActivityLevel 1..1` |

This is useful in further specifying fields that are analagous to lists.

In the above example, the field `Foo` is _required_ to include `1` `PhysicalActivityLevel`.

>**Note:** "As of CIMPL 6.0, the use of the `Includes` keyword for specifying inclusions subclasses or properties is now obsolete and discouraged for use."

For example, the following statement block is no longer supported.

**CIMPL 5.0: (no longer supported)**
```
Element: VitalSign
1..* BloodPressurePanel
    includes 1..1 SystolicPressure
    includes 1..1 DiastolicPressure
```

**CIMPL 6.0:**
```
Element: VitalSign
 BloodPressurePanel  1..*
    SystolicPressure  1..1
    DiastolicPressure 1..1
```

>**Note:** "The collective cardinality of all inclusions must fit within the field, i.e. if you are including 3 distinct elements, then the field must have a maximum cardinality greater than `3` (like `*`)."

## Array Population

```
Category += OBSCAT#laboratory
```

Indicates that _Category_, an element of multiple cardinality, must contain the specified code.  The `+=` operator works with any fixed value type.

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

| Keywords | Example |
|----------|---------|
| `ValueSet` | `ValueSet: ProposedStatusVS` |

>**Note:** "By convention, all value sets should be Pascal case and end with `VS`."

### Value Set Description
Defines the name of the value set.

| Keywords | Example |
|----------|---------|
| `Description` | `Description: "The status of a proposal."` |

### Code-Value Declaration
Define the code or value.

| Keywords | Example |
|----------|---------|
|| `#proposed "The proposal has been proposed, but not accepted or rejected."` |
|| `CAP#29915		"None/Negative"`|

Each value is prefaced by a hash symbol (`#`) and followed by a description enclosed in quotation marks. By convention, each value should use lowercase [snake_case formatting](https://en.wikipedia.org/wiki/Snake_case).

This can either be a new user defined `code`, or it can be individual codes from separate codesystems, such as the `CAP#29915` example, in which `29915` is an existing code within the `CAP` codesystem.

### Includes

Extends a custom defined `ValueSet` to include codes from other codesystems.

| Keywords | Example |
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

| Keywords | Example |
|----------|---------|
| `maps to` | `FinancialSituation maps to DiagnosticResult:` |


In CIMPL map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in your target output.

If no FHIR mapping is defined for an element and it inherits no mappings from its parents, it defaults to a mapping to the [Basic resource](https://www.hl7.org/fhir/basic.html).  In general, this is not recommended because it has no inherent semantic meaning for implementers.  To promote interoperability and reuse, CIMPL modelers are encouraged to find target models that already exist and use constraints or extensions to modify them according to project needs.

## Field Mapping
Maps a CIMPL element field to a target field.

| Keywords | Example |
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

| Keywords | Example |
|----------|---------|
| `fix` | `fix status to #completed` |

_Note about syntax: The element being fixed on left hand side of this syntax is a FHIR element, not a CIMPL element. The right hand syntax allows for any code or primitive value to be fixed._

## Constrain
The `constrain` keyword allows users to constrain the cardinality of elements in output files.

| Keywords | Example |
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

| Keywords | Example |
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

| Keywords | Example |
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

<br />

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

>**Note:** "As of CIMPL 6.0, `must be`, `should be`, `could be`, and `if covered` value set constraints are obsolete and replaced by `(required)`, `(preferred)`, `(extensible)`, and `(example)`"

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