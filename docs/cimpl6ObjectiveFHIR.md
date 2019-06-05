# Objective FHIR User Guide

>**Note:** This documentation is in draft form.

_This is an introductory guide to Objective FHIR Version 1.0, implemented in the Clinical Information Modeling and Profiling Language (CIMPL). The Guide assumes some knowledge of CIMPL. If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md). If you're looking for a more in-depth introduction, try the [Tutorial](cimpl6Tutorial_detail.md). Or, see the [CIMPL 6 Reference Manual](cimpl6Reference.md)._

***

**Table of Contents**

[TOC]

***

## Overview

Objective FHIR ("OBF") is an [**object-oriented**](https://en.wikipedia.org/wiki/Object-oriented_programming) abstraction of FHIR. It provides modelers a way to define a detailed clinical information model by subclassing, extending, and constraining a pre-existing class library. These classes can then be translated automatically into FHIR profiles, FHIR Implementation Guides, data dictionaries, schemas, and other assets, using a choice of the three major FHIR versions: DSTU 2, STU 3, and R4.

### Philosophy

OBF classes resemble FHIR R4, but differ in carefully considered ways that increase consistency and reusability of the resulting models and profiles. Objective FHIR allows data structures of all sorts to be reused. This means that individual data elements and frequently-occurring structures can be defined once and used repeatedly.

Objective FHIR addresses one of the frequent criticisms of FHIR, specifically its [lack of horizontal consistency](https://wolandscat.net/2019/05/05/a-fhir-experience-consistently-inconsistent/). FHIR not only uses different _names_ for equivalent things in different resources, but often, altogether different modeling approaches. This is a result of having resources managed by separate groups. OBF creates a layer that smooths over many of these differences, not for aesthetic or theoretical reasons, but to make the whole framework easier to learn, enable greater code reuse, and most importantly, to make the resulting clinical models **more interoperable**.

OBF also insulates modelers from differences between FHIR versions. The OBF classes are based on FHIR R4, but the same content is mapped to previous versions, specifically, DSTU 2 and STU 3. This means you can model once and publish the same content across multiple FHIR versions.

### Meta-Model

Objective FHIR has been developed using the Clinical Information Modeling and Profiling Language (CIMPL). CIMPL is a very powerful, FHIR-aware, high-level language for creating clinical models. Expressing the model in CIMPL means that Objective FHIR models can automatically be turned into FHIR Profiles, Implementation Guides, data dictionaries, and other useful artifacts, in multiple FHIR versions.

Conceptually, there is nothing that prevents the same model from be expressed in other formalisms, some of which are mentioned in the [Appendix](#Appendix:-Relationship-to-Other-Initiatives). However, OBF in CIMPL is a complete, ready-to-use solution.

### Mapping to FHIR

One of the significant benefits of the OBF framework, compared to using CIMPL alone, is that mapping to FHIR has already been done for you. In most cases, you don't have to worry about mapping, and you can focus entirely on modeling. The only exceptions are when you create a new class that doesn't inherit from a pre-mapped OBF class, or add an extension to a pre-mapped class that requires mapping to a nested extension.

### Coverage

Not all FHIR R4 resources are covered by Objective FHIR. We are working to expand the coverage. The model documentation is the best source to determine if OBF covers your needs.

### Naming

Attribute names in OBF may differ from FHIR names, but if they do so, it is usually to make the meaning of the attribute more explicit. OBF names are meant to be meaningful outside of the context of a single class. For example, the attribute `period` in `Encounter` is not entirely self-explanatory. To be more reusable, OBF renames it `OccurrencePeriod`, to better reflect its definition. Coupled with a different event, such as a procedure, the renamed attribute's meaning is more clear. Although an attribute name is rarely a sufficient definition, OBF moves the needle in that direction.

### Subclassing

To continue the scenario above, suppose you want a more specific concept of `OccurrencePeriod` applied to a surgical procedure. In this case, we can subclass:

```
Element:  SurgicalProcedureOccurrencePeriod
Parent:  OccurrencePeriod
Description: The period of time for a surgery, from the first incision time to the last incision close time, as defined by https://manual.jointcommission.org/releases/archive/TJC2010B/DataElem0127.html.
```
The structure and content of `OccurrenceTime` is inherited by the new class (cardinalities, data type, the fact that the start time must be less than the end time, etc.), so repeating that information is unnecessary. [Don't repeat yourself (DRY)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) is a major benefit of inheritance. The DRY principle is stated as "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system".

## Subclassing, Part 2

That's a trivial example, but almost everything you do in OBF will involve creating new classes from existing ones. Here's a more involved example:

```
Entry:             GenomicsReport
Parent:            DiagnosticReport
Description:       "Genetic analysis summary report. The report may include one or more tests, with two distinct test types... (truncated)"
Property:          SpecimenType 0..1
Property:          RegionStudied 0..*
                   Code from https://www.ncbi.nlm.nih.gov/gtr (preferred)
                   Category 1..1
                   Category = DS#GE "Genetics"
                   Observation
                      includes GeneticVariantFound 0..* 
                      includes GeneticVariantTested 0..*
                   SpecimenType from GeneticSpecimenTypeVS (extensible)
```
Although the purpose of this guide is not to teach CIMPL, it's worth pulling this apart:

* On the first two lines, we're creating a new class, `GenomicsReport`, based on `DiagnosticReport`. `Entry` is a CIMPL building block that roughly corresponds to a FHIR resource, and `Parent` obviously identifies the parent class.
* Following the `Description`, two new properties are introduced: `SpecimenType` and `RegionStudied`. Note that you don't need to fuss with extensions -- that will happen automatically when the class is mapped to FHIR.

After the keyword section, there are several constraint statements. Without delving into details, these statements say:

* The Code identifying the report should perferrably come from the [Genetic Test Registry](https://www.ncbi.nlm.nih.gov/gtr).
* The Category will occur exactly once, and will be fixed to the code `GE` (Genetics) drawn from a code system aliased to `DS` (mapped elsewhere to <http://terminology.hl7.org/CodeSystem/v2-0074>).
* The `Observation` attribute should include zero or more GeneticVariantFound observations and zero or more GeneticVariantTested observations. *Did you guess that we've just sliced an array?*
* Finally, the SpecimenType, which we've just added as a new class property, is extensibly bound to the value set `GeneticSpecimenTypeVS`.

We now have a general-purpose genomics report. We can use this class as-is, in the form of a FHIR profile, or as a parent for defining more specific genomics reports, e.g., `AncestryDotComGenomicsReport`.

### Comparison Between Profiling Tools
In FHIR terms, subclassing is akin to profiling profiles, which can be achieved with a number of tools, notably [Forge](https://fire.ly/products/forge/) and [Trifolia](https://trifolia-fhir.lantanagroup.com/home). Both these tools are extremely well-done, and generously supported by commercial entities. 

Both tools are essentially visual editors for StructureDefinitions, the "assembly language" of FHIR. By contrast, CIMPL is like a high-level programming language. The source code for your project is the CIMPL itself. The difference is that creating and maintaining a complex project is **much, much, much** easier when you use a language, compared to a visual editor. That's why programming languages are always text, and visual programming has had comparatively little uptake.

You will find that when projects grow to a certain size, many of your activities will involve revising, refactoring, renaming, and redoing. Global search and replace across text files will be an indispensible weapon. Text also opens the door to meaningful source code control. The ability distribute work across multiple branches, compare changes, and merge contibutions allows your projects to scale in ways that visual editors can't support.

### Primitives and Complex Data Types

The primitive types used in OBF are those defined in CIMPL, which correspond exactly to FHIR primitives, except for [the way CIMPL handles coded types](cimpl6Reference.md#concept-codes).

The complex data types in OBF are the same as FHIR R4. They are found in the `obf.datatype` namespace. Since complex types like Quantity are ubiquitous, you will almost certainly need to import the `obf.datatype` into your namespace. This is done using the `Uses` keyword.

## Class Hierarchy Overview
The heart of Objective FHIR is its class hierarchy. At the top of the hierarchy are classes Resource and DomainResource, same as FHIR. Below that are several classes that don't occur in FHIR. The purpose of these classes is to uniformly define properties that are used across many classes/resources. For example, many classes should have an author. It shouldn't be left to individual classes to define `author`, because inevitably, it will be defined differently, or worse, omitted. Inheriting from a common parent prevents that.

FHIR's approach is to define certain patterns, such as the [request pattern](https://www.hl7.org/fhir/request.html). However, FHIR stops short of actually implementing these patterns across resources. Implementations can't assume all requests are the same, and can't write general code for processing requests. Instead, each type of request must be implemented as a one-off.

In addition to high-level classes, Objective FHIR provides some extended classes you can use as ready-made extension points. An example is `QuantitativeLaboratoryObservation`. Moreover, because Objective FHIR is organized into namespaces that can be shared, you can re-use user-created classes such as `CancerCondition` or `WoundCondition`.

In this section, we describe some of the key classes in Objective FHIR. For full details, see the [model documentation](#model-documentation)





## Model Documentation

## Future
Moreover, the classes you define can be used directly in implementations, leveraging automatically generated Javascript* classes and methods that, at runtime, translate FHIR resources to and from the logical model classes. This allows implementers to use object oriented programming in much more powerful ways than available in native FHIR, with its flat class structure and complex extension representations.






## Appendix: Relationship to Other Initiatives 

### Modeling Formalisms

Conceptually, Objective FHIR could be expressed in other modeling frameworks, besides CIMPL. Some of the potential frameworks include:

* [Unified Modeling Language (UML)](https://www.uml.org/) for structure coupled with [Object Constraint Language](https://www.omg.org/spec/OCL) for constraint representation,
* [Basic Meta-Model](https://specifications.openehr.org/releases/LANG/latest/bmm.html) for class hierarchy, coupled with [Archetype Description Language](https://specifications.openehr.org/releases/AM/latest/ADL2.html) for constraint representation,
* [Clinical Element Models](http://www.clinicalelement.com/#/).

Objective FHIR is an [open source project](https://github.com/standardhealth/shr-models), and we welcome contributions, especially expressions of OBF in other meta-modeling languages.

## Class Hierarchies

* [CIMI Reference Model](http://models.opencimi.org/)
* FHIR Patterns

