# CIMPL 6.0 In-depth Tutorial
_This is an extensive tutorial, meant to educate people about many different aspects of CIMPL.  If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](cimpl6Tutorial_helloWorld.md).  If you're looking for detailed guidance on CIMPL syntax, try the [CIMPL6 Reference documentation](cimpl6Reference.md)._

***

**Table Of Contents**

[TOC]

***

# Pre-requisite

This tutorial assumes that you already have the following software components installed:

* [CIMPL shr-cli](cimplInstall.md) (preferably installed in the `~/cimpl/shr-cli` directory)
* A text-editor (preferably VSCode with the _vs-code-language-cimpl_ extension, but not required)

>**Note:** _This tutorial was run using a MacOS environment.  While the content of CIMPL authoring are identical regardless of platform, the command lines run in a command line terminal will differ in file path specifications.  Namely, for a Windows command line terminal, replace all the path references of forward-slash `/` to back-slash `\`._


## Tutorial Pre-configuration

To keep this tutorial more focused on how to model, we're first going to setup the environment with some supporting configuration and core data type files.

We're also going to assume the following directory structure for your configuration:

```
Directory:  cimpl
            |_ shr-cli
```

>**Note:** _Your top level directory where shr-cli was install might differ. If so, then you'll be instead referencing the relative directory where you have installed shr-cli._

Create a sub-directory under `cimpl` called **`myExampleC6`**
Now your directory structure should look like this:

```
Directory:  cimpl
            |_ shr-cli
            |_ myExampleC6
