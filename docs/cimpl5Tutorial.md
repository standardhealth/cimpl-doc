# CIMPL 5.0 Tutorial
_This is an extensive tutorial, meant to educate people about many different aspects of CIMPL.  If you're looking for a quick introduction to CIMPL and `shr-cli` environment setup, try the [Hello World](https://github.com/standardhealth/shr-cli/wiki/Hello-World).  If you're looking for detailed guidance on CIMPL syntax, try the [CIMPL documentation](https://github.com/standardhealth/shr-cli/wiki/CIMPL-Documentation)._

## Table of Contents
[TOC]

# Background
This tutorial assumes that you are already comfortable with:

* Using a code editor (e.g., [Visual Studio Code](https://code.visualstudio.com/))
* Using the command line on a unix-based OS (e.g., Linux, macOS)
* Data modeling (e.g., what separates a code from a value, what is a codeable concept)
* Data constraints (e.g., an element SHALL contain a specific code, another element MAY exist)

## Why use a modeling tool?
It can be difficult for people to jump directly into resource modeling, especially for extensive, domain-specific implementations like FHIR.  CIMPL is purpose-built to create clear, concise object models independent of their particular implementations.  Abstraction between data models and implementations allows for cleaner code and a more repeatable and understandable process for building implementation code.

## Data Elements, Maps, and Value Sets
CIMPL input files are either Data Elements, Maps, or Value Sets.

* **Data Element** files are the backbone of your input - they define individual data elements, whether those data elements are nested within other data elements, and the cardinalities of those data elements, among other things
* **Map** files dictate how the elements defined in your data model should be represented in a target implementation (for now, FHIR is the only output supported).
* **Value Set** files can be used to define custom codes and/or value sets for use in your data models.

# Building Your First Data Model
Let's get started with a data model using an example.  Say we've just [purchased a zoo](https://www.imdb.com/title/tt1389137/) and we want to define a data model for the animal clinic at our new zoo, which could include things like:

* Animals
* Staff
* Facilities
* Medicines
* Food
* etc.

## CIMPL Environment Setup
Before we develop any models, let's make sure our environment is set up.  If you haven't already done so, follow the [CIMPL Installation](https://github.com/standardhealth/shr-cli/wiki/CIMPL-Installation) instructions and check that everything is working using the [Hello World](https://github.com/standardhealth/shr-cli/wiki/Hello-World) test.

Create a directory that shares the same parent folder as your `shr-cli` installation and name it `zoo`:
```
mkdir ~/cimpl/zoo
```
Your directory structure should look like this:
```
cimpl
|- shr-cli
|- zoo
```
We'll need to copy some files from the SHR specification into the zoo directory to ensure that the `shr-cli` tool runs properly.  Download the following files and place them in the `zoo` directory:

* [base_map.txt](https://raw.githubusercontent.com/standardhealth/cimpl-tutorial/master/base_map.txt)
* [base.txt](https://raw.githubusercontent.com/standardhealth/cimpl-tutorial/master/base.txt)
* [core_map.txt](https://raw.githubusercontent.com/standardhealth/cimpl-tutorial/master/core_map.txt)
* [core_vs.txt](https://raw.githubusercontent.com/standardhealth/cimpl-tutorial/master/core_vs.txt)
* [core.txt](https://raw.githubusercontent.com/standardhealth/cimpl-tutorial/master/core.txt)

<!-- TODO: this is not great.  These files should be incorporated into the `shr-cli` tool itself. -->

Your directory structure should now look like this:
```
cimpl
|- shr-cli
|- zoo
  |- base_map.txt
  |- base.txt
  |- core_map.txt
  |- core_vs.txt
  |- core.txt
```

## Data Elements Header
The core part of any data model is the definition of its data elements.  Many modeling tools use diagrams or database tables to define their data elements, but CIMPL is defined using plain text.

Create a file called `zoo_animal.txt` and open it.

To give our `zoo_animal.txt` file some context, we need to include some header information. Let's start with a line that describes what the file is:
```
Grammar: DataElement 5.0
```
Every CIMPL file starts by defining the `Grammar`.  The common CIMPL files are `DataElement`, `Map`, and `ValueSet`.  We'll get into `Map` and `ValueSet` later.  This is a `DataElement` file that is using CIMPL version `5.0`.

Next, add your file to a `namespace`:
```
Grammar: DataElement 5.0
Namespace: zoo.animal
```
The `namespace` allows you to define a scope for value sets, data elements, and data mappings. By convention, the `namespace` is period-delimited, with the project (`zoo`) preceding the specific data model that we are defining (`animal`).  The filename (`zoo_animal.txt`) doesn't have to match, but it's good practice to keep the two consistent. Filenames should use underscores in place of periods to avoid confusion with file types.

While we're at it, let's give this file a description:
```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
```
Descriptions can include any characters (except quotation marks) and be as long as you need, but it is good practice to keep them short and simple.

By default, we'll need to include some basic modeling building blocks with the `uses` keyword:
```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base
```
Once our zoo gets more complicated, we will need to add more to the `uses` statement, but for now we're just including the basic building blocks that all CIMPL models would not want to redefine (e.g., `Entry`, `Quantity`, `CodeSystem`, etc.).

Great!  That's it for the `DataElement` file header.

## Create an Animal
With the header information out of the way, we can start defining our first data element:
```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base

EntryElement: Animal
```
Since we plan to map this element to some output (we'll get into mapping later), we're using the term `EntryElement` to describe our `Animal`.  Eventually, this element will have data _entered_ directly into it, thus `EntryElement` and not just `Element`.

Let's add some attributes to our animal:
```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base

EntryElement: Animal
1..1 DateOfBirth
0..* Name
1..1 DeceasedStatus
0..1 AnimalType
```
We've added four separate attributes to this `Animal`: `DateOfBirth`, `Name`,  `AnimalType`, and `DeceasedStatus`.  Users familiar with traditional [cardinality notation for data modeling](https://en.wikipedia.org/wiki/Cardinality_(data_modeling)#Application_program_modeling_approaches) will notice that the `DateOfBirth` and `DeceasedStatus` fields are required and can only appear once, while the `AnimalType` is optional and our `Animal` can have an arbitrary number of `Name`s.

CIMPL doesn't know how to interpret these attributes yet, so let's define them as child `Element`s, starting with `DateOfBirth`:
```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base

EntryElement: Animal
1..1 DateOfBirth
0..* Name
1..1 DeceasedStatus
0..1 AnimalType

  Element: DateOfBirth
  Description: "A date of birth"
  Value: date
```
We've defined `DateOfBirth` as an `Element`, added a short `Description`, and assigned a `Value` of `date`, which is one of the [primitive data types that CIMPL accepts](https://github.com/standardhealth/shr-cli/wiki/CIMPL-Documentation#primitives).  CIMPL is whitespace-agnostic, so the indentation is not necessary, but it is good practice to indent child elements underneath their parents.  `Element`s can get more complicated but let's leave it at that.

Now that you know the basic structure of an `Element`, let's define `Name` and `DeceasedStatus` (as a `string` and `boolean`, respectively):
```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base

EntryElement: Animal
1..1 DateOfBirth
0..* Name
1..1 DeceasedStatus
0..1 AnimalType

  Element: DateOfBirth
  Description: "A date of birth"
  Value: date

  Element: Name
  Description: "A name used to refer to an animal"
  Value: string

  Element: DeceasedStatus
  Description: "An indication that the animal is no longer living, given by a boolean value which, when true, indicates the animal is deceased."
  Value: boolean
```
Since we want `AnimalType` to be restricted to a certain set of values, we need to define a custom `ValueSet` file.  Create a file called `zoo_animal_vs.txt` and open it.

## Value Sets
Just like we did for the `DataElement` file, we need to define some header information for our `ValueSet` file:
```
Grammar: ValueSet 5.0
Namespace: zoo.animal
```
That's it!  Since the `ValueSet` file is in the same `namespace` as our `DataElement` file, we don't need to add a `Description` or set up inheritance with a `Uses` statement.

Start defining our `AnimalType` value set by giving it a name:
```
Grammar: ValueSet 5.0
Namespace: zoo.animal

ValueSet: AnimalTypeVS // These are the major classes of animal that we want to model.
```
By convention, all value sets should be Pascal case and end with `VS`.  You may notice that we've added a comment to the value set after the double-slash (`//`).  CIMPL uses [JavaScript comment syntax](https://www.w3schools.com/js/js_comments.asp) and you can add comments in your files to give extra context to certain items.  They aren't required, but could be helpful later on when you are reviewing the file.

With the value set named, let's go ahead and add some values:
```
Grammar: ValueSet 5.0
Namespace: zoo.animal

ValueSet: AnimalTypeVS // These are the major classes of animal that we want to model.
#invertebrate   "Invertebrates are characterized by their lack of backbones and internal skeletons"
#fish           "Fish breathe using gills, and are equipped with 'lateral lines' that detect water currents and even electricity"
#amphibian      "Amphibians are characterized by their semi-aquatic lifestyles (they have to stay near bodies of water, both to maintain the moisture of their skin and to lay their eggs)"
#reptile        "Reptiles are characterized by their cold-blooded metabolisms, their scaly skin, and their leathery eggs, which, unlike amphibians, they can lay some distance away from bodies of water"
#bird           "Birds are characterized by their coats of feathers, their warm-blooded metabolisms, their memorable songs (at least in certain species), and their ability to adapt to a wide range of habitats"
#mammal         "Mammals are characterized by their hair or fur (which all species possess during some stage of their life cycles), the milk with which they suckle their young, and their warm-blooded metabolisms"
```
Each value is prefaced by a hash symbol (`#`) and followed by a description enclosed in quotation marks. By convention, each value should use lowercase [snake_case formatting](https://en.wikipedia.org/wiki/Snake_case).

We've defined our custom `ValueSet` but we still need to add it to our `DataElement` file, so open `zoo_animal.txt` and add the full definition for our `AnimalType`:
```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base

EntryElement: Animal
1..1 DateOfBirth
0..* Name
1..1 DeceasedStatus
0..1 AnimalType

  Element: DateOfBirth
  Description: "A date of birth"
  Value: date

  Element: Name
  Description: "A name used to refer to an animal"
  Value: string

  Element: DeceasedStatus
  Description: "An indication that the animal is no longer living, given by a boolean value which, when true, indicates the animal is deceased."
  Value: boolean

  Element: AnimalType
  Description: "Classification code of an animal"
  Value: code from AnimalTypeVS
```
Now, `AnimalType` will be restricted to the values defined in our custom value set, thanks to the `code from [ValueSet]` syntax.

That's it!  We have defined our `DataElement` file and a custom `ValueSet` for `AnimalType`, so we can map our model to its output.  Create a file called `zoo_animal_map.txt` and open it.

## Maps
Our data model is fully defined, but we still need to map it to an output format.  Currently, CIMPL only supports output to various versions of the [FHIR specification](http://hl7.org/fhir/patient.html).

Let's create the file header for our map:
```
Grammar: Map 5.0
Namespace: zoo.animal
Target: FHIR_STU_3
```
In this case, we plan to map our data model to FHIR resources, so we are using `FHIR_STU_3` as our target model.
<!-- TODO: link to available target options? -->

Next, we map our `EntryElement` to its FHIR counterpart:
```
Grammar: Map 5.0
Namespace: zoo.animal
Target: FHIR_STU_3

Animal maps to Patient:
```
For our clinic, it makes sense to map animals as [`Patient` resources in FHIR](http://hl7.org/fhir/patient.html), since they will be the entities receiving care.  In CIMPL map files, the item to the left of `maps to` comes from your CIMPL data model and the item to the right is an element in your FHIR output.

Let's map our animal attributes. The `name` and `birthDate` fields exist in the [FHIR Patient resource](http://hl7.org/fhir/patient.html), so it looks like they will be simple mappings from our `Name` and `DateOfBirth` attributes:
```
Grammar: Map 5.0
Namespace: zoo.animal
Target: FHIR_STU_3

Animal maps to Patient:
  Name maps to name
  DateOfBirth maps to birthDate
```
The FHIR Patient resource represents the deceased status using the `deceased[x]` field.  The `[x]` indicates that this field provides a choice of data types; in this case `boolean` or `dateTime`.  Since our `DeceasedStatus` element is a `boolean` value, we can map it to `deceased[x]` and the CIMPL framework will properly identify it as mapping to the `boolean` choice:
```
Grammar: Map 5.0
Namespace: zoo.animal
Target: FHIR_STU_3

Animal maps to Patient:
  Name maps to name
  DateOfBirth maps to birthDate
  DeceasedStatus maps to deceased[x]
```
That just leaves `AnimalType`, which...doesn't seem to fit nicely into any of the fields on the FHIR Patient resource.  There are fields for `animal.species`, `animal.breed`, and `animal.genderStatus`, but there isn't anything that maps well to our custom animal class codes.  That's okay!  If we simply leave off a mapping for the `AnimalType`, it will be created as an extension on the FHIR Patient resource.  More information about [FHIR extensions](https://www.hl7.org/fhir/extensibility.html) is available in the FHIR documentation.

With all of our data elements, custom value sets, and mapping defined for our `Animal` element, it's time to run the `shr-cli` tool.

## Command-Line Arguments
Before we run the `shr-cli` tool, let's break down the syntax for calling it using the command line:
```
node {location_of_shr_cli_tool} {location_of_cimpl_files} {optional_arguments}
```
* `node`: The `shr-cli` tool is written in [node.js](https://nodejs.org/en/), so we start our statement by calling for that application.
* `{location_of_shr_cli_tool}`: Next, we need to tell `node` which code to run by giving it the directory where the `shr-cli` tool lives.  If you're in the `shr-cli` directory, you can use a single period (`.`) as a shorthand for that directory.
* `{location_of_cimpl_files}`: Now that node has the code it needs to run, we need to give it the directory where our CIMPL files reside.
* `{optional_arguments}`: There are a few other options that you can pass to the `shr-cli` tool.

Check the [README for all of the possible arguments](https://github.com/standardhealth/shr-cli#exporting-shr-to-json-and-fhir).

## Execution and Error Identification
Run the shr-cli tool using the following commands:
```
cd ~/cimpl/shr-cli
node . ../zoo -o ../zoo/out
```
Uh-oh.  You should see the following output:
```
[15:21:21.846Z]  INFO shr: Starting CLI Import/Export
[15:21:24.435Z] ERROR shr: Mismatched types. Cannot map zoo.animal.Name[Value: string] to [HumanName]. ERROR_CODE:13022 (module=shr-fhir-export, shrId=zoo.animal.Animal, target=Patient, mappingRule="Name maps to name")
[15:21:25.410Z]  INFO shr: Compiling Documentation for 3 namespaces... (module=shr-json-javadoc)
[15:21:25.492Z]  INFO shr: Building documentation pages for 31 elements... (module=shr-json-javadoc)
[15:21:25.669Z]  INFO shr: Finished CLI Import/Export
------------------------------------------------------------
Elapsed time: 3.829s
1 errors (shr-fhir-export)
0 warnings
------------------------------------------------------------
```
Huh - it looks like we have an error.  That's okay!  This is pretty common in developing CIMPL files and the output from the `shr-cli` tool can help us troubleshoot.  In, particular, this line tells us where we need to make changes:
```
[15:21:24.435Z] ERROR shr: Mismatched types. Cannot map zoo.animal.Name[Value: string] to [HumanName]. ERROR_CODE:13022 (module=shr-fhir-export, shrId=zoo.animal.Animal, target=Patient, mappingRule="Name maps to name")
```
We defined our `Name` element as a `string`, but the `name` attribute on the FHIR `Patient` record has the type of `HumanName`.  We have two options:
1. We can redefine the `Name` element as a `HumanName`, define the `HumanName` data type, and redo the mapping so the `Name` element is properly mapped to FHIR fields
2. We can remove the mapping from `Name` to the `name` attribute on the FHIR patient and allow the `Name` to be created as an extension

_Note: for more information about possible errors, see the [Error Message Documentation](https://github.com/standardhealth/shr-cli/wiki/Error-Message-Documentation)_

Let's take the easier path of removing the mapping for `Name`.  Change your `zoo_animal_map.txt` file to:
```
Grammar: Map 5.0
Namespace: zoo.animal
Target: FHIR_STU_3

Animal maps to Patient:
  DateOfBirth maps to birthDate
  DeceasedStatus maps to deceased[x]
```
With the `Name` field removed from the mapping, the `shr-cli` output should run without errors:
```
computer:shr-cli username$ node . ../zoo -o ../zoo/out
[18:06:56.010Z]  INFO shr: Starting CLI Import/Export
[18:07:00.775Z]  INFO shr: Compiling Documentation for 3 namespaces... (module=shr-json-javadoc)
[18:07:00.881Z]  INFO shr: Building documentation pages for 31 elements... (module=shr-json-javadoc)
[18:07:01.135Z]  INFO shr: Finished CLI Import/Export
------------------------------------------------------------
Elapsed time: 5.136s
0 errors
0 warnings
------------------------------------------------------------
```
Excellent.  You should find that the `out` directory in your `zoo` folder now contains quite a bit of material.  We'll explore its contents in the next section.

## CIMPL Outputs
By default, `shr-cli` takes CIMPL models and exports files into the following directories (under _out/_):
* **cimcore**: The serialized, machine-readable CIMCORE (Clinical Information Modeling Computable Representation) files representing the CIMPL input. For more information, see the [CIMCORE Documentation](https://github.com/standardhealth/shr-cli/wiki/CIMCORE-Documentation).
* **es6**: ES6 JavaScript classes that support interacting with instances of the SHR elements within JS code. Typically, a JavaScript application will load SHR data from a JSON or FHIR format and then represent the entries in memory using these ES6 classes.
* **fhir**: HL7 FHIR ([Fast Healthcare Interoperability Resources](http://hl7.org/fhir/)) is an open, international standard for electronic exchange of healthcare information that is governed by HL7.  The files in this folder define FHIR profiles, extensions, logical models, value sets, code systems, and the source needed to generate an HL7 FHIR Implementation Guide.
* **json**: A legacy serialization format used to generate parts of [standardhealthrecord.org](http://standardhealthrecord.org). This format has been deprecated in favor of [CIMCORE](https://github.com/standardhealth/shr-cli/wiki/CIMCORE-Documentation).
* **json-schema**: JSON schemas used to define and validate the structure of instance data.
* **modeldoc**: [Javadoc](https://en.wikipedia.org/wiki/Javadoc)-style HTML documentation representing the CIMPL data elements.  Casual users may find this documentation easier to review than the CIMPL text input files.

Each of the generated files in these directories would take a considerable amount of time to create by hand, but CIMPL's concise syntax combined with the `shr-cli` generators allow for quick initial and subsequent drafts.

## FHIR Output

For this tutorial, we will focus on the FHIR output.  Open the following file:
```
zoo/out/fhir/profiles/zoo-animal-Animal.json
```
This is a JSON representation of a FHIR profile.  It's a bit difficult to read in this form, but we can make it slightly more legible by creating a FHIR Implementation Guide.

In the development of FHIR standards, an Implementation Guide is the document that most people review when writing software that includes a data standard.  It is a human-readable webpage that can contain multiple profiles, extensions, value sets, code sets, etc. and gives context to how those items should be used in a software application.

Let's add some project-specific context to our Implementation Guide.  Open the following file:
```
zoo/config.json
```
It should look something like this:
```
{
    "projectName": "Example Project",
    "projectShorthand": "EXAMPLE",
    "projectURL": "http://example.com",
    "fhirURL": "http://example.com/fhir",
    "implementationGuide": {
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
Let's edit this file to make it more specific to our project:
```
{
    "projectName": "We bought a zoo",
    "projectShorthand": "ZOO",
    "projectURL": "http://thatzoowebought.com",
    "fhirURL": "http://thatzoowebought.com/fhir",
    "implementationGuide": {
        "npmName": "zoo",
        "version": "0.0.1",
        "indexContent": "zoo.html"
    },
    "publisher": "Zoo Owner",
    "contact": [
      {
        "telecom": [
          {
            "system": "url",
            "value": "http://thatzoowebought.com"
          }
        ]
      }
    ],
    "provenanceInfo": {
      "leadAuthor": {
          "name":"Zoe the Zoo Owner",
          "organization": "Zoo Owner",
          "email": "zoe@thatzoowebought.com"
      },
      "license": "Creative Commons CC-BY <https://creativecommons.org/licenses/by/3.0/>",
      "copyright": "Copyright (c) Zoo Owner <http://thatzoowebought.com>"
    }
}
```
That's better.  Note that we also added two new fields to the `implementationGuide` object: `npmName` and `version`.  These are new requirements from the FHIR IG Publisher and haven't yet been integrated into the default config template -- but we need them to successfully run the IG Publisher.

You'll also notice that the `implementationGuide` object's `indexContent` value references a file named `zoo.html`.  This file contains the HTML content that will always show up on the homepage of your generated Implementation Guide.  If you have a mission statement, a project timeline, or other general context for your project, this is a good place to include them.  Note that this file will be embedded within the `<body>` tag of the IG home page, so you should not include `<html>`,`<head>`, or `<body>` tags yourself.  Let's create the `zoo/zoo.html` file with the following contents:
```
<h1>We Bought a Zoo!</h1>
<p>In 2018, after coming into an absurd amount of money, we decided to buy a zoo!  Now, we need to figure out how to care for all the animals.  This Implementation Guide describes the data models for our new zoo clinic IT system.</p>
```
After running the `shr-cli` tool again, we can generate the IG using HL7's [FHIR IG Publishing tool](http://wiki.hl7.org/index.php?title=FHIR_IG_Publishing_tool).  Run the following commands:
```
cd ~/cimpl/shr-cli
cp -r ../zoo/out .
yarn run ig:publish
```

_Note: if this fails, see the `shr-cli` README for more [detailed instructions on running the FHIR IG generator](https://github.com/standardhealth/shr-cli#creating-the-fhir-implementation-guide)._

Open the following file:
```
~/cimpl/shr-cli/out/fhir/guide/output/index.html
```
It should look something like this:

![IG Home](https://github.com/standardhealth/cimpl-tutorial/blob/master/images/ig_home.png?raw=true)

Let's take a look at the items that CIMPL generated.  Click the 'Profiles' link and you should see this:

![IG Profiles](https://github.com/standardhealth/cimpl-tutorial/blob/master/images/ig_profiles.png?raw=true)

Since we only have one profile, that screen isn't terribly helpful.  Click on the 'Animal' link to see the profile we just generated:

![IG Animal Profile](https://github.com/standardhealth/cimpl-tutorial/blob/master/images/ig_animal_profile.png?raw=true)

That's much more helpful.  You can see a few of the things we defined in `config.json`, the `Name` and `AnimalType` extensions defined in our mapping, and other bindings and constraints created by the `shr-cli` tool.  For a more compact view of the profiled resource, click on the `Differential Table` tab:

![IG Animal Profile Differential](https://github.com/standardhealth/cimpl-tutorial/blob/master/images/ig_animal_profile_differential.png?raw=true)

This looks a lot more like the `DataElement` file.  All four of our `Animal` attributes are present and the `AnimalType` is explicitly bound to our `AnimalTypeVS` value set.  You may notice that the `Type` for `birthDate` is not listed here.  That is because it is already explicitly defined on the FHIR Patient resource as a `date`.  This 'Differential Table' view only shows the differences between the generated profile and the FHIR resources on which they are based.

# Improving your First Data Model

Now that we've built a very simple model for our animals, we can expand it using other aspects of the CIMPL language.

## Inheritance and Constraints

Let's start by making a few more animals in our `zoo_animal.txt` file:

```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base

EntryElement: Animal
1..1 DateOfBirth
0..* Name
1..1 DeceasedStatus
0..1 AnimalType

  Element: DateOfBirth
  Description: "A date of birth"
  Value: date

  Element: Name
  Description: "A name used to refer to an animal"
  Value: string

  Element: DeceasedStatus
  Description: "An indication that the animal is no longer living, given by a boolean value which, when true, indicates the animal is deceased."
  Value: boolean

  Element: AnimalType
  Description: "Classification code of an animal"
  Value: code from AnimalTypeVS

EntryElement: Lion
Based on: Animal
Description: "The king of the jungle."
AnimalType is #mammal

EntryElement: Shark
Based on: Animal
Description: "The king of the ocean."
AnimalType is #fish

EntryElement: Eagle
Based on: Animal
Description: "The king of the sky."
AnimalType is #bird
```

Looks like this zoo has quite the display! We've added three animals to our model (`Lion`, `Shark`, and `Eagle`) to show additional CIMPL functionality:

### Element inheritance using `Based on`
Since all three are animals, we can go ahead and make all three of them `Based on` on the `Animal` class we've already defined. This lets `Lion`, `Shark`, and `Eagle` inherit all of the properties of `Animal` (i.e., `DateOfBirth`, `Name`, etc.), so we don't have to redefine them on the more specific animals. This includes mapping inheritance, so if we don't add new fields, we don't have to create new mappings!

<!-- Note: I'm not sure it's helpful to reiterate this - we want to get into the more advanced stuff and can mention elsewhere that users can add a `Description` to basically anything

### `Description` for an `EntryElement`
We've added a `Description` to each of the animals. You can see from the `Animal` element that this description is optional but it can be helpful to give additional detail. -->

### Constraints
We've constrained each of these animals' `AnimalType` using `{element} is {value}` to make the model more specific to our creatures. In our `AnimalType` definition, we established that its code must come from the custom `AnimalTypeVS` value set. From that value set, the `#mammal`, `#fish`, and `#bird` codes are relevant for those animals.

_Constraints (e.g., `is`) are an important capability in CIMPL that lead to more accurate, specific models. We will keep coming back to this concept, but it will be helpful to refer to the [CIMPL documentation](cimpl-documentation#field-constraints)._

## Managing Multiple Namespaces

Now that our zoo is full of animals, where do we put them? We haven't defined any enclosures yet, and it wouldn't exactly make sense to put them in the `zoo.animal` namespace, so let's make a new `zoo.facility` namespace.

Similarly to how we made the `zoo.animal` namespace, let's go ahead and make a new file called `zoo_facility.txt` and add header information:
```
Grammar: DataElement 5.0
Namespace: zoo.facility
Description: "Enclosure environments of the zoo"
Uses: shr.core, shr.base
```

Now let's add an `Enclosure` and some specific types of enclosures:
```
Grammar: DataElement 5.0
Namespace: zoo.facility
Description: "Enclosure environments of the zoo"
Uses: shr.core, shr.base

CodeSystem: MTH = http://ncimeta.nci.nih.gov

EntryElement: Enclosure
Concept: MTH#C2986715
Value: code from EnclosureVS
1..1 Dimensions
Dimensions.DimensionComponent
  includes 1..1 Length
  includes 1..1 Width

EntryElement: Jungle
Based on: Enclosure
Value is #tropical

EntryElement: Aquarium
Based on: Enclosure
Value is #aquatic

EntryElement: Aviary
Based on: Enclosure
Value is #sky
```

In addition to this new namespace (`zoo.facility`), we've introduced a few new CIMPL concepts:

### `Concept` Codes
`Concept` codes are numerical codes organized in hierarchies that identify clinical terms. They can be useful for correlating your element with a specific clinical concept but do not impact the structure of your elements or add any specific constraints about how they are used. For instance, the `Enclosure` can be described by the [NCI Metathesaurus](https://ncimeta.nci.nih.gov/ncimbrowser/) code [`MTH#C2986715`](https://ncimeta.nci.nih.gov/ncimbrowser/ConceptReport.jsp?dictionary=NCI%20Metathesaurus&code=C2986715). In the FHIR output, this outputs as `StructureDefinition.keyword`.
<!-- TODO: It's nice that the concept code goes in `StructureDefinition.keyword`, but what consequence does the `StructureDefinition.keyword` have for the FHIR IG or other IRL systems? -->

### `CodeSystem` Declarations
In order to use that concept code, we must first define the [`CodeSystem`](cimpl-documentation#codesystems) in the header of our file. This allows us to reference the NCI Metathesaurus with the `MTH` abbrevation. All abbrevations must be uppercase.

### The `includes` Constraint
The `Dimensions` element `includes` `DimensionComponent` values for both a `Length` and a `Width`. We don't have to define those elements here, because they are already defined in `shr.core`. This `includes` structure is a constraint (specifically the [`includes type`](cimpl-documentation#includes-type) constraint) that allows you to specify members of a list.  In this case, since both `Length` and `Width` are `Based on DimensionComponent` and the `Dimensions` element can include multiple `DimensionComponent` elements, we are specifying that `Dimensions` _must_ include both `Length` and `Width` (though it _may_ also contain other `DimensionComponent` elements).

### Inherited `Value`s
In `Jungle`, `Aquarium`, and `Aviary`, we introduce the concept of constraining an inherited `Value`. While the `Enclosure` element's `Value` is constrained to a value set (using `code from EnclosureVS`), we can further constrain the `Value` for `Jungle`, `Aquarium`, and `Aviary` to a specific code using the `is` keyword (e.g., `Value is #sky`).

We've used `AnimalEnclosureVS` but it isn't defined yet. Let's create a new ValueSet file named `shr_facility_vs.txt` and add that value set:
```
Grammar: ValueSet 5.0
Namespace: zoo.facility

ValueSet: EnclosureVS // These are the environments in which our animals live.
#tropical   "An environment with average temperature of above 18 degrees Celsius and considerable precipitation during at least part of the year"
#aquatic    "An ecosystem in a body of water"
#sky        "An ecosystem in the sky"
```

Now, going back to our `zoo_animal.txt` file, let's add this information to our animals:

```
Grammar: DataElement 5.0
Namespace: zoo.animal
Description: "Animal residents of the zoo"
Uses: shr.core, shr.base, zoo.facility

EntryElement: Animal
1..1 DateOfBirth
0..* Name
1..1 DeceasedStatus
0..1 AnimalType
1..* Enclosure

  Element: DateOfBirth
  Description: "A date of birth"
  Value: date

  Element: Name
  Description: "A name used to refer to an animal"
  Value: string

  Element: DeceasedStatus
  Description: "An indication that the animal is no longer living, given by a boolean value which, when true, indicates the animal is deceased."
  Value: boolean

  Element: AnimalType
  Description: "Classification code of an animal"
  Value: code from AnimalTypeVS

EntryElement: Lion
Based on: Animal
Description: "The king of the jungle."
AnimalType is #mammal
Enclosure is type Jungle

EntryElement: Shark
Based on: Animal
Description: "The king of the ocean."
AnimalType is #fish
Enclosure is type Aquarium

EntryElement: Eagle
Based on: Animal
Description: "The king of the sky."
AnimalType is #bird
Enclosure is type Aviary
```

We've made two very important edits:
1. We added the `zoo.facility` namespace to the `Uses` statement in the file header, allowing our `zoo.animal` files to reference any item in the `zoo.facility` files.
2. We've added `Enclosure` as a child element of `Animal` and further constrained the types of enclosure allowed for each of our animals (`Lion`, `Shark`, and `Eagle`) using the [`is type`](cimpl_documentation#type).

<!-- TODO: Narrative Prose -->

<!-- #### Additional Concepts -->

<!-- * TODO: Concept (How are concept codes defined/provisioned? Does this have any consequence for FHIR profiling? How is the concept reflected in the output?) -->
<!-- * TODO: CodeSystem declarations (why can I declare CodeSystems in this file?  why would/should I declare this in the Value Set file instead?) -->
<!-- * TODO: Other Data Models -->

<!-- # Advanced Topics
* TODO: config.json and IG declarations
* TODO: Advanced slicing
* TODO: Mapping to a base FHIR resource vs. mapping to a profiled FHIR resource
* TODO: various shorthands/naming conventions
* TODO: Debugging
* TODO: Others? -->