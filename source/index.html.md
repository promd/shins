---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - json
  - c
  - kotlin
  - swift

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors
  - test-images.md.erb

search: true
---

<aside class="warning">
This document is Work in Progress!
</aside>

# Introduction

This document describes the data exchange between devices and P&Gs cIoT cloud, based on the requirements of Project Badger (smartCC). 
Other Projects might deviate in Detail, but the overall mechanics should remain unchanged. 

___
#General


<div class="mermaid">
sequenceDiagram
SY ->> PH:         Start Provisioning
PH ->> SY:         Activate WIFI ap-scan\n(non blocking)
PH ->> BH:         Start Bluetooth
BH ->> BS:         Start Advertising
APP ->> BS:        Finds device and connect       
BS ->> BH:         Callback Event\nESP_GATTS_CONNECT_EVT 
BH ->> PH:         Callback Event\nCONNECTED 
APP -->> PH:       Get device ID (DEVICE_ID)
APP -->> PH:       Set certificates and key (AWS_KEY, AWS_CERT)
APP -->> PH:       Set consumer ID (CONSUMER_ID)
APP -->> PH:       Set timezone ID (TIMEZONE_ID)
APP -->> PH:       Set host & port (HOST_PORT)
APP -->> PH:       Set root certificate (AWS_ROOT_CA)
APP -->> PH:       Get list of available wifis (WIFI_AP_LIST)
PH ->> SY:         Get AP-list\n(Blocking with timeout)
SY ->> PH:	  Need a message
PH -->> APP:       Send AP-list
APP -->> PH:       Set Wifi ssid (SSID)
APP -->> PH:       Set Wifi password (PSWD)
APP -->> PH:       Start test of Wifi credentials
PH ->> SY:         Restart Wifi ap-connect \nwith new credentials
SY ->> PH:         Wifi test done\nprov_AddEvent(WIFI_TEST_DONE)
PH ->> SY:         Get Wifi test result
APP ->> PH:        Get WIFI ap-connect result
APP -->> PH:       End provisioning
PH ->> SY:         Store credentials
PH ->>+ SY:         Initiate a reboot
</div>


<div class="mermaid">
sequenceDiagram
User ->> App: Hello Bob, how are you?
App-->>Cloud: How about you John?
App--x Device: I am good thanks!
App-x Cloud: I am good thanks!
Note right of Cloud: Bob thinks a long<br/>long time, so long<br/>that the text does<br/>not fit on a row.
App-->User: Checking with John...
User->Cloud: Yes... John, how are you?
</div>

##development process guide
###ideal development process
###request process cIoT platform team
###issue reporting and tracking
###expected response times / resource request process
###deviating from / adding to OOTB features
##Why native Apps ?
##Gateway connected vs. direct connected devices
___
#App Developer
##Fundamentals
* Data Privacy standard implementation (GDPR)
* JSON
* AppSync
* GraphQL
* Kotlin / Swing
* BLE
* AWS cognito and federation
##Data Storage requirements
##Data Transport Encryption requirements
___
#Firmware Developer
##Fundamentals
* Data Privacy basics (GDPR)
* JSON
* MQTT
MQTT, as protocol supported by the Message Broker component, is the backbone of Amazons’ IoT core services[1]. 
It is used to realize several standard offerings. 
Because of various reasons, the current P&G cIoT cloud only leverages a subset of them. 
The ones of relevance for firmware development are
	* "direct” MQTT publish/subscribe [2]
	* Jobs [3]
* BLE
* aws-sdk
* AWS jobs
Jobs are only used for Over-the-Air update scenarios. 
While they offer greater traceability and control compared to “plain” MQTT messaging, they also come at a much higher cost. 

##recommended chipsets
* ESP32 and esp-idf
##Data Storage requirements
##Device UUID requirements