```

Copy the following files to the `myExampleC6` sub-directory.  These file names are hyperlinked for retrieval:

* [shr_core_datatype.txt](./cimplTutorial/shr_core_datatype.txt)
* [shr_core_datatype_vs.txt](./cimplTutorial/shr_core_datatype_vs.txt)
* [shr_core_datatype_map_r4.txt](./cimplTutorial/shr_core_datatype_map_r4.txt)
* [ig-myExampleR4-config.json](./cimplTutorial/ig-myExampleR4-config.json.txt)

# Introduction

This tutorial will guide the user in a step by step process to build a basic clinical data model using the "Clean Slate" method of CIMPL authoring. It will also illustrate the modeling practice briefly described in Appendix A of the [CIMPL 6 Model Authoring Guide](cimpl6UserGuide.md).

The steps are briefly summarized here for conciseness:

* Define use cases.
* Create a high-level conceptual model.
* Create a data element requirement list.
* Create the logical model.
* Create logical element-to-FHIR mappings.
* Generate the FHIR Implementation Guide.
* Create FHIR examples for each profile. (optional).

## Define Use Cases

For our example, we'll focus on a simple use case scenario involving [obstructive sleep apnea (OSA)](https://www.webmd.com/sleep-disorders/guide/understanding-obstructive-sleep-apnea-syndrome#1).

The goal is to measure the prevalence of patients who are actively diagnosed OSA, categorized by different age groups. A secondary desirable but not required analysis would further identify the population by gender. Electronic Health Records (EHRs) are the primary source of this information.

## Create a High-level Conceptual Model

Based on the use case scenario, we determine that there are two clinical domains which are involved:

* Patient
* Disease (scoped to just Obstructive Sleep Apnea)

A basic conceptual diagram can quickly be drawn as shown in the figure below:

![Tutorial1 ConceptualModel](img_cimpl/cimplTutorial1_ConceptModel.png)

While it appears very simple, this illustration quickly identifies to a non-technical community (who also likely defined the use case), the relevant information which will drive the design of the model.  Here, we can clearly see that we are associating the a disease, narrowed down to just OSA, with a patient population where we require knowing the patient's age, and preferably the patient's gender.

## Create a Data Elements Requirements List

Creating a data elements requirements list further refines the data types, cardinality, and any clinical terminologies that would be needed to drive the model. Domain-specific research on existing standards and related models occur.

For our use case scenario, we're keeping it simple. We're looking at what's been done in the HL7 FHIR standard as a reference model.  This is made more convenient for our model as `shr-cli` supports FHIR exports in DSTU2, STU_3, and R4.

The table below is a high level summary of the elements we seek for this use case:

| Entity | Data Element Name | Cardinality | Data type | Value Set (if applicable) |
|-------|-------|-------|-------|-------|
| Patient | BirthDate | 1..1 | date | |
| Patient | Gender | 0..1 | concept | HL7 Gender |
| OSADisorder | OSADisorderCode | concept | SNOMED codes for OSA |
| OSADisorder | OSADisorderStatus | concept | HL7 status codes |

## Create the Logical Model in CIMPL

Now we can start authoring models in the CIMPL development environment.

In this example, we are using the "clean slate" method of defining our logical model elements without using the pre-existing "ObjectiveFHIR" class library.

CIMPL has 3 main files involved in the creation of a logical model. We list them here for brevity, but reference the CIMPL 6 Reference Guide for further details on each:

* data element file
* value set file
* mapping file

Let's first create the data element file:

Create a new file under the `myExample6` subdirectory called **`myExample.txt`**.

Type in the CIMPL header information designating the namespace and version of the DataElement parser:

```
Grammar:	    DataElement 6.0
Namespace:	    myExample
Description:    "CIMPL Tutorial: myExample for an OSA patient."
Uses:  		    shr.core
```
Where
* `Grammar:` is a fixed keyword with the Data Element parser supported.  At this time, fix the value to _DataElement 6.0_
* `Namespace:` is the name of your defined logical model namespace.
* `Uses:` is the list of all namespaces which your model elements will reference.  For our example, we fix this to _shr.core_.

Next, we will create our first profile called **`MyPatient`** which we determined will also have the following properties:
* `MyBirthDate` (a required field which only accepts one value)
* `MyGender` (an optional field that we want to have the receive accept if it so happens to have a value sent in any FHIR instance.)

Let's build our Patient logical model. Add the following lines below your CIMPL header.

```
Entry:			MyPatient
Description:    "an individual awaiting or under medical care and treatment. (source: Webster's Dictionary)"
Property:       MyBirthDate 1..1
Property:		MyGender 0..1

	Element:		MyBirthDate
	Description:	"The date when the patient was born."
	Value:			date

	Element:		MyGender
	Description:	"Either of the two sexes (male and female), especially when considered with reference to social and cultural differences rather than biological ones. (source: Oxford Dictionary)"
	Value:			concept from http://hl7.org/fhir/ValueSet/administrative-gender
```

What we just represented here is that there is a `Patient` entity called `MyPatient` which has 2 properties: `MyBirthdate` and `MyGender`.  

We determined from our clinical requirements that `MyBirthDate` has a data type of _date_, which is one of the primitive data types supported in CIMPL.

We also determined that `MyGender` is a value set that can be _male_, _female_, or _unknown_.  For the sake of simplicity, we find that HL7 has a matching value set identical to our requirements so we'll use that one for its value.

Now let's work on the OSA Disorder entity.

Add the following text into the file:
```
Entry:	ObstructiveSleepApneaDisorder
Description:	"Obstructive sleep apnea (OSA) is a sleep-related breathing disorder that involves a decrease or complete halt in airflow despite an ongoing effort to breathe. It occurs when the muscles relax during sleep, causing soft tissue in the back of the throat to collapse and block the upper airway. (source: American Academy of Sleep Medicine)"
Property:	OSADisorderCode 1..1
Property:	OSACurrentStatus 0..1

	Element: 		OSADisorderCode
	Description:	"The diagnosis or problem list code assigned to the OSA disorder."
	Value:			concept from OSADisorderVS

	Element: 		OSACurrentStatus
	Description:	"The diagnosis or problem list code assigned to the OSA disorder."
	Value:			concept from OSACurrentStatusVS
