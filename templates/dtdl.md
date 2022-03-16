# DTDL Generator

## Conversion concept
The intention is to generate correct DTDL according to the [specification](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md), keeping as much as possible of the information in the VSC definition. For now the focus is on concepts used in the [VSC example service](https://github.com/COVESA/vehicle_service_catalog/blob/master/comfort-service.yml). The reason for this limitation is that the functionality of the tooling in the [COVESA vsc-tools repo](https://github.com/COVESA/vsc-tools) currently has significant limitations, concerning e.g. nested namespaces and inclusion of data from other files. In this section potential mappings are described for all VSC concepts used in the example file. 

### VSC Namespace
Represented as [DTDL Interface](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#interface). Proposed default naming for [example VSC service](https://github.com/COVESA/vehicle_service_catalog/blob/master/comfort-service.yml) is `"dtmi:global:covesa:comfort:seats"`, to be aligned with the homepage [http://covesa.global/](http://covesa.global/) of COVESA. In more advanced tooling the prefix (`global:covesa`) could be an input parameter.

It must be noted that for now VSC has not yet aligned on a specific tree/naming structure, e.g. whether there should exist a top level called `Vehicle` similar to [VSS](https://github.com/COVESA/vehicle_signal_specification).

### VSC Primitive Types
General approach is to convert from [VSC native datatype](https://github.com/COVESA/vehicle_service_catalog#native-data-types) to [DTDL datatype](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#primitive-schemas).

- `uint8`, `int8`, `uint16`, `int16`, `int32` -> `integer` (signed 4-byte)
- `uint32`, `uint64`, `int64` -> `long` (signed 8-byte). Some values of uint64 cannot be represented, but likely has no impact. Should possibly in long term give warning during conversion.
- `boolean` -> `boolean`
- `float` -> `float`
- `double` -> `double`
- `string` -> `string`

 A conversion tool could possibly add a DTDL comment giving a textual description of the base type, like `"Comment":"Original VSC type: uint16"`

### VSC Struct
[VSC Structs](https://github.com/COVESA/vehicle_service_catalog#namespace-list-object-structs) can be represented as [DTDL Interface Schema](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#interface-schemas)

### VSC Typedef
No good DTDL representation exist for [VSC Typedefs](https://github.com/COVESA/vehicle_service_catalog#namespace-list-object-typedefs). Suggested conversion is to use base-type, i.e. for the example below `integer`. A conversion tool could possibly add a DTDL comment giving a textual description of the base type, like `"Comment":"Original VSC type: movement_t (int16, min: -1000, max:1000"`


```
 typedefs:
      - name: movement_t
        datatype: int16
        min: -1000
        max: 1000
        description: |
          The movement of a seat component
```

### VSC Enumerations
[VSC Enumerations](https://github.com/COVESA/vehicle_service_catalog#namespace-list-object-enumerations) should be straightforward to represent as [DTDL enum](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#enum).

### VSC Methods
[VSC Methods](https://github.com/COVESA/vehicle_service_catalog#namespace-list-object-methods) should be possible to represent as [DTDL Command](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#command)

If the VSC method has more than 1 in-param or more than 1 out-param an inline struct needs to be used in DTDL representation. The struct must then have a name, a possible approach is to use `in` for the in-parameter and `out` for the out-parameter.
 
### Events
Assuming the [VSC Events](https://github.com/COVESA/vehicle_service_catalog#namespace-list-object-events) are sent by the vehicle, it could be represented as [DTDL Telemetry](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#telemetry). 

### Properties
[VSC Properties](https://github.com/COVESA/vehicle_service_catalog#namespace-list-object-properties) is intended to be able to represent [VSS attributes and signals (sensors/actuators)](https://github.com/COVESA/vehicle_service_catalog). They match [DTDL Property](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#property) quite well, but to a certain extent also [DTDL Telemetry](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#telemetry).

In the DTDL documentation examples for [Telemetry](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#telemetry-examples) and [Properties](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#property-examples) it seems that Telemetry is used for data similar to VSS sensors and Properties for items similar to VSS actuators and VSS attributes.
But in VSS an actuator can also act as a sensor, i.e. when you set it you set the "wanted" value, but when you read it you get the "actual" value rather than the "wanted" value.
An example is `Vehicle.Cabin.Seat.Row1.Pos1.Position`.

For now, the following mapping is used:

- VSS Actuator: Writeable Property
- VSS Sensor: Telemetry
- VSS Attribute: Non-writeable Property

### Handling of generic fields
- VSC description can be represented as DTDL description property.
- VSC does not have structured comments (only `#` comments that are ignored when creating internal model), not possible to convert to DTDL comment property.

### Open Topics

* Representation of VSC Properties (VSS Actuators/Sensors) need further discussion.It might be desirable to be able to e.g. both write to an actuator and to subscribe the same actuator to get notified by changes. It is currently unclear if this can be fulfilled by having a single DTDL entity (Property or Telemetry), or if some VSC/VSS items needs to be represented twice in DTDL, e.g. by both a Telemetry and Property.

## Validating DTDL files

Validation of DTDL can be used by an [Azure example project](https://github.com/azure-samples/dtdl-validator/).

Steps required for compiling and running the tools on Windows 10 includes:

- Install Dotnet. When verifing the example DTDL file NET Core SDK 5.0 was used, others might work
- `git clone https://github.com/azure-samples/dtdl-validator/`
- Edit the `*.csproj` file to include the correct NET version, e.g. `<TargetFramework>netcoreapp5.0</TargetFramework>`
- Make sure environment variables `HTTP_PROXY`and `HTTPS_PROXY` are defined if needed
- Build it with `dotnet build DTDLValidator.csproj`

When running the DTDL file to analyze shall be put in a separate directory, as the tool analyzes all files found in current directory.

Example when errors are found in file:

``` bat 
MYUSER@MYCOMPUTER MINGW64 /c/myuser/dtdl-validator/DTDLValidator-Sample/DTDLValidator/tmp (master)
$ ../bin/Debug/netcoreapp5.0/DTDLValidator.exe
Simple DTDL Validator (dtdl parser library version 3.12.5.0)
Validating *.json files in folder 'C:\myuser\dtdl-validator\DTDLValidator-Sample\DTDLValidator\tmp'.
Recursive is set to True

Read 1 files from specified directory
Validated JSON for all files - now validating DTDL
*** Error parsing models. Missing:
  dtmi:com:example:SeatAdjusterVehicleApp;3
  dtmi:com:example:SwdcComfortSeat;3
  dtmi:com:example:VSSService;3
Could not resolve required references

```
Example of successful validation:

``` bat 
MYUSER@MYCOMPUTER MINGW64 /c/myuser/dtdl-validator/DTDLValidator-Sample/DTDLValidator/tmp (master)
$ ../bin/Debug/netcoreapp5.0/DTDLValidator.exe
Simple DTDL Validator (dtdl parser library version 3.12.5.0)
Validating *.json files in folder 'C:\myuser\dtdl-validator\DTDLValidator-Sample\DTDLValidator\tmp'.
Recursive is set to True

Read 1 files from specified directory
Validated JSON for all files - now validating DTDL

**********************************************
** Validated all files - Your DTDL is valid **
**********************************************
Found a total of 18 entities

```

## DTDL Generator Template

A Proof-of-Concept (PoC) [generator template](dtdl.tpl) has been created using the template framework of the vsc-tools project.
It can be used like below:

```
# go to vsc-tools if not already there
cd vsc-tools
# make sure that vsc has been cloned
git clone https://github.com/COVESA/vehicle_service_catalog/
python model/vsc_generator.py vehicle_service_catalog/comfort-service.yml dtdl.tpl > dtdl_generated.json
```

The generated file can be validated as described in previous section.

The PoC shows that the vsc-tooling can be used to generate DTDL.
The [generator template](dtdl.tpl) does however look quite ugly as logical indentation for loops and if-statements in the template are interleaved with actual indentation to use in output.
The focus of the PoC has been to represent the [VSC example service](https://github.com/COVESA/vehicle_service_catalog/blob/master/comfort-service.yml), which works without any problems.
More complex features of the VSC language like file inclusions and nested namespaces have not been within the scope of the PoC.

Known limitations of the PoC implementation:

* Code contain quite a lot of code duplication - could possibly be avoided with addition Jinja macros for code reuse
* Handling of commas between entities not optimal
* No support to put files under other prefix, `global:covesa` always used
* No support for namespaces of arbitrary depth. Currently everythiong expected to be within the same namespace
* No support for including other files or referencing types in other namespaces
* No support for including error messages defined in VSC.


In general all data in the example VSC service can be converted, but minor details are lost:

- Explicit min/max values (from VSC typedefs)
- Implicit min/max values (from VSC types like `uint8` as a bigger type like `integer` must be used in DTDL)
