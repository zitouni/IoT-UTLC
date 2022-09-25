# Urban Traffic-light Control in IoT (IoT-UTLC) Testbed
## Conference papers 
Zitouni, R., Petit, J., Djoudi, A., & George, L. (2019). Iot-based urban traffic-light control: Modelling, prototyping and evaluation of mqtt protocol. In Proceedings - 2019 IEEE International Congress on Cybermatics: 12th IEEE International Conference on Internet of Things, 15th IEEE International Conference on Green Computing and Communications, 12th IEEE International Conference on Cyber, Physical and So. https://doi.org/10.1109/iThings/GreenCom/CPSCom/SmartData.2019.00051

Petit, J., Zitouni, R., & George, L. (2018). Prototyping of Urban Traffic-Light Control in IoT. In 2018 IEEE International Smart Cities Conference (ISC2) (pp. 1–2). IEEE. https://doi.org/10.1109/ISC2.2018.8656717

![Maquette](https://zupimages.net/up/18/35/np9t.jpg)

## Introduction
In this wiki page, we will explain how we implemented our project. We propose a concept of a Traffic-light control through the Internet of Things (IoT). We have prototyped a proof of concept using a number of technological solutions such as 6LowPAN, MQTT protocol, IEEE 802.15.4, Contiki Os, Re-motes, .etc. You would be able throughout the Wiki steps to rebuild our project and to reuse our source codes to prototype new IoT solutions.  

[FR]
Il s’agit de détailler et d’expliquer le fonctionnement des Re-motes de Zolertia et nos implémentations additionnelles
Ce projet est pour l'heure actuelle, une preuve de concept.
Même si cette documentation possède des parties assez techniques, elle a pour but d'être pour un maximum de personnes et donc certaines notions pourront être abstraites et certains aspects moins proches de la réalité.


Vidéo du projet : https://youtu.be/QottWpnE1zk

## Situation
The aim of this project is to implement a traffic light system using IoT and a Wireless Sensor Network (WSN). Zolertia's Re-motes are the motes of our WSN and Ubidots is the IoT cloud plateform. 
We built a Maquette with a 3D crossroad model, mini traffic-lights and wireless sensors/actuators. Our contribution is the combination of a number of solutions in order to ensure the interoperability between objects, which use different communication's technologies. For example, if vehicles use ITS-G5 for V2V communication, and road's sensors are connected through 6LowPAN network, our solution would allow the both to interact. In our case, we have established an IoT Cloud Platform to perform that indirect communication between roads and traffic-lights. 
We imagine that a vehicle could interrupt the normal cycle of a crossroad traffic-lights. 
For example, emergency vehicles such as ambulance, police, firefighters could change their road signal to green and minimize the overall waiting time. 

To reproduce our results of this project, you need minimum background of these technologies:
- Zolertia Re-motes
- Contiki Os
- 6LoWPAN
- IPv6
- MQTT
- QoS

To achieve the maquette, you need 9 Zolertia Re-motes (6-7 minimum) divided into 3 parts:
- 1 Border Router Re-mote to forward packets between the 6LoWPAN network (Re-motes) and the Internet. 
- 4 Re-motes for the four traffic-lights of the crossroad
- 2 to 4 Re-motes are the deployed road's sensors. We choose a touch sensor to detect and identify the arrival of priority vehicles, but other types of sensors could be used.

## Border router

For simplicity, we use a Re-mote as Border Router (BR). However, it is not capable by itself to send and receive data from the Internet. So few options should be taken. To send and receive directly from IP network (or Internet), an additional module can be used like the ENC28J60 to connect an Ethernet cable to Re-mote BR. The advantage is that the system is autonomous.
[https://github.com/Zolertia/Resources/wiki/How-to-build-your-own-Ethernet-Zoul-Router]
Another way to have Internet connection is to plug the Border Router to a host computer (PC, Raspberry Py, etc.). In this case, the packets are forwarded to the IoT Cloud Platform through a python script or a Middelware executed by the host computer. 


## Traffic light Re-mote

Those Re-motes share a same main source code with some minor changes. For each traffic light mote, we have to define in the source code their IDs. In addition, they play two roles: traffic lights masters and slaves resulting on some extra source code.

One traffic light will periodically send a packet to notify when it will change its colors (their states). It will not change its state locally, but it has to wait until receiving confirmation from the Middelware. The objective of this coordination is to satisfy synchronization between traffic lights. After that, the cycle from green to yellow and then red will be trigged locally.

The traffic lights have three states (or colors) corresponding to values:
- 0: Red light
- 1: Yellow light
- 2: Green light

[FR]
Ici pour un effort de présentation, un feu à base de leds ont été fabriqués afin d'obtenir un meilleur affichage et compréhension pour ce projet.
Pour la mise en place de cette solution, 3 leds (rouge, jaune, vert) ont été disposés et branchés sur les pins RGB du Re-mote et la masse a été placée sur une des GRD (ground) présente. Ainsi pour les brancher correctement il faut brancher dans l'ordre sur la LED1 le rouge, la LED2, le vert et enfin sur la LED3 l’orange.
[Image ci-dessous]. Le Re-mote fourni assez d'énergie directement via ses pins pour allumer les différentes leds.
![PinDisposition](https://zupimages.net/up/18/35/lxxd.png)


## Road's sensors (touch sensor)
The deployed road's sensors could interrupt the classic cycle of the traffic lights. In our case, we have touch sensors driven by Re-motes. When Re-motes detect the arriving of priority vehicles, an event-driven communication is triggered. Our system (IoT-UTLC) orders the safe change of traffic light signal from red to green.  

Technically, the touch sensor is connected to the Re-mote via ADC port. The sensor gives a number (0 or 0<). 
The Re-mote will periodically read the sensor's values and do:
- If the value is equal to 0, no touch has been detected by the sensor, no action required
- If the value is above 20 000, we suppose a touch has occurred and packet will be sent to notify its will to change its road to green.

The obtained values of the touch sensor are around 32 000 (for example, 32 764). You can see if you run the touch sensor application that the printed values in this range. 

If you obtain values around 2268 or so, check again if you have connected the touch sensor to the right ADC port. Indeed, such values suggest a bad connection or bad understanding of the value sent to the Re-mote.

![TouchSensor](https://zupimages.net/up/18/35/c28v.jpg

[FR]
Similairement à ce que l'on a effectué précédemment, une fois cette condition respectée, le Re-mote va lancer la fonction `send_packet_event()`que l'on a vu précédemment. C'est dans celle-ci que l'on interviendra pour définir l'ID du feu et donc de la rue qui sera prioritaire.
Le code est basé sur zoulmate-demo.c et udp-client.c que l'on peut retrouver respectivement dans `~/contiki/examples/zolertia/zoul/`et `~/contiki/example/zolertia/project/3-sensors/`


## Set up of the Project with host computer as Interface for Internet Network

### Step 1 : Set up the Border Router
The source code is based on the example that can be found in `~/contiki/examples/ipv6/UDP-border-router/`.

The first step is the upload of the border-router application. In the directory `~/contiki/examples/zolertia/project/1-border-router/`, you should execute this command `make border-router.upload` in your terminal. The source code will be compiled and then the binary will be written on the Re-mote.
The second step is creating a virtual bridge between the Border router and the host computer. You connect your Re-mote to your host machine with USB port. To link our Border Router and the host machine, use the tunslip6 tool built into Contiki Os. Go to  `~/contiki/tools/` and execute the command `./tunslip6 -s /dev/ttyUSB0 aaaa::1/64`. Here serial port is fixed to ttyUSB0, because we connect the Re-mote via USB first. To avoid any problem of device recognition, you should check the number of the port opened for it. By doing a `dmesg | grep USB` you can see on which port your USB device has been linked.

```bash
etudiant@ubuntu:~/contiki/tools$ ./tunslip6 -s /dev/ttyUSB0 aaaa::1/64
********SLIP started on ''/dev/ttyUSB0''
opened tun device ''/dev/tun0''
ifconfig tun0 inet 'hostname' mtu 1500 up
ifconfig tun0 add aaaa::1/64
ifconfig tun0 add fe80::0:0:0:1/64
ifconfig tun0

tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:127.0.1.1  P-t-P:127.0.1.1  Mask:255.255.255.255
          inet6 addr: fe80::1/64 Scope:Link
          inet6 addr: aaaa::1/64 Scope:Global
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:500
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

*** Address:aaaa::1 => aaaa:0000:0000:0000
Got configuration message of type P
Setting prefix aaaa::
Server IPv6 addresses:
 aaaa::212:4b00:60d:b3ef
 fe80::212:4b00:60d:b3ef
```
The Border Router has a webserver, accessing via our navigator Firefox by installing the plugin Copper (Cu) and go to the url `http://[aaaa::212:4b00:60d:b3ef]`
(Your ipv6 addressee will be different from this one, use the one obtained with the last command).
You can see in the web page the list of neighbors that the Border Router is capable to reach and its routing table.
You should have an empty list but it will be useful to confirm if all our Re-motes are well connected to our 6LoWPAN network.

![BorderRouterWebServer](https://zupimages.net/up/18/35/z0qz.png)


### Step 2 : Setup of Traffic lights 
The source codes can be found in `~/contiki/examples/zolertia/tutorial/02-ipv6/03-udp-client-and-server/` 
The goal is to obtain a behavior close to what we can find in a crossroad. Our maquette has been made base an an actual crossroad in Paris with measure of its lights delays. We have established to have a change of the traffic light states every 30 seconds.

For synchronization, we defined master and slave traffic lights. Only masters send periodically packets to notify a will to change their states. To have a good working cycle of a change every 30 seconds, Re-motes have to be started at the same time using the RESET button (right button of the Re-mote). In addition, we specified two roads A and B and for each one we have one master traffic light. The master of the road A got a lead of 30 seconds compared to the master of the road B. Coupling this specification with a transmission rate of 1 packet every minutes, you obtain this timeline:

![Timeline Traffic lights](https://zupimages.net/up/18/35/yf0k.png)


Slave traffic lights will obey changing their states after receiving the master’s packets. 

Every Re-mote have 2 possible actions:
- Packets sent periodically to notify a will to change its state. 
- As a help for debugging or understanding of the system, action can be triggered using the user button of the Re-mote (left button of the Re-mote). You can see that it is customizable in the `send_packet_sensor` function.

Now, to configure the traffic lights, connect the Re-motes to your host computer (it can be one after another if you don't have lots of USB ports).
Go to `~/contiki/examples/zolertia/project/2-trafficlight-control` and upload the 4 Re-motes programs with the command `make trafficlight-control.upload PORT=/dev/ttyUSBx`. Where x is the port number defined by your machine for the Re-mote connected to it. However, a personalization is required. Those traffic light will have an ID to define themselves to the system.
In the code it is in the `send_packet_event` and `send_packet_sensor` functions.
```c
msg.id = 0x1; /* Set trafficlight/sensor ID */
```
You can modify in the order to have traffic light 1 with an 0x1 ID, traffic light 2 with an 0x2 ID ...
In our representation, for RoadA, we have traffic lights 1 and 3 and for RoadB we have traffic lights 2 and 4.
Moreover, we have some initialization values of the traffic lights to simplify our model.
In `trafficlight_control_process`
```c
leds_on(LEDS_RED); //Traffic lights 2 & 4 start at green, 1 & 3 at red
```
Where we have defined traffic lights 1 and 3 at red and 2 and 4 at green.

To define our traffic lights master and slave, we have 2 separated codes which have the same objective but one will not have the ability to periodically send data to the Internet. In particular, this:
```c
/* Send data to the server */
    /* QoS 0: Non-priority data sent every 30 seconds */
    if (ev == PROCESS_EVENT_TIMER)
    {
      send_packet_event();
      if (etimer_expired(&periodic))
      {
        etimer_reset(&periodic);
      }
    }
```
Modification for ID is the same.
Once, those 4 Re-motes are configured and ready, you should see them on the webserver of the Border Router.
You can unplug them from your host computer, let them rely on their batteries.
If you need to debug something, at any moment, you can display the output of the Re-motes still connected to your host machine via the command `make login PORT=/dev/ttyUSBx` where x is the port defined by your machine for the connected Re-mote.

### Step 3: Set up touch sensors

Configuring the touch sensor is similarly to the one used for the traffic lights. A specific ID has to be defined in the `send_packet_sensor` function in the code. The ID is related to the road where the sensor has been deployed. For RoadA, its ID will be 5 or 7 and so for RoadB it will be 6 or 8. 
4 Re-motes can be used to simulate priority vehicle for every roads, but, as far as it is a proof of concept, we limited the use to only 2 Re-motes to be present on the 2 roads.
Go to `~/contiki/examples/zolertia/project/3-sensors/` and flash the Re-motes with the command `make touch-sensor.upload PORT=/dev/ttyUSBx` where x is the port defined by your machine for the connected Re-mote.
You can unplug them from your host machine, let them rely on their batteries.
Check by using the webserver of the Border Router that those new Re-motes are recognized and active in the system.

### Step 4 : Prepare Ubidots for the reception of data

A Cloud architecture is needed to get and display data. Here, Ubidots has been chosen to answer this problem.
You have to create an account on the platform. If you are still in Education, it is free but with limited functions. Don’t worry, it is enough to satisfy the specification of our project.
https://ubidots.com/education/

Once you registered and logged in, click on your name then `API Credentials` and copy the default token

Code base for the python script `UDP-MQTT-server.py` can be found in `~/contiki/examples/zolertia/tutorial/02-ipv6/03-udp-client-and-server/`. It will start an MQTT client with Ubidots, our MQTT borker. It will also start a UDP server and listen to the IPv6 address `aaaa::1` to retreinve every message from 6LoWPAN.

Go to `~/contiki/examples/zolertia/project/1-border-router/paho-MQTT/` and open the UDP-MQTT-final-server.py.
In the `start_client()` function, seek for the MQTT connection part to find this line:
```python
client.username_pw_set("A1E-restOfYourToken", "")
```
And put your token here.
Topics, which are information channel through which our messages goes, they define an hierarchy of our data in the Cloud Platform. They will be created automatically along with the variables under the name zolertia, the first time script has been launched and connected to the IoT Cloud Platform.


### Step 5 Final implementation for the project
Border Router is operational, Re-motes are configured and working, all is needed now to send and receive packets from Ubidots.
You have to run the python script that will be an MQTT client and a UDP client.
Go to `~/contiki/examples/zolertia/project/1-border-router/paho-MQTT/`  and launch the python script `python UDP-MQTT-final-server.py`.
Go to Ubidots and the Devices page, you should see a new source named zolertia with 2 variables (for now) and updated data from a few second ago. Check that all your variables have been created and that data right and will come up every 30 seconds or so. If you have had too many variables updates in one message, it is possible that some could be missing from Ubidots. It will be better to split that message in several messages to send them correctly.

Create a dashboard to monitor the system.
To add a new widget:
- Select the `indicator` type,
- Select  `On/Off`
- Select variables to control (here roadA and roadB of the device zolertia)
- Define the values:
    - 0   Red light   #ff0000
    - 1   Yellow light  #ff9900
    - 2   Green light    #00d415



## Set up of the Project with independent Re-mote 
Second version of this project have similarities with the first one. However, code is different for the Border Router as well as for the traffic light and sensor Re-motes.

This code is on the `autonomous-border-router` branch of the project on Github

### Step 1 : Set up Ethernet Router
Code is based on the examples of Ethernet Routers that can be found in `~/contiki/examples/zolertia/tutorial/99-apps/mqtt-node/Binairies`.

First, flash a Re-mote by going in `~/contiki/examples/zolertia/project/v2/1-ethernet-router/` and execute the command `python ../../../../../tools/cc2538-bsl/cc2538-bsl.py -e -w -v -a 0x200000 Binaries/router/cetic_6lbr_router_eth_gw.bin`. It will write the Re-mote's binary file.
Then some wiring is necessary to connect the external Ethernet module to the Re-mote.
Follow this tutorial to realize it
[https://www.hackster.io/alinan/a-diy-iot-ethernet-router-with-ip64-over-6lowpan-365280]


Once all connections have been made and the Re-mote is on, execute `make login` to obtain the following results:
```bash
etudiant@ubuntu:~/contiki/examples/zolertia/tutorial/02-ipv6/03-udp-client-and-server$ zoul-login
../../../../../tools/sky/serialdump-linux -b115200 /dev/ttyUSB0
connecting to /dev/ttyUSB0 (115200) [OK]
Contiki-develop-20160421-4-gde85971
Zolertia eth-gw
Rime configured with address 00:12:4b:00:06:16:0f:5f
 Net: sicslowpan
 MAC: CSMA
 RDC: nullrdc
NOTICE: 6LBR: Starting 6LBR version 1.4.x (Contiki-develop-20160421-4-gde85971)
INFO: NVM: Reading 6LBR NVM
INFO: NVM: NVM Magic : 2009
INFO: NVM: NVM Version : 5
NOTICE: 6LBR: Log level: 30 (services: ffffffff)
INFO: ETH: ENC28J60 init
INFO: ETH: Eth MAC address : 06:00:06:16:0f:5f
INFO: ENC: resetting chip
INFO: ETH: ENC-28J60 Process started
INFO: LLSEC: Using 'noncoresec' llsec driver
INFO: 6LBR: Security layer initialized
INFO: 6LBR: Tentative local IPv6 address fe80::212:4b00:616:f5f
INFO: 6LBR: Tentative global IPv6 address (WSN) aaaa::212:4b00:616:f5f
INFO: 6LBR: Tentative global IPv6 address (ETH) bbbb::100
INFO: 6LBR: RA Daemon enabled
INFO: 6LBR: Checking addresses duplication
INFO: NVM: Flashing 6LBR NVM
ERROR: NVM: verify NVM failed, retry
INFO: NVM: Flashing 6LBR NVM
INFO: 6LBR: Configured as DODAG Root aaaa::212:4b00:616:f5f
INFO: 6LBR: Starting IP64
Starting DHCPv4
INFO: 6LBR: Starting as RPL ROUTER
INFO: NODECFG: Node Config init
INFO: HTTP: Starting webserver on port 80
INFO: UDPS: UDP server started
INFO: DNS: DNS proxy started
INFO: 6LBR: CETIC 6LBR Started
INFO: 6LBR: Set IPv4 address : 10.4.89.107
```
With these information, we have everything we need to know. Ethernet Router is working properly and you can access its parameters from the web service created at the addressee 10.4.89.107 (in our case, yours will be different).
Depending on how your connection to Internet is, this IP address can vary and so, it could be useful to check if it has not change sometimes.
This IPv4 address is accessible everywhere from the internet and it is the gateway of our 6LoWPAN network.

![EthernetRouterWebServer](https://zupimages.net/up/18/35/kx88.png)

### Step 2 : Set up traffic lights
Now, flash your 4 Re-motes for traffic lights.
Go to `contiki/examples/zolertia/tutorial/99-apps/mqtt-node/` and open the Makefile.
Definition of what kind of sensor/element will be flash is there.
```Makefile
MQTT_SENSORS  ?= trafficlight
```

Moreover, to personalize the traffic light (like the ID system on the previous project), we define the right topic in `internals/ubidots.h`
```c
#define DEFAULT_SUBSCRIBE_STATE       "/roada/lv" /* Define /roada/lv ou /roadb/lv depending of your trafficlight*/
```
Here because we don't use ID system, our topic architecture is the same but a different functioning.
Every Re-mote has it own topic with its variables (temperature, signal state, battery...) but because of the limitation of MQTT engine in Contiki Os, multi topic publishing and subscribing is not available.
So the modifications of the zolertia topic, which contains the global variables of our system (RoadA, RoadB), will be updated in the Ubidots Platform in a next step.

### Step 3 : Set up touch sensors
Like we have done for the traffic lights, process is the same, the codes are in the same folder.

Defining the DEFAULT_SUBSCRIBE_STATE is no longer required because it will not receive data.
In the Makefile, MQTT_SENSORS should be modified.
```Makefile
MQTT_SENSORS  ?= remote
```

### Step 4 : Set up events on Ubidots
The topic, like in the previous project, will have a `zolertia`topic created along with topics for every Re-mote sending data. As it has been said in a previous section, modification in the `zolertia` topic for modifications in the Re-mote's topics would be automatic and handled in Ubidots using events.

Those events can only modify one value at a time, so you will have to create as many events as its needed for your expected result.

4 Events should be created for every traffic lights:
- When `trafficlight` value (for instance in traffic light 1) goes to 0 => value roada in the topic `zolertia` goes to 0
- Same for roadb to 2
- When `trafficlight` value (for instance in traffic light 1) goes to 2 => value roada in the topic `zolertia` goes to 2
- Same for roadb to 0

For the touch sensor, 2 events are required:
- If value (of touch sensor for roadA) > 3000 (the value divided by 10 is sent to Ubidots), then `zolertia/roada` goes to 2
- Same condition but for `zolertia/feu2` to go to 0

Project is now working. All topics should be receiving adequate data and the topic `zolertia` and its variable should be automatically modified.

This project has some downsides. For instance, not being able to publish and subscribe to multiple topics is not perfect. 
To be noted in the MQTT engine on Contiki Os, only level 0 and 1 of QoS has been developed.
Finally, our traffic lights are not synchronized, due to issues mentioned above and lack of solution for the time implementation has been made.

## Credit
This project has been initiated as student project at ECE Paris. 

### Proposed and supervised by: 

Rafik ZITOUNI

### Participants: 

Jérémy PETIT (Four months' internship project)

Jonathan HAUTERVILLE

Sophie DUBIEF 

Pierre BENEDICK

Aghiles Djoudi
 
