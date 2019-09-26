# WARNING: This repository is no longer maintained :warning:

> This repository will not be updated. The repository will be kept available in read-only mode.

**__Skill Level__**: Intermediate
<br>**__N.B__**: All services used in this repo are Lite plans.

# [Creating a Cognitive IOT Application that Monitors Household Devices](https://developer.ibm.com/patterns/detect-voltage-anomalies-in-household-iot-devices/)


This Cognitive pattern demonstrates how to build a Cognitive IoT application. In cognitive IoT solutions, machine learning takes place in an edge computing architecture. Note that Edge computing means that you push computing away from the cloud or data center out towards the sensors. Computations happens on the edge gateway (e.g. Simulated Raspberry Pi in this scenario) near the sensors and actors. Let's use a simple scenario for illustration. Assume that you have a Washing Machine with 3 sensors.

* Fluid temperature
* Hardness of H2O
* Speed of the motor

One can use the sensors to monitor the washing machine for any abnormalities for these specific parts of the machine.  When any of these become unstable, information is triggered to perform an action and to record for later analytics processing.

## Application Workflow Diagram

![Architectural Diagram](images/arch-diagram.png "Architectural Diagram")

## Flow

1. **Sensors**: The heart of every IOT solution is the sensors and actors. We sense the environment, make (cognitive) decisions, and act using actors. In the washing machine use case; the 3 sensors are 1) fluid temperature 2) hardness of H2O 3) Speed of the motor.  These sensors would be in the washing machine.
2. [**Node-RED Edge (Node-RED running on your device)**](https://github.com/IBM/CognitiveIoT/tree/master/node-red): edge means "it's near the device" which one manages; in this case that is the simulated Raspberry Pi; therefore it is called Node Red on the edge. Imagine the Raspberry Pi is attached to the washing machine. Data is flowing constantly from the sensors to the Raspberry Pi. The Node Red Edge (or Raspberry Pi) is hooked up to the Internet by ethernet and is streaming data to the Watson IoT Platform.
3. **Edge model**: It's a built-in piece of code that filters and transforms values transitioned to the IoT Platform
4. **Watson IoT Platform**: is using the MQTT Message broker. It publishes data back to the Node Red Edge (in a real world case, if it detects an anomaly, it would send a message back to the machine to deactivate its motor)
5. [**Node-RED IBM Cloud (YOUR_APP_NAME.mybluemix.net)**](https://github.com/IBM/CognitiveIoT/tree/master/defaults): There is a second Node-RED on IBM Cloud and it is subscribing to the data coming from the IoT Platform.


## Included Components
* [Node-Red Edge](https://flows.nodered.org/node/node-red-node-watson) Node-RED is a flow-based programming tool.
* [Watson IOT Platform](https://developer.ibm.com/iotplatform/)  Tools and services you need to connect, manage, analyze and secure IoT devices.



## Featured Technologies
* MQTT Message Broker
* Cloud
* IoT
* Node.js


# Steps

These are high-level steps:

* Deploy NodeRED + AnomalyDetector app to the cloud
* Configure Watson IoT Components
* Stream data into Watson IoT Platform
* Analyze Edge-based Anomaly Detector using moving zscore in NodeRED Dashboard

## Deploying the Application to IBM Cloud

A) **Using the Deploy to IBM Cloud button**

Clicking on the button will automatically deploy a NodeRED data flow tool acting as a device simulator in the IBM Cloud. It also creates a pre-configured "edge" implementation of the zScore algorithm.

The `Deploy to IBM Cloud` button automates a majority of the deployment. Afterward, you need to wire up the Watson IoT Platform to your simulated device.

[![](images/deploy_btm_cloud.png)](https://cloud.ibm.com/devops/setup/deploy?repository=https://github.com/IBM/CognitiveIoT)


1. We can see that the app is ready to be deployed, and we need to ensure that the App name, region, Organization, Space is valid before pressing 'Deploy'.
![](images/deploy-to-bm1.png)

2. In Toolchain, the app is deployed. There is also an option to edit code via eclipseIDE, git changes if required.
![](images/deploy-to-bm2.png)

3. You should see two stages pass successfully once your Deploy Stage completes
![](images/deploy-to-bm3.png)

4. To see the app and services created and configured for this pattern, use the IBM Cloud dashboard. The app is named `CognitiveIoT-20170724231917178` and it is running the following services:

  * [cognitive-iot-iotf-service](https://www.ibm.com/internet-of-things/solutions/iot-platform/watson-iot-platform)

B) **Go into IBM Cloud** and click on the url route associated with your application. you will be taken to the NodeRED flow editor where you can see the already deployed and running application.
![](images/runtheapp.png)

Please have a look; you should see the following:
![](images/edgeflow.png)

So let's walk you through each element:
1. NodeRED is free, Open Source and runs everywhere! In the IBM Cloud, in every other cloud or data center, on your laptop and even on an IoT Gateway like a Raspberry Pi. So consider this flow to run on an IoT Gateway connected to an elevator and measuring voltage for the main driver motor. As we don’t have a Raspberry Pi in place we are just simulating these sensor values using an “Inject” node in NodeRED. Otherwise you would see a dedicated sensor node here

2. In order to create some randomness this JavaScript function node adds some random noise to the signal and also occasionally adds an anomaly which we want to detect

3. In addition we want to send data upstream to the cloud so let’s add a time-stamp. It is always good to generate a time-stamp (temporally and spatially) as close as possible to the sensor. So this value can be referred to as “event time” rather than “processing time”

4. To stream these data to the IBM Watson IoT Platform via MQTT only one simple node is necessary

5. In addition we want to create a little dashboard to monitor the voltage sensor values

6. The voltage dashboard displays the voltage data. In order to achieve this we need to extract the voltage value from the input message

7. Now we are at the heart of the system. This JavaScript function will actually calculate the moving z-score for us.

8. Of course we want to plot the moving z-score as well – in parallel to the voltage in order to really understand what’s going on

9. Again we need to extract the z-score from the input message

10. Now we generate an alert messages in case the z-score drops below -0.5 which means some major fluctuation has been taken place recently

11. We just display this alert message under the two other charts

12. In order to get rid of the message once in a while we reset it

13. And delay the deletion 5s, so the message keeps displayed for 5s


C) **Observe what's going on** using the real-time dashboard: in order to open the dashboard just click on the dashboard tab and on the dashboard icon as shown here:
![](images/opendashboard.png)

When you click on it, you will observe two time series charts, one for voltage and one for the moving z-score.
![](images/dashboard.png)

Wait for some time until you observe a z-score below -0.5 and you’ll see that an alert message is being generated. Of course you can also trigger something more important like initiate an emergency shutdown of the system or raise an alert, either by sending an email/SMS from the Edge directly using a NodeRED node (see twilio or email for this) or you can also send the alert upstream to the cloud using MQTT. The latter would be a perfect example on how Edge analytics can reduce the amount of data transferred to the cloud by adding intelligence to the Edge gateway device:
![](images/dashboardalert.png)

## Setting up your IBM Cloud Application with the Watson IoT Components

There are a few more steps required to wire up your Node-RED application to Watson IoT Platform. This gives you a dashboard that shows the real-time values of your simulated sensor, as well as shows connectivity over time and values over time.

## Setting up the RemoteDevice Device Type in Watson IoT Platform

Each device connected to the IBM Watson IoT Platform is associated with a device type. Device types are intended to be groups of devices which share common characteristics. So in order to add devices in IBM Watson IoT Platform, one need to create a device type. What you are actually doing is registering a device in the IBM Watson IoT Platform.

1) Go to the menu in IBM Cloud console

2) Select Apps

3) Select Dashboard

