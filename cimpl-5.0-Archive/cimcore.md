_(note: this documentation is in the process of being further expanded and is an actively updating document.)_

## General Overview

The SHR CIMCORE (**C**linical **I**nformation **M**odeling **Co**mputable **Re**presentation) is an easily computable health care modeling representation, designed to be comprehensive and intelligently structured, as well as **simple** enough to easily store model information for parsing and cross language interoperability.

The architecture of it has three clinical information modeling and profile components: `DataElements`, `ValueSets`, and Profile `Mappings`, with each separate definition generated into a discrete file. These files are organized into folders on the namespace level. Additionally, there should be project level and namespace level meta data files. The structures of these files are defined below.

***

## Table of Reference

#### [General Overview](#general-overview)

#### [Data Elements](#data-element)
* [name](#name-string) (_string_)
* [namespace](#namespace-string) (_string_)
* [fqn](#fqn-string) (_string_)
* [description](#description-string) (_string_)
* [isEntry](#isentry-boolean) (_boolean_)
* [isAbstract](#isabstract-boolean) (_boolean_)
* [basedOn](#basedon-string) (_array_)
* [hierarchy](#hierarchy-array) (_array_)
* [concepts](#concepts-array) (_array_)
* [value](#value-dictionary) (_dictionary_)
  * [valueType](#valuetype-string) (_string_)
  * [card](#card-dictionary) (_dictionary_)
  	* [min](#min-integer) (_integer_)
	* [max](#max-integer) (_integer_)
	* [history](#history-array) (_array_)
  * [constraints](#constraints-dictionary) (_dictionary_)
    * [valueSet](#valueset-dictionary) (_dictionary_)
    * [type](#type-dictionary) (_dictionary_)
    * [includesType](#includestype-array) (_array_)
    * [includesCode](#includescode-array) (_array_)
    * [fixedValue](#fixedvalue-dictionary) (_dictionary_)
    * [subpaths](#subpaths-dictionary) (_dictionary_)
  * [inheritance](#inheritance-array) (_dictionary_)
    * [status](#status-string) (_string_)
    * [from](#from-string) (_string_)
* [fields](#fields-array) (_array_)

#### [Value Sets](#value-set)
* [name](#name-string-1) (_string_)
* [namespace](#namespace-string-1) (_string_)
* [fqn](#fqn-string-1) (_string_)
* [description](#description-string-1) (_string_)
* [concepts](#concepts-1) (_array_)
* [url](#url-url) (_url_)
* [values](#values-array) (_array_)
* [rules](#rules-dictionary) (_dictionary_)
  * [includesDescendants](#includesDescendants-array) (_array_)
  * [includesFromCode](#includesFromCode-array) (_array_)
  * [includesFromCodeSystem](#includesFromCodeSystem-array) (_array_)
  * [excludesDescendants](#excludesDescendants-array) (_array_)

#### [Mappings](#mapping)
* [name](#name-string-2) (_string_)
* [namespace](#namespace-string-2) (_string_)
* [fqn](#fqn-string-2) (_string_)
* [targetSpec](#targetSpec-string) (_string_)
* [targetItem](#targetItem-string) (_string_)
* [mappings](#mappings-dictionary) (_dictionary_)
  * [fieldMapping](#fieldMapping-array) (_array_)
  * [cardMapping](#cardMapping-array) (_array_)
  * [fixedValueMapping](#fixedValueMapping-array) (_array_)

***

## Data Element

#### `name` (_string_)

The name of the element.

_Example:_
```
  "name": "DiseaseProgression"
```

#### `namespace` (_string_)

The namespace to which the element belongs. It is good practice to prefix the namespace with an organization identifier, e.g. `shr.oncology`.

_Example:_
```
  "namespace": "shr.condition"
```

#### `fqn` (_string_)

The full identifier of an element, as a combination of its name and namespace, e.g. `shr.oncology.TNMStage`. 

_Example:_
```
  "fqn": "shr.condition.DiseaseProgression"
```

#### `description` (_string_)

The textual description of the element.

_Example:_
```
  "description": "A finding related to the current trend of a disease. This concept is most often used for chronic and incurable diseases where the status and trendline of the disease is an important determinant of therapy and prognosis. The specific disorder being evaluated must be cited in the AssessmentFocus as a reference to a Condition."
```

#### `isEntry` (_boolean_)

The status of the element as an health record entry.

_Example:_
```
  "isEntry": true
```

#### `isAbstract` (_boolean_)

The status of the element as an abstract element.

_Example:_
```
  "isAbstract": false,
```

#### `concepts` (_array_)

An array of the element's corresponding concept codes. Each concept in the array is represented by a dictionary object with the following fields:

* ###### `system` (_uri_)

  The uri of the concept code's code system. 

* ###### `code` (_string_)

  The concept code.

* ###### `display` (_string_)

  An optional external display name for this concept.

_Example:_

```
  "concepts": [
    {
      "system": "http://ncimeta.nci.nih.gov",
      "code": "C0421176",
      "display": "Progression"
    }
  ]
```


#### `value` (_dictionary_)

This is, in essence, what the element _is_. This generally takes the form of a `primitive`, a more advanced concept such as a `Coding` or a `CodeableConcept`, or an entirely separate element altogether (e.g. `Finding`, `ObservationComponent`, etc.)

The `value` field takes the form of a dictionary object with following properties:

* ##### `valueType` (_string_)

  An attribute for the type of the value. A value can be one of three separate types:
  1. `IdentifiableValue`: A value that can be properly identified as a compositional element or primitive.
  2. `ChoiceValue`: A list of individual `value` options.
  3. `RefValue`: A reference to a preexisting value, (as opposed to a new instance).

  A value of type `ChoiceValue` has a additional field `options` that is an array of `values`s.
  
  _Example:_
  ```
    "valueType": "IdentifiableValue"
  ```


* ##### `card` (_dictionary_)
  The final cardinality of element after all cardinality constraints have been applied, in essence, how _many_ instances of the value are to be expected.

  * ###### `min` (_integer_)

   The minimum number of instances to be expected. 

  * ###### `max` (_integer_)

   The maximum number of instances to be expected.

  * ###### `history` (_array_)
   The history of value's cardinality constraints through each of the element's base elements, tracing it from its original definition. (TBD: expand documentation on history dictionary objects in array)

  _Example:_
  ```
    "card": {
      "min": 1,
      "max": 1,
      "history": [
        {
          "source": "shr.finding.Observation",
          "min": 0,
          "max": 1
        }
      ]
    },
  ```

* ##### `constraints` (_dictionary_)
    The collection of constraints applied to the value and its subfields (see: [subpaths](#subpaths) for the subfield constraints). This includes both constraints defined on the element level, and constraints inherited from the element's base elements. 
    
    The structure of the `constraints` field divides the constraints into _types_ of constraint. Each type of constraint is either a single value in the form of a dictionary object, or it is an array of multiple dictionary objects, as defined below. 
    
    Additionally, each constraint contains `lastModifiedBy` information detailing where in the element's hierarchy the constraint was latest defined or modified, identified in the form of a fully qualified name (FQN).

    _Structure Example:_
    ```
      "constraints": {
        "valueSet": {
          ...
        }, 
        "type": {
          ...
          }, 
        "includesType": [
          ...
        ], 
        "includesCode": [
          ...
        ], 
        "fixedValue": {
          ...
        }, 
        "subpaths": {
          ...
        }, 
    ```
    _(see below for individual type examples)_

    1. ###### `valueSet` (_dictionary_)
        A constraint that fixes a value to be from a specific value set, with different levels of requirement. The value set constraint object has the following fields:
        * `uri`: The uri reference of the value set. 
        * `bindingStrength`: The level of requirement for this constraint. The binding strength can have the following strengths:
            1. `REQUIRED`
            2. `EXTENSIBLE`
            3. `PREFERRED`
            4. `EXAMPLE`
        * `lastModifiedBy`

        _Example:_
         ```
             "valueSet": {
               "uri": "http://standardhealthrecord.org/shr/condition/vs/#ProgressionVS",
               "bindingStrength": "REQUIRED",
               "lastModifiedBy": "shr.condition.DiseaseProgression"
             }
         ```
    2. ###### `type` (_dictionary_)
        A constraint that further defines a value to be a more specific _type_ of the value, for example `Animal is type Dog`. (This is akin to casting to a subclass). The type constraint object has the following fields:
        * `fqn`: The fully qualified name that identifies the specific target type, e.g. `shr.finding.QuestionAnswer`.
        * `onValue`: A boolean that determines whether the value object itself is being constrained, or if the _value object's value_ is receiving the constraint. (TBD: refine documentation that explains `onValue`)
        * `lastModifiedBy`

        _Example:_
         ```
             "type": {
               "fqn": "shr.condition.Condition",
               "onValue": false,
               "lastModifiedBy": "shr.condition.DiseaseProgression"
             }
         ```

    3. ###### `includesType` (_array_)
        For a value that is a list of objects, this constraint specifies the specific quantity of particular types of objects within the value. The `includesType` field is an array that contains constraint objects of the following form:
        * `fqn`: The fully qualified name that identifies the included type, e.g. `shr.finding.QuestionAnswer`.
        * `card`: The final quantity of the included type of object, in essence, how _many_ instances of the object are to be expected. Contains a range of minimum and maximum expected quantity. 
           * `min`
           * `max`
        * `onValue`: (TBD: this may be taken out of CIMCORE, documentation will be shortly updated to reflect decision)
        * `lastModifiedBy`

        _Example:_
         ```
           "includesType": [
             {
               "fqn": "shr.behavior.FrequencyOfUse",
               "card": {
                 "min": 0,
                 "max": 1
               },
               "onValue": false,
               "lastModifiedBy": "shr.behavior.SubstanceUse"
            },
         ```

    4.  ###### `includesCode` (_array_)
        A constraint that specifies specific codes to be included in the value. The `includesCode` field is an array that contains constraint objects of the following form:
        * `system`: See [system](######system)
        * `code`: See [code](######code)
        * `lastModifiedBy`

        _Example:_
         ```
            "includesCode": [
              {
                "system": "http://loinc.org",
                "code": "54511-1",
                "lastModifiedBy": "shr.behavior.BehavioralFinding"
              }
         ```

    5. ###### `fixedValue` (_dictionary_)
        A constraint that fixes the value to a specifc `code` or `boolean` value. The fixed value constraint object takes the following form: 
        * `type`: The type of the fixed value, either `code` or `boolean` (TBD: potentially expanding to other primitives, confirm for documentation) 
        * `value`: 
           In the case of `boolean`, `value` is simply a `true` or `false` boolean.
           In the case of `code`, the `value` field is an object of the following form:
           * `system`: See [system](######system)
           * `code`: See [code](######code)
        * `lastModifiedBy`

        _Example:_
         ```
            "fixedValue": {
              "type": "code",
              "value": {
                "system": "http://ncimeta.nci.nih.gov",
                "code": "C1963761"
              },
              "lastModifiedBy": "shr.adverse.NoAdverseEvent"
            }
         ```

    6. ###### `subpaths` (_dictionary_)
        To represent constraints on _subfields_ of values, e.g. (ObservationComponent.RouteIntoBody.CodeableConcept is MTH#C1522726), CIMCORE structures the constraints layer by layer with respective to each level of access. 
        Each layer organizes all of the level's constraints in a similar structure as [constraints](#constraints) under the key of the element being constrained. If there are additional constraints at a deeper level, under the same key is another `subpaths` field, with the exact same structure. 

        For example, for the `ObservationComponent` example stated above, it's subpath tree would look similar to this:
        ~~~~
          "constraints": {
            "includesType": [...],
            "subpaths": {
              "shr.medication.RouteIntoBody": {
                "card": {...},
                "subpaths": {
                  "shr.core.CodeableConcept": {
                    "fixedValue": {...}
                  }
                }
              },
              "shr.environment.ExposureMethod": {
                "card": {...},
                "subpaths": {
                  "shr.core.CodeableConcept": {
                    "fixedValue": {...}
                  }
                }
              }
              ...
            }
          }
        ~~~~

* ##### `inheritance` (_dictionary_)
    This gives the information regarding the inheritance of the `value`, with respect to from which parent base class the value was inherited _from_, and the modification _status_ with respect to the original definition.
    *  ##### `status` (_string_)
        Tells us the whether the value was cleanly _inherited_ or if it has been modified and _overridden_ since the original definition. The field has the option between two values:
        1. `inherited`
        2. `overridden` 
    *  ##### `from` (_string_)
        Identifies the original definer of the value by [fully qualified name](#fqn-string)

    _Example:_
    ```
       "inheritance": {
         "status": "overridden",
         "from": "shr.finding.Observation"
       }
    ```
#### `fields` (_array_)
Fields is an array of object properties on the element, both defined at the element level and inherited from parents. 
These are identical in form to [value](#value-dictionary).

_Example:_
```
   "fields": [
     {
       "fqn": "shr.finding.Finding",
       "valueType": "RefValue",
       ...
     }, 
     ...
   ]
```

***

## Value Set

#### `name` (_string_)

The name of the value set.

_Example:_
```
  "name": "BrcaReceptorStatusVS"
```

#### `namespace` (_string_)

The namespace to which the value set belongs. It is good practice to prefix the namespace with an organization identifier, e.g. `shr.oncology`.

_Example:_
```
  "namespace": "shr.oncology"
```

#### `fqn` (_string_)

The full identifier of a value set, as a combination of its name and namespace, e.g. `shr.oncology.BrcaReceptorStatusVS`. 

_Example:_
```
  "fqn": "shr.oncology.BrcaReceptorStatusVS"
```

#### `description` (_string_)

The textual description of the value set.

_Example:_
```
  "description": "Classification of receptor status in breast cancer."
```

#### `concepts` (_array_)

An array of the element's corresponding concept codes. Each concept in the array is represented by a dictionary object identical in form to [DataElement's concepts](#concepts-array)

_Example:_
```
  "concepts": [
    {
      "system": "http://ncimeta.nci.nih.gov",
      "code": "C0925205",
      "display": "Anatomical laterality"
    }
  ]
```

#### `url` (_url_)

The URI Location of the value set.

_Example:_
```
  "url": "http://standardhealthrecord.org/shr/oncology/vs/#BrcaReceptorStatusVS"
```

#### `values` (_array_)

The values defined to be in the value set. These can either be custom newly created codes, or codes from existing code systems.

* ##### `system` (_string_)
  The code system to which the value belongs.
* ##### `code` (_string_)
  The code itself (either an existing code from an existing code system, or a custom defined one)
* ##### `description` (_string_)
  The description of the code.

_Example:_
```
  "values": [
    {
      "system": "http://standardhealthrecord.org/shr/base/cs/#ProposedStatusCS",
      "code": "proposed",
      "description": "The proposal has been proposed, but not accepted or rejected."
    },
    {
      "system": "http://ncimeta.nci.nih.gov",
      "code": "C1519275",
      "description": "Severe Adverse Event. Daily activity is markedly reduced; some assistance usually required; medical intervention/therapy required, hospitalization or hospice care possible."
    }
  ]
```

#### `rules` (_url_)

The set of rules defining inclusion and exclusion relation with external code systems, codes, and descendants.

_Example:_
```
  "rules": {
    "includesDescendants": [
      ...
    ]
    "includesFromCode": [
      ...
    ]
    "includesFromCodeSystem": [
       ...
    ],

```

* ##### `includesDescendants` (_array_)
  
  The rule which defines which descendants are to be included in the value set:

  * ###### `system` (_string_)
  The code system to which the descendant belongs.
  * ###### `code` (_string_)
  The code of the descendant.
  * ###### `description` (_string_)
  The description of the descendent.

  _Example:_
  ```
    "includesDescendants": [
      {
        "system": "http://snomed.info/sct",
        "code": "64572001",
        "description": "disease"
      }
    ]
  ```

* ##### `includesFromCode` (_array_)

  The rule which defines values from which codes are to be included in the value set:

  * ###### `system` (_string_)
  The code system to which the code belongs.
  * ###### `code` (_string_)
  The code itself.
  * ###### `description` (_string_)
  The description of the code.

  _Example:_
  ```
    "includesFromCode": [
      {
        "system": "https://evs.nci.nih.gov/ftp1/CDISC/SDTM/",
        "code": "C99074",
        "description": "CDISC SDTM Directionality Terminology"
      }
    ]
  ```

* ##### `includesFromCodeSystem` (_array_)

  The rule which defines values from which code systems are to be included in the value set:

  * ###### `system` (_string_)
  The code system itself.
  _Example:_
  ```
    "includesFromCodeSystem": [
      {
        "system": "http://loinc.org"
      },
      {
        "system": "http://ncimeta.nci.nih.gov"
      },
      {
        "system": "http://snomed.info/sct"
      }
    ]
  ```

* ##### `excludesDescendants` (_array_)
  
  The rule which defines which descendants are to be included in the value set:

  * ###### `system` (_string_)
  The code system to which the descendant belongs.
  * ###### `code` (_string_)
  The code of the descendant.
  * ###### `description` (_string_)
  The description of the descendent.

  _Example:_
  ```
    "excludesDescendants": [
      {
        "system": "http://snomed.info/sct",
        "code": "66091009",
        "description": "Congenital Disease"
      }
    ]
  ```

***

## Mappings

#### `name` (_string_)

The name of the element being mapped.

_Example:_
```
  "name": "AdverseEvent"
```

#### `namespace` (_string_)

The namespace to which the element and mapping belongs. It is good practice to prefix the namespace with an organization identifier, e.g. `shr.oncology`.

_Example:_
```
  "namespace": "shr.adverse"
```

#### `fqn` (_string_)

The full identifier of the element being mapped, as a combination of its name and namespace, e.g. `shr.oncology.TNMStage`. 

_Example:_
```
  "fqn": "shr.adverse.AdverseEvent"
```

#### `targetSpec` (_string_)

The version of the specification resources to which the element is being mapped, (e.g. FHIR).

_Example:_
```
  "targetSpec": "FHIR_STU_3"
```

#### `targetItem` (_string_)

The specific item of the resource to which the element is being mapped.

_Example:_
```
  "targetItem": "http://hl7.org/fhir/us/core/StructureDefinition/us-core-condition"
```

#### `mappings` (_dictionary_)

The mappings rules for the element onto the resources. 

_Example:_
```
  "mappings": {
    "fieldMapping": [
       ...
    ],
    "cardMapping: [
       ...
    ],
    "fixedValueMapping": [
       ...
    ]
```

* ##### `fieldMapping` (_array_)
  A mapping rule to map a field to a specific resource.

  * ###### `target` (_string_)
    The target resource.
  * ###### `sourcePath` (_array_)
    The hierarchical path of the field being mapped (i.e. an expansion of the 'dot structure').

  _Example:_
  ```
    "fieldMapping": [
      {
        "target": "subject",
        "sourcePath": [
          "shr.base.Subject"
        ]
      }
    ]
  ```

* ##### `cardMapping` (_array_)
  A mapping rule to map a resource to a specific cardinality.

  * ###### `target` (_string_)
    The target resource.

  * ###### `cardinality` (_dictionary_)
    The fixed cardinality

  _Example:_
  ```
    "cardMapping": [
      {
        "target": "notDone",
        "cardinality": {
          "min": 0,
          "max": 0
        }
      }
    ]
  ```

* ##### `fixedValueMapping` (_array_)
  A mapping rule to map a resource to a specific fixed value.

  * ###### `target` (_string_)
    The target resource.

  * ###### `fixedValue` (_string_)
    The specific value or code to which the resource is mapped.

  _Example:_
  ```
    "fixedValueMapping": [
      {
        "target": "related.type",
        "fixedValue": "#has-member"
      }
    ]
  ```

***

## Project Meta Data