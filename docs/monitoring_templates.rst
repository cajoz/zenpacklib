.. _monitoring-templates:

####################
Monitoring Templates
####################

Monitoring templates are containers for monitoring configuration. Specifically
datasources, thresholds and graphs. A monitoring template must be created to
perform periodic collection of data, associate thresholds with that data, or
define how that data should be graphed.


.. _location-and-binding:

********************
Location and Binding
********************

Two important concepts in understanding how monitoring templates are used are
location and binding. Location is the device class in which a monitoring
template is contained. Binding is the device class, device or component to
which a monitoring template is bound.

A monitoring template's location is important because it restricts to which
devices a the template may be bound. Assume you have a device named *widgeter1*
in the /Server/ACME/Widgeter device class that as a monitoring template named
*WidgeterHealth* bound. Zenoss will attempt to find a monitoring template
named *WidgeterHealth* in the following places in the following order.

1. On the *widgeter1* device.
2. In the /Server/ACME/Widgeter device class.
3. In the /Server/ACME device class.
4. In the /Server device class.
5. In the / device class.

The first template that matches by name will be used for the device. No
template will be bound if no matching template is found in any of these
locations.

It is because of this search up the hierarchy that allows the monitoring
template's location to be used to restrict to which devices it can be bound.
For example, by locating our monitoring template in the /Server/ACME device
class we make it available to be bound for all devices in /Server/ACME and
/Server/ACME/Widgeter, but we also make unavailable to be bound in other device
classes such as /Server or /Network/Cisco.

After deciding on the right location for a monitoring template should then
decide where it should be bound. Remember that to cause the template to be used
it must be bound. This is done by adding the template's name to the
*zDeviceTemplates* zProperty of a device class. See the following example that
shows how to bind the *WidgeterHealth* monitoring template to the
/Server/ACME/Widgeter device class.

.. code-block:: yaml

    name: ZenPacks.acme.Widgeter

    device_classes:
      /Server/ACME/Widgeter:
        zProperties:
          zDeviceTemplates:
            - WidgeterHealth
          
          templates:
            WidgeterHealth: {}

Note that zDeviceTemplates didn't have to be declared in the ZenPack's
zProperties field because it's a standard Zenoss zProperty.

.. note::

    Binding templates using zDeviceTemplates is only applicable for monitoring
    templates that should be bound to devices. See
    :doc:`classes_and_relationships` for information on how monitoring
    templates are bound to components.


.. _alternatives-to-yaml:

********************
Alternatives to YAML
********************

It's possible to create monitoring templates and add them to a ZenPack entirely
through the Zenoss web interface. If you don't have complex or many monitoring
templates to create and prefer to click through the web interface, you may
choose to create your monitoring templates this way instead of through the
`zenpack.yaml` file.

There are some advantages to defining monitoring templates in YAML.

* Using text-editor features such as search can be an easier way to make
  changes than clicking through the web interface.

* Having monitoring templates defined in the same document as the zProperties
  they use, and the device classes they're bound to can be easier to
  understand.

* Changes made to monitoring templates in YAML are much more diff-friendly than
  the same changes made through the web interface then exported to objects.xml.
  For those keeping ZenPack source in version control this can make changes
  clearer. For the same reason it can also be of benefit when multiple authors
  are working on the same ZenPack.

See :doc:`command_line` for information on the `dump_templates` option if
you're interested in exporting monitoring templates already created in the
web interface to YAML.


.. _adding-monitoring-templates:

***************************
Adding Monitoring Templates
***************************

To add a monitoring template to `zenpack.yaml` you must first add the device
class where it is to be located. Then within this device class entry you must
add a templates field. The following example shows a *WidgeterHealth*
monitoring template being added to the /Server/ACME/Widgeter device class. It
also shows that template being bound to the device class by setting
zDeviceTemplates.

.. code-block:: yaml

    name: ZenPacks.acme.Widgeter

    device_classes:
      /Server/ACME/Widgeter:
        zProperties:
          zDeviceTemplates:
            - WidgeterHealth
          
          templates:
            WidgeterHealth:
              description: ACME Widgeter monitoring.

              datasources:
                health:
                  type: COMMAND
                  parser: Nagios
                  commandTemplate: "echo OK|percent=100"

                  datapoints:
                    percent:
                      rrdtype: GAUGE
                      rrdmin: 0
                      rrdmax: 100

              thresholds:
                unhealthy:
                  dsnames: [health_percent]
                  eventClass: /Status
                  severity: Warning
                  minval: 90

              graphs:
                Health:
                  units: percent
                  miny: 0
                  maxy: 0

                  graphpoints:
                    Health:
                      dpName: health_percent
                      format: "%7.2lf%%"