```

Here we specify the entity called `ObstructiveSleepApneaDisorder` containing two properties to define the disorder code and current status.  Both properties have been defined as concepts which  reference value sets that need to be defined.

### Creating Custom Value Sets

Create a new file called **`myExample_vs.txt`** underneath the `myExampleC6` directory.

Add the CIMPL value set header information below:
```
Grammar:	ValueSet 5.0
Namespace:	myExample
```

Where:
* `Grammar:` is the keyword which specifies that the file created is a valueset file conforming to CIMPL value set parser version 5.0.
* `Namespace:` is the keyword whose value contains the namespace defined for your model.  In this case, it's `myExample`.

Next we specify the terms in the value set which we've defined in our clinical data requirements.

Add the following text below the CIMPL value set header information:

```
CodeSystem:     SCT = http://snomed.info/sct
CodeSystem:     HL7CONDSTAT = http://terminology.hl7.org/CodeSystem/condition-clinical

ValueSet:           OSADisorderVS
Description:        "The list of disorders related to Obstructive Sleep Apnea."
SCT#1101000119103   "Obstructive sleep apnea of adult (disorder)"
SCT#1091000119108   "Obstructive sleep apnea of child (disorder)"

ValueSet:               OSACurrentStatusVS
Description:            "The current disease status of an OSA patient"
HL7CONDSTAT#active      "Active"
HL7CONDSTAT#inactive    "Inactive"
HL7CONDSTAT#resolved    "Resolved"
```

Here we have defined the terminology coding system reference by the value set, specified using the `CodeSystem:` keyword up front.  This allows us to tag a short-hand notation of the coding system to the code for each term in the value set.  It also allows for the flexibility of specifying terms with different coding systems within a value set, if applicable.

The format for specifying each term in the value set is as follows:
`<CodeSystem Abbreviation>#<Term Code>`

For example: `HL7CONDSTAT#active      "Active"`
Where:
* HL7CONDSTAT is the value assigned to `CodeSystem:` which contains the canonical URL or urn: of the coding system.
* `#` is a delimiter separating the coding system and the code
* _`active`_ is the term code for the concept
* `"Active"` is the display name associated with the term code.

## Create Logical Model to FHIR mappings

Create a file called **`myExample_map_r4.txt`** under the `myExampleC6` sub-directory.

Copy the following header information to the top of the file:

```
Grammar:	Map 5.1
Namespace:	myExample
Target:		FHIR_R4
```
Where:
* `Grammar:` is the version supported for the mapping grammar
* `Namespace:` is the name of the namespace for your logical model
* `Target:` is the FHIR release version targeted for mapping elements in your logical model.  These could be one of the following values:
    FHIR_DSTU2
    FHIR_STU_3
    FHIR_R4

Our mapping will be to FHIR_R4.

Cut and paste the following code below the map header information:

```
MyPatient maps to Patient:
    MyBirthDate maps to birthDate
    MyGender maps to gender

ObstructiveSleepApneaDisorder maps to Condition:
    OSADisorderCode maps to code
    OSACurrentStatus maps to clinicalStatus
```

Here's the thought process behind our code block:

