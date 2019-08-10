# CIMPL 6.0 Tooling Reference

>**Note:** This documentation is a draft.

_This is a comprehensive guide to CIMPL 6.0 Tooling, including the command line interface, auxiliary files, and configurations needed to produce a FHIR Implementation Guide (IG). If you're looking for a quick introduction to CIMPL and `SHR-CLI` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md) or [In-Depth Tutorials](cimpl6Tutorial_detail.md). For details of the CIMPL language itself, see the [CIMPL Language Reference Manual](cimpl6LanguageReference.md)._

***

**Table of Contents**
[TOC]

***

## Overview

CIMPL (**C**linical **I**nformation **M**odeling **P**rofiling **L**anguage) is a specially-designed language for defining clinical information models. It is simple and compact, with tools to produce [Fast Healthcare Interoperability Resources (FHIR)](https://www.hl7.org/fhir/overview.html) profiles, extensions and Implementation Guides (IGs). The CIMPL language itself, however, is static and must be processed to produce these outputs. This reference manual describes the configurations, files, and commands needed to create a FHIR IG from CIMPL language files.

The CIMPL Tooling, also called SHR-CLI (Standard Health Record Command Line Interface), is the engine that imports a set of inputs, including CIMPL language files, and exports FHIR and other outputs, as shown below:

#### _Figure 1: CIMPL Tooling Overview_ ####
![CIMPL Tooling Overview](img_cimpl/cli-overview.png)

The inputs to this process include:

* **CIMPL Language files**, including class files, value set files, and mapping files that define your clinical information model,
* A **Configuration file** that contains directives to the tooling, and points to other resources,
* An optional **Content Profile** file, which specifies Must-Support elements and profiling options specific to an IG,
* One or more **Front Matter** files, which are the narratives and graphics that introduce the IG,
* **FHIR Examples** that are to be included in the IG, and
* A **Package List** that has information required for building the IG.

Depending on the configuration, the outputs of the CIMPL Tooling may include:

* **FHIR Profiles, Extensions, Value Sets** that form the core content of the IG,
* A **Logical Model** corresponding to the CIMPL class definitions, expressed as FHIR StructureDefinitions,
* **JSON Schema** for the profiles defined by the IG,
* A **Data Dictionary** that lists the Must-Support data elements in the IG, as well as value sets and value set members,
* **Model Documentation** in the form of a Javadoc-like browser that allows one to see the hierarchical class relationships in the logical model.

In the following, we describe these inputs and outputs, and the process of running the CIMPl tooling, to produce a FHIR Implementation Guide.

This reference manual assumes that the CIMPL Language files (classes, value sets, and maps) have been defined (see [CIMPL Language Reference Manual](cimpl6LanguageReference.md) for details). It also assumes that the CIMPL SHR-CLI tooling has been installed according to the directions in the [Setup and Installation Guide](cimplInstall.md).

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
* **IG Directories** - these directories, under the specification directory contain files specific to a given implementation guide. There are subdirectories containing the frontmatter and examples, and individual configuration, content profile, and package list files. These files can be specific to a FHIR release, since the same IG could be created using different versions of FHIR.

**Note:** There is currently no way to combine profiles for multiple FHIR versions in single IG.

## Configuration File

SHR-CLI requires a configuration file to run. The name of this file is typically specified on the command line using the -c command line option. If the name is not specified, the tooling looks for a file named `config.json` in the working directory. If it cannot be found, or the contents of the file are invalid, an error message is returned.

The configuration file is a JSON file with the following parameters:

|Parameter            |Type    |Description                                                    |
|:--------------------|:-------|:--------------------------------------------------------------|
|`projectName`        |`string`|The full, official name of the project, for example "HL7 FHIR Implementation Guide: minimal Common Oncology Data Elements (mCODE) Release 1 - US Realm, STU Ballot 1"  |
|`projectShorthand`   |`string`|A shorthand name for the project, such as "mcode".                              |
|`projectURL`         |`string`|The primary URL for the project, such as "http://hl7.org/fhir/us/mcode/"                             |
|`fhirURL`            |`string`|The FHIR IG URL for the project, often the same as the projectURL. **(TO DO: clarify the difference)**  |
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
After the import stage (see  [Figure 1](#figure-1-cimpl-tooling-overview)), there is an opportunity to remove unwanted elements to limit the scope of the exports. The filter strategy controls which classes are included in the export stage, and subsequently, the IG. If this parameter is omitted, no filtering will occur.

The contents of the `filterStrategy` object are as follows:

|Parameter |Type |Description |
|---------|------|------------|
|`filter`  |`boolean`|A value indicating whether to enable filtering. If `filter` is `true`, then the filtering operation will occur. Otherwise, no filtering will occur. |
|`strategy`|`string` | The strategy for specification filtering, either "namespace", "element", or "hybrid".|
|`target`  |`[]`     |An array of strings containing the names of Entries, Namespaces, or both. |

* The "element" strategy filters the imported classes to include each Entry listed in the `target` array, and their recursive dependencies.
* The "namespace" strategy filters the imported classes to include only Entries in the namespaces listed in the `target` array, and their recursive dependencies.
* The "hybrid" strategy filters the imported classes to include each Entry listed in the `target` array and all Entries in every namespace listed in the `target` array, and their recursive dependencies.

When specifying an Entry in the `target` array, it is best to use the fully qualified name (FQN) format for doing so. For example, a namespace could be `oncology` and an Entry could be `oncology.BreastCancerStage`.

### Implementation Guide Configuration Parameters
These configurations are used to control the production of the IG. The contents of the `implementationGuide` object are as follows:

|Parameter                 |Type     |Description                                                          |
|:-------------------------|:--------|:--------------------------------------------------------------------|
|`npmName`                 |`string` |The assigned node package manager name for this IG, for example "hl7.fhir.us.mcode". The npm name is usually assigned by HL7.   |
|`version`                 |`string` |The version of this IG (not necessarily the version of FHIR).  |
|`ballotStatus`            |`string` |The HL7 ballot status of the IG (e.g., STU1 Ballot, Continuous Integration Build, etc.)      |
|`packageList`             |`string` |The name of the file to use as the package-list.json for publication, relative to the Specification Directory, for example, "ig-mcode/ig-mcode-plist.json". |
|`includeLogicalModels`    |`boolean`|A "true" or "false" value indicating whether to include logical models in the IG.      |
|`includeModelDoc`         |`boolean`|A "true" or "false" value indicating whether to include the model documentation in the IG.       |
|`indexContent`            |`string` |The name of the file or folder containing the frontmatter, relative to the Specification Directory, for example, "ig-mcode/IndexFolder-Oncocore".    |
|`extraResources`          |`string` |The name of the folder containing extra JSON resources to include in the IG. **(TO DO: clarify where and how this is used)**|
|`examples`                |`string` |The name of the folder containing examples to include in the IG, relative to the Specification Directory, for example, "ig-mcode/Examples-mCODE-r4" |
|`historyLink`             |`string` |The URL for the page containing the IG's history information.  **(TO DO: clarify where and how this is used)**   |
|`changesLink`             |`string` |The URL to a site where users can request changes (shown in page footer) **(TO DO: clarify where and how this is used)** |
|`primarySelectionStrategy`|`{}`     |The strategy for selection of what is primary in the IG. |

The file indicated by `packageList` will be used as the `package-list.json` file for publication.  This file is required for all HL7 IGs.  If a `packageList` file is not indicated, it will default to `package-list.json`.  If a package file exists at the configured location, it will be used.  Otherwise, if the configured `fhirURL` is an hl7.org or fhir.org URL (indicating it is an HL7 publication), a basic package list file will be created.  In this case, the IG author should review and modify the file as needed and then check it into the version control system.  For more information about the `package-list.json` file, see: [http://wiki.hl7.org/index.php?title=FHIR_IG_PackageList_doco](http://wiki.hl7.org/index.php?title=FHIR_IG_PackageList_doco).

If the `indexContent` value is a path to a folder (relative to the spec directory), then it should contain an `index.html` file whose contents will be used as the body of the IG landing page.

The folder indicated by `extraResources` should include one file per JSON-formatted resource to include.  Currently, the following resource types are supported: `SearchParameter`, `OperationDefinition`, `CapabilityStatement` (STU3+), `Conformance` (DSTU2).  If files are detected, links are added to the navigation menu as necessary.

The folder indicated by `examples` should include one file per JSON-formatted example.  Each example may be named as the author wishes, but we recommend the example name match the example `id` in the file (with a `.json` file extension added).  The example JSON must contain an `id` and the example's `meta.profile` should include the canonical URL for the profile it exemplifies (e.g., `"meta": { "profile": [ "http://hl7.org/fhir/us/breastcancer/StructureDefinition/oncology-BreastCancerPresenceStatement" ] }`).

_NOTE: For backwards compatibility, if no `examples` folder is specified in the config, and a folder named "fhir-examples" exists in the spec directory, it will be used as the examples folder._

The contents of the `primarySelectionStrategy` object are as follows:

|Parameter |Type    |Description |
|--------|------|-----------|
|`strategy`|`string`|The strategy to follow for primary selection (`"namespace"`, `"hybrid"`, or default `"entry"`). |
|`primary` |`[]    `|An array containing the namespaces and entries (only used for `"namespace"` and `"hybrid"` `strategy`). |

The primary selection strategy causes certain profiles to be listed in a "Primary" section at the top of their respective pages in the IG, displaying them as most directly relevant. All other profiles exported in step 3 are listed in a "Supporting" section below the "Primary" section.

The options for the configuration file's `implementationGuide.primarySelectionStrategy` are described below.

* The `"entry"` `strategy` selects every profile in the filtered set as primary in the IG.
* The `"namespace"` `strategy` selects every profile found in the namespaces of the `primary` array as primary in the IG.
* The `"hybrid"` `strategy` for primary selection sets every entry listed in the `primary` array or found in the namespaces in the `primary` array as primary in the IG.
* If there is no `strategy` set in the `implementationGuide.primarySelectionStrategy`, the default operation is the `"entry"` `strategy`.

When specifying a namespace or element in the `primary` array, it is best to use the fully qualified name (FQN) format for doing so. For example, a namespace could be `"shr.oncology"` and an element could be `"shr.oncology.BreastCancerStage"`.

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

### Executing CLI

When using this command-line interface and IG publisher, the general order of operations is as follows:

The command-line interface (CLI) imports SHR definitions that have been written (as in the shr_spec repo) and parses them through a text importer.
CLI applies filters, according to the filterStrategy (see below).
CLI exports the filtered SHR definitions into desired formats, such as FHIR, ES6, etc. The exports can be selected through command line options, as explained above.
(separate, optional step) The IG publisher takes the SHR FHIR export and generates an IG from the information in these files, following the implementationGuide configuration (see below).
To control this process, CLI requires a configuration file. Configuration files must be valid JSON, have at least the projectName property, and use the .json file extension. The configuration file must be located in the path to the SHR specification definitions.



$ node <path-to-cli>  <path-to-shr-definitions> [options]



It is also possible to override the default logging level or format, skip exports, override the default output folder, or change the configuration file to use:

$ node . --help

  Usage: shr-cli <path-to-shr-definitions> [options]

  Options:

    -l, --log-level <level>  the console log level <fatal,error,warn,info,debug,trace> (default: info)
    -m, --log-mode <mode>    the console log mode <short,long,json,off> (default: short)
    -s, --skip <feature>     skip an export feature <fhir,cimcore,json-schema,model-doc,data-dict,all> (default: <none>)
    -o, --out <out>          the path to the output folder (default: out)
    -c, --config <config>    the name of the config file (default: config.json)
    -d, --duplicate          show duplicate error messages (default: false)
    -j, --export-es6         export ES6 JavaScript classes (experimental, default: false)
    -i, --import-cimcore     import CIMCORE files instead of CIMPL (default: false)
    -n, --clean              Save archive of old output directory and perform clean build (default: false)
    -h, --help               output usage information
For example:

node . ../shr_spec/spec -l error -o out2 -c other_config.json


### Error Messages

Interpretation of errors.

Fixing errors.

## Exporters

### FHIR Export

###

