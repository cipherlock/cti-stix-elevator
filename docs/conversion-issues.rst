.. _conversion_issues:

Conversion Issues
=====================

This section discusses some techniques to facilitate the conversion of
STIX 1.x data to STIX 2.x. These techniques cover non-obvious issues
that might present an impediment to re-using STIX 1.x data.

Assumptions
-----------------

Timestamps, Identifiers and Object Creators
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In STIX 1.x most properties were optional. This includes properties that
correspond to required properties in STIX 2.x. In particular, all STIX
SDOs, SMOs and SROs in 2.x are required to have ``id`` and ``created``
properties. In STIX 2.1, all SCOs must have the ``id`` property.
These are often not specified in a STIX 1.x object, but can sometimes
be inferred from another STIX 1.x object in the same package.

Content in STIX 1.x was often hierarchical unlike content in STIX 2.x which is relatively flat, and
this can help to determine required properties. For instance, a
timestamp on a STIX 1.x package could be construed as the timestamp for
all objects it contains. Likewise, an object could assume that its
parent object's timestamp is also the timestamp of that object, unless
that object possessed its own timestamp. Of course, if no timestamp is
present for any of the objects, included the top level package, some
other timestamp outside of the content must be used. In most cases, this
would probably result in using the current timestamp when the conversion
is made.

Most top-level STIX 1.x objects contained an ``id`` (or an ``idref``), however when
converting STIX 1.x TTPs and Exploit Targets the id must be assigned to
the STIX 2.x object that results. For instance, a TTP might have contain
an attack pattern object, but the id was not a property of the attack
pattern, but the TTP.

In certain circumstances, no id is available or in the case of TTPs and
Exploit Targets, there may be more than one STIX 2.x object created. In
these cases, a new ``id`` must be used.

In STIX 1.x, all top-level objects had a ``Information_Source`` property to
hold information about, among other things, the object creator. However,
this property was optional. ``created_by_ref``, which is a common
property on all STIX 2.x SDOs, SMOs and SROs, is often optional. It should be noted
however, that the object creator can also be "inherited" from its parent
object, as with the timestamp. This fact can be useful to derive a more
robust STIX 2.x object.  Note that SCOs do not have a ``created_by_ref`` property.

Special Considerations for TTPs and Exploit Target Conversions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When converting a STIX 1.x TTP or Exploit Target certain properties
exist at the top-level, and not in the subsidiary object which will form
the basis of the STIX 2.x object. However, those properties must be used
when creating the subsidiary object. See section :ref:`attack_pattern`
for an example. The conversion of that
STIX 1.x TTP will yield a STIX 2.x Attack Pattern, whose ``name`` and
``created_by_ref`` are determined from the TTP itself, and not the
STIX 1.x Attack Pattern.

Minor Issues
~~~~~~~~~~~~~~~~~~~~

-  The ``condition`` property was optional in STIX 1.x Observables. If it was not
   specified for an Observable used for patterning, the condition
   used in the STIX 2.x pattern will be assumed to be "=".

-  The title property should be used for the ``name`` property, when necessary.

-  STIX 1.2 introduced versioning of objects. Currently, there is no
   guidance to converting STIX 1.2 versioning to STIX 2.x versioning. In most cases, a STIX 1.x relationship between object
   instances of the same type will be converted to a ``related-to`` relationship in STIX 2.x, which could be undesirable.

Optional vs. Required
---------------------------

Certain properties are required in STIX 2.x object that were optional in
STIX 1.x. This goes beyond the properties such as ids, created/modified timestamps. The most
frequently occurring example is the ``labels`` property in 2.0.
The elevator will use a default value - ``unknown``. Other SDOs have similarly named properties.

​Issues with Patterns
------------------------

Patterns in STIX 2.x have certain restrictions that didn't explicitly
appear in STIX 1.x. A pattern in STIX 2.x has explicit rules about if
the expression can refer to only one or many observed data instances.
Because STIX 1.x patterns did not have any of these restrictions, a
reasonable conversion of the pattern by the elevator might be illegal in STIX 2.x.

