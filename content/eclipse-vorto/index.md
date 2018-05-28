---
date: 2016-03-08T21:07:13+01:00
title: Eclipse Vorto
weight: 10
---
![Material Screenshot](/images/Vorto_Ar.png)

## Why Vorto?
Standardization organizations and industry consortia work hard on device abstraction related standards. Some of them are domain specific, some are very generic, and all of them are useful for a large number of use cases. However in most cases there is no tooling available that allows for creating and managing standard conform device representations. It is also the case that many standards are very complex and it is not easy to validate existing abstract representations of devices against the standard.

## What is Vorto?
Vorto provides an Eclipse Toolset that lets you describe the device functionality and characteristics as Information Models. These models are managed in a [Vorto Repository](http://vorto.eclipse.org/). [Code generators](http://vorto.eclipse.org/#/generators) convert these models into device - specific "stubs" that run on the device and send Information Model compliant messages to an IoT Backend. In order to process these messages in the IoT backend, Vorto offers a set of technical components, for example parsers and validators. For devices sending arbitrary, non-Vorto, messages to an IoT backend, the Vorto Mapping Engine helps you to execute device message transformations to IoT Platform specific meta-models, e.g. Eclipse Ditto or AWS IoT Shadow.