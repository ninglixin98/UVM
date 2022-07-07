# UVM —— 第三章 组件家族（uvm_component）
## 3.1 概述
- SV环境中的验证组件按照功能需要，被称之为激励器（stimulator），检测器(monitor)和检查器（checker）。这三个核心组件与验证环境的三个关键特性相对应
- UVM组件家族就是从UVM基类继承的一个核心分支，即uvm_component
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707162153.png)

## 3.2 uvm_driver

- 该类会从uvm_sequencer中获取事务（transaction），经过转化进而在接口中对DUT进行时序激励
- 定义
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707162828.png)

	- 是参数类

- uvm_driver在uvm_component的基础上没有扩展新的函数，而是扩展了一些用于通讯的端口和变量
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707163028.png)

- driver和sequencer类之间的通信就是为了获取新的事务对象，这一操作是通过pull方式实现的
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707163036.png)

## 3.3 uvm_monitor
- 这个类是为了监测接口数据，而任何需要用户自定义数据监测行为的monitor都应该继承该类
- uvm_monitor与它的父类相比，没有添加任何新的方法和变量
- 功能包括
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707163422.png)

- 监测数据例子：
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707163442.png)

## 3.4 uvm_sequencer
- uvm_sequencer就如同一个管道，传递uvm_sequence发送的连续激励事务，并最终通过TLM端口送至driver一侧
- uvm_sequencer也可以从driver获取RSP来检查数据通道是否正常
- uvm_sequencer同样是一个参数类，需要在定义时声明REQ类型
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707164137.png)

- 定义
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707164154.png)

## 3.5 uvm_agent
- uvm_agent是一个标准单位，它通常包含一个driver、一个monitor以及一个sequencer
- 为了复用，有些时候uvm_agent只需要例化一个monitor，而不需要例化driver和sequence
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707164425.png)

	- is_active
		- UVM_ACTIVE：active模式需要发送激励，必须全部例化三个组件
		- UVM_PASSIVE：passive模式只监测，不发送数据，因此只需要例化monitor

- 按照总线的传输方向划分，可分为
	- master agent：用来向DUT发送transaction
	- slave agent：用来接收相应DUT的events
- 实例：
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707164437.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707164449.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707164457.png)

## 3.6 uvm_scoreboard
- uvm_scoreboard担任着同SV中checker一样的功能，即进行数据对比和报告
- uvm_scoreboard相比父类没有添加任何额外的成员变量和方法
- UVM除了uvm_scoreboard还有其他两个类进行数据比较
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707171000.png)

		- 是一个参数类，并且有两个端口before_export和after_export，可以分别写入期望的数据和实际监测到的数据，只能进行简单比较

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707171009.png)

		- 也是一个参数类，可以比较不同类型的参数，并可设置一个类似参考模型的转换类型TRANSFORMER

- 在uvm_scoreboard中通常要声明TLM端口以供monitor传输数据
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707171019.png)

	- 也可以不用自定义类，直接声明一个m_comp的句柄，但这样实例化时需要带上类型参数，如 m_comp = uvm_in_order_comparator#(bus_xact)::type_id
	- comparator有两个作用
		- 缓存数据
		- 数据比较

## 3.7 uvm_env
- 从环境层次结构而言，uvm_env包含多个agent和其他component，共同组成完整的验证环境
- uvm_agent不能嵌套uvm_agent，它的上级必须是uvm_env
- uvm_env可以被其他的uvm_env嵌套
- 实例
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707171814.png)

## 3.8 uvm_test
- uvm_test本身没有新的成员，它和SV中一样，决定环境的结构和连接关系，也决定着使用哪个虚类
- 在uvm_test中，可以先例化config_object类，将配置参数传递到组件中，再做子一级组件的例化
- 实例
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707172000.png)