Additionally, the use of the NOT operator in STIX 2.x is restricted to
be used only with Comparison operators. Therefore, it is not possible to
express a pattern such as ``NOT (file.name == foo.bar" AND 'file.size ==
123)`` directly. To yield an equivalent pattern expression in STIX 2.x,
DeMorgan's Law would need to be used to reduce the scope of the NOT operator:
``(file.name != foo.bar" OR 'file.size != 123)``, but the elevator does not perform this functionality.

​Single vs. Multiple
-------------------------

Some properties in STIX 1.x allowed for multiple values, but the
corresponding property in STIX 2.x does not. In these cases, the first
value is used.

In certain situations, something specific to the properties can be
helpful in handling this issue. For instance, the first entry in the
STIX 1.x Threat Actors ``motivation`` property should be assumed to be the
``primary_motivation``. Any others should be listed in the
``secondary_motivations`` property.

Data Markings
--------------

The stix-elevator currently supports global markings and object-level markings. Through the use of hashing,
the elevator make the best effort to detect duplicate markings to prevent excessive object creation.
Also, the marking types supported by the elevator is limited to: Simple, Terms of Use, TLP and AIS.
AIS is a data marking used when submitting STIX content to DHS/CISA.

Missing Policy
--------------

Certain STIX 1.x properties cannot be converted to a STIX 2.x property defined in the STIX 2.x specification.  The elevator
provides a command line option to determine how to handle these STIX 1.x properties.

- ``add-to-description``:  Add the property name, property value pair to the description property to the ``description`` property.
- ``use-custom-properties``: STIX 2.x provides the ability to add *custom* properties to any STIX object.
  Missing properties can be included using this facility.  Note, that custom property names will have a prefix of ``x_<CUSTOM_PROPERTY_PREFIX>``,
  where ``CUSTOM_PROPERTY_PREFIX`` is provided as a command line option.  It defaults to ``elevator``.  This option has been deprecated, use ``use-extensions`` instead.
- ``use-extensions``: STIX 2.x provides the ability to "extend" any STIX object, using the extension-definition object.
- ``ignore``: The content is dropped, and does not appear in the STIX 2.x object

Note that the handling of missing properties is not complete - not every STIX 1.x property is handled.  The Mapping section
of this documentation lists what properties are handled for each SDO.

The disposition of all missing properties is presented in warning messages.

It is possible to create custom cyber observables in STIX 1.x through use of the CustomObjectType.  This can only be done within an
Observable Object, therefore the resulting STIX 2.1 object will be a SCO. For STIX 2.0, it will be similar to any other
cyber observable object.

``Incident`` and ``Infrastructure`` are object types in STIX 1.x, but it is not representable in STIX 2.0.
However, through the use of the options --incidents and --infrastructure, a custom object (or extensions) will be created.
Both of these object types exsit in STIX 2.1.

**An Example**

STIX 1.x

.. code-block:: xml

    <stix:Course_Of_Action id="example:coa-495c9b28-b5d8-41e3-b7bb-000c29789db9" xsi:type='coa:CourseOfActionType' version="1.2">
            <coa:Title>Block outbound traffic</coa:Title>
            <coa:Stage xsi:type="stixVocabs:COAStageVocab-1.0">Response</coa:Stage>
            <coa:Type xsi:type="stixVocabs:CourseOfActionTypeVocab-1.0">Perimeter Blocking</coa:Type>
            <coa:Objective>
                <coa:Description>Block communication between the PIVY agents and the C2 Server</coa:Description>
                <coa:Applicability_Confidence>
                    <stixCommon:Value xsi:type="stixVocabs:HighMediumLowVocab-1.0">High</stixCommon:Value>
                </coa:Applicability_Confidence>
            </coa:Objective>
            <coa:Impact>
                <stixCommon:Value xsi:type="stixVocabs:HighMediumLowVocab-1.0">Low</stixCommon:Value>
                <stixCommon:Description>This IP address is not used for legitimate hosting so there should be no operational impact.</stixCommon:Description>
            </coa:Impact>
        </stix:Course_Of_Action>

STIX 2.x using ``add-to-description``

.. code-block:: json

    {
            "created": "2015-07-31T11:24:39.090Z",
            "description": "\n\nSTAGE:\n\tResponse\n\nOBJECTIVE: Block outbound traffic\n\nOBJECTIVE CONFIDENCE: High\n\nIMPACT:Medium: Some description about the indicator.",
            "id": "course-of-action--3dbfccad-1fbb-4e9f-8307-f2d1a5c651cc",
            "labels": [
                "perimeter-blocking"
            ],
            "modified": "2015-07-31T11:24:39.090Z",
            "name": "Block outbound traffic",
            "spec_version": "2.1",
            "type": "course-of-action"
    }

STIX 2.x using ``use-extensions``

.. code-block:: json

    {
            "created": "2015-07-31T11:24:39.090Z",
            "extensions": {
                "extension-definition--a46b18de-0b41-4a95-9d2d-67a360f2d859": {
                    "extension_type": "property-extension",
                    "impact": {
                        "description": "Some description about the indicator.",
                        "value": "Medium"
                    },
                    "objective": "Block outbound traffic",
                    "objective_confidence": "High",
                    "stage": "Response"
                }
            },
            "id": "course-of-action--3dbfccad-1fbb-4e9f-8307-f2d1a5c651cc",
            "labels": [
                "perimeter-blocking"
            ],
            "modified": "2015-07-31T11:24:39.090Z",
            "name": "Block outbound traffic",
            "spec_version": "2.1",
            "type": "course-of-action"
    }

STIX 2.x using ``use-custom-properties``

.. code-block:: json

        {
            "created": "2015-07-31T11:24:39.090Z",
            "id": "course-of-action--3dbfccad-1fbb-4e9f-8307-f2d1a5c651cc",
            "labels": [
                "perimeter-blocking"
            ],
            "modified": "2015-07-31T11:24:39.090Z",
            "name": "Block outbound traffic",
            "spec_version": "2.1",
            "type": "course-of-action",
            "x_elevator_impact": {
                "description": "Some description about the indicator.",
                "value": "Medium"
            },
            "x_elevator_objective": "Block outbound traffic",
            "x_elevator_objective_confidence": "High",
            "x_elevator_stage": "Response"
        }

STIX 2.x using ``ignore``

.. code-block:: json

        {
            "created": "2015-07-31T11:24:39.090Z",
            "id": "course-of-action--3dbfccad-1fbb-4e9f-8307-f2d1a5c651cc",
            "labels": [
                "perimeter-blocking"
            ],
            "modified": "2015-07-31T11:24:39.090Z",
            "name": "Block outbound traffic",
            "type": "course-of-action"
        }

Extensions
----------

Extensions are based on the Extension Definition object.  The key in the ``extension`` property dictionary contains the id of the
Extension Definition object used to define the extension.  Extensions are explained in detail in section 7.3 of the STIX 2.1
specification document.

Currently, the schemas associated with the Extension Definition object do not exist.  However, the Extension Definition objects themselves
can be found in extension_definitions.py.  They will be more fully defined in a future release of the elevator.

Note that these extensions are not used by the predefined extension (e.g., Archive File), because those are fully defined within the
specification.