Because we have specified our target map to be _`FHIR_R4`_, our source for performing a structural and semantic mapping analysis will be using the [FHIR R4 Release resource list](https://www.hl7.org/fhir/resourcelist.html).

The goal for our mapping is to ask the following:

* Is there a FHIR resource which could be a good fit for our logical model entities?
* If yes, then is there an equivalent FHIR attribute for each of the properties in our logical model?
* If there is an equivalent FHIR attribute then do the data types, cardinality constraints, and terminology bindings align?

For simplicity, we made the example such that the answer is yes to all questions.  We have identified that the `MyPatient` entity can be based off of the **[FHIR R4 Patient resource](https://www.hl7.org/fhir/patient.html)**. Similarly our `ObstructiveSleepApneaDisorder` entity could be a customization of the **[FHIR R4 Condition resource](https://www.hl7.org/fhir/condition.html)**.
The logical model properties also match to equivalent FHIR attributes for each resource we've chosen to build our profiles. And because FHIR profiling does allow to further constrain and not loosen cardinality constraints and binding strengths, we can confirm that our elements also satisfy those requirements.

This mapping `Target:` to _`FHIR_R4`_ also configures the CIMPL toolchain to not only verify the mappings we specify to FHIR R4, and also generates the FHIR `StructureDefinition` files for our profile, filling in the specifications all of the other targeted resource elements which we haven't further customized.

## Generate the FHIR IG

In this section, we cover some of the extra CIMPL configuration steps that might be needed to customize the output of your FHIR IG.

### Specify the FHIR "Must-Support" elements

The CIMPL toolchain provides a separate configuration file called a "Content Profile" which serves 2 purposes:
* specifies which elements in your logical model that you want to designate in FHIR as **["must-support"](https://www.hl7.org/fhir/conformance-rules.html#mustSupport)**.
* instructs the CIMPL toolchain which elements you want autogenerated in a data dictionary output.

Create a file called **`ig-myExample-cp.txt`** under the `myExampleC6` sub-directory.

Paste the following code snippet into your file and save:

```
Grammar:        ContentProfile 1.0
/*---------------
Levels of Data Element Support
1) 1..1 = required 
2) 0..1  MS = required if known
3) 0..1 MS I = conditionally required  0..1 MS I  --- defer
4) 0..1 MS I = conditionally required if known  0..1 MS I  --- defer
See
http://wiki.hl7.org/index.php?title=Conformance_Implementation_Manual
------------------*/

Namespace: myExample

    MyPatient:
        MyGender MS

    ObstructiveSleepApneaDisorder:
        OSADisorderCode MS

```

### Create a Homepage for the IG

Create a new file called **`index.html`** under the `myExampleC6` directory.

Copy the following contents in this page:

```
<h1>myExample Implementation Guide</h1>
<p>CIMPL Tutorial demonstrating how to create FHIR profiles and an IG.</p>
```

### Configuring your output

For purpose of the tutorial's focus on modeling, we have already created a configuration file called `ig-myExampleR4-config.json` for our example and had you copy this file to the `myExampleC6` directory at the start of this tutorial.

But let's review some highlights of this file.  Click **[here](./cimplTutorial/ig-myExampleR4-config.json.txt)** to view the contents of `ig-myExampleR4-config.json`.

The table below shows some of the notable fields and values in this file. We add in the file line numbers so you can follow along with the example configuration. 

Not every field will be covered.  More advanced configuration options are outside of this tutorial.

| File Line Number | Configuration Field | Value | Additional Comments |
|-------|-------|-------|-------|
| 2 | `"ProjectName:"` | _"My Example Project"_ | The name of your modeling project. This will appear on the home page title of your generated IG. |
| 5 | `"fhirURL:"` | _"http://example.com/fhir"_ | The URL which will be prepended to all of your profile StructureDefinition canonical URLs. |
| 8 | `"contentProfile:"` | _"ig-myExample-cp.txt"_ | The name of the file which sets FHIR _must-support_ flags for any of the properties in your logical model. |
| 12 | `"strategy:"` | _"namespace"_ | This is the primary selection strategy. Set in this example to _"namespace"_, this identifies the CIMPL filter strategy that you are bringing in all of the logical model elements defined in your model namespace into the IG. Further details on the options for a primary selection strategy are covered outside of this tutorial. |
| 21 | `"indexContent":` | _"index.html"_| The location of the generated IG homepage. |
| 25 | `"strategy":` | _"namespace"_ | This sets the primary selection strategy for the CIMPL model data elements. Typically, this strategy setting matches the filter strategy noted in line 12 of this file. |

### Running scripts to compile CIMPL and IG Generation

Generating the FHIR IG is a 2-step process:

**Step 1:**  Run the SHR Command Line Interface (shr-cli) compiler.

* Open a command line terminal and navigate to the ./cimpl/shr-cli directory (or whichever directory you installed `shr-cli`).

* Run the following command:

    `node . ../myExampleC6 -l error -o myExampleC6r4 -c ig-myExampleR4-config.json`

Where:
* the first parameter after the node command, `../myExampleC6` is the path where your CIMPL modeling and configuration files are located.
* the `-l` parameter and _`error`_ value specifies logging to only show errors.
* the `-o` parameter and _`myExampleC6r4`_ value specify the name of the directory in which you want to generate the IG output.
* the `-c` parameter and _`ig-myExampleR4-config.json`_ value specify the name of the CIMPL configuration file to reference for the output.

After you run this first command, as an interim check, navigate to the `~/cimpl/shr-cli` folder.  You should see a new subdirectory created called `myExampleC6r4` which will eventually contain the StructureDefinition and html output of the generated IG. This is the value that was specified in the `-o` parameter in the above command.

**Step 2:**  Run the IG Publisher.

* From the same command line terminal, run the following:

    `java $JAVA_OPTS -jar myExampleC6r4/fhir/guide/org.hl7.fhir.publisher.jar -ig myExampleC6r4/fhir/guide/ig.json`

>**Note:** _The IG Publisher command in this detailed tutorial differs from the one referenced in the `HelloWorld` quick tutorial. This is on purpose because this more advanced command will allow you to specify the location where you want your IG rather than it being hard-coded to an ./out directory.  Moreover, if you're developing from a shared repository like GitHub, this will allow you to separate your CIMPL modeling source code from an output which might contain extraneous files irrelevant to your modeling code._

Navigate to the `~/cimpl/shr-cli/myExampleC6r4/fhir/guide/output` directory and open the **`index.html`** file in a browser.  This will display the home page of the generated FHIR IG for your example project.

![Tutorial 1 IG Homepage](img_cimpl/cimplTutorial1_IGHomepage.png)

## Create FHIR examples

We're now going to add a FHIR example that conforms to our profile.  Since the details creating FHIR examples are out of scope for this tutorial, we're going to supply one for you.

Create a subdirectory called **`examples_myFhirExamplesFolder`** under the `myExampleC6` folder.

```
Directory:  cimpl
            |_ shr-cli
            |_ myExampleC6
               |_ examples_myFhirExamplesFolder
```

Copy the file [myPatientExample1.json](./cimplTutorial/myPatientExample1.json) and its contents into the `examples_myFhirExamplesFolder`.

>**Note:** _The FHIR example can contain more than what the IG needs and still pass validation. What matters is that both of the required conformances specified in the FHIR profile are met._

Add the following line to your cimpl configuration file within the `"implementationGuide":` JSON object:

**`        "examples": "examples_myFhirExamplesFolder"`**

We have already added it for you so you can just check that the parameter is there.

The following screenshot shows an exmample of where you would find this parameter within the CIMPL configuration file `ig-myExampleR4-config.json`.

![Tutorial1 FHIR Example Configuration Location](img_cimpl/cimplTutorial1_FHIRExampleConfigLocation.png)

Now run shr-cli and the IG publisher commands again:

`node . ../myExampleC6 -l error -o myExampleC6r4 -c ig-myExampleR4-config.json`

`java $JAVA_OPTS -jar myExampleC6r4/fhir/guide/org.hl7.fhir.publisher.jar -ig myExampleC6r4/fhir/guide/ig.json`

Open the index.html file from the generated IG output in your browser and navigate to the **Profiles** tab and select the MyPatient profile.

You'll now find a tab called `Examples` which will contain a URL to the example within the `examples_myFhirExamplesFolder`.

![Tutorial1 FHIR Example IG Location](img_cimpl/fhirexample1Display.png)

So you might be thinking, "How does the IG know which profile to insert your example?"

The answer is that the name of the related FHIR StructureDefinition for the profile is specified within the FHIR example itself...

![Tutorial1 FHIR Example Configuration Location](img_cimpl/cimplTutorial1_FHIRExampleProfileReference.png)

The IG Publisher will match the canonical URL specified in the example with what is specified in the IG. Note that the value of the profile parameter must match the canonical URL of the profile where the StructureDefinition is located in order for the example to appear.
