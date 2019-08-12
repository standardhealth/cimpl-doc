# CIMPL 6.0 Tooling Reference

>**Note:** This documentation is a draft.

_This is a comprehensive guide to CIMPL 6.0 Tooling, including the command line interface, auxiliary files, and configurations needed to produce a FHIR Implementation Guide (IG). If you're looking for a quick introduction to CIMPL and `SHR-CLI` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md) or [In-Depth Tutorials](cimpl6Tutorial_detail.md). For details of the CIMPL language itself, see the [CIMPL Language Reference Manual](cimpl6LanguageReference.md)._

***

**Table of Contents**
[TOC]

***

## Overview

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles, extensions and Implementation Guides (IGs). The CIMPL language itself, however, is static and must be processed to produce these outputs. 

This reference manual describes the configurations, files, and commands needed to create a FHIR IG from CIMPL language files. It assumes that the CIMPL Language files (classes, value sets, and maps) have been defined (see [CIMPL Language Reference Manual](cimpl6LanguageReference.md) for details). It also assumes that the CIMPL SHR-CLI tooling has been installed according to the directions in the [Setup and Installation Guide](cimplInstall.md).

The CIMPL Tooling, also called SHR-CLI (Standard Health Record Command Line Interface), is the engine that imports a set of inputs, including CIMPL language files, and exports FHIR and other outputs, as shown below:

#### _Figure 1: CIMPL Tooling Overview_ ####
![CIMPL Tooling Overview](img_cimpl/cli-overview.png)

### Inputs