4) Select your App's Name (not route) (CognitiveIoT-<timestamp>)

5) Under "connections", select "cognitive-iot-iotf-service"

6) Under the "Welcome to Watson IoT Platform" text, click "Launch"

7) On the WIoT page, Click the microchip icon on the left-hand side. It is labeled "Devices"
 <p align="center">
    <img width="40" height="40" src="images/Microchipicon.png">
  </p>

8) Click the "+ Add Device" button on the right side.
<p align="center">
   <img width="50" height="60" src="images/add-device.png">
 </p>

9) Click the "Create device type" button.

10) Click the "Create device type" button on the next dialog as well.

11) On the "general information" dialog, enter the name "RemoteDevice", and click the "Next" button.

12) Let's skip this page, but this is where you'd fill in metadata about an actual physical device. Click "next".

13) Click "Next" again, then "Create"

## Setting up the VoltageSensor01 Device in Watson IoT Platform

Now that your device type is created, you will add the device to the IoT Platform. A device can be defined that has a connection to the internet and has data it wants to get into the cloud. And devices can accept commands from applications as well. You need to add the device in IBM Watson IoT Platform Organization before connecting it to the IoT.

1) Select "RemoteDevice" in the dropdown under "Add Device", and click the "Next" button.

2) For Device ID, put "VoltageSensor01". This matches details under the ibmiot out node in Node-RED, which you can investigate by double-clicking the node.

