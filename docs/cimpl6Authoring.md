# CIMPL Authoring

**NOTE:** This guide is under development. Please refer to the Tutorials and Reference Manuals as primary documentation.

_The purpose of this guide is to educate people about many different aspects of creating CIMPL models and its supporting utilities.  If you're looking for a quick introduction to CIMPL and SHR-CLI environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md).  If you're looking for detailed guidance on CIMPL syntax, try the [CIMPL6 Language Reference documentation](cimpl6LanguageReference.md)._

***

## Table Of Contents

[TOC]

***

# Using CIMPL to Create FHIR-based Models

CIMPL is designed to be modular and extensible, allowing for the reuse of other logical models, and inheriting its properties. The figure below illustrates this notion.

![CIMPL Modularity](img_cimpl/CIMPLBuildingBlocks.png)

The modeling author has multiple ways in CIMPL to represent  FHIR profiles:

* **Define all of the customized resource attributes you need from scratch (_"Clean slate"_)** - In this approach, the modeling author already knows which FHIR base resources they will be customize, and just define the element constraints or extensions in the FHIR profile.
* **Leverage CIMPL's "ObjectiveFHIR" (OBF) base FHIR models** -  In this approach, the modeling author will define their profile and specify a `Parent` class from the **ObjectiveFHIR** elements. Reference the [ObjectiveFHIR User Guide](cimpl6ObjectiveFHIR.md) for details.

Each profiling specification approach has advantages and disadvantages.

The CIMPL "clean slate" authoring approach might be beneficial when prototyping models containing only small number of profiles with minimal repetition of elements.  However, as the number of customizations increase, maintenance becomes more cumbersome and difficult to keep consistent between FHIR profiles.

On the other hand, using ObjectiveFHIR (OBF) base FHIR models has the significant benefits which include but are not limited to:

* saves the modeling author time in mapping common elements to its equivalent FHIR attribute.
* ensures consistency in the representation of commonly used attributes in different FHIR resources.

The user must however invest time to understand the ObjectiveFHIR logical model. Also, ObjectiveFHIR does not comprehensively support all FHIR resources, especially the new ones in R4 which have a low maturity level.

## Mapping to FHIR

Logical model elements are mapped to FHIR by creating and editing map files.
As a best practice, CIMPL map files must have the *_map.txt* naming convention.

Each map file should start with the following 3 lines:

* Grammar: Map _mapping_grammar_version_
* Namespace:  _data element group name_
* Target: _fhir target version_

Where:

* _mapping_grammar_version_ is the version number for parsing the contents of the map file. The current version is 5.1.
* _data element group name_ is the name of the grouping of elements that you have identified.
* _fhir target version_ is a choice of FHIR_DSTU_2, FHIR_STU_3, or FHIR_R4

The snippet below shows an example:
```
Grammar:	Map 5.1
Namespace:	shr.core
Target:		FHIR_R4
```


## Support

Questions on using CIMPL and its toolchain can be addressed on the HL7 Zulip chat channel [#cimpl](https://chat.fhir.org/#streams/197290/cimpl)

Report any issues on one of the following GitHub repositories:

* Related to running the SHR-CLI compiler, configuration files, or generating the FHIR Implementation Guide (IG): https://standardhealthrecord.atlassian.net/projects/CIMPL/issues
* Related to CIMPL base classes (Objective FHIR): https://standardhealthrecord.atlassian.net/projects/SHRM/issues

# Appendix A: An Approach to CIMPL Modeling for FHIR

Keeping in mind that CIMPL is primarily a way to create logical models with the capability to "model-once, translate-many", the modeling author should consider requirements-gathering and high level modeling steps.  While one approach is proposed below, the modeling author is not limited to following these steps and might find better approaches to creating detailed clinical models.

* Define the use cases behind the creation of a model.
* Create a high-level conceptual model which addresses your defined use case and can be easily understood by both technical and clinical communities.
* Create a data dictionary listing the data elements, cardinality, and potential value sets involved if the data type is a coded element. This provides a convenient summary for implmenters presented in a way that can be understood by non-technical subject matter experts involved in defining the use cases more than the model's design and implementation.
* Create the logical model in CIMPL which aligns with the high-level conceptual model and data elements noted in the previous steps.
* Create [FHIR mappings](#Mapping-to-FHIR) from the logical model to its equivalent FHIR resource and attributes, noting which elements you've defined in your logical model will be extensions.
* Generate the FHIR Implementation Guide.
* Create FHIR examples for each of the key profiles which define your IG and configure them within the CIMPL authoring environment so that they are validated by the FHIR IG Publisher.
