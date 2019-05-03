# CIMPL Authoring
_The purpose of this guide is to educate people about many different aspects of creating CIMPL models and its supporting utilities.  If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md).  If you're looking for detailed guidance on CIMPL syntax, try the [CIMPL6 Reference documentation](cimpl6Reference.md)._

***

**Table Of Contents**

[TOC]

***
# Authoring Environment

Any text editor can be used to write CIMPL grammar. However, [VSCode editor](https://code.visualstudio.com/) is recommended to take advantage of custom-developed to better navigate CIMPL constructs.  

## Setting up the VSCode Authoring

>**Note:** VSCode UI screenshots in this section were taken from a MacOS enironment. While the overall functionality is the same across supported OS platforms, installation and configuration specifics might differ. Reference the VSCode documentation pertinent to your OS platform.

* Download the [VSCode editor](https://code.visualstudio.com/). 
* Open VSCode and search for the extension **vscode-lang-cimpl**.  The figure below shows where to find VSCode extensions. 

![CIMPL VSCode Extenstion](img_cimpl/VSCodeLangCimplExtension_small.png)

### Navigating a CIMPL Model within VSCode

Elements properties can be previewed in the following ways:

1. hovering over the element.
2. placing the cursor on the element text and right-clicking option _Peek Definition_
3. placing the cursor on the element text and right-clicking option _Go to Definition_

<br />

Hovering over the element:
![Hovering over the element](img_cimpl/VSCode_Peek01.png)

Using _Peek Definition_:
![Using Peek](img_cimpl/VSCode_Peek02.png)

Using _Go to Definition_:
![Using Go to Definition](img_cimpl/VSCode_GotoDef.png)

# Support
Questions on using CIMPL and its toolchain can be addressed on the HL7 Zulip chat channel [#cimpl](https://chat.fhir.org/#streams/197290/cimpl)

Report any issues on one of the following GitHub repositories:

* Related to modeling of CIMPL constructs or its FHIR-based classes: https://github.com/standardhealth/shr_spec/issues 
* Related to running the CIMPL `shr-cli` compiler, CIMPL export configuration files, or generating the FHIR Implementation Guide (IG): https://github.com/standardhealth/shr-cli/issues