All UUIDs are following the [UUID Version 4](https://de.wikipedia.org/wiki/Universally_Unique_Identifier) type.
Device UUIDs must be stored in read-only parts of the device memory. 

##OTA requirements
##Data Transport Encryption requirements
___
#APIs
##provisioning (HL E2E and LL app-device & app-cloud)
##Developer Dashboard
##Event-driven, asynchrounus communication model (E2E with examples)
##Data models
###available Data Types
AWS supports a limited number of variable types, summarizing many more low-level types on the C-code of the device firmware.
* Integer (signed or unsigned)
* Float 
* UUIDs
* AWS-DateTime
* AWS-JSON
####naming convention
####storing complex data in JSON Attributes
* use cases
* limitations (size?)
####handling multiple physical devices (gateway use case)
###device-state

The Device State Structure is used to provide the cloud with all relevant device state information needed to drive consumer benefits and information required to operate the device management in the cloud systems. 
It also allows updates at the cloud that are being send down to the device.
Device State Schema is validated at cloud entry. 
Messages are withdrawn in case one or more attributes fail validation.

####mandatory fields

```json
{												
	"consumerId": "us-east-1:88859bc9-0149-4183-bf10-39e36EX",	
	"status": "PROVISIONED",							
}
```

**consumerId:**
field identifying the consumer currently assigned to this device.
Value is provided to the device during provisioning process and needs to be part of **every** message.

**status:**
ENUM field that indcates the status of the device-state. 
Currently supported values:

Value       | Description 
----------- | ------------
FACTORY     | Indicates that the device-state is a stub generated ad provisioning. No data received yet.
PROVISIONED | To be contained in any message send by the device. Indivates actual received data.


####reserved words (platform attributes)

```json
{												
	"createdBy"	: "GQL",
	"deviceId"	: "c436596407ab4f64ac0ec9eabcc5d33c",
	"deviceType"	: "BADGER",
	"receivedTime"	: "",
	"stateId"	: "2s789-xsas8-xasyui",
	"thingName"	: "badger_c436596407ab4f64ac0ec9eabcc5d33c",
	"traceId"	: "2s789-xsas8-xasyui"
}
```
Reserved words are being used by the cIoT platform and shall not be used for device specific attributes.
These Attributes are maintained by the cloud and must **not** be included be the device when sending updates to the cloud.

###user
####mandatory fields
####reserved words (platform attributes)
###sessions

Sessions are used to send information from the device to the cloud that represent “usage sessions”.
The definition of usage sessions differs for different device types but is always limited to condensed / short session information. 
Streaming data (like IMU sensor data) is not send by this channel.

####mandatory fields
####reserved words (platform attributes)
###log

Logs are a “general purpose” channel that can be used to send structured data up to the cloud. 
All Log messages will flow into IoT analytics and can therefore be used for analysis.

####mandatory fields
####reserved words (platform attributes)
###additional data models
####example use cases
####process
##Data connections
###Application Developer
####Account creation process
####Account log-in process
####discovery service
#####process flow
#####data exchanged
####AppSync and GraphQL
####Braze communication
###Firmware Developer
####MQTT Topic Nomenclature 

<%= image_tag "mqtt_topics.png" %>

####device->cloud
#####status updates 
#####logs 
#####session data
#####raw data
####cloud->device
#####status updates 
#####device operations
**OTA_UPDATE**
Instructs the device to initiate an over the air firmware update.

```json
Format
{
"operation": "ota_update",
"target": "<target>",
"url": "https://fw.iotdev.
alchemy.codes/<brand>/<deviceType>/<target>/<version>/<deviceType>-<target>-<version>.<fileExtension>",
"fileName": "<deviceType>-<target>-<version>.<fileExtension>",
"version": "<version>",
"fileSize": "<fileSize>",
"md5": "<md5>",
"force": "<force>"
}

Example
{
"operation": "ota_update",
"target": "full",
"url": "https://fw.iotdev.alchemy.codes/oralb/boombox_alexa/full/111/boombox_alexa_full_111.zip",
"fileName": "boombox_alexa_full_111.zip",
"version": "18202429",
"fileSize": "81400584",
"md5": "58552f2ac69ce8576a0c894974cab6f4",
"force": "true"
}
```

Parameter | Values | Description
--------- | ------ | -----------
target    | full, rtl, f0, f4 | full: default value for all devices dreamworks (Opte): rtl, f4, f0
url       |        | Download URL - domain: https://fw.iot-dev.alchemy.codes
fileName  |        | <deviceType>_<target>_<version>.<fileExtension> Supported File Extensions: .zip, .bin, .ota
version   |        | version of the firmware image
fileSize  |        | size of the firmware image file in bytes as reported from OS kernel (Terminal command ls -l)
md5       |        | md5 hash of the firmware image (terminal command md5 <file>)
force     | false, true | true: device will update whether the device is in use or not false: device will update only if not in use

**CLEAR_CONFIG**
**GET_STATUS**

Instructs the device to publish a status payload to the logs topic

```json
Format
{
"operation": "get_status"
}
```

**REBOOT**

Instructs the device to reboot.

```json
Format
{
"operation": "reboot",
"force": "true"
}
```

Parameter | Values | Description
--------- | ------ | -----------
force     | false, true | true: device will reboot whether the device is in use or not false: device will reboot only if not in use

**GET_SYSLOG**

Instructs the device to send its system log data.

```json
Format
{
"operation": "get_syslog"
}
```

####Streaming scenarios
#####after session streaming
#####direct session streaming
####OTA
#####direct OTA
#####aws jobs OTA

# Authentication

> To authorize, use this code:

```ruby
require 'kittn2'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