* [CIMPL Language files](cimpl6LanguageReference.md), including class files, value set files, and mapping files that define your clinical information model,
* A [Configuration file](#configuration-file) that contains directives to the tooling, and points to other resources,
* An optional [Content Profile](#content-profile-file) file, which specifies Must-Support elements and profiling options specific to an IG,
* One or more [Front Matter](#front-matter-files) files, which are the narratives and graphics that introduce the IG,
* [FHIR Examples](#fhir-examples) that are to be included in the IG, and
* A [Package List](#package-list) that has information required for building the IG.

### Processing Sequence

* The user issues a command through the command-line interface (CLI).
* The CIMPL tooling imports definitions from CIMPL files (class, value set, and map files). SHR-CLI will report any errors in the CIMPL definitions.
* SHR-CLI applies filters, according to the [filter strategy](#filter-strategy-configuration-parameters).
* SHR-CLI exports the filtered SHR definitions into desired formats, such as FHIR profiles, data dictionaries, etc. The exports can be selected through [command line options](#executing-shr-cli). SHR-CLI will report any errors during the export process.
* The users issues a command to produce the IG.

### Outputs

Depending on the configuration options selected, the SHR-CLI produces one or more of the following outputs:

* **FHIR Profiles, Extensions, Value Sets** that form the core content of the IG,
* A **Logical Model** corresponding to the CIMPL class definitions, expressed as FHIR StructureDefinitions,
* **JSON Schema** for the profiles defined by the IG,
* A **Data Dictionary** that lists the Must-Support data elements in the IG, as well as value sets and value set members,
* **Model Documentation** in the form of a Javadoc-like browser that allows one to see the hierarchical class relationships in the logical model.

## Directory Structure
The SHR-CLI tooling does not require a particular directory structure. However, following conventions makes the process of managing the requisite files much easier. Here is an example of the suggested arrangement that assumes the same IG can be produced under FHIR STU3 and R4:

```
├── shr-cli-6.5.0                   // Tooling location
|
...
|
├── spec                            // Main specification directory
|   ├── ns-namespace1                   // Namespace directories
|   |   ├── namespace1-foo.txt               // class file(s)
|   |   ├── namespace1-bar.txt
|   |   ├── namespace1-foo-vs.txt            // value set file(s)
|   |   ├── namespace1-bar-vs.txt
|   |   ├── namespace1-foo-map-stu3.txt      // map(s) for FHIR STU3
|   |   ├── namespace1-bar-map-stu3.txt
|   |   ├── namespace1-foo-map-r4.txt        // map(s) for FHIR r4
|   |   └── namespace1-bar-map-r4.txt
|   ├── ns-namespace2
|   | ...
|   ├── ns-namespace3
|   | ...
|   | ...
|   ├── ig-myguide1                 // IG directories
|   |   ├── myguide1-examples           // directory of examples
|   |   |   ├── example1
|   |   |   └── example2
|   |   ├── myguide1-frontmatter        // directory containing narratives
|   |   |   ├── index.html
|   |   |   ├── picture.png
|   |   |   └── another-page.html
|   |   ├── myguide1-stu3-cp.txt        // content profile file for FHIR STU3 IG
|   |   ├── myguide1-r4-cp.txt          // content profile file for FHIR R4 IG
|   |   ├── myguide1-stu3-plist.json    // package list for FHIR STU3 IG
|   |   ├── myguide1-r4-plist.json      // package list for FHIR R4 IG
|   |   ├── myguide1-stu3-config.json   // config file for FHIR STU3 IG
|   |   └── myguide1-r4-config.json     // config file for FHIR R4 IG
|   ├── ig-myguide2
|   |  ...
```

* **Tooling Directory** - this the directory where the SHR-CLI tooling has been installed. Any convenient directory can be used. For easy identification, the tooling version number should be included in the directory name.
* **Specification Directory** - this is the top level directory where the input files and are located, usually arranged in sub-folders. Any convenient directory can be used. If you are using Github to manage your development, this could be where the repository is checked out to your local machine.
* **Namespace Directories** - these directories, located under the Specification Directory, contains CIMPL language files for a single namespace. The name of the directory should be prefixed with `ns-` followed by the shortname or acronym of the namespace followed .
* **IG Directories** - these directories, under the specification directory contain files specific to a given implementation guide. There are subdirectories containing the front matter and examples, and individual configuration, content profile, and package list files. These files can be specific to a FHIR release, since the same IG could be created using different versions of FHIR.

**Note:** There is currently no way to combine profiles for multiple FHIR versions in single IG.

## Naming Conventions
The naming of configuration, content profiles, and package list files is arbitrary, but it is useful for different teams to follow similar conventions. The suggested approach uses variations on the IG short name, as follows:

* Configuration file: `ig-<guide-name>-config.json`
* Content Profile file: `ig-<guide-name>-cp.txt`
* Package List file: `ig-<guide-name>-plist.json`

If more than one FHIR version will be supported, the FHIR version should be included:

* Configuration file: `ig-<guide-name>-<FHIR Version>-config.json`
* Content Profile file: `ig-<guide-name>-<FHIR Version>-cp.txt`
* Package List file: `ig-<guide-name>-<FHIR Version>-plist.json`

where FHIR version is dstu2, stu3, or r4.

## Configuration File

SHR-CLI requires a configuration file to run. The name of this file is typically specified on the command line using the -c command line option. If the name is not specified, the tooling looks for a file named `config.json` in the working directory. If it cannot be found, or the contents of the file are invalid, an error message is returned.

The configuration file is a JSON file with the following parameters:

|Parameter            |Type    |Description                                                    |
|:--------------------|:-------|:--------------------------------------------------------------|
|`projectName`        |`string`|The full, official name of the project, for example "HL7 FHIR Implementation Guide: minimal Common Oncology Data Elements (mCODE) Release 1 - US Realm, STU Ballot 1"  |
|`projectShorthand`   |`string`|A shorthand name for the project, such as "mcode".                              |
|`projectURL`         |`string`|The primary URL for the project, such as "http://hl7.org/fhir/us/mcode/"                             |
|`fhirURL`            |`string`|The FHIR IG URL for the project, often the same as the projectURL. **(TO DO: clarify the difference between projectURL, fhirURL, and entryTypeURL)**  |
|`fhirTarget`         |`string`|The FHIR version this IG will be based on, currently a choice of `"FHIR_R4"`, `"FHIR_STU_3"`, or `"FHIR_DSTU_2"`|
|`entryTypeURL`       |`string`|The root URL for the JSON schema `EntryType` field. **(TO DO: clarify where and how this is used)**   |
|`filterStrategy`     |`{}` |A JSON object containing configuration for filtering ([see below](#filter-strategy-configuration-parameters)).              |
|`contentProfile`     |`string`| The file name of the content profile for the project.    |
|`implementationGuide`|`{}`    |A JSON object containing configuration for IG publishing ([see below](#implementation-guide-configuration-parameters)).          |
|`copyrightYear`      |`string`|The copyright year to include in the documentation.            |
|`publisher`          |`string`|The name of the publisher for the project, which for HL7 projects, should be the sponsoring workgroup, for example, "HL7 International Clinical Interoperability Council".  |
|`contact`            |`[]`    |A JSON array containing HL7 FHIR R4 [ContactPoint objects](http://hl7.org/fhir/R4/datatypes.html#ContactPoint).  |
|`provenanceInfo`     |`{}`    | A JSON object specifying author and other information ([see below](#provenance-information-configuration-parameters)) |

### Filter Strategy Configuration Parameters
Between the import stage and the export stage, there is a filtering stage (see  [Figure 1](#figure-1-cimpl-tooling-overview)). Filtering is useful when [specification directory](#directory-structure) contains namespaces that or entries that are outside the scope of the current IG. The filter stage can remove unwanted namespaces and entries to limit the scope of the exports, and subsequently, the IG.

The contents of the `filterStrategy` object are as follows:

|Parameter |Type |Description |
|---------|------|------------|
|`filter`  |`boolean`|A value indicating whether to enable filtering. If `filter` is `true`, then the filtering operation will occur. Otherwise, no filtering will occur. (Also, if the `filterStrategy` parameter is entirely omitted, no filtering will occur.) |
|`strategy`|`string` | The strategy for specification filtering, either "namespace", "element", or "hybrid".|
|`target`  |`[]`     |An array of strings containing the names of Entries, Namespaces, or both. |

* The "element" strategy filters the imported classes to include each Entry listed in the `target` array, and their recursive dependencies.
* The "namespace" strategy filters the imported classes to include only Entries in the namespaces listed in the `target` array, and their recursive dependencies.
* The "hybrid" strategy filters the imported classes to include each Entry listed in the `target` array and all Entries in every namespace listed in the `target` array, and their recursive dependencies.

When specifying an Entry in the `target` array, use the [fully qualified name (FQN)](cimpl6LanguageReference.md#fully-qualified-names) format.

### Implementation Guide Configuration Parameters
These configurations are used to control the production of the IG. The contents of the `implementationGuide` object are as follows:

|Parameter                 |Type     |Description                                                          |
|:-------------------------|:--------|:--------------------------------------------------------------------|
|`npmName`                 |`string` |The assigned node package manager name for this IG, for example "hl7.fhir.us.mcode". The npm name is usually assigned by HL7.   |
|`version` |`string` |The version of this IG (not necessarily the version of FHIR), usually in the form _major.minor.revision_, for example, "3.0.1"  |
|`ballotStatus`            |`string` |The HL7 ballot status of the IG (e.g., STU1 Ballot, Continuous Integration Build, etc.)      |
|`packageList` |`string` |The name of the file to use as the [IG's package list](#package-list), relative to the Specification Directory. |
|`includeLogicalModels`    |`boolean`|A "true" or "false" value indicating whether to include logical models in the IG.     |
|`includeModelDoc`         |`boolean`|A "true" or "false" value indicating whether to include the model documentation in the IG. |
|`indexContent` |`string` |The name of the file or folder containing the [front matter](#front-matter-files), relative to the Specification Directory, for example, "ig-mcode/IndexFolder-Oncocore". If the `indexContent` is a folder, then it must contain an `index.html` file whose contents will be used as the body of the IG home page.  |
|`extraResources`          |`string` |The name of the folder containing extra JSON resources to include in the IG, one file per resource. Currently, the following resource types are supported: `SearchParameter`, `OperationDefinition`, `CapabilityStatement` (STU3+), `Conformance` (DSTU2).  If files are detected, links are added to the navigation menu as necessary. |
|`examples` |`string` |The name of the folder containing examples (one example per file) to include in the IG, for example, "ig-mcode/Examples-mCODE-r4". We recommend the individual example file name match the `id` in the example file (with `.json` extension added). The example's `meta.profile` must match the canonical URL for the profile it exemplifies (e.g. `"meta": { "profile": [ "http://hl7.org/fhir/us/breastcancer/StructureDefinition/oncology-BreastCancerPresenceStatement" ] }`). If no `examples` folder is specified, and a folder named "fhir-examples" exists in the specification directory, it will be used as the examples folder. | 
|`historyLink`             |`string` |The URL for the page containing the IG's history information.  **(TO DO: clarify where and how this is used)**   |
|`changesLink`             |`string` |The URL to a site where users can request changes (shown in page footer) **(TO DO: clarify where and how this is used)** |
|`primarySelectionStrategy`|`{}`     |The strategy for selection of what is primary in the IG (see below). |

The primary selection strategy causes certain profiles to be displayed in a "Primary" section at the top list of profiles. All other exported profiles are listed in a "Supporting" section below the "Primary" section. The contents of the `primarySelectionStrategy` object are as follows:

|Parameter |Type    |Description |
|--------|------|-----------|
|`strategy`|`string`|The strategy to follow for primary selection, either "namespace", "hybrid", or "entry" (default). |
|`primary` |`[]`|An array of strings containing the namespaces and entries (only used for "namespace" and "hybrid" strategy). |

The `strategy` options are as follows:

* "entry" selects every exported profile as primary.
* "namespace" selects every profile found in the namespaces of the `primary` array as primary.
* "hybrid" selects every entry listed in the `primary` array or found in the namespaces in the `primary` array as primary.

When specifying an Entry in the target array, use the fully qualified name (FQN) format.

### Provenance Information Configuration Parameters

**(TO DO: need to document what this structure can contain)**

Here is an example of provenanceInformation:

```
    "provenanceInfo": {
        "leadAuthor": {
            "name":"Example Author",
            "organization": "Example Publisher",
            "email": "example@example.org"
        },
        "license": "Creative Commons CC-BY <https://creativecommons.org/licenses/by/3.0/>",
        "copyright": "Copyright (c) The Example Organization <http://example.org>"
    }
```

## FHIR Examples

Configuring FHIR examples to appear in the generated IG involves the following steps:

* Create a folder which will contain your FHIR examples
* Modify the CIMPL configuration file to specify the folder containing your examples

The folder created can be any name, as long as it is specified within the CIMPL configuration file. However, following [directory structure](#directory-structure) and [naming conventions](#naming-conventions) is recommending.

The folder location is specified using the `"examples:"` parameter in the CIMPL configuration file.  This is illustrated in the figure below:
![CIMPL Examples Configuration](img_cimpl/fhirexampleconfig01.png)


## Front Matter Files

Every IG contains some amount of narrative content. We call this information the "front matter" of the IG. The front matter is typically a set of hyperlinked HTML files, graphics, and sometimes, downloads. These files are manually authored using any convenient tool; there are no special facilities in CIMPL to author the front matter.

If multiple files are involved, they must be placed into a single folder, which is then named in the `indexContent` parameter of the [configuration file](#configuration-file). This folder must contain a file named `index.html` whose contents will be used as the body of the IG home page.

If a single file is used, the file is named in the `indexContent` parameter of the configuration file, and a folder is not required.

## Package List File

The format and content of this file follows the [FHIR specification](http://wiki.hl7.org/index.php?title=FHIR_IG_PackageList_doco). The package list is required for IGs published by HL7. The file containing the package list is named by the `packageList` parameter of the [configuration file](#configuration-file). If the `packageList` parameter is not supplied and no file is found at the default location (packageList.json), and `fhirURL` is an hl7.org or fhir.org URL (indicating it is an HL7 publication), then a basic package list file will be created. In this case, the IG author should review and modify the file as needed and then check it into the version control system.

Here is an example of a package list file:

```
{
  "package-id": "hl7.fhir.us.mcode",
  "title": "HL7 FHIR Implementation Guide: minimal Common Oncology Data Elements (mCODE) Release 1 - US Realm | STU Ballot 1",
  "canonical": "http://hl7.org/fhir/us/mcode",
  "list": [
    {
      "version": "current",
      "desc": "Continuous Integration Build (latest in version control)",
      "path": "https://build.fhir.org/ig/HL7/fhir-mCODE-ig",
      "status": "ci-build",
      "current": true
    },
    {
      "version": "0.9.1",
      "date": "2019-06-10",
      "desc": "Initial version",
      "path": "http://hl7.org/fhir/us/mcode/2019Sep",
      "status": "draft",
      "sequence": "STU 1",
      "fhir-version": "4.0.0"
    }
  ]
}
```

## Content Profile File

In CIMPL, Implementation Guides are viewed as "overlays" that specify how parts of the common data model are applied in the context of specific use cases. The common data model is shared across multiple IGs, promoting interoperability. This way of thinking about models and IGs contrasts with the conventional approach wherein each IG defines its own data model, with little regard for data models specified in other IGs.

This idea is illustrated below. There is a set of common data models defined in namespaces A through D, but each IG uses a different set of entries (profiles) from those models in the context of its particular use cases.

![Data model IG relationship](img_cimpl/Data-model-use-case-model.png)

We have already introduced [filtering](#filter-strategy-configuration-parameters) as the mechanism by which entries (profiles) and namespaces are selected for an IG. In addition, certain properties within each profile may be designated as "Must-Support". In FHIR, [Must-Support](https://www.hl7.org/fhir/conformance-rules.html#mustSupport) is a boolean flag which allows a profile to indicate that an implementation must be able to process that element in a FHIR instance if it exists.

An important point is that different use cases may require different elements to be supported. For example, a certain use case may not require ethnicity, even though US Core designates ethnicity as Must-Support. In conclusion, Must-Support is contexual, and should be designated at the level of the IG, not in the data model.

CIMPL uses a separate file, the Content Profile, so that MustSupport elements are configured at the IG level. The syntax of a Content Profile file is:

```
Grammar:  ContentProfile 1.0

Namespace:  <namespace-1>
    <Entry-name-1>:
        <Element-name-1> MS
        <Element-name-2> MS
        ...
    <Entry-name-2>:
    ...
Namespace:  <namespace-2>
    <Entry-name-1>:
        <Element-name-1> MS
        <Element-name-2> MS
        ...
    <Entry-name-2>:
    ...
```







## Executing SHR-CLI

The general form of the execution command is as follows (where $ stands for the command prompt, which could be different on your system):

$ `node <tooling-directory> <specification-directory> [options]`

where options include:

```
-c, --config <config>    the name of the config file (default: config.json)
-l, --log-level <level>  the console log level <fatal, error, warn, info, debug, trace> (default: info)
-s, --skip <feature>     skip an export feature <fhir, json-schema, model-doc, data-dict, all> (default: <none>)
-o, --out <out>          the path to the output folder (default: out)
-m, --log-mode <mode>    the console log mode <short,long,json,off> (default: short)
-d, --duplicate          show duplicate error messages (default: false)
-j, --export-es6         export ES6 JavaScript classes (experimental, default: false)
-n, --clean              Save archive of old output directory and perform clean build (default: false)
-h, --help               output usage information
```

The options are not order-sensitive. Here is an example of a SHR-CLI command and an explanation of its parts:

$ `node . ../shr_spec/spec -c ig-mcode/ig-mcode-r4-config.json -l error`

* `node` is the command that starts the SHR-CLI application.
* The dot `.` represents the current directory in Windows and macOS. In this example, the tooling directory is the current working directory.
* `../shr_spec/spec` represents the location of the specification directory. The the double dot `..` represents the directory above the current working directory, in Windows and macOS. In this case, `/shr_spec` is at the same level as the tooling directory, and `/spec` is one level below that.
* `-c ig-mcode/ig-mcode-r4-config.json` directs the execution engine to the configuration file. Note that the configuration file location is relative to the specification directory, implying the full path to the configuration is `../shr_spec/spec/ig-mcode/ig-mcode-r4-config.json`
* `-l error` is an option that sets tells the system to surpress any messages that don't rise to the level of an `error`. This reduces the amount of output to the console window.

### Error Messages
When you are building a model, it is inevitable that you encounter error messages from SHR-CLI. These messages indicate some inconsistency in the way the CIMPL language has been used, or in the specification of the model. Debugging the model is an iterative process, and it could take some time and experience to arrive at a "clean" compile.

A detailed list of CIMPL compilation errors and troubleshooting suggestions are available **[in Appendix A](#appendix-a-error-messages)**. In this section, we give some general tips on approaching debugging your model:

* Eliminate [parsing errors](#parsing-errors) first. **Parsing errors can be identified because they include a file, line number, and column number.** This is essential because parsing errors usually trigger large cascades of meaningless errors, so you can't truly start debugging your model until parsing errors have been eliminated.
* Once all parsing errors have been eliminated, start working on the first (or first few) errors. Often, subsequent errors are a consequence of the first error.
* Don't be discouraged by the number of errors, since a single correction can silence multiple logged errors.
* Read the error messages carefully. Although the messages might be cryptic, especially at first, the names of classes and paths are often excellent clues.

## FHIR Export

## Logical Model Export

## Model Doc Export

## Data Dictionary Export

## JSON Schema Export

## Creating the Implementation Guide

The final step in the IG creation process is to run the **[FHIR IG Publisher](http://wiki.hl7.org/index.php?title=IG_Publisher_Documentation)**. This tool is maintained and owned by HL7 FHIR.

At a command prompt, use one of the 2 options:

* **Option 1 (_if the defaults were used in running shr-cli_):** yarn run ig: publish
* **Option 2 (_if not using defaults and specifying a directory in running shr-cli_):** java -Xms4g -Xmx8g -jar <_directory specified when running shr-cli_>/fhir/guide/org.hl7.fhir.publisher.jar -ig <_directory specified when running shr-cli_>/fhir/guide/ig.json

By default, the FHIR IG Publisher will perform validation checks on the  StructureDefinition of specified FHIR profiles, value sets, and examples which reference any base resources or FHIR profiles.  An output of these checks are found in the CIMPL output, *qa.html*.

An example QA output is shown in the figure below:

![qa.html example output](img_cimpl/igpublisher_output.png)


## Appendix A: Error Messages

In the following, `$` prefix indicates a variable that will be filled in with specific information.

### Table of Contents

* **[Error Code Structure](#error-code-structure)**
* **[Parsing Errors](#parsing-errors)**
* **[Warning Codes](#warning-codes)**
* **[Compilation Error Codes](#compilation-error-codes)**
* **[Mapping Error Codes](#mapping-error-codes)**

### Error Code Structure

CIMPL Compilation Errors are structured in the following format:

***1*** 2 3 4 5 <br>
&nbsp;↳&nbsp;First digit tells whether it is an warning or error. 0 = warning, 1 = error

1 ***2*** 3 4 5 <br>
&nbsp;&nbsp;&nbsp;↳&nbsp; Second digit indicates the phase of processing where the error appeared: 1 = grammar and importing of the text files, 2 = logical verification of the model, 3 = exporting of FHIR profiles, 4 = exporting of the JSON profiles

1 2 ***3 4 5*** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳&nbsp;The last three digits are for unique identification and have no particular meaning.

### Parsing Errors

Parsing errors are generated when the importer cannot make sense of the contents of a CIMPL statement. **Parsing error messages include the file, line, and column number, and can be linked directly to a location in a CIMPL file where parsing failed.** Most other types of errors (with the exception of mapping errors) cannot be linked to a specific file location because they represent some type of a logical inconsistency.

A typical parsing error looks like this:

```
extraneous input 'Entry' expecting {<EOF>, 'CodeSystem:', 'Abstract:', 'Element:', 'Entry:', 'Group:'}.
ERROR_CODE:11023 (module=shr-text-input, file=..\shr_spec\spec\ns-onco-core\oncocore.txt, line=9, column=0)
```
When a parsing error occurs, the associated error code number and subsequent errors are not meaningful, since the entire model could not be read.

### Warning Codes

| Warning Code | Warning Message | Potential Solution |
| ------------ | --------------- | -------- |
| 01001 | No project configuration file found, currently using default EXAMPLE identifiers. Auto-generating a proper 'config.json' in your specifications folder | Open the 'config.json' file and customize it for your project. |
| 01002 | Config file missing key: `$KEY`, using default key: `$DEFAULT VALUE` instead.   | Open the 'config.json' file and add your project specific details for that key.
| 02001 | Potentially mismatched targets: `$CLASS` maps to `$ITEM`, but based on class (`$CLASS`) maps to `$ITEM`, and `$ITEM` is not based on `$ELEMENT` in `$CLASS`.' | You are overriding an inherited mapping. |
| 03001 | Trying to map `$PROFILE` to `$CODE`, but `$PROFILE` was previously mapped to it | |
| 03002 | Choice has equivalent types, so choice options may overwrite or override each other when mapped to FHIR. | |
| 03003 | Overriding extensible value set constraint from `$VS1` to `$VS2`.  Only allowed when new codes do not overlap meaning of old codes. | The "extensible" binding means that a code from outside the given value set should be used only if the value set does not contain a suitable code. |
| 03004 | Element profiled on Basic. Consider a more specific mapping. | The Basic profile should not be used in most cases. Consider a more specific profile mapping that categorizes the Element being mapped. |
| 03005 | No mapping to '`$ELEMENT PATH`'. | This property is core to the target resource and usually should be mapped. |  |
| 03006 | The `$PROPERTY` property is not bound to a value set, fixed to a code, or fixed to a quantity unit. This property is core to the target resource and usually should be constrained |Usually a result of not setting the Code attribute of an Observation. |
| 04001 | Unsupported code system: '`$CODESYSTEM`' | |

### Compilation Error Codes

| Error Code    | Error Message | Solution |
| ------------  | ------------- | -------- |
| 11001          | Element name '`$NAME`' should begin with a capital letter | Rename the specified Element |
| 11002          | Entry name '`$NAME`' should begin with a capital letter    | Rename the specified Entry |
| 11003          | Unable to resolve value set reference: `$VALUESET`                 | Invalid value set reference, double check the name and the path
| 11004          | Unsupported binding strength: `$BINDING_STRENGTH`.  Defaulting to REQUIRED     | Binding strength has to be one of the following: example, preferred, extensible, or required (default). |
| 11005          | Error parsing source path: `$PATH`                                 | Invalid path to definitions. Double check path. |
| 11006          | Invalid config file. Should be valid JSON dictionary               | Make sure your 'config.json' file is using a valid format for JSON. |
| 11007          | Unsupported grammar version: `$VERSION`                            | Grammar Version for file must be 5.0 (or above) |
| 11008          | Defining value sets by URL has been deprecated in ValueSet files.  ValueSet `$VALUESET` ignored.           | Define the value set with a name using proper syntax. |
| 11009          | Defining value sets by URN has been deprecated in ValueSet files.  ValueSet `$VALUESET` ignored.           | Define the value set with a name using proper syntax. |
| 11010          | Couldn’t resolve code system for alias: `$ALIAS`                   | Invalid Codesystem, double check spelling |
| 11011          | Uses statements have been deprecated in ValueSet files.  Uses statement ignored.                  | Uses statement is unnecessary. Refer to documentation for proper syntax |
| 11012          | Only default path definitions are allowed in ValueSet files.  Path definition ignored.            | Use one of the preset path definitions defined in the documentation. |
| 11013          | Failed to resolve definition for `$ELEMENT_NAME`                   | The referenced Element doesn't exist in the current namespace, or in any of its inherited parents. Check spelling errors as well as imports. |
| 11013          | Failed to resolve definition for `primitive`                       | Only certain primitives are supported. Please refer to the documentation to see the full list.
| 11015          | token recognition error at: `$CHARACTER` | This is usually a typo issue. Investigate keywords and missing colons around the specificed text input.
| 11016          | mismatched input `$INPUT` expecting `$LIST_OF_KEYWORDS` | This is usually a typo issue. Investigate spelling and keywords used around the specificied text input.
| 11017          | Cannot resolve path without namespaces | There was a failure to parse the namespace. Ensure the namespace is correctly defined.
| 11018          | Failed to resolve path for `$NAME`. |
| 11019          | Found conflicting path for `$NAME` in multiple namespaces: `$NAMESPACES` | 
| 11020          | Failed to resolve vocabulary for `$NAME`. | 
| 11021          | Found conflicting vocabularies for `$NAME` in multiple namespaces: `$NAMESPACES` | 
| 11022          | Found conflicting definitions for `$NAME` in multiple namespaces: `$NAMESPACES` | 
| 11023          | Elements cannot be based on "Value" keyword |
| 11024          | Elements cannot use "Value:" modifier and specify "Value" field at same time. |
| 11025          | Fields cannot be constrained to type "Value" |
| 11026          | ref(Value) is an unsupported construct; treating as Value without the reference. |
| 11027          | Unable to import property `$FQN`, unknown value type: `$VALUE_TYPE` | The type either does not exist, or the import tool needs to be updated.
| 11028          | Unable to import unknown constraint type: `$CONSTRAINT_TYPE`| The type either does not exist, or the import tool needs to be updated.
| 11029          | Unable to import mapping, unknown rule type: `$RULE_TYPE` | The type either does not exist, or the import tool needs to be updated.
| 11030          | Unable to import VS rule, unknown rule type: `$RULE_TYPE` | The type either does not exist, or the import tool needs to be updated.
| 11031          | Unable to import FixedValueConstraint, unknown fixed value type: `$RULE_TYPE` | The value type either does not exist, or the import tool needs to be updated.
| 11032          | Project configuration not found! Exiting the program. | There was an error finding or loading the configuration file. Please double check that it exists and is valid.
| 11033          | Name $ELEMENT_ENTRY_NAME already exists. | The entity or element name already exists within the namespace and the most recently defined element or entry name will be used.
| 11034          | ValueSet name $VS_NAME already exists. | The value set name already exists within the namespace.
| 11035          | Definition not found for data element in content profile path: `$ELEMENT_FQN`. | Could indicate a missing definition, misspelling, or missing `Uses` declaration. |
| 11036          | Path not found for `$ELEMENT_FQN`: `$PATH`. | Usually when sub-elements in a dotted path (foo.bar.baz) can't be traced. See [CIMPL path](#cimpl6LanguageReference.md#cimpl-paths) for more information. |
| 11037          | Could not find content profile file: `$FILE_NAME`. | Check the name or path to the content profile file specified in your configuration file. |
| 11038          | Namespace declaration not found. | File needs a namespace declaration at the top of the file under the Grammar declaration. |
| 11039          | Grammar declaration not found. | File needs a Grammar declaration at the top of the file. |
| 11040          | Property `$PROPERTY` already exists. | Extra declaration of Property needs to be removed. |
| 11041          | Choice value constrained without specifying the specific choice. |
| 11042          | Constraint refers to previous identifier. |
| 11043          | Value should not be declaring cardinality. | Remove cardinality declared for the value. |
| 11044          | Missing a value element. |
| 12001          | Cannot resolve element definition.                                 | Element doesn't exist. Double check spelling and inheritance |
| 12002          | Reference to non-existing base: `$ELEMENT_NAME`                    | Base doesn't exist. Double check spelling and inheritance. |
| 12003          | No cardinality found for value: `$VALUE`                           | Explicitly define cardinality for that value. |
| 12004          | No cardinality found for field: `$FIELD`                           | Explicity define cardinality for that field. |
| 12005          | Cannot override `$OLD_VALUE` with `$NEW_VALUE`                                                | Double check types match. |
| 12006          | Cannot override `$OLD_VALUE` with `$NEW_VALUE` since it is not one of the options             | Verify Identifiers match. |
| 12007          | Cannot override `$OLD_VALUE` with `$NEW_VALUE`                                                | Verify Identifiers match. |
| 12008          | Cannot override `$OLD_VALUE` with `$NEW_VALUE` since overriding ChoiceValue is not supported  | Verify Identifiers match. |
| 12009          | Unsupported constraint type: `$CONSTRAINT` Invalid constraint syntax.    | Consult documentation to see what constraints are supported |
| 12010          | Cannot constrain cardinality of `$NAME` from `$SMALL_CARDINALITY` to `$BIGGER_CARDINALITY`              | You can only narrow the cardinality. You cannot constrain it to have a larger range than its parent |
| 12011          | Cannot further constrain cardinality of `$NAME` from `$CARDINALITY` to `$CARDINALITY`      | You can only narrow the cardinality. You cannot constrain it to have a larger range than its parent |
| 12012          | Cannot constrain type of `$NAME` to `$TYPE`                        | Make sure base types match |
| 12013          | Cannot constrain type of `$NAME` since it has no identifier        | Invalid Element |
| 12014          | Cannot constrain type of `$NAME` to `$TYPE`                        | Make sure base types match |
| 12015          | Cannot further constrain type of `$NAME` from `$TYPE` to `$TYPE`   | The two elements aren't based on the same parent. You cannot constrain an element to one that is completely distinct. |
| 12017          | Cannot constrain type of `$NAME` since it has no identifier        | |
| 12018          | Cannot constrain element `$NAME` to `$TARGET` since it is an invalid sub-type | Element has to be based on `$s` or otherwise is a child of `$s`. |
| 12020          | Cardinality of `$NAME` not found                                   | Please explicitly define the cardinality. |
| 12021          | Cannot include cardinality on `$NAME`, cardinality of `$CARD` doesnt fit within `$CARD` | The cardinality of included parameters must be as narrow or narrower than the  property it contains. |
| 12022          | Cannot constrain valueset of `$NAME` since it has no identifier    | |
| 12023          | Cannot constrain valueset of `$NAME` since neither it nor its value is a code, Coding, or CodeableConcept                              | ? |
| 12024          | Cannot constrain valueset of `$NAME` since it is already constrained to a single code                                                  | ? |
| 12025          | Cannot constrain code of `$NAME` since neither it nor its value is a code, based on a Coding, or based on CodeableConcept              | ? |
| 12026          | Cannot constrain included code of `$NAME` since neither it nor its value is a code, based on a Coding, or based on CodeableConcept     | ? |
| 12027          | Cannot constrain boolean value of `$NAME` since neither it nor its value is a boolean                                                  | ? |
| 12028          | Cannot constrain boolean value of `$NAME` to `$VALUE` since a previous constraint constrains it to `$VALUE`                                        | ? |
| 12029          | Cannot resolve element definition for `$NAME`                      | This is due to a incomplete definition for an element. Please refer to the document for proper definition syntax. |
| 12030          | Cannot determine target item                                       | System error. |
| 12031          | Cannot resolve data element definition from path: `$PATH`          | Check spelling for field or value. |
| 12032          | Cannot resolve data element definition from path: `$PATH`          | Check spelling for field or value. |
| 12033          | Cannot map Value since element does not define a value             | Define a value for your element |
| 12034          | Cannot map Value since it is unsupported type: `$VALUE_TYPE`       | ? |
| 12035          | Found multiple matches for field `$FIELD`                          | Please use fully qualified identifier. |
| 12036          | Could not find expanded definition of `$ELEMENT`. Inheritance calculations will be incomplete. | Double check `shr.base.Entry` is defined within the specifications. |
| 12037          | Could not find based on element `$ELEMENT` for child element `$ELEMENT`. | Double check the `basedOn` element is defined within the specifications and correctly referenced. |
| 14001          | Unsupported value set rule type: `$s` |
| 14002          | Unknown type for value `$VALUE` |
| 14003          | Unknown type for constraint `$CONSTRAINT` |

### Mapping Error Codes

| Error Code    | Error Message | Solution |
| ------------  | ------------- | -------- |
| 13001          | Invalid FHIR target: `$TARGET` | Could not find the FHIR resource or profile you're trying to map to. Check spelling and FHIR version. |
| 13002          | Cannot flag path as mapped |
| 13003          | Slicing on include type constraints with paths is not supported |
| 13004          | Slicing required to disambiguate multiple mappings to `$TARGET` |
| 13005          | Invalid source path |
| 13006          | Invalid or unsupported target path |
| 13007          | Cannot unroll contentReference `$CONTENT_REFERENCE` on `$ELEMENT` because it is not a local reference |
| 13008          | Invalid content reference on `$ELEMENT`: `$CONTENT_REFERENCE` |
| 13009          | Cannot unroll `$ELEMENT`. Create an explicit choice element first. |
| 13010          | Cannot unroll `$ELEMENT` at `$ELEMENT`: invalid SHR element. |
| 13011          | Cannot make choice element explicit since it is not a choice ([x]): `$ELEMENT` |
| 13012          | Cannot make choice element explicit at `$ELEMENT`. Invalid SHR identifier: `$IDENTIFIER`. |
| 13013          | Invalid target path. Cannot apply cardinality constraint. |
| 13014          | Cannot constrain cardinality from `$CARD` to `$CARD` |
| 13015          | Invalid target path. Cannot apply fixed value. |
| 13016          | Currently, only fixing codes is supported (value must contain "#").  Unable to fix to `$VALUE`. |
| 13017          | Incompatible cardinality (using aggregation). Source cardinality `$CARD` does not fit in target cardinality       | |
| 13018          | Cannot constrain cardinality to `$CARD` because cardinality placement is ambiguous. Explicitly constrain          | parent elements in target path.
| 13019          | Cannot constrain cardinality to `$CARD` because there is no tail cardinality min that can get us there |
| 13020          | Cannot constrain cardinality to `$CARD` because there is no tail cardinality max that can get us there |
| 13021          | Cannot constrain cardinality to `$CARD` because there is no tail cardinality that can get us there |
| 13022          | Mismatched types. Cannot map `$SOURCE_VALUE` to `$MAPPING`. | Find the `EntryElement` referenced in the error details and change it to match the data type of the target field.
| 13023          | Cannot resolve element definition for `$ELEMENT` |
| 13024          | Failed to resolve element path from `$ELEMENT` to `$PATH` |
| 13025          | Applying constraints to profiled children not yet supported. SHR doesn\ |
| 13026          | Failed to resolve path from `$ELEMENT` to `$PATH` |
| 13027          | Unsupported binding strength: `$BINDING_STRENGTH` |
| 13028          | Cannot change binding strength from `$BINDING_STRENGTH` to `$BINDING_STRENGTH` |
| 13029          | Cannot override value set constraint from `$URI` to `$URI` |
| 13030          | Found more than one value set to apply to `$ELEMENT`. This should never happen and is probably a bug in the tool. |
| 13031          | Found more than one code to fix on `$ELEMENT`. This should never happen and is probably a bug in the tool. |
| 13032          | Can’t fix code on `$ELEMENT` because source value isn’t code-like. This should never happen and is probably a bug in the tool. |
| 13033          | Can’t fix code on `$ELEMENT` because source value isn’t code-like. This should never happen and is probably a bug in the tool. |
| 13034          | Cannot override code constraint from `$VALUE` to `$VALUE` |
| 13035          | Cannot override boolean constraint from `$VALUE` to `$VALUE` |
| 13036          | Found more than one boolean to fix on `$ELEMENT`. This should never happen and is probably a bug in the tool. |
| 13037          | Conversion from `$VALUE` to one of `$TYPE` drops boolean constraints |
| 13038          | Conversion from `$VALUE` to one of `$TYPE` drops value set constraints |
| 13039          | Conversion from `$VALUE` to one of `$TYPE` drops code constraints |
| 13040          | Conversion from `$VALUE` to one of `$TYPE` drops includesCode constraints |
| 13041          | Unable to establish namespace for `$ELEMENT` |
| 13042          | No slice name supplied for target. This should never happen and is probably a bug in the tool. |
| 13043          | Couldn’t find target in slice `$SLICE` | (Exporting) |
| 13044          | Target resolves to multiple elements but is not sliced |
| 13045          | Unable to establish namespace for `$FIELD` | (Extensions) |
| 13046          | Mapping to `MAP_TARGET`'s `RULE_TARGET`: slice could not be found. |
| 13047          | Couldn't find sd to unroll | 
| 13048          | Cannot override code constraint from `$SYSTEM`\|`$CODE` to `$SYSTEM`\|`$CODE`' |
| 13049          | Unexpected error processing mapping to FHIR. |
| 13050          | Unexpected error processing mapping rule. |
| 13051          | Unexpected error adding extension. |
| 13054          | Using profile that is currently in the middle of processing: `$PROFILE_ID`. |
| 13055          | Using extension that is currently in the middle of processing: `$EXTENSION_ID`. |
| 13056          | Can't fix `$TARGET` to `$VALUE` since `$TARGET` is not one of: `$ALLOWABLE_TYPES`. |
| 13057          | Could not fix `$TARGET` to `$VALUE`; failed to detect compatible type for value `$VALUE`. |
| 13058          | Cannot fix `$TARGET` to `$VALUE` since it is not a `$TYPE` type. |
| 13059          | Cannot fix `$TARGET` to `$VALUE` since it is already fixed to `$OTHER_VALUE`. |
| 13060          | Could not determine how to map nested value (`$ELEMENT_PATH`) to FHIR profile. | Occurs on the FHIR profile export when there are multiple levels of reference specified. e.g.: LaboratoryObservationTopic.Specimen.CollectionSite within LaboratoryObservationTopic.  Resolved by creating a reference in the cimi entity to FHIR map in cimi_entity_map.txt. |
| 13061          | Mapping `$ELEMENT` sub-fields is currently not supported. |
| 13062          | Cannot make choice element explicit at `$ELEMENT`. Could not find compatible type match for: `$TYPE`. |
| 13063          | Could not find FHIR element with path `$ELEMENT_PATH` for content profile rule with path `$RULE_PATH`. |
| 13064          | Could not find FHIR element for content profile rule with path `$RULE_PATH`. |
| 13065          | Could not find FHIR element subextension for content profile rule with path `$RULE_PATH`. |