Many different entry types are shown in the above example. See the references
below for more information on each.


.. _monitoring-template-reference:

*****************************
Monitoring Template Reference
*****************************

The following fields are valid for a monitoring template entry.

name
  :Description: Name (e.g. WidgeterHealth). Must be a valid Zenoss object ID.
  :Required: Yes
  :Type: string
  :Default Value: *(implied from key in templates map)*

description
  :Description: Description of the templates purpose and function.
  :Required: No
  :Type: string
  :Default Value: "" *(empty string)*

targetPythonClass
  :Description: Python module name (e.g. ZenPacks.acme.Widgeter.Widgeter) to which this template is intended to be bound.
  :Required: No
  :Type: string
  :Default Value: "" (empty string is equivalent to Products.ZenModel.Device)

datasources
  :Description: Datasources to add to the template.
  :Required: No
  :Type: map<name, :ref:`Datasource <datasource-reference>`>
  :Default Value: {} *(empty map)*

thresholds
  :Description: Thresholds to add to the template.
  :Required: No
  :Type: map<name, :ref:`Threshold <threshold-reference>`>
  :Default Value: {} *(empty map)*

graphs
  :Description: Graphs to add to the template.
  :Required: No
  :Type: map<name, :ref:`Graph <graph-reference>`>
  :Default Value: {} *(empty map)*

.. _datasource-reference:

Datasource Reference
====================

The following fields are valid for a datasource entry.

name
  :Description: Name (e.g. health). Must be a valid Zenoss object ID.
  :Required: Yes
  :Type: string
  :Default Value: *(implied from key in datasources map)*

sourcetype
  :Description: Type of datasource. See :ref:`datasource-sourcetypes`.
  :Required: Yes
  :Type: string *(must be a valid source type)*
  :Default Value: None. Must be specified.

enabled
  :Description: Should the datasource be enabled by default?
  :Required: No
  :Type: boolean
  :Default Value: true

component
  :Description: Value for the *component* field on events generated by the datasource. Accepts TALES expressions.
  :Required: No
  :Type: string
  :Default Value: "" *(empty string)* -- can vary depending on sourcetype.

eventClass
  :Description: Value for the *eventClass* field on events generated by the datasource.
  :Required: No
  :Type: string
  :Default Value: "" *(empty string)* -- can vary depending on sourcetype.

eventKey
  :Description: Value for the *eventKey* field on events generated by the datasource.
  :Required: No
  :Type: string
  :Default Value: "" *(empty string)* -- can vary depending on sourcetype.

severity
  :Description: Value for the *severity* field on events generated by the datasource.
  :Required: No
  :Type: integer
  :Default Value: 3 *(0=Clear, 1=Debug, 2=Info, 3=Warning, 4=Error, 5=Critical)* -- can vary depending on sourcetype.

cycletime
  :Description: How often the datasource will be executed in seconds.
  :Required: No
  :Type: integer -- can vary depending on sourcetype.
  :Default Value: 300 -- can vary depending on sourcetype.

datapoints
  :Description: Datapoints to add to the datasource.
  :Required: No
  :Type: map<name, :ref:`Datapoint <datapoint-reference>`>
  :Default Value: {} *(empty map)*

Datasources also allow other ad-hoc options to be added not referenced in the
above list. This is because datasources are an extensible type in Zenoss, and
depending on the value of *sourcetype*, other fields may be valid.

.. _datasource-sourcetypes:

Datasource Sourcetypes
----------------------

The following datasource sourcetypes are valid on any Zenoss system. They are
the default sourcetypes that are part of the platform. This list is not
exhaustive as datasources sourcetype are commonly added by ZenPacks.

SNMP
  :Description: Performs an SNMP GET operation using the *oid* field.
  :Availability: Zenoss Platform
  :Additional Fields:
    oid
      :Description: The SNMP OID to get.
      :Required: Yes
      :Type: string
      :Default Value: "" *(empty string)*

COMMAND
  :Description: Runs command in *commandTemplate* field.
  :Availability: Zenoss Platform
  :Additional Fields:
    commandTemplate
      :Description: The command to run.
      :Required: Yes
      :Type: string
      :Default Value: "" *(empty string)*

    usessh:
      :Description: Run command on bound device using SSH, or run it on the Zenoss collector server?
      :Required: No
      :Type: boolean
      :Default Value: false

    parser:
      :Description: Parser used to parse output from command.
      :Required: No
      :Type: string *(must be a valid parser name)*
      :Default Value: Nagios

