---
title: "Getting Started"
date: 2018-05-09T10:58:37+08:00
weight: 20
---
# Connecting a Java based device to Eclipse Honogit:q Sandbox with Vorto
![Material Screenshot](/images/getting-started-ar.png)

This getting-started explains how to generate a simple **Java application** that sends **distance** sensor data via **MQTT**.

## Prerequisites
* Maven
* IDE of your choice

## 1. Choosing an Information Model
- In this example, we will use an existing `Vorto Information Model` describing a [distace sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor).

## 2. Generate Code
- Go to the Information Model of your device in the `Vorto Repository`, e.g. [distance sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor).
- On the right side, click on the Hono generator that will generate and download a Java application.
- Unzip the project and import it as an existing maven project in the IDE of your choice, e.g. Eclipse

## 3. Edit configuration details
Edit configuration details in `src/main/java/device/distancesensor/Distancesensor.java`. You can copy and paste the following example for the [distance sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor):

	// Hono MQTT Endpoint
	private static final String MQTT_ENDPOINT = "ssl://hono.eclipse.org:8883";

	// Your Tenant
	private static final String HONO_TENANT = "DEFAULT_TENANT";

	// Your DeviceId
	private static final String DEVICE_ID = "112233";
	
	// Device authentication ID
	private static final String AUTH_ID = "112233@DEFAULT_TENANT";
	
	// Ditto Namespace
	private static final String DITTO_NAMESPACE = "org.eclipse.vorto";

	// Device authentication Password
	private static final String PASSWORD = "secret123";

**Download Certificate**

> Copy and paste the [certificate](https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt) to `src/main/resources/certificate/hono.crt` in your project.

## 4. Run and verify data
- Right click on `Distancesensor.java` in you IDE and `run as Java Application` to start sending data. 
- Follow [Consuming Messages from Java for Hono] (https://www.eclipse.org/hono/dev-guide/java_client_consumer/) to check if the data is sent succesfully.
All you need to change is the Host to the Sandbox URL in the src/main/org.eclipse.hono.devices/HonoHttpDevice.java File.