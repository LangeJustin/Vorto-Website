---
title: "Getting Started"
date: 2018-05-09T10:58:37+08:00
weight: 20
---
# Connecting a Java based device to Eclipse Hono/Ditto Sandbox with Vorto
![Material Screenshot](/images/getting-started-ar.png)

This getting-started explains how to generate a simple **Java application** that sends **distance** sensor data via **MQTT**. In 4 simple steps, we will create and verify a digital twin of our device in Eclipse Ditto.

## Prerequisites
* Maven
* IDE of your choice
* Curl

## 1. Choosing an Information Model
- In this example, we will use an existing `Vorto Information Model` describing a [distace sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor).

## 2. Generate Code
- Go to the Information Model of your device in the `Vorto Repository`, e.g. [distance sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor).
- On the right side, click on the Hono generator that will generate and download a Java application.
- Unzip the project and import it as an existing maven project in the IDE of your choice, e.g. Eclipse

## 3. Edit configuration details
Edit configuration details in `src/main/java/device/distancesensor/Distancesensor.java`. You can copy and paste the following example for the [distace sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor):

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
- See new state of your digital twin:

```sh
curl -X GET https://ditto.eclipse.org/api/2/things/hono.eclipse.org:112233 -H 'authorization: Basic ZGVtbzE6ZGVtbw==' -H 'Accept: application/json'
```

**Response:**
```sh
{  
   "thingId":"org.eclipse.vorto:112233",
   "attributes":{  
      "schema":{  
         "distance":"com.ipso.smartobjects.Distance:0.0.1"
      },
      "_modelId":"demo.iot.device.DistanceSensor:1.0.0",
      "thingName":"DistanceSensor",
      "createdOn":"2018-05-15 10:08:29+0000",
      "deviceId":"112233"
   },
   "features":{  
      "distance":{  
         "properties":{  
            "status":{  
               "sensor_value":74,
               "sensor_units":"",
               "min_measured_value":76,
               "max_measured_value":96,
               "min_range_value":3,
               "max_range_value":69,
               "current_calibration":"",
               "application_type":""
            }
         }
      }
   }
}
```