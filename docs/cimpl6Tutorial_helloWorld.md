# CIMPL 6.0 'Hello World' Tutorial

## Preface

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles, extensions and implementation guides (IG). Because it is a _language_, written in text statements, CIMPL encourages distributed, team-based development using conventional source-code control tools such as Github. CIMPL provides tooling that enables you to define a model once, and publish that model to multiple versions of FHIR.

### Purpose of this Document

In the tradition of "Hello, World" examples, this tutorial the the bare minimum to get you started using CIMPL. Upon completion of this tutorial, you will be able to try the [CIMPL In-Depth Tutorial](cimpl6Tutorial_detail.md) and get started creating a FHIR Implementation Guide (IG).

### Intended Audience

The CIMPL Hello World Tutorial is targeted to software developers or people comfortable with programming languages. Familiarity with FHIR is helpful as the tutorial references FHIR artifacts (such as Resources, Elements, etc.)

### Prerequisite

This guide assumes you have: 
* Installed the latest version of the SHR-CLI software as documented in [CIMPL SetUp and Installation](cimplInstall.md) (preferably installed in the `~/cimpl/shr-cli` directory)
* A text editor (preferably VSCode with the _vs-code-language-cimpl_ extension, but not required)

**Note:** _This tutorial is written assuming a MacOS environment.  While the contents of CIMPL authoring are identical regardless of platform, the command lines run in a command line terminal will differ in file path specifications.  Namely, for a Windows command line terminal, replace all the path references of forward-slash `/` to back-slash `\`._

***

## Table of Contents

[TOC]

***

## Tutorial Pre-configuration

This tutorial is focused on how to create a model that will be used as input to create an FHIR Implementation Guide. Supporting configuration and core data type files have been defined for you.

The following directory structure is assumed for your configuration:

```
Directory:  cimpl
            |_ shr-cli
            |_ hello-world
```
The shr-cli directory should already exist as a result of installing the SHR-CLI tool.

To create `hello-world` directory, use File Explorer on Windows, or open your favorite command-line tool and enter the following:

```
cd ~/cimpl
mkdir hello-world
```


# Tutorial Steps - Hello World!

## 1) Define the CIMPL Model

Now, create a file called `HelloWorld.txt` in the `~/cimpl/hello-world` folder and add the following data definitions:

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
This is a CIMPL class file.

## 2) Define a Mapping to FHIR

Next, create a file in the same directory called `HelloWorld_map.txt`. This file is even simpler:

```
Grammar:     Map 5.1
Namespace:   hello
Target:      FHIR_STU_3

HelloWorld maps to Basic:
```

[`Basic`](https://www.hl7.org/fhir/basic.html) is a "blank slate" FHIR resource. We didn’t map the `SayHello` data element to an existing element inside this resource, so it will automatically appear as an extension in the profile on Basic.

## 3) Create the Home Page of your IG

Next, we need some simple HTML for the Implementation Guide pages, so create an `exampleIndexContent.html` file with the following content:
```
HELLO HELLO HELLO
```

## 4) Create a Configuration File

To point the SHR-CLI tool at the right files, and assign the right names to things, we need a configuration file. Create an `ig-hello-world-config.json` file with these contents:

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

## 5) Run SHR-CLI

Now, go back to your SHR-CLI installation and run the hello world:

```
cd ~/cimpl/shr-cli
node . -c ig-hello-world-config.json ../hello-world
```

When the program runs, it will output a warning message alerting you that mapping to Basic usually isn’t the best choice, but in this case, it is intentional. After the program runs, the generated profile (a FHIR `StructureDefinition`) will be found in `~/cimpl/shr-cli/out/fhir/profiles/`.  A `StructureDefinition` can be verbose, and this one clocks in at several hundred lines.

## 6) Generate the "Hello, World" Implementation Guide

_Note: If your organization uses a proxy server, you may have to run the IG publishing process from outside your organization's firewall._

A friendlier view of the profile is created when we create an implementation guide (IG). To do this, we use the [existing FHIR implementation guide (IG) publisher](http://wiki.hl7.org/index.php?title=IG_Publisher_Documentation).

To generate the IG, run:

```
cd ~/cimpl/shr-cli
yarn run ig:publish
```

The generated profile page will be located at `~/cimpl/shr-cli/out/fhir/guide/output/StructureDefinition-hello-HelloWorld.html`.  Open this page in your favorite web browser.

You should see a FHIR profile that looks like a traditional [FHIR Implementation Guide](https://www.hl7.org/fhir/implementationguide.html).  For a more comprehensive understanding of the CIMPL grammar and how to use it for your project, continue to the [Tutorial]().


