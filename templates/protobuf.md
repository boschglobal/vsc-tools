
# Protobuf Generator

## Introduction

The Protobuf Generator converts VSC services to a [Google Protocol Buffer](https://developers.google.com/protocol-buffers/) definition.

 
## Conversion Concept

* VSC typedefs are not represented in protobuf, instead the base type is used.
* VSC Structs are represented in protobuf.
* VSC methods are represented as rpc in protobuf.
* VSC events are represented as messages in protobuf.
* VSC properties are represented as a read rpc and a write rpc (if actuator) in Protobuf.
* VSC Error Messages are currently not imported.

## Preparations

No additional installations needed to run the generator, but to validate output a protobuf compiler is needed 

```
apt install -y protobuf-compiler
```

## Running protobuf generator

The code below runs the prototype protobuf generator and verifies that the generated code is syntactically correct.

```
# go to vsc-tools if not already there
cd vsc-tools
# make sure that vsc has been cloned
git clone https://github.com/COVESA/vehicle_service_catalog/
python model/vsc_generator.py vehicle_service_catalog/comfort-service.yml protobuf.tpl > vsc.proto
# 
mkdir tmp
protoc --cpp_out=tmp vsc.proto
```

## Limitations with current protobuf generator

The general tooling framework provided by VSC is currently not that advanced, partially depending on that VSC as language not yet is that stable.
The open source tooling perform syntactic checks, but it does not perform semantic checks, e.g. verifying that a referenced datatype actually exists.

The protobuf template has a number of limitations. It is intended to support the example service only, it does not intend to support all possible use-cases of VSC. 
* For example it does not support use of types from other namespaces. 