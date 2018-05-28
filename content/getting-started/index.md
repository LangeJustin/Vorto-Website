---
title: "Getting Started"
date: 2018-05-09T10:58:37+08:00
weight: 20
---
![Material Screenshot](/images/getting-started-ar2.png)

This getting-started explains how to generate a simple **Java application** that sends **distance** sensor data via **MQTT** based on a Vorto Information Model.

## Prerequisites
* Maven
* Java 8
* Eclipse IDE with DSL editors
* [Eclipse Vorto Toolset](http://marketplace.eclipse.org/content/vorto-toolset)

## Defining a Project
1. In the Vorto Model Project Browser, click the + button.
2. The Create new Vorto Project dialog opens:
![Material Screenshot](/images/tutorials/getting_started/vorto_create_new_vorto_project_dialog.png)
3. Create new Vorto Project dialog
4. Enter a name as Project Name, for example, `MyVortoProject`.
5. Click on Finish.
6. The new project is created.

![Material Screenshot](/images/tutorials/getting_started/vorto_new_vorto_project_created.png)

## Defining a Function Block
> A function block provides an abstract view on a device to applications that want to employ the devices’ functionality. Thus, it is a consistent, self-contained set of (potentially re-usable) properties and capabilities.

1. Right-click in the **Functionblocks** area and choose **New Functionblock** from the context menu.
![Material Screenshot](/images/tutorials/getting_started/m2m_tc_create_function_block_designer_dialog_2.png)
2. Enter a name as **Function Block Name**, for example, Distance
3. Adjust the entries for the input fields **Namespace** and **Version**, if necessary.
4. Optionally, enter a description in the **Description** entry field.
5. Click **Finish**. ![Material Screenshot](/images/tutorials/getting_started/m2m_tc_create_function_block_generated_source_1.png)
6. Edit the function block project by extending the generated source file in the function block DSL editor.

**Example:**
```sh
namespace org.eclipse.vorto
version 1.0.0
displayname "Distance"
description "Function block model for Distance"
functionblock Distance {

	configuration {
	}

	status {
		mandatory distance as double <MIN 0, MAX 1000>
		optional sensor_unitt as string
	}

	fault {
	}

	operations {
	}
}
```
## Defining an Information Model
> Information models represent the capabilities of a particular type of device entirety. An information model contains one or more function blocks.

1. Right-click in the **Information Models** area and choose **New Information Model** from the context menu.
![Material Screenshot](/images/tutorials/getting_started/m2m_tc_create_information_model_dialog.png)
2. Enter a name as **Information Model Name**, for example, DistanceSensor.
3. Adjust the entries for the input fields **Namespace** and **Version**, if necessary.
4. Optionally, enter a description in the **Description** entry field.
5. Click **Finish**.![Material Screenshot](/images/tutorials/getting_started/m2m_tc_information_model_dsl_editor.png)
6. Drag and drop the created and edited function block (distance) from the **Functionblocks** area into the information model **DistanceSensor** area to create the reference. ![Material Screenshot](/images/tutorials/getting_started/m2m_tc_drag_drop_function_block_to_information_model.png)

7. Right click on the Information Model **DistanceSensor** in the Information Models area and click **share**.

>	Info: this will make your Information Model public available in the [Vorto Repository](http://vorto.eclipse.org/)

## Generate Code
- Right click on the Information Model **DistanceSensor** in the Information Model area
- Click `Generate Code -> Eclipse Hono Generator`
- Choose **Java** and click **Generate** that will download a Java application.
- Unzip the project and import it as an existing maven project in the IDE of your choice, e.g. Eclipse

## Registering to Eclipse Hono
Eclipse Hono provides a remote service interface where we will send our data to. To use Hono, we need to `register our device` and `add device credentials`

{{< warning title="Please choose a hard to guess Tenant" >}}{{< /warning >}}

### Registering our Device

```sh
curl -X POST http://hono.eclipse.org:28080/registration/<your_tenant>
-i -H 'Content-Type: application/json'
-d '{"device-id": "1234","thingId":"org.eclipse.vorto:1234","modelId":"org.eclipse.vorto.informationmodels.DistanceSensor:1.0.0"}'
```

### Adding device credentials
```sh
PWD_HASH=$(echo -n "<your_passwort>" | openssl dgst -binary -sha512 | base64 | tr -d '\n') 
curl -X POST http://hono.eclipse.org:28080/credentials/<your_tenant>
-i -H 'Content-Type: application/json'
-d 'JSON payload' 
```

Example JSON payload:
```sh
{
  "device-id": "1234",
  "auth-id": "1234",
  "type": "hashed-password",
  "secrets": [
    {
      "hash-function": "sha-512",
      "pwd-hash": "'$PWD_HASH'"
    }
  ]
}
```

## Edit configuration details
Edit details in `src/main/java/device/distancesensor/Distancesensor.java` that fits your configuration:

	// Hono MQTT Endpoint
	private static final String MQTT_ENDPOINT = "ssl://hono.eclipse.org:8883";

	// Your Tenant
	private static final String HONO_TENANT = "<your_tenant>";

	// Your DeviceId
	private static final String DEVICE_ID = "1234";
	
	// Device authentication ID
	private static final String AUTH_ID = "1234@<your_tenant>";
	
	// Ditto Namespace
	private static final String DITTO_NAMESPACE = "org.eclipse.vorto";

	// Device authentication Password
	private static final String PASSWORD = "passwort";

**Download Certificate**

> Copy and paste the [certificate](https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt) to `src/main/resources/certificate/hono.crt` in your project.

## Run and verify data
- Right click on `Distancesensor.java` in you IDE and `run as Java Application` to start sending data:

![Material Screenshot](/images/run_java.PNG)


- Follow [Consuming Messages from Java for Hono] (https://www.eclipse.org/hono/dev-guide/java_client_consumer/) to check if the data is sent succesfully.
All you need to change is the Host to the Sandbox URL in the src/main/org.eclipse.hono.devices/HonoHttpDevice.java File.

{{< warning title="The Sandbox will delete it's registered devices daily!" >}}{{< /warning >}}