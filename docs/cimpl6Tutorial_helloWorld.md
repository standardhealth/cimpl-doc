# CIMPL 6.0 'Hello World' Tutorial
<br />
Let’s do a “Hello, World” using CIMPL v6!

_Note: Make sure you have completed the [CIMPL Installation instructions](cimplInstall.md) before proceeding, and that `shr-cli` (minimum version 6.2.0) is installed in `~/cimpl/shr-cli`._

Start by creating a folder structure for our new project.  Open up your favorite (Unix-based) command-line tool and enter the following:
```
cd ~/cimpl
mkdir hello_world
```

Now, create a file called `HelloWorld.txt` in the `~/cimpl/hello_world` folder and add the following data definitions:

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
Next, create a file called `HelloWorld_map.txt` that defines the mapping to FHIR. Since this doesn’t align with any particular resource, it will map to [`Basic`](https://www.hl7.org/fhir/basic.html). This file is even simpler:

```
Grammar:     Map 5.1
Namespace:   hello
Target:      FHIR_STU_3

HelloWorld maps to Basic:
```

We didn’t map the `SayHello` data element to an existing element inside this resource, so it will automatically appear as an extension.

Next, we need some simple HTML for the Implementation Guide pages, so create an `exampleIndexContent.html` file with the following content:
```
HELLO HELLO HELLO
```

Finally, we need a configuration file. Create an `ig-hello_world-config.json` file with these contents:

```
{
    "projectName": "Example Project",
    "projectShorthand": "EXAMPLE",
    "projectURL": "http://example.com",
    "fhirURL": "http://example.com/fhir",
    "implementationGuide": {
      "npmName": "hello_world",
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

Now, go back to your `shr-cli` installation and run the hello world:

```dos
cd ~/cimpl/shr-cli
node . -c ig-hello_world-config.json ../hello_world
```

When the program runs, it will output a warning message alerting you that mapping to Basic usually isn’t the best choice, but in this case, it is intentional. After the program runs, the generated profile (a FHIR `StructureDefinition`) will be found in `~/cimpl/shr-cli/out/fhir/profiles/`.  `StructureDefinition`s are verbose, and this one clocks in at several hundred lines.

A friendlier view is created when we create an implementation guide (IG). To do this, we use the [existing FHIR implementation guide (IG) publisher](http://wiki.hl7.org/index.php?title=IG_Publisher_Documentation).

## IG Generation

_Note: If your organization uses a proxy server, you may have to run the IG publishing process from outside your organization's firewall. For more information, see FHIR's GForge tracker as issue [#19815](https://gforge.hl7.org/gf/project/fhir/tracker/?action=TrackerItemEdit&tracker_item_id=19815).

To generate the IG, run:

```dos
cd ~/cimpl/shr-cli
yarn run ig:publish
```

The generated profile page will be located at `~/cimpl/shr-cli/out/fhir/guide/output/StructureDefinition-hello-HelloWorld.html`.  Open this page in your favorite web browser.

You should see a FHIR profile that looks like a traditional [FHIR Implementation Guide](https://www.hl7.org/fhir/implementationguide.html).  For a more comprehensive understanding of the CIMPL grammar and how to use it for your project, continue to the [Tutorial](https://github.com/standardhealth/shr-cli/wiki/Tutorial).

> **Note:** the IG is automatically branded as part of the Standard Health Record (SHR). We are working on “de-branding” the IG, so please ignore the SHR references for now.