.. todo:: Document COMMAND datasource parsers.

PING
  :Description: Pings (ICMP echo-request) an IP address.
  :Availability: Zenoss Platform
  :Additional Fields:
    cycleTime
      :Description: How many seconds between ping attempts. (note capitalization)
      :Required: No
      :Type: integer
      :Default Value: 60

    attempts:
      :Description: How many ping attempts to perform each cycle.
      :Required: No
      :Type: integer
      :Default Value: 2

    sampleSize
      :Description: How many echo requests to send with each attempt.
      :Required: No
      :Type: integer
      :Default Value: 1

Built-In
  :Description: No collection. Assumes associated data will be populated by an external mechanism.
  :Availability: Zenoss Platform
  :Additional Fields:
    None

.. todo:: Document commonly-used sourcetypes added by ZenPacks.

.. _datapoint-reference:

Datapoint Reference
===================

The following fields are valid for a datapoint entry.

name
  :Description: Name (e.g. percent). Must be a valid Zenoss object ID.
  :Required: Yes
  :Type: string
  :Default Value: *(implied from key in datapoints map)*

description
  :Description: Description of the datapoint's purpose and function.
  :Required: No
  :Type: string
  :Default Value: "" *(empty string)*

rrdtype
  :Description: Type of datapoint. Must be GAUGE or DERIVE.
  :Required: No
  :Type: string *(must be either GAUGE or DERIVE)*
  :Default Value: GAUGE

rrdmin
  :Description: Minimum allowable value that can be written to the datapoint. Any lower values will be ignored.
  :Required: No
  :Type: int
  :Default Value: None *(no lower-bound on acceptable values)*

rrdmax
  :Description: Maximum allowable value that can be written to the datapoint. Any higher values will be ignored.
  :Required: No
  :Type: int
  :Default Value: None *(no upper-bound on acceptable values)*

aliases
  :Description: Aliases for the datapoint.
  :Required: No
  :Type: map<name, formula>
  :Default Value: {} *(empty map)*

.. todo:: Document datapoint alias formulas.

Datapoints also allow other ad-hoc options to be added not referenced in the
above list. This is because datapoints are an extensible type in Zenoss, and
depending on the value of the datasource's *sourcetype*, other fields may be
valid.

.. _threshold-reference:

Threshold Reference
===================

The following fields are valid for a threshold entry.

name
  :Description: Name (e.g. unhealthy). Must be a valid Zenoss object ID.
  :Required: Yes
  :Type: string
  :Default Value: *(implied from key in thresholds map)*

type
  :Description: Type of threshold. See :ref:`Threshold Types <threshold-types>`.
  :Required: No
  :Type: string *(must be a valid threshold type)*
  :Default Value: MinMaxThreshold

enabled
  :Description: Should the threshold be enabled by default?
  :Required: No
  :Type: boolean
  :Default Value: true

dsnames
  :Description: List of *datasource_datapoint* combinations to threshold.
  :Required: No
  :Type: list
  :Default Value: [] *(empty list)*

.. todo:: Better explain syntax for threshold.dsnames.

eventClass
  :Description: Value for the *eventClass* field on events generated by the threshold.
  :Required: No
  :Type: string
  :Default Value: /Perf/Snmp -- can vary depending on type.

severity
  :Description: Value for the *severity* field on events generated by the threshold.
  :Required: No
  :Type: int
  :Default Value: 3 *(0=Clear, 1=Debug, 2=Info, 3=Warning, 4=Error, 5=Critical)* -- can vary depending on type.

Thresholds also allow other ad-hoc options to be added not referenced in the
above list. This is because thresholds are an extensible type in Zenoss, and
depending on the value of the threshold's *type*, other fields may be valid.

.. _threshold-types:

Threshold Types
---------------

The following threshold types are valid on any Zenoss system. They are the
default types that are part of the platform. This list is not exhaustive as
additional threshold types can be added by ZenPacks.

MinMaxThreshold:
  :Description: Creates an event if values are below or above specified limits.
  :Availability: Zenoss Platform
  :Additional Fields:
    minval
      :Description: The minimum allowable value. Values below this will raise an event.
      :Required: No
      :Type: string -- Must evaluate to a number. Accepts Python expressions.
      :Default Value: None *(no lower-bound on allowable values)*

    maxval
      :Description: The maximum allowable value. Values above this will raise an event.
      :Required: No
      :Type: string -- Must evaluate to a number. Accepts Python expressions.
      :Default Value: None *(no upper-bound on allowable values)*

