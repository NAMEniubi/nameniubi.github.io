---
title: ros-topic通讯方式
description: 这是一个副标题
date: 2021-06-07 18:01:10 +0800
slug: ros
image: the-creative-exchange-unsplash.jpg
categories:
    - 代码
tags:
    - ros
---


[TOC]
## Topic

#### Topic通讯概念


多个Node节点都需要到ROS Master进行注册。

每个Node完成自己的功能逻辑。有的时候Node和Node间需要有数据的传递，这个时候ROS提供了一种数据通讯机制。

![rostopic2](/images/posts/rostopic2.png)

Node间进行通讯，其中发送消息的一方，ROS将其定义为**Publisher(发布者)，将接收消息的一方定义为**Subscriber(订阅者)。考虑到消息需要广泛传播，ROS没有将其设计为点对点的单一传递，而是由**Publisher**将信息发布到**Topic(主题)**中，想要获得消息的任何一方都可以到这个**Topic**中去取数据。我们理解Topic的时候，可以认为**Topic**相当于一个聚宝盆，东西放进去后，不管同时有多少人来取，都可以拿到数据。


#### Topic通讯实现

C++实现Publisher

```cpp

#include "ros/ros.h"
#include <iostream>
#include "std_msgs/String.h"

using namespace std;

int main(int argc, char **argv) {

    string nodeName = "cpppublisher";
    string topicName = "cpptopic";

    //初始化ros节点
    ros::init(argc, argv, nodeName);
    ros::NodeHandle node;

    //通过节点创建publisher
    const ros::Publisher &publisher = node.advertise<std_msgs::String>(topicName, 1000);

    //按照一定周期，发布消息
    int index = 0;
    ros::Rate rate(1);
    while (ros::ok()) {
        std_msgs::String msg;

        msg.data = "hello pub " + to_string(index);
        publisher.publish(msg);

        cout << msg.data << endl;

        rate.sleep();
        index++;
    }
    return 0;
}
```
C++实现Subscriber
> topicCallback为收到订阅消息时的回调
在此处，必须要用ros::Subscriber进行接收。 否则系统会回收对象。

```cpp
#include "ros/ros.h"
#include <iostream>
#include "std_msgs/String.h"

using namespace std;

void topicCallback(const std_msgs::String::ConstPtr &msg) {
    cout << (*msg).data << endl;
}

int main(int argc, char **argv) {
    string nodeName = "cppsubscriber";
    string topicName = "cpptopic";

    //初始化节点
    ros::init(argc, argv, nodeName);
    ros::NodeHandle node;
    //创建订阅者
    const ros::Subscriber &subscriber = node.subscribe(topicName, 1000, topicCallback);
    // 阻塞线程
    ros::spin();
    return 0;
}
```

python实现Publisher
> 在编程过程中，必须加上以上两行代码。
#!/usr/bin/env python 用来表示当前为python脚本，如果不加，系统会默认为bash脚本
#coding:utf-8 提供中文支持


```python
#!/usr/bin/env python
#coding:utf-8

import rospy
from std_msgs.msg import String

if __name__ == '__main__':
    nodeName = "pypublisher"
    topicName = "pytopic"
    # 初始化节点
    rospy.init_node(nodeName)

    # 创建发布者
    publisher = rospy.Publisher(topicName, String, queue_size=1000)

    rate = rospy.Rate(10)
    count = 0
    while not rospy.is_shutdown():
        # 发布消息
        publisher.publish("hello %d" % count)
        rate.sleep()
        count += 1;
```
python实现subscriber

```
#!/usr/bin/env python
# coding:utf-8
import rospy
from std_msgs.msg import String

def topicCallback(msg):
    print msg

if __name__ == '__main__':
    nodeName = "pysubscriber"
    topicName = "pytopic"
    # 初始化节点
    rospy.init_node(nodeName)
    # 创建订阅者
    rospy.Subscriber(topicName, String, topicCallback)
    # 阻塞线程
    rospy.spin()
```
> topic调试工具
rosrun rqt_topic rqt_topic
publisher调试工具
rosrun rqt_publisher rqt_publisher

#### Topic自定义消息
 1. 在pakage目录下新建<span class='highlight'>msg</span>目录
 2. 新建msg文件¶
创建的这个Student.msg文件就是自定义消息文件，需要去描述消息的格式。
我们可以编辑代码如下
string name
int64 age
这个自定义的消息包含两个数据形式，name和age，name的类型 是string，age的类型是int64。
这个msg文件其实遵循一定规范的，每一行表示一种数据。前面是类型，后面是名称。
ros不只是提供了int64和string两种基本类型供我们描述，其实还有很多，具体可以参考附录的内容 
 3. 配置package.xml文件
 在package.xml种添加如下配置:
	><build_depend>message_generation</build_depend>
	<exec_depend>message_runtime</exec_depend>

	message_generation是消息生成工具，在打包编译时会用到
	message_runtime运行时消息处理工具
 4. 配置CMakeLists.txt
find_package配置👇
在<span class='highlight'>find_package</span>添加<span class='highlight'>message_generation</span>，结果如下:
	```cpp
	find_package(catkin REQUIRED COMPONENTS
	  roscpp
 	  rosmsg
	  rospy
  	message_generation
	)
	```
	添加<span class='highlight'>add_message_file</span>，结果如下:👇

	```cpp
	add_message_files(
        	FILES
        	Student.msg
	)
	```
	这里的Student.msg要和你创建的msg文件名称一致，且必须时在msg目录下，否则编译会出现问题

	generation_msg配置👇
	```
	generate_messages(
        DEPENDENCIES
        std_msgs
	)
	```
	这个配置的作用是添加生成消息的依赖，默认的时候要添加std_msgs
	catkin_package配置👇
	```
	catkin_package(
	#  INCLUDE_DIRS include
	#  LIBRARIES demo_msg
	   CATKIN_DEPENDS roscpp rosmsg rospy message_runtime
	#  DEPENDS system_lib
	)
	```
	为catkin编译提供了依赖message_runtime


 5. 检验自定义消息
 编译项目
来到工作空间目录下，运行编译
catkin_make
 查看生成的消息文件
c++的头文件
来到devel的include目录下，如果生成了头文件说明，自定义消息创建成功。
python的py文件
来到devel的lib/python2.7/dist-package目录下，查看是否生成和package名称相同的目录，以及目录内是否生成对应的py文件。
 通过rosmsg工具校验
rosmsg show demo_msgs/Student
查看运行结果，运行结果和自己定义的相一致，说明成功。

#### 使用自定义消息
 在开发过程种，自定义消息是以一个package存在的，其他package需要用到这个自定义消息的package，是需要添加依赖的。
package.xml依赖配置👇
来到package.xml中，添加如下:
><build_depend>demo_msgs</build_depend>
<build_export_depend>demo_msgs</build_export_depend>
<exec_depend>demo_msgs</exec_depend>

CMakeLists.txt依赖配置👇
来到CMakeLists.txt文件中，找到find_package，添加demo_msgs自定义消息依赖，添加结果如下：

```
find_package(catkin REQUIRED COMPONENTS
roscpp
rosmsg
rospy
demo_msgs
)
```
