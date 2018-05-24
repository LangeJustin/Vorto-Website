---
date: 2016-03-09T20:08:11+01:00
title: Tutorials
weight: 30
---

## Connecting a Java based Device
This tutorial explains how to generate a simple Java application that sends distance sensor data via MQTT. In 4 simple steps, we will create a digital twin of our device in Eclipse Ditto.

### Prerequisites
* Maven
* IDE of your choice
* Curl

### 1. Choosing an Information Model
- In this example, we will use an existing `Vorto Information Model` describing a [distace sensor] (http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor).

{{< warning title="Sandbox has limited resources" >}}
In this tutorial, we are using the Hono/Ditto Sandbox for demonstration purposes. Since they have (very) limited resources, we would strongly suggest to use a local instance that fits your needs.
{{< /warning >}}

### 2. Creating a digital twin
With [Eclipse Ditto](https://www.eclipse.org/ditto/), we are able to create a digital twin of our device. In other words, we are creating a virtual, cloud based, representation that is described by our `Vorto Information Model`.

> Note: You might choose a different thingId & deviceId since they might already exists

```sh
BASE64_CRED=$(echo -n "demo1:demo" | base64 | tr -d '\n') 
curl -X PUT https://ditto.eclipse.org/api/2/things/org.eclipse.vorto:112233 
-H 'authorization: Basic '$BASE64_CRED'' 
-H "Accept: application/json" 
-d 'JSON payload' 
```

Example JSON payload:
```sh
{
  "thingId": "org.eclipse.vorto:112233",
  "policyId": null,
  "attributes": {
    "schema": {
      "distance": "com.ipso.smartobjects.Distance:0.0.1"
    },
    "_modelId": "demo.iot.device.DistanceSensor:1.0.0",
    "thingName": "DistanceSensor",
    "createdOn": "2018-05-14 10:08:29+0000",
    "deviceId": "112233"
  },
  "features": {}
}
```

> Optional: check if your device was created:

```sh
BASE64_CRED=$(echo -n "demo1:demo" | base64 | tr -d '\n')
curl -X GET https://ditto.eclipse.org/api/2/things/org.eclipse.vorto:112233 
-H 'authorization: Basic '$BASE64_CRED'' 
-H "Accept: application/json"
```
### 3. Registering to Eclipse Hono
Eclipse Hono provides a remote service interface to connect our device to eclipse Ditto, our digital twin. To use Hono, we need to `register our device` and `add device credentials`

**3.1 Registering our Device**

```sh
curl -X POST http://hono.eclipse.org:28080/registration/DEFAULT_TENANT
-i -H 'Content-Type: application/json'
-d '{"device-id": "112233","thingId":"org.eclipse.vorto:112233","modelId":"demo.iot.device.DistanceSensor:1.0.0"}'
```

> Optional: Check registration of your device:

```sh
curl -i http://hono.eclipse.org:28080/registration/DEFAULT_TENANT/112233
```

**3.2 Adding device credesntials**

```sh
PWD_HASH=$(echo -n "secret123" | openssl dgst -binary -sha512 | base64 | tr -d '\n') 
curl -X POST http://hono.eclipse.org:28080/credentials/DEFAULT_TENANT
-i -H 'Content-Type: application/json'
-d 'JSON payload' 
```

Example JSON payload:
```sh
{
  "device-id": "112233",
  "auth-id": "112233",
  "type": "hashed-password",
  "secrets": [
    {
      "hash-function": "sha-512",
      "pwd-hash": "'$PWD_HASH'"
    }
  ]
}
```

 > Optional: Check credentials of your device

```sh
curl -i http://hono.eclipse.org:28080/credentials/DEFAULT_TENANT/112233 
```

### 4. Using Code Generators
- Go to the Information Model of your device in the Vorto Repository, eg [distance sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor).
- On the right side, click on the Hono generator that will generate and download a Java application.
- Unzip the project and import it as an existing maven project in the IDE of your choice, e.g. Eclipse
- copy and paste the certificate of https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt to `src/main/resources/certificate/hono.crt` in your project
- Edit configuration details in `src/main/java/device/distancesensor/Distancesensor.java`

**To check configurations details:**

```sh
curl -i http://hono.eclipse.org:28080/credentials/DEFAULT_TENANT/112233
```
> Note: AUTH_ID needs to be followed by "@YOUR_TENANT". See example below.

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

### 5. Run and verify data
- Right click on `Distancesensor.java` in you IDE and `run as Java Application` to start sending data. 
- See new state of your digital twin:

```sh
BASE64_CRED=$(echo -n "demo1:demo" | base64 | tr -d '\n')
curl -X GET https://ditto.eclipse.org/api/2/things/org.eclipse.vorto:112233 
-H 'authorization: '$BASE64_CRED'' 
-H "Accept: application/json"
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

## Connecting a ESP8266 with Arduino
This tutorial explains how to generate an Arduino sketch for a given Information Model and send the device data via MQTT.

### Prerequisites
* [Arduino IDE](https://www.arduino.cc/en/Main/Software)
    * ESP8266 Package
    * [PubSubClient](https://pubsubclient.knolleary.net/)
* OpenSSL
* Curl

### 1. Setup your development environment (on your development machine).
 *   Download and install the [Arduino IDE](https://www.arduino.cc/en/Main/Software).
        
    *   Install the ESP8266 Board Package.
        
        1.  Select **File -> Preferences -> Additional Board Manager URLs** and add `http://arduino.esp8266.com/stable/package_esp8266com_index.json`.
            
        2.  Select **Tools -> Board -> Boards Manager…**, look for the esp8266 package and install the latest version.
            
        3.  Select **Tools -> Board**.
            
            You should now find all the supported ESP8266 boards listed, chose the one you are working with.
            
        4.  Plug-in the NodeMCU board and check in the Device Manager, whether you have the necessary USB serial driver installed. In case it is missing and Windows Update can’t find the driver, get the latest version of the driver from the [NodeMCU github repository](https://github.com/nodemcu/nodemcu-devkit/blob/master/Drivers/).
            
    *   Select **Sketch -> Include Library -> Manage Libraries**, look for the PubSubClient and install the latest version.
        
        > Note: You can use the search field to search for the PubSubClient.
        
    
    The PubSubClient will get installed in the Arduino/libraries directory, i.e. in `{HOME}/Arduino/libraries`. As you might need to adjust the buffer size for the MQTT payload, it is a good thing to verify the location of the library at this point.



### 2. Generating an arduino sketch using the Arduino Generator (Arduino project).
    
- Go to the Information Model of the [distace sensor] (http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor) in the Vorto Repository
- On the right side, click on the Arduino generator
- Store the ZIP file and extract the source code.
- Open the INO file in your Arduino IDE.

### 3. Adjust the Arduino Project according to your needs.
The following important changes have to be made:
```sh
/* Your tenant in Eclipse Hono / Bosch IoT Hub */
#define hono_tenant "DEFAULT_TENANT"

/* Device Configuration */
String hono_deviceId = "112233";
String ditto_namespace = "org.eclipse.vorto";

/* MQTT broker endpoint */
const char* hono_endpoint = "ssl://hono.eclipse.org:8883";
const char* hono_password = "secret123";
String hono_authId;

#if (USE_SECURE_CONNECTION == 1)
    /* SHA-1 fingerprint of the server certificate of the MQTT broker, UPPERCASE and spacing */
    const char* mqttServerFingerprint = "E6 A3 B4 5B 06 2D 50 9B 33 82 28 2D 19 6E FE 97 D5 95 6C CB";
#endif

/* WiFi Configuration */
const char* ssid = "<ENTER YOUR WIFI SSID>";
const char* password = "<ENTER YOUR WIFI PASSWORD>"; 
```

* For a secure connection, you need a fingerprint of the server certificate. This finger print is an SHA-1 hash of the certificate of the MQTT broker.

* If you want to create the fingerprint yourself [get the certificate](https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt) and copy and paste the content in a .crt file. The fingerprint for the certificate can be extracted by invoking:

```sh
openssl x509 -noout -fingerprint -sha1 -inform pem -in [certificate-file.crt]
```

* Connect your ESP8266 to your computer and select the virtual COM port to which your device is connected in the Arduino IDE under **Tools -> Port** and upload the sketch.

* Follow [Consuming Messages from Java for Hono] (https://www.eclipse.org/hono/dev-guide/java_client_consumer/) to check if the data is sent succesfully.
All you need to change is the Host to the Sandbox URL in the src/main/org.eclipse.hono.devices/HonoHttpDevice.java File.

```sh
BASE64_CRED=$(echo -n "demo1:demo" | base64 | tr -d '\n')
curl -X GET https://ditto.eclipse.org/api/2/things/org.eclipse.vorto:112233 
-H 'authorization: '$BASE64_CRED'' 
-H "Accept: application/json"
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
-------------




## Connecting a GroviPi sensor
This tutorial explains how to generate a simple Pyhton application for you GrovePi that sends sensor data using MQTT.

### Prerequisites

*   [Python 3.x](https://www.python.org/)
    
*   IDEs - for Python: [Visual Studio Code with the Python extension](https://code.visualstudio.com/docs/languages/python)
    
*   [Paho Python Client](https://eclipse.org/paho/clients/python/)
    
*   [GrovePi Sensor Kit](https://www.dexterindustries.com/grovepi/)
    


### 1.  Setup your device.
In this example Raspberry Pi is used but you can use any device which can run python.

- [Install raspbian on the Raspberry Pi](https://www.raspberrypi.org/learning/software-guide/).
- [Connect the pi to wifi](https://www.raspberrypi.org/learning/software-guide/wifi/).
- [Enable ssh connection on your pi](https://www.raspberrypi.org/documentation/remote-access/ssh/) .
- [Install python and required modules](https://github.com/eclipse/vorto/blob/development/tutorials/tutorial_install_python_and_required_python_modules.md).

        
### 2. Setup your development environment (on your development machine).

- [Install Visual Studio Code with the Python extension](https://code.visualstudio.com/docs/languages/python).
- [Install python and required modules](https://github.com/eclipse/vorto/blob/development/tutorials/tutorial_install_python_and_required_python_modules.md).
        
### 3.  Generate application code using the Python generator.
    
- Go to the Information Model of your device in the Vorto Repository, eg [distance sensor](http://vorto.eclipse.org/#/details/demo.iot.device/DistanceSensor/1.0.1?s=distancesensor).
- On the right side, click on Eclipse Hono and select Pyhton
- Click Generate
- Store the ZIP file and extract the source code.
- [Get the certificate](https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt) and copy and paste the content in a .crt file where you stored your source code

        
### 4.  Install GrovePi Python dependencies on the Raspberry Pi.
    
- Install necessary dependencies for GrovePi on the Raspberry Pi. To do so, just execute the commands

```sh
sudo curl https://raw.githubusercontent.com/DexterInd/Raspbian_For_Robots/master
upd_script/fetch_grovepi.sh | bash
       
sudo reboot
```

- You can find full instructions [here](https://www.dexterindustries.com/GrovePi/get-started-with-the-grovepi/setting-software/).
    
### 5.  Connect the sensor to the Raspberry Pi.
    
- Select a sensor from the kit.
- Connect the selected sensor to the Raspberry Pi. In this tutorial, ultrasonic sensor is used, which measeurs a non-contact distance. The ultrasonic sensor is connected to **pin “3”** on the GrovePi.
        
### 6.  Update the application code.
    
The ZIP file contains the necessary libraries for your project and a sample Python main program. The main program is named after the thing type and stored in the main directory of the project. Open this file in your editor of choice to make the necessary adjustments.
    
Code sections which can be customized for your needs are marked with
    
```sh
        ### BEGIN SAMPLE CODE
```         
and
```sh
    
        ### END SAMPLE CODE
```
    
{{< warning title="The following sections are of particular interest" >}}{{< /warning >}}
    
- Import the GrovePi dependencies:

```sh      
      from grovepi import *
```     
        
- Update the Device Configuration values like:
        
```sh
      hono_tenant = "DEFAULT_TENANT"
      hono_password = "secret123"
      hono_endpoint = "ssl://hono.eclipse.org:8883"
      hono_deviceId = "112233"
      hono_clientId = hono_deviceId
      hono_authId = hono_deviceId + "@" + hono_tenant
      hono_certificatePath = "PATH TO YOUR CERTIFICATE"
      ditto_namespace = "org.eclipse.vorto"
```
            
- Define a constant and assign the pin number of the pin to which the sensor is physically connected:

```sh
      # constants
      ultrasonic_ranger = 3
```            
              
- The function `periodicAction` is called periodically every `timePeriod` seconds. It is meant to periodically read your sensor’s values and transmit them using the thing type as a data model layer and the Paho MQTT Client.
        
- Read the data and assign it to the functional block properties:

```sh     
      infomodel.distance.sensor_value = ultrasonicRead(ultrasonic_ranger)
      infomodel.distance.sensor_units = "cm"
```       
        
- The variable `timePeriod` is set further down in the file. In this example, the variable `timePeriod` is set to 10.
        
### 7.  Run the application on the device.
- Copy the code to the device ([How to copy files to Raspberry Pi?](https://www.raspberrypi.org/documentation/remote-access/ssh/scp.md)).
- Open the Terminal and navigate to the source code folder.
- Run the application by typing the following command:

```sh  
      python GENERATED_MAIN_PYTHON_FILE.py
```      
`example output:`

![grovePi Screenshot](/images/tutorials/grovepi/output_screenhot_grovepi.png)

<Verify incoming sensor data still needs to be demonstrated> 
        