ValueChangeThreshold
  :Description: Creates an event if the value is different than last time it was checked. 
  :Availability: Zenoss Platform
  :Additional Fields: None

.. _graph-reference:

Graph Reference
===============

The following fields are valid for a graph entry.

name
  :Description: Name (e.g. Health). Must be a valid Zenoss object ID.
  :Required: Yes
  :Type: string
  :Default Value: *(implied from key in graphs map)*

units
  :Description: Units displayed on graph. Used as the y-axis label.
  :Required: No
  :Type: string
  :Default Value: None

miny
  :Description: Value for bottom of y-axis.
  :Required: No
  :Type: integer
  :Default Value: -1 *(-1 causes the minimum y-axis to conform to the plotted data)*

maxy
  :Description: Value for top of y-axis.
  :Required: No
  :Type: integer
  :Default Value: -1 *(-1 causes the maximum y-axis to conform to the plotted data)*

log
  :Description: Should the y-axis be a logarithmic scale?
  :Required: No
  :Type: boolean
  :Default Value: false

base
  :Description: Is the plotted data in base 1024 like storage or memory size?
  :Required: No
  :Type: boolean
  :Default Value: false

hasSummary
  :Description: Should the graph legend be shown?
  :Required: No
  :Type: boolean
  :Default Value: true

height
  :Description: The graph's height in pixels.
  :Required: No
  :Type: integer
  :Default Value: 100

width
  :Description: The graph's width in pixels.
  :Required: No
  :Type: integer
  :Default Value: 500

graphpoints
  :Description: Graphpoints to add to the graph.
  :Required: No
  :Type: map<name, :ref:`Graphpoint <graphpoint-reference>`>
  :Default Value: {} *(empty map)*

comments
  :Description: List of comments to display in the graph's legend.
  :Required: No
  :Type: list<string>
  :Default Value: [] *(empty list)*

.. _graphpoint-reference:

Graphpoint Reference
====================

The following fields are valid for a graphpoint entry.

name
  :Description: Name (e.g. Health). Must be a valid Zenoss object ID.
  :Required: Yes
  :Type: string
  :Default Value: *(implied from key in templates map)*

legend
  :Description: Label to be shown for this graphpoint in the legend. The name field will be used if legend is not set.
  :Required: No
  :Type: string
  :Default Value: None

dpName
  :Description: *datasource_datapoint* combination to plot.
  :Required: Yes
  :Type: string
  :Default Value: None

.. todo:: Better explain syntax for graphpoint.dpName.

lineType
  :Description: How to plot the data: "Line", "Area" or "Not Drawn".
  :Required: No
  :Type: string
  :Default Value: Line

lineWidth
  :Description: How thick the line should be for the line type.
  :Required: No
  :Type: integer
  :Default Value: 1

stacked
  :Description: Should this graphpoint be stacked (added) to the last? Ideally both area "Area" types.
  :Required: No
  :Type: boolean
  :Default Value: false

color
  :Description: Color for the line. Specified as RRGGBB (e.g. 1f77b4).
  :Required: No
  :Type: string
  :Default Value: Cycles through a preset list depending on graphpoint's sequence.

colorindex
  :Description: Color index for the line. Can be used instead of color to specify the color sequence number rather than the specific color.
  :Required: No
  :Type: integer
  :Default Value: None

format
  :Description: String format for this graphpoint in the legend (e.g. %7.2lf%s).
  :Required: No
  :Type: string
  :Default Value: "%5.2lf%s"

.. todo:: Better explain graphpoint.format syntax.

cFunc
  :Description: Consolidation function. One of AVERAGE, MIN, MAX, LAST.
  :Required: No
  :Type: string
  :Default Value: AVERAGE

limit
  :Description: Maximum permitted value. Value larger than this will be nulled. Not used if negative.
  :Required: No
  :Type: integer
  :Default Value: -1

rpn
  :Description: RPN (Reverse Polish Notation) calculation to apply to datapoint.
  :Required: No
  :Type: string
  :Default Value: None

..tood:: Better explain graphpoint.rpn syntax.

includeThresholds
  :Description: Should thresholds associated with *dpName* be automatically added to the graph?
  :Required: No
  :Type: boolean
  :Default Value: false
