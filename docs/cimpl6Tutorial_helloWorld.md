# CIMPL Hello World Tutorial

## Table of Contents

[TOC]

***

## Introduction

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles, extensions and [implementation guides](https://wiki.hl7.org/index.php?title=FHIR_Implementation_Guides) (IG). Because it is a _language_, written in text statements, CIMPL encourages distributed, team-based development using conventional source-code control tools such as Github. CIMPL provides tooling that enables you to define a model once, and publish that model to multiple versions of FHIR.

### Purpose

In [the tradition of _Hello, World_ examples](https://en.wikipedia.org/wiki/"Hello,_World!"_program), this tutorial presents a very simple example to get you started using CIMPL. Upon completion of this tutorial, you will be able to try the [CIMPL In-Depth Tutorial](cimpl6Tutorial_detail.md) and get started creating a FHIR IG.

### Intended Audience

The CIMPL _Hello World Tutorial_ is targeted to people with some programming experience. Familiarity with FHIR is helpful as the tutorial references FHIR artifacts (e.g. resources and profile.)

### Prerequisite

This guide assumes you have:

* Installed the latest version of the SHR-CLI software as documented in [CIMPL Setup and Installation](#cimplInstall.md) (preferably installed in the `~/cimpl/shr-cli` directory)
* A text editor (preferably VSCode with the _vs-code-language-cimpl_ extension)

### Initial Setup

This tutorial is focused on how to create a model that will be used as input to create a FHIR IG. Supporting configuration and core data type files have been defined for you.

The following directory structure is assumed for your configuration:

```
Directory:  cimpl
            |_ shr-cli
            |_ hello-world
```
The shr-cli directory should already exist as a result of [installing the SHR-CLI tool](cimplInstall.md).

To create _hello-world_ directory, use File Explorer on Windows, or open your favorite command-line tool and enter the following:

```
cd ~/cimpl
mkdir hello-world
```


## Tutorial Steps - Hello World

### 1) Define the CIMPL Model

Create a file called _HelloWorld.txt_ in the _~/cimpl/hello-world_ folder and add the following data definitions:

```
Grammar:         DataElement 6.0
Namespace:       hello
Description:     "A simple example of CIMPL."

Entry:           HelloWorld
Description:     "A silly profile."
Property:        SayHello 0..1

Element:         SayHello
Description:     "An extension indicating whether to say hello"
Value:           boolean
```
This is a CIMPL class file that defines a class called `HelloWorld` and gives that class one property, `SayHello`. The `Property` has an `Element` class with a `boolean` data type.

### 2) Define a Map to FHIR

Next, create a file in the same directory called _HelloWorld_map.txt_. This file is even simpler:

```
Grammar:     Map 5.1
Namespace:   hello
Target:      FHIR_STU_3

HelloWorld maps to Basic:
```

[**Basic**](https://www.hl7.org/fhir/basic.html) is a _blank slate_ FHIR resource. It has no elements, so the unmapped `SayHello` `Property` will automatically appear as an extension in the profile on **Basic**.

### 3) Create the Home Page of your IG

Next, we need some simple HTML for the IG pages. Create an _exampleIndexContent.html_ file with the following content:
```
HELLO HELLO HELLO
```

### 4) Create a Configuration File

To point the SHR-CLI tool at the right files, and assign the right names to things, we need a configuration file. Create a file named _ig-hello-world-config.json_, and copy and paste this content:

```
{
    "projectName": "Example Project",
    "projectShorthand": "EXAMPLE",
    "projectURL": "http://example.com",
    "fhirURL": "http://example.com/fhir",
    "implementationGuide": {
      "npmName": "hello-world",
      "indexContent": "exampleIndexContent.html"
    },
    "publisher": "Example Publisher",
    "contact": [
        {
            "telecom": [
                {
                    "system": "url",
                    "value": "http://example.com"
                }
            ]
        }
    ],
    "provenanceInfo": {
        "leadAuthor": {
            "name":"Example Author",
            "organization": "Example Publisher",
            "email": "example@example.org"
        },
        "license": "Creative Commons CC-BY <https://creativecommons.org/licenses/by/3.0/>",
        "copyright": "Copyright (c) The Example Organization <http://example.org>"
    }
}
```

### 5) Run SHR-CLI

Now, go back to your SHR-CLI installation and run the hello world:

```
cd ~/cimpl/shr-cli
node . ../hello-world -c ig-hello-world-config.json
```

When the program runs, it will output a warning message alerting you that mapping to **Basic** usually isn’t the best choice, but in this case, it is intentional. After the program runs, the generated profile (a FHIR **StructureDefinition**) will be found in _~/cimpl/shr-cli/out/fhir/profiles/_.  A **StructureDefinition** can be verbose, and this one clocks in at several hundred lines.

### 6) Generate the _Hello, World_ Implementation Guide

>Note: If your organization uses a proxy server, you may have to run the IG publishing process from outside your organization's firewall.

A friendlier view of the profile is created when we create an IG. To do this, we use the [existing FHIR implementation guide (IG) publisher](http://wiki.hl7.org/index.php?title=IG_Publisher_Documentation).

To generate the IG, run:

```
cd ~/cimpl/shr-cli
yarn run ig:publish
```

The generated profile page will be located at _~/cimpl/shr-cli/out/fhir/guide/output/StructureDefinition-hello-HelloWorld.html_.  Open this page in your favorite web browser.

You should see a FHIR profile for _HelloWorld_ in a (mostly empty) [FHIR Implementation Guide](https://www.hl7.org/fhir/implementationguide.html). 

_You did it!_

For a more comprehensive understanding of the CIMPL grammar and how to use it for your project, continue to the [detailed tutorial](cimpl6Tutorial_detail.md).

# Appendix A - Document Conventions

| What you see | Explanation |
|:----------|:---------|
| Bolded Text  | FHIR reference, first letter in what will become an acronym
| Italics | Code substitution text, emphasis to call attention to a word in a sentence or heading, example file or folder names|
| `Code Block` | CIMPL reserved word or phrase, CIMPL code block or partial code block example |
|Capitalization|CIMPL reserved words or references that are capitalized, specific instances of FHIR artifacts|