3) Continue to click the "Next" button, until you get to the Summary view, and then click "Add".

4) You will now have a view of details about your VoltageSensor01 device. Scroll down to "Recent Events" and you should see voltage events occurring continuously. Under "Sensor Information", you can see the most recent data being received, including voltage, frequency, count, and timestamp ('ts'). Click the 'x' to the upper-right to close this view.

5) On the upper-left, click the menu item called "Boards".
<p align="center">
   <img width="100" height="50" src="images/board-icon.png">
 </p>

6) Click the "Device-Centric Analytics" board. You'll now see cards representing much of the same info we saw in the device details. However, this view is a dashboard to easily allow seeing multiple devices at the same time, and you could create additional cards to display further analytics.
<p align="center">
   <img width="400" height="200" src="images/dev-centric-analytics.png">
 </p>

# Performance/production considerations

In a production environment, you would run Node-RED on each of your edge devices, and wire each to Watson IoT Platform. There, you could monitor and manage all of them from a central dashboard.

# Extension considerations

For alerting, it you could run a Node-RED instance from IBM Cloud like above, and change only a few things to expand the Node-RED dashboard to show your other devices.

Another option would be to run the dashboard on your edge devices themselves, and check per-device. This is probably less than optimal though.

**Note:** This pattern was originally part of a larger tutorial which can be found [HERE](https://developer.ibm.com/patterns/detect-voltage-anomalies-in-household-iot-devices/).

## <h2>Learn more</h2>
<ul>
<li><strong>Artificial Intelligence Code Patterns</strong>: Enjoyed this Code Pattern? Check out our other <a href="https://developer.ibm.com/technologies/artificial-intelligence/" rel="nofollow">AI Code Patterns</a>.</li>
<li><strong>Data Analytics Code Patterns</strong>: Enjoyed this Code Pattern? Check out our other <a href="https://developer.ibm.com/technologies/data-science/" rel="nofollow">Data Analytics Code Patterns</a></li>
<li><strong>AI and Data Code Pattern Playlist</strong>: Bookmark our <a href="https://www.youtube.com/playlist?list=PLzUbsvIyrNfknNewObx5N7uGZ5FKH0Fde" rel="nofollow">playlist</a> with all of our Code Pattern videos</li>
<li><strong>With Watson</strong>: Want to take your Watson app to the next level? Looking to utilize Watson Brand assets? <a href="https://www.ibm.com/watson/with-watson/" rel="nofollow">Join the With Watson program</a> to leverage exclusive brand, marketing, and tech resources to amplify and accelerate your Watson embedded commercial solution.</li>
<li><strong>Watson Studios</strong>: Master the art of data science with IBM's <a href="https://www.ibm.com/cloud/watson-studio" rel="nofollow">Watson Studios</a></li>
<li><strong>PowerAI</strong>: Get started or get scaling, faster, with a software distribution for machine learning running on the Enterprise Platform for AI: <a href="https://www.ibm.com/ms-en/marketplace/deep-learning-platform" rel="nofollow">IBM Power Systems</a></li>
<li><strong>Spark on IBM Cloud</strong>: Need a Spark cluster? Create up to 30 Spark executors on IBM Cloud with our <a href="https://cloud.ibm.com/catalog/services/apache-spark" rel="nofollow">Spark service</a></li>
<li><strong>Kubernetes on IBM Cloud</strong>: Deliver your apps with the combined the power of <a href="https://www.ibm.com/cloud/container-service" rel="nofollow">Kubernetes and Docker on IBM Cloud</a></li>
</ul>

### License
-----------------------

This code pattern is licensed under the Apache Software License, Version 2.  Separate third party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](https://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache Software License (ASL) FAQ](https://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
