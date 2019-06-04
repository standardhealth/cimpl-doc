# Objective FHIR User Guide

>**Note:** This documentation is in draft form.

_This is an introductory guide to Objective FHIR Version 1.0. Objective FHIR is implemented in the Clinical Information Modeling and Profiling Language (CIMPL). The Guide assumes some knowledge of CIMPL. If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md). If you're looking for a more in-depth introduction, try the [Tutorial](cimpl6Tutorial_detail.md). Or, see the [CIMPL 6 Reference Manual](cimpl6Reference.md)._

***

**Table of Contents**

[TOC]

***

## Overview

Objective FHIR ("OBF") is an [**object-oriented**](https://en.wikipedia.org/wiki/Object-oriented_programming) abstraction of FHIR. It provides modelers a way to define a detailed clinical information model by subclassing, extending, and constraining a pre-existing class library. These classes can then be translated automatically into FHIR profiles, FHIR Implementation Guides, data dictionaries, schemas, and other assets, using a choice of the three major FHIR versions: DSTU 2, STU 3, and R4.

### Philosophy

OBF classes resemble FHIR R4, but differ in carefully considered ways that increase consistency and reusability of the resulting models and profiles. Objective FHIR is implemented in CIMPL (Clinical Information Modeling and Profiling Language), a language that allows data structures of all sorts to be reused. This means a frequently-occurring structures like BodySite can be defined once and used repeatedly.

Objective FHIR addresses one of the frequent criticisms of FHIR, specifically its [lack of horizontal consistency](https://wolandscat.net/2019/05/05/a-fhir-experience-consistently-inconsistent/). FHIR not only uses different _names_ for equivalent things in different resources, but often, uses altogether different modeling approaches. This is almost certainly a result of having resources managed by separate groups. OBF creates a layer that smooths over many of these differences, not for aesthetic or theoretical reasons, but to make the whole framework easier to learn, to enable greater code reuse, and most importantly, to make the resulting clinical models **more interoperable**.

OBF also insulates modelers from differences between FHIR versions. The OBF classes are based on FHIR R4, but the same content is mapped to previous versions, specifically, DSTU 2 and STU 3. This means you can model once and publish the same content across multiple FHIR versions.

### Meta-Model

Objective FHIR has been developed using the Clinical Information Modeling and Profiling Language (CIMPL). CIMPL is a very powerful, FHIR-aware, high-level language for creating clinical models. Expressing the model in CIMPL means that your models can automatically be turned into FHIR Profiles, Implementation Guides, data dictionaries, and other useful artifacts, in multiple FHIR versions.

Conceptually, there is nothing that prevents the same model from be expressed in other formalisms, some of which are mentioned in the [Appendix](#Appendix:-Relationship-to-Other-Initiatives), but Objective FHIR in CIMPL is a complete, ready-to-use solution.

### Mapping to FHIR

One of the significant benefits of the OBF framework is that mapping to FHIR has already been done for you. In most cases, you don't have to worry about mapping at all. The only exceptions are when you create a new class that doesn't inherit from a pre-mapped OBF base class, or add an extension to a pre-mapped class that requires mapping to a nested extension.

### Coverage

Not all FHIR R4 resources are covered by Objective FHIR. We are working to expand the coverage. The model documentation is the best source to determine if OBF covers your needs.

### Naming

Attribute names in OBF sometimes differ from FHIR names, but if they do so, it is usually to make the meaning of the attribute more explicit. OBF names are meant to be meaningful outside of the context of a single class. For example, the attribute `period` in `Encounter` needs additional description, but far more so if removed from the encounter context. To be more reusable, OBF redefines it as `OccurrencePeriod`, so when coupled with a different event, perhaps the performance of a procedure, the meaning (occurrence period of the procedure) is fairly specific and self-explanatory. Although a name by itself can never precisely define an attribute or class, OBF moves the needle in that direction.

### Subclassing

To continue the scenario above, if `OccurrencePeriod` needs a tighter definition in the context of a surgical procedure, you can do it through subclassing. Specifically, you can define `SurgicalProcedureOccurrencePeriod` like this:

```
Element:  SurgicalProcedureOccurrencePeriod
Parent:  OccurrencePeriod
Description: The period of time for a surgery, from the first incision time to the last incision close time, as defined by https://manual.jointcommission.org/releases/archive/TJC2010B/DataElem0127.html.
```
The datatype and other constraints of `OccurrenceTime` are inherited (e.g., cardinalities, start time must be before the end time, etc.), so that's all there is to it.

That's a trivial example, but almost everything you do in OBF will involve creating new classes from existing ones. Here's a less trivial example:

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
Although the purpose of this guide is not to introduce CIMPL, it's worth pulling this apart:

* The two lines define the class. `Entry` is a CIMPL building block that roughly corresponds to a FHIR resource, and `Parent` identifies the parent class. Clearly, we're creating a new class, `GenomicsReport`, based on `DiagnosticReport`.
* Following the `Description`, we add two new properties that don't exist in the parent: `SpecimenType` and `RegionStudied`. Don't worry about extensions -- that will happen automatically.
* After the two `Property` statements, there are several constraint statements. Without delving into too much detail, these statements say:
    * The Code for the report should perferrably come from the [Genetic Test Registry](https://www.ncbi.nlm.nih.gov/gtr).
    * The Category will occur exactly once, and will be fixed to the code `GE` (Genetics) drawn from a code system aliased to `DS` (mapped elsewhere to <http://terminology.hl7.org/CodeSystem/v2-0074>).
    * The `Observation` attribute should include zero or more GeneticVariantFound observations and zero or more GeneticVariantTested observations. *Did you guess that we've just sliced an array?*
    * Finally, the SpecimentType, which we've just defined as a property, is extensibly bound to the value set `GeneticSpecimenTypeVS`.

Now that we've defined a general-purpose genomics report, we can use this class as-is, or as a parent for more specific genomics reports, perhaps `ConsumerAncestryGenomicsReport` or `NextGenerationSequencingGenomicsReport`.

### Comparison Between Profiling Tools
In FHIR terms, subclassing is akin to profiling profiles, which can be achieved in a number of tools, notably Forge and Trifolia. Both these tools are extremely well-done, and generously supported by commercial entities. Essentially, they are both visual editors for the "assembly language" of FHIR, namely, StructureDefinitions.

In contrast, CIMPL is like a high-level programming language. The source code for your project is the actual CIMPL itself. The difference is that revising, refactoring, renaming, reverting is **much, much** easier when you use a language, compared to a visual editor. That's why programming languages are always text, and comparatively, visual programming has had little uptake.

You will find that when projects grow to a certain size, global search and replace is an indispensible weapon, as is source code control, such as Github. The ability to branch, distribute work, compare changes, and merge automatically allows scalability that is far beyond any other approach. This is open to you when you use CIMPL.

### Primitives and Complex Data Types

The primitive types used in OBF are those defined in CIMPL, which correspond exactly to FHIR primitives, except for the way CIMPL handles coded types.

The complex data types in OBF are the same as the complex types in FHIR R4. They are found in the `obf.datatype` namespace. Since complex types like Quantity are ubiquitous, you will almost certainly need to import `obf.datatype` into your namespace. This is done using the `Uses` keyword.

## Class Hierarchy Overview


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

