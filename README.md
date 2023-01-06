# Air Monitoring

## Introduction

The aim of the project is to simulate an air monitoring system. Whenever it detects unhealthy air it starts a purifier and notifies the user via Alexa.

To present the project, the sensors are simulated using node red. Meanwhile, purifier management is simulated using the `ProcessThreshold` function.

## Architecture

METTI L'IMMAGINE

The project is composed by 1 function:

- `ProcessThreshold`: processes the data received from the sensors to determine in which threshold values fall into and sends a command to the purifier setting its power.

And an Alexa Skill [`Voice Monkey`](https://voicemonkey.io/).

All of this is achieved by using `RabbitMQ`, `Nuclio`, `Node Red`, `IFTTT` and `Docker`.

### RabbitMQ

RabbitMQ supports multiple messaging protocols and can be deployed to meet high-scale and high-availability requirements. The used protocol for the project is [MQTT](https://mqtt.org/).

To start a `RabbitMQ` instance with Docker:

```
docker run -p 9000:15672  -p 1883:1883 -p 5672:5672  cyrilix/rabbitmq-mqtt 
```

RabbitMQ dashboard is deployed on http://localhost:9000 and can be accessed using `guest` as username and `guest` as password.

### Nuclio

Nuclio is High-Performance Serverless event and data processing platform.

To start a `Nuclio` instance with Docker:

```
docker run -p 8070:8070 -v /var/run/docker.sock:/var/run/docker.sock -v /tmp:/tmp nuclio/dashboard:stable-amd64
```
Nuclio dashboard is deployed on http://localhost:8070.

### Node Red

Node-RED is a flow-based development tool for visual programming for cabling hardware devices, APIs, and online services as part of the Internet of Things.

To start a `Node Red` instance with Docker:
```
docker run -it -p 1880:1880 -v node_red_data:/data --name mynodered nodered/node-red
```
Node Red dashboard is deployed on http://localhost:1880.

### IFTTT

[IFTTT](https://ifttt.com/explore) allows you to automate certain actions given an event, actions that can go from simply opening an app, turning on a device to management of the domotic devices of an entire house.

### Voice Monkey
[Voice Monkey](https://voicemonkey.io/)’s API allows you to trigger Alexa routines externally from services such as IFTTT and Home Assistant and offers dynamic text-to-speech (TTS) capabilities.

### Docker

Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow developers to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package. Thanks to the container, the application will run on any other Linux machine regardless of any customized settings.

Docker is used in the architecture to deploy functions in application containers and each of the functions is a Docker container that is listening on a socket port and can be invoked by any of the configurable triggers.

## How to use
The project is fully functional using docker instances. 

1) Start `RabbitMQ` instance

2) Start `Nuclio` instance

3) Start `Node Red` instance 

4) Sign in to `Voice Monkey` with your Amazon account and in the `Manage Monkey` tab click on `Add a monkey`. Choose a name and save (let's say `Echo Monkey`). 

5) Download `Voice Monkey` on your Alexa app and create a new routine. 
    - In the "When" tab select the _monkey_ previously created (Echo Monkey)
    - Add "Alexa says" action and write whatever you want Alexa to say (ex. "Caution! Poor air quality detected!")
    - Save

6) Register and log in on `IFTTT` then:
    - Create a new Applet
    - `IF` action will be a Webhook 
    - `THEN` reaction will be a Trigger Monkey. 
You need to save your request url and IFTTTkey by clicking on your webhook [documentation](https://ifttt.com/maker_webhooks)

7) After Nuclio instance has been successfully started, browse to http://localhost:8070 to access to Nuclio dashboard.
Then, simply follow these steps:

    - Create a new project named `airMonitoring`.
    - Click on `Create Function` and then on `Import` to upload `processThreshold`  functions by using `processThreshold.yaml` file. You can find this file in `./yaml-function` folder.
    - Click on `Create`.
    - Go to the `Code` tab and change the `iftttKey` and `mqttUrl` variables using your IFTTT key and MQTT url (should be mqtt:your-local-ip).
    - Click on `Deploy` button.

8) Once the `Node Red` container is started and running:
    - Open the Node Red [dashboard](http://localhost:1880).
    - Import the airMonitoringFlow in `./flow` folder.
    - Press Manage palette and from install tab search and install the following palette:
        - node-red-dashboard
        - node-red-contrib-mqtt-broker
    - Set the MQTT server property (Server, Port, Username and Password) on `pmTopic` or `fanTopic` (this will automatically change both).
