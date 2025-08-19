# ROS2 Workshop

![ROS-2_logo](https://github.com/user-attachments/assets/734acefc-bf0a-456d-a5cb-c2e577d20cf5)

## Basic Overview & Installation

---

### ROS 1 Distributions

- **Box Turtle**
- **C Turtle**
- **Diamondback**
- **Electric Emys**
- **Fuerte Turtle**
- **Groovy Galapagos**
- **Hydro Medusa**
- **Indigo Igloo**
- **Jade Turtle**
- **Kinetic Kame**
- **Lunar Loggerhead**
- **Melodic Morenia**
- **Noetic Ninjemys**

### ROS 2 Distributions

- **Ardent Apalone**
- **Bouncy Bolson**
- **Crystal Clemmys**
- **Dashing Diademata**
- **Eloquent Elusor**
- **Foxy Fitzroy**
- **Galactic Geochelone**
- **Humble Hawksbill**
- **Iron Irwini**
- **Jazzy Jalisco**
- **Kilted Kaiju**

---
![compatible-robots-ros-en](https://github.com/user-attachments/assets/cda26829-5a03-496b-922d-2ce9b1d6437d)

### Prerequisites

Before installing ROS 2, make sure you have Git installed. You can install Git using the following command:

```bash
sudo apt-get update
sudo apt-get install git
```
### Official Documentation

   https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html
   
#### ROS 1

- **Ubuntu 14.04** - ROS Indigo
- **Ubuntu 16.04** - ROS Kinetic
- **Ubuntu 18.04** - ROS Melodic
- **Ubuntu 20.04** - ROS Noetic

#### ROS 2

- **Ubuntu 18.04** - ROS Eloquent Elusor
- **Ubuntu 20.04** - ROS Galactic Geochelone
- **Ubuntu 22.04** - ROS Humble Hawksbill

### If your getting Sudoers error , see below :
 First Switch to root user
 ```bash
su
```
if getting authentication failure error , Check User Privileges: Make sure your user is in the sudo group. You can verify this by running:
```bash
groups <your-username>
```
If your username is listed under sudo, you have the correct privileges. If not, you'll need to add your user to the sudo group

use  'whoami' command to check user
```bash
whoami
```
then , 
```bash
apt install sudo

usermod -aG sudo <your_username>

exit
```
After that restart your virtual machine
```bash
reboot
```

#### ROS 2 Installation Script

You can use the following script to automatically detect your Ubuntu release and install a compatible ROS 2 distro:

```bash
sudo git clone https://github.com/linorobot/ros2me

cd ros2me

./install

```

#### To check the current Distro installed

```bash
printenv
```
```bash
echo $ROS_DISTRO
```
### Add sourcing to your shell startup script

```bash
gedit .bashrc

source /opt/ros/humble/setup.bash
```
OR
```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```
### By default, ROS 2 communication is not limited to localhost. You can set the environment variable with the following command:
```bash
echo "export ROS_LOCALHOST_ONLY=1" >> ~/.bashrc
```

## Creating a Workspace
```bash
mkdir -p ~/<ws_name>/src
```

### Initialize the Workspace 
```bash
colcon build
```
#### The colcon tool is used for building ROS 2 packages. If colcon is not installed, you can install it using:
```bash
sudo apt-get install python3-colcon-common-extensions
```
## Source the Workspace
```bash
source install/setup.bash
```

 # Creating a Package

## https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html

## Using cpp

 ```bash
cd <your_workspace>/src

ros2 pkg create --build-type ament_cmake --license Apache-2.0 <package_name>

```
## Creating a Node (Simple Publisher and Subscriber)

![ROSpublisher](https://github.com/user-attachments/assets/0ba9a1b9-f4fd-4e2f-aa1d-8ef9064497d5)

#### Then navigate to src folder inside your created package and lets create our First NODE

```bash
cd <package_name>/src

gedit publisher.cpp

```
### publisher.cpp

```bash
#include <chrono>
#include <functional>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using namespace std::chrono_literals;

/* This example creates a subclass of Node and uses std::bind() to register a
* member function as a callback from the timer. */

class MinimalPublisher : public rclcpp::Node
{
  public:
    MinimalPublisher()
    : Node("minimal_publisher"), count_(0)
    {
      publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
      timer_ = this->create_wall_timer(
      500ms, std::bind(&MinimalPublisher::timer_callback, this));
    }

  private:
    void timer_callback()
    {
      auto message = std_msgs::msg::String();
      message.data = "Hello, world! " + std::to_string(count_++);
      RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
      publisher_->publish(message);
    }
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
    size_t count_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  rclcpp::shutdown();
  return 0;
}
```
### subscriber.cpp
```bash
#include <memory>
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
using std::placeholders::_1;

class MinimalSubscriber : public rclcpp::Node
{
public:
  MinimalSubscriber()
  : Node("minimal_subscriber")
  {
    subscription_ = this->create_subscription<std_msgs::msg::String>(
      "topic", 10, std::bind(&MinimalSubscriber::topic_callback, this, _1));
  }

private:
  void topic_callback(const std::shared_ptr<const std_msgs::msg::String> msg) const
  {
    RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg->data.c_str());
  }

  rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalSubscriber>());
  rclcpp::shutdown();
  return 0;
}
```
### Update the CmakeLists.txt file
```bash
cmake_minimum_required(VERSION 3.5)
project(mypackage)
 
# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
 
# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
 
# Declare a C++ executable for talker
add_executable(talker src/publisher.cpp)
ament_target_dependencies(talker rclcpp std_msgs)
 
# Declare a C++ executable for listener
add_executable(listener src/subscriber.cpp)
ament_target_dependencies(listener rclcpp std_msgs)
 
# Install the executables
install(TARGETS
  talker
  listener
  DESTINATION lib/${PROJECT_NAME}
)
 
# Export the dependencies
ament_package()
```

#### Then come to workspace and build / source the workspace using :
```bash
cd <your_workspace>

colcon build

source install/setup.bash

```

### To Run The Talker NODE ;
```bash

ros2 run <your_package_name> talker

```
### To Run The Subscriber NODE ;
```bash

ros2 run <your_package_name> listener

```

## Using python
```bash
cd <your_workspace>
cd <src>

ros2 pkg create --build-type ament_python --license Apache-2.0 <package_name>

cd <package_name>

touch <package_name>/publisher.py
chmod +x <package_name>/publisher.py

touch <package_name>/subscriber.py
chmod +x <package_name>/subscriber.py

```
### publisher.py
```bash
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalPublisher(Node):

    def __init__(self):
        super().__init__('minimal_publisher')
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        timer_period = 0.5  # seconds
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.i = 0

    def timer_callback(self):
        msg = String()
        msg.data = f'Hello, world! {self.i}'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.i += 1

def main(args=None):
    rclpy.init(args=args)
    minimal_publisher = MinimalPublisher()
    rclpy.spin(minimal_publisher)
    minimal_publisher.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()

```

### subscriber.py
```bash
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalSubscriber(Node):

    def __init__(self):
        super().__init__('minimal_subscriber')
        self.subscription = self.create_subscription(
            String,
            'topic',
            self.listener_callback,
            10)
        self.subscription  # prevent unused variable warning

    def listener_callback(self, msg):
        self.get_logger().info(f'I heard: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    minimal_subscriber = MinimalSubscriber()
    rclpy.spin(minimal_subscriber)
    minimal_subscriber.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()

```
## Modify setup.py
```bash
from setuptools import setup

package_name = '<package_name>'

setup(
    name=package_name,
    version='0.0.0',
    packages=[package_name],
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='Your Name',
    maintainer_email='your.email@example.com',
    description='TODO: Package description',
    license='TODO: License declaration',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            'publisher = <package_name>.publisher:main',
            'subscriber = <package_name>.subscriber:main',
        ],
    },
)
```

#### Now build/source your workspace and Run the Node
```bash
ros2 run <package_name> publisher

ros2 run <package_name> subscriber
```

# TURTLESIM

#### Turtlesim is the Flagship example application for ROS and ROS 2. It demonstrates in simple but effective ways the basic concepts.

## Getting Start with the Turtle simulator
```bash
sudo apt update

sudo apt install ros-humble-turtlesim
```
#### Check that the package is installed:
```bash
ros2 pkg executables turtlesim
```
#### To start turtlesim, enter the following command in your terminal:
```bash
ros2 run turtlesim turtlesim_node
```
#### To move the turtle using keyboard
```bash
ros2 run turtlesim turtle_teleop_key
```
#### Inspect the Active Nodes
```bash
ros2 node list
```
#### Check Active Topics
```bash
ros2 topic list
```
# micro-ROS

#### Micro-ROS is a framework designed to enable the integration of small devices, such as microcontrollers and embedded systems, with the Robot Operating System (ROS) ecosystem. 
### Supported Hardware : https://micro.ros.org/docs/overview/hardware
## Installation
```bash
mkdir microros_ws
cd microros_ws

git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup

sudo apt update && rosdep update
rosdep install --from-paths src --ignore-src -y

sudo apt-get install python3-pip

colcon build
source install/local_setup.bash

```
#### make sure you are still inside the micro-ros ws

```bash
ros2 run micro_ros_setup build_firmware.sh
source install/local_setup.bash
```

```bash
ros2 run micro_ros_setup create_agent_ws.sh   
ros2 run micro_ros_setup build_agent.sh

source install/local_setup.bash

```

### Now, let’s give a dry run by running the micro-ROS agent by following the command:

```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0

```

### The result should show something like this:
![image](https://github.com/user-attachments/assets/4e75d483-c0e3-4dbe-ac07-aee6213da7e6)

This means the installation of the agent is successful. 
# Add yourself to  dialout group
### The dialout group in Linux is a user group that grants permissions to access serial devices, which include things like USB-to-serial converters and other serial communication ports (like your ESP32).
```bash
sudo usermod -aG dialout <your_username>

sudo reboot
```

## Installing Arduino
```bash
wget https://downloads.arduino.cc/arduino-ide_latest_Linux_64bit.tar.xz
tar -xf arduino-ide_latest_Linux_64bit.tar.xz
cd arduino-ide_*/
sudo ./install.sh

```

INstalllation 

```bash
sudo apt install snapd

sudo snap install arduino
```

#### after installing arduino , add the url in the prefernces
```bash
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json, http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

### then add esp32 board to library from board manager select version V2.0.2

## Then we need to add the micro-ros-arduino library

https://github.com/micro-ROS/micro_ros_arduino/tree/humble

### add this zip file to the library

### then select the example publisher code and upload to esp32!

## take a new terminal and build the micro-ros ws
```bash
cd micro_ros_ws/
colcon build
source install/setup.bash

ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0

```
### in a new terminal check the topic list
```bash
ros2 topic list
```

#### you should see this : "/micro_ros_arduino_node_publisher"

```bash
ros2 topic echo /micro_ros_arduino_node_publisher  
```

# Code for ESP32 - ultrasonic sensor Node

```bash

#include <micro_ros_arduino.h>
#include <stdio.h>
#include <rcl/rcl.h>
#include <rcl/error_handling.h>
#include <rclc/rclc.h>
#include <rclc/executor.h>
#include <std_msgs/msg/int32.h>

rcl_publisher_t publisher;
std_msgs__msg__Int32 msg;
rclc_executor_t executor;
rclc_support_t support;
rcl_allocator_t allocator;
rcl_node_t node;
rcl_timer_t timer;

#define LED_PIN 13
#define TRIG_PIN 12
#define ECHO_PIN 14

#define RCCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){error_loop();}}
#define RCSOFTCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){}}

void error_loop() {
  while(1) {
    digitalWrite(LED_PIN, !digitalRead(LED_PIN));
    delay(100);
  }
}

// Function to measure the distance using the ultrasonic sensor
long measure_distance() {
  // Ensure trigger pin is low
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  
  // Trigger the ultrasonic pulse
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Measure the pulse duration on the echo pin
  long duration = pulseIn(ECHO_PIN, HIGH);

  // Calculate the distance in cm (speed of sound is ~343 m/s)
  long distance_cm = duration * 0.034 / 2;

  return distance_cm;
}

void timer_callback(rcl_timer_t * timer, int64_t last_call_time) {  
  RCLC_UNUSED(last_call_time);
  if (timer != NULL) {
    // Measure the distance and update the message
    msg.data = measure_distance();
    RCSOFTCHECK(rcl_publish(&publisher, &msg, NULL));
  }
}

void setup() {
  set_microros_transports();
  
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, HIGH);
  
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  
  delay(2000);

  allocator = rcl_get_default_allocator();

  // Create init_options
  RCCHECK(rclc_support_init(&support, 0, NULL, &allocator));

  // Create node
  RCCHECK(rclc_node_init_default(&node, "ultrasonic_sensor_node", "", &support));

  // Create publisher
  RCCHECK(rclc_publisher_init_default(
    &publisher,
    &node,
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32),
    "ultrasonic_sensor_distance"));

  // Create timer (100 ms interval)
  const unsigned int timer_timeout = 100; // Adjusted to 100 ms
  RCCHECK(rclc_timer_init_default(
    &timer,
    &support,
    RCL_MS_TO_NS(timer_timeout),
    timer_callback));

  // Create executor
  RCCHECK(rclc_executor_init(&executor, &support.context, 1, &allocator));
  RCCHECK(rclc_executor_add_timer(&executor, &timer));

  msg.data = 0; // Initialize message data
}

void loop() {
  delay(10); // Reduced delay to minimize blocking
  RCSOFTCHECK(rclc_executor_spin_some(&executor, RCL_MS_TO_NS(10)));
}

```
## Command to list USB serial devices
```bash

ls -l /dev/ttyUSB* /dev/ttyACM*

```
# Code for ESP32 - ultrasonic sensor BASIC

```bash

#define TRIG_PIN  5   // Change pin numbers as needed
#define ECHO_PIN  18

void setup() {
  Serial.begin(115200);  // ESP32 default Serial speed is often 115200
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
}

void loop() {
  // Clear the trigger pin
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);

  // Send a 10 microsecond pulse to trigger pin
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Read the echo pin: how long was it HIGH?
  long duration = pulseIn(ECHO_PIN, HIGH);

  // Calculate the distance (speed of sound = 34300 cm/s)
  float distance_cm = (duration * 0.0343) / 2;

  Serial.print("Distance: ");
  Serial.print(distance_cm);
  Serial.println(" cm");

  delay(500);  // Wait half a second before next reading
}
```
