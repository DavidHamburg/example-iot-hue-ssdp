# Example-IoT-Hue [![Build Status](https://dev.azure.com/lganzzzo/lganzzzo/_apis/build/status/oatpp.example-crud?branchName=master)](https://dev.azure.com/lganzzzo/lganzzzo/_build?definitionId=9?branchName=master)

Example project how-to create an Philips Hue compatible REST-API that is discovered and controllable by Hue compatible Smart-Home devices like Amazon Alexa or Google Echo.

For this discoverability, an UDP/SSDP stack is implemented.

This REST-API was implemented with the help of the Hue API unofficial reference documentation by burgestrand.se

See more:

- [Oat++ Website](https://oatpp.io/)
- [Oat++ Github Repository](https://github.com/oatpp/oatpp)
- [Get Started](https://oatpp.io/docs/start)
- [Philips Hue API — Unofficial Reference Documentation](http://www.burgestrand.se/hue-api/)

## Overview

This project is using [oatpp](https://github.com/oatpp/oatpp) and [oatpp-swagger](https://github.com/oatpp/oatpp-swagger) modules.

### Project layout

```
|- CMakeLists.txt                        // projects CMakeLists.txt
|- src/
|   |
|   |- connection/                       // Folder where the UDP/SSDP stack is implemented
|   |- controller/                       // Folder containing UserController where all endpoints are declared
|   |- db/                               // Folder with database mock
|   |- dto/                              // DTOs are declared here
|   |- SwaggerComponent.hpp              // Swagger-UI config
|   |- AppComponent.hpp                  // Service config
|   |- App.cpp                           // main() is here
|
|- test/                                 // test folder
|- utility/install-oatpp-modules.sh      // utility script to install required oatpp-modules.
```

---

### Build and Run

#### Using CMake

**Requires**

- `oatpp` and `oatpp-swagger` modules installed. You may run `utility/install-oatpp-modules.sh` 
script to install required oatpp modules.

```
$ mkdir build && cd build
$ cmake ..
$ make 
$ ./example-iot-hue-exe        # - run application.
```

#### In Docker

```
$ docker build -t example-iot-hue .
$ docker run -p 8000:8000 -t example-iot-hue
```

---

### Endpoints declaration

All implemented endpoints are compatible to a Philips Hue bridge (V1 and V3).
**Their path and structure are fixed!**

#### SSDP: Search Responder
```c++
ENDPOINT("M-SEARCH", "*", star)
```
This Endpoint accepts and answers to `M-SEARCH` SSDP packets like a Philips Hue hub would do.

#### HTTP: description.xml
```c++
ENDPOINT("GET", "/description.xml", description)
```
In the discovery answer, a reference to this endpoint is send back.
This endpoints emulates a static `desciption.xml` which includes all necessary information required to act as an Philips Hue hub.

See [Bridge discovery (burgestrand.se)](http://www.burgestrand.se/hue-api/api/discovery/)

#### HTTP: One-Shot 'user' registration
```c++
ENDPOINT("POST", "/api", appRegister, BODY_DTO(oatpp::Object<UserRegisterDto>, userRegister))
```

This endpoint just emulates a valid user-registration on a Philips Hue hub.

See [Application registration (burgestrand.se)](http://www.burgestrand.se/hue-api/api/auth/registration/)

#### HTTP: Get all 'lights'
```c++
ENDPOINT("GET", "/api/{username}/lights", getLights, PATH(String, username))
```

This endpoint returns a **object** of all devices in a Philips Hue compatible fashion.
However, formally this endpoint should just return the names. But returning the full list is fine too.

See [Lights (burgestrand.se)](http://www.burgestrand.se/hue-api/api/lights/)

#### HTTP: Get state of a specific light
```c++
ENDPOINT("GET", "/api/{username}/lights/{hueId}", getLight, PATH(String, username), PATH(Int32, hueId))
```
This endpoint returns the state of the light given in `{hueId}` in a Philips Hue compatible fashion.

See [Lights (burgestrand.se)](http://www.burgestrand.se/hue-api/api/lights/)

#### HTTP: Set state of a specific light
```c++
ENDPOINT("PUT", "/api/{username}/lights/{hueId}/state", updateState,
      PATH(String, username),
      PATH(Int32, hueId),
      BODY_DTO(Object<HueDeviceStateDto>, state))
```

This endpoint accepts a Philips Hue compatible state-object and sets the state in the internal database accordingly.
It is called e.g. by Alexa if you tell it  "Alexa, turn on <devicename>".
Finally it returns a "success" or "error" object.

See [Lights (burgestrand.se)](http://www.burgestrand.se/hue-api/api/lights/)
