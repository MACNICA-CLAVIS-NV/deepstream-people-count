# Reference Apps using DeepStream 5.0

This app contains the reference applications for person counting using the USB camera and the DeepStream SDK 5.0.
Before utilizing it, you should have Jetson developer kit on which Jetpack 4.4 is installed.

Requirements:
		1.Jetson product:TX2, NX, AGX Xavire (pick one)
		2.Jetpack version:Jetpack4.4 [L4T 32.4.3]
		3.DeepStream version:DeepStream 5.0.1 for Jetson
		4.USB camera(Logicool webcam C270 HD 720P)
		5.CUDA:10.2.89, TensorRT:7.1.3.0, cuDNN:8.0.0.180
		6.Node-RED version: v1.2.2, Node.js version: v12.19.0
		7.kafka version: 2.5.0

How to setup:
---
1. To set up Kafka
	1. Install Kafka
		$ sudo apt install openjdk-11-jre
		$ java --version

	2. Download from the below site
		https://downloads.apache.org/kafka/2.5.0/kafka_2.12-2.5.0.tgz

	3. Build Kafka server into Jetson (Noted that you should keep to open terminal screen)
		$ cd ~/Downloads
		$ tar -xzf kafka_2.12-2.5.0.tgz
		$ cd kafka_2.12-2.5.0
		$ bin/zookeeper-server-start.sh config/zookeeper.properties

	4. Start Kafka sever (Noted that you should keep to open terminal screen)
		$ cd ~/Downloads/kafka_2.12-2.5.0
		$ bin/kafka-server-start.sh config/server.properties

	5. Create the topic (This is an exsample with test for topic in this project)
		$ cd ~/Downloads/kafka_2.12-2.5.0
		$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
		$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092

2. To set up Node-RED to watch the number of person on dashboard
	1. Install Node-RED and confirm Node-RED version
		$ sudo apt install -y nodejs npm
		$ sudo npm install n -g
		$ sudo n stable
		$ sudo apt purge -y nodejs npm
		$ exec $SHELL -l
		$ node -v
		$ npm -v

	2. Start Node-RED (Noted that you should keep to open terminal screen)
		$ sudo npm install -g --unsafe-perm node-red
		$ node-red

	3. Access to Node-RED on Browser with "URL:localhost:1880"

	4. Add the below items on manage palette in Node-RED in order to connect to Kafka sever
		"node-red-contrib-kafka-manager"
		"node-red-dashboard"
		"node-red-node-email"(if you need it)

	5. Import "flows.json" file to create the flow of person counting in the Node-RED

	6. You can see E-mai notification on gmail when the counting number of person be exceeded the threshold you set in the Node-RED

	7. Click the "Deploy" button in the Node-RED

3. To start deepstream to count the person number with the USB camera
	1. Follow "To install additional packages", "To install librdkafka", "To install latest NVIDIA V4L2 GStreamer plugin", and "To install the DeepStream SDK" in the below URL
		https://docs.nvidia.com/metropolis/deepstream/5.0/dev-guide/index.html#page/DeepStream_Development_Guide/deepstream_quick_start.html#wwpID0E0GI0HA

	2. To set up additional enviroment
		$ sudo apt-get install libglib2.0-dev libjson-glib-dev uuid-dev
		$ sudo apt-get install libglib2.0 libglib2.0-dev
		$ sudo apt-get install libjansson4  libjansson-dev
		$ sudo apt-get install librdkafka1=0.11.3-1build1
		
	3. Replace the news files you downloaded with the original one.
		$ mkdir work
		$ sudo cp -r /opt/nvidia/deepstream/deepstream-5.0/sources ~/work
		$ sudo mv deepstream_test4_app.c ~/work/sources/apps/sample_apps/deepstream-test4/deepstream_test4_app.c
		$ sudo mv nvmsgconv.cpp ~/work/sources/libs/nvmsgconv/nvmsgconv.cpp
		
	4. Make 2 files
		$ cd ~/work/sources/libs/nvmsgconv
		$ sudo make
		$ sudo make install

		$ cd ~/work/sources/apps/sample_apps/deepstream-test4
		$ sudo make
		$ sudo make install
				
	5. Confirm your webcam
		$ sudo apt-get install v4l-utils
		$ v4l2-ctl --list-devices
		
	6. Start deepstream sample app
		$ cd ~/work/sources/apps/sample_apps/deepstream-test4/
		$ deepstream-test4-app -p /opt/nvidia/deepstream/deepstream-5.0/lib/libnvds_kafka_proto.so --conn-str="127.0.0.1;9092;test"
		
4. Modify the flow to adapt your environment in the Node-RED
		1. Click the red mark
		<img src="https://github.com/MACNICA-CLAVIS-NV/deepstream-people-count/blob/main/images/The placement of the debug icon.png">
		2. Please find the "person_count" on the debug messages and check the number(the original number is "123" but, it is "93" in the below image)
		<img src="https://github.com/MACNICA-CLAVIS-NV/deepstream-people-count/blob/main/images/person_count on the debug messages.png">
		3. Double click the red mark
		<img src="https://github.com/MACNICA-CLAVIS-NV/deepstream-people-count/blob/main/images/The placement of the object function.png">
		4. You should find the number of the element in the next "person_count" on the debag messages and apply it to the payload in the object function as the below:
			for example : var msgnew = {payload: a[124]} (the original number of object function:"124", the number in the following image:"94")
		<img src="https://github.com/MACNICA-CLAVIS-NV/deepstream-people-count/blob/main/images/The value of the payload in the object function.png">
		5. Click the "Deploy" button

5. Check the result on Browser
		You can realize the result of the people-count when you click the red marks according to the order in the below
		<img src="https://github.com/MACNICA-CLAVIS-NV/deepstream-people-count/blob/main/images/The order of displaying for the dashboad.png">
