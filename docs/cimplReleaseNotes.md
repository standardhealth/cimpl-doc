# Release Notes

## Table of Contents

[TOC]

***

## Introduction

CIMPL consists of two parts:

Models written in CIMPL, include base classes and models for certain Implementation Guides, maintained in Github in the [shr-spec repository](https://github.com/standardhealth/shr-spec) and SHR-CLI tools (primarily in the [shr-cli repository](https://github.com/standardhealth/shr-cli)).

Please refer to the Github repository for [complete log of all releases of SHR-CLI](https://github.com/standardhealth/shr-cli/releases).

## SHR-CLI 6.0

**Release Date**: June 5, 2019

CIMPL 6.0 contains a number of significant changes to both syntax and its base classes which support translation to FHIR. The table below summarizes these changes:

| Change Type | Change Description | CIMPL 5.0 Example | CIMPL 6.0 Example | Section |
|:---- |:----------|:---------------------- |:-------------------|:----------------- |
| New | keyword `only` eliminates all value choices except one | None | `FindingResult only concept` | [Only Constraint](#only-constraint) |
| New |keyword `Property` is required to define properties for an Entry or Element. |`0..1 TreatmentIntent`| `Property:  TreatmentIntent 0..1` | [Property Keyword](#property) |
| New |keyword `Group` is required to a reusable collection of properties, that is not an `Entry`. | None | `Group: Address` | [Group Keyword](#group) |
| Replace | `EntryElement` keyword replaced by `Entry` | `EntryElement: CourseOfTreatmentPerformed`| `Entry:  CourseOfTreatmentPerformed` | [Element Keyword](#element) |
| Replace | `Based on` keyword replaced by `Parent` | `Based on: Observation` | `Parent:  Observation` | [Parent Keyword](#parent) |
| Syntax change | Cardinality is specified _after_ the property or class name | `0..1 TreatmentIntent` | `Property:  TreatmentIntent 0..1` | [Property Keyword](#property), [Cardinality Constraint](#cardinality-constraint) |
| Replace | `is` constraint for fixed values replaced by `=` | `FindingTopicCode is LNC#48676-1` | `FindingTopicCode = LNC#48676-1` | [Field Constraints](#field-constraints) |
| Replace | substitution of a more specific element derived from a parent element using `is type` keyword replaced by `substitute`. | `Specimen is type BreastSpecimen` | `Specimen substitute BreastSpecimen` | [Substitute](#substitute) |
| Replace | `code`, `Coding`, and `CodeableConcept` are replaced by a new primitive `concept` | `Value: CodeableConcept from AttributionCategoryVS` | `Value: concept from AttributionCategoryVS` | [Primitives](#primitives) |
| Replace | `must be`, `should be`, `could be`, and `if covered` value set constraints are obsolete and replaced by `(required)`, `(preferred)`, `(extensible)`, and `(example)` | `Type from BreastSpecimenTypeVS if covered` | `Type from BreastSpecimenTypeVS (extensible)` | [Value Set Binding Constraint](#value-set-binding-constraint) |
| Replace | `ref()` is now obsolete. `value is type` replaced by `substitute` and bracket notation denoting value choices| `SourceSpecimen value is type ref(BreastSpecimen)` | `SourceSpecimen[Specimen] substitute BreastSpecimen` | [Mapping to References](#mapping-to-references) |

## SHR-CLI 5.0

Release Date: September 5, 2017

* support for the 5.0.0 grammar
* detect file types by contents rather than extensions
* simpler declaration of value set constraints
* binding strengths for value sets
* abstract elements
* includes type constraints (indicates a list should include certain sub-types)
* updated FHIR to 3.0.1
* fixed value mappings
* bug fixes

## SHR-CLI 4.0

Release Date: March 6, 2017

## SHR-CLI 3.0

Release Date: January 4, 2017

## SHR-CLI 2.0

Release Date: September 17, 2016

## SHR-CLI 1.0

Release Date: June 1, 2016
