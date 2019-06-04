# Objective FHIR User Guide

>**Note:** This documentation is in draft form.

_This is an introductory guide to Objective FHIR Version 1.0. Objective FHIR is implemented in, the Clinical Information Modeling and Profiling Language (CIMPL). The Guide assumes some knowledge of CIMPL, at least for examples. If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md). If you're looking for a more in-depth introduction, try the [Tutorial](cimpl6Tutorial_detail.md). Or, see the [CIMPL 6 Reference Manual](cimpl6Reference.md)._

***

**Table of Contents**

[TOC]

***

## Overview

Objective FHIR ("OBF") is an [**object-oriented**](https://en.wikipedia.org/wiki/Object-oriented_programming) abstraction of FHIR. It provides modelers a way to define a detailed clinical information model by subclassing, extending, and constraining a pre-existing class library. These classes can then be translated automatically into FHIR profiles, FHIR Implementation Guides, data dictionaries, schemas, and other assets, using a choice of the three major FHIR versions: DSTU 2, STU 3, and R4.

### Philosophy

OBF classes resemble FHIR R4, but differ in carefully considered ways that increase consistency and reusability of the resulting models and profiles. Objective FHIR is implemented in CIMPL (Clinical Information Modeling and Profiling Language), a language that allows data structures of all sorts to be reused. This means a frequently-occurring group of elements like BodySite can be defined once and used anywhere.

Objective FHIR addresses one of the frequent criticisms of FHIR, specifically its [lack of consistency](https://wolandscat.net/2019/05/05/a-fhir-experience-consistently-inconsistent/). FHIR not only uses different _names_ for equivalent things, but often, altogether different modeling approaches. This is almost certainly a result of having resources managed by separate groups, coupled with loose oversight. OBF creates a layer that smooths over many of these differences, not for aesthetic reasons, but to make the whole framework easier to learn, and enable greater code reuse.

OBF also insulates modelers from differences between FHIR versions. The OBF classes are based on FHIR R4, but the same content is mapped to previous versions, specifically, DSTU 2 and STU 3. This means you can model once and publish the same semantic content across multiple FHIR versions.

### Meta-Model

Objective FHIR has been developed using the Clinical Information Modeling and Profiling Language (CIMPL). CIMPL is a very powerful, FHIR-aware, high-level language for creating clinical models. Expressing the model in CIMPL means that your models can automatically be turned into FHIR Profiles, Implementation Guides, data dictionaries, and other useful artifacts, in multiple FHIR versions.

Conceptually, there is nothing that prevents the same model from be expressed in other formalisms, some of which are mentioned in the [Appendix](#Appendix:-Relationship-to-Other-Initiatives), but no alternative formalism comes close to the mature, powerful, FHIR-specific tooling of CIMPL.

### Mapping to FHIR

One of the significant benefits of the OBF framework is that mapping to FHIR has already been done for you. In most cases, you don't have to worry about mapping at all. The only exceptions are when you create a new class that doesn't inherit from a pre-mapped OBF base class, or add an extension to a pre-mapped class that requires mapping to a nested extension.

### Coverage

Not all FHIR R4 resources are covered by Objective FHIR. We are working to expand the coverage. The model documentation is the best source to determine if OBF covers your needs.

### Naming

Attribute names in OBF sometimes differ from FHIR names, but if they do so, it is usually to make the meaning of the attribute more explicit. OBF names are chosen to be meaningful outside of the context of a single class. For example, the attribute `period` in `Encounter` is very vague, especially when removed from the encounter event context. To be more reusable, OBF redefines it as `OccurrencePeriod`, so when coupled with an event, perhaps the performance of a procedure, the meaning "occurrence period of the procedure" is fairly specific and self-explanatory. Although renaming does not answer all definition-related questions (for example, how is the start time of the procedure defined?), it does increase the specificity.

### Subclassing

To continue the example above, if procedure occurrence period needs a tighter definition, subclassing can be used. Specifically, you can take OccurrencePeriod and turn it into SurgicalProcedureOccurrencePeriod, something like this: 
```
Element:  SurgicalProcedureOccurrencePeriod
Parent:  OccurrencePeriod
Description: The period of a surgery, from the first incision time to the incision close time, as defined by https://manual.jointcommission.org/releases/archive/TJC2010B/DataElem0127.html.
```
The datatype and other constraints are inherited (cardinalities, low time < high time, etc.), so that's all there is to it.

## Primitives and Complex Data Types

The primitive types used in OBF are defined by CIMPL to be the same as FHIR, with one exception, which involves coded types.

The complex data types in OBF are the same as the complex types in FHIR R4. They are found in the `obf.datatype` namespace. Since complex types like Quantity are ubiquitous, you will almost certainly need to import `obf.datatype` into your namespace.





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

