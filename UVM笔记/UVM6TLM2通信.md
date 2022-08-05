# UVM —— 第六章 TLM2通信、同步通信原件
## 6.1 TLM2通信
### 6.1.1 概述
- 之前介绍的UVM各个组件间的通信是通过TLM1.0方式实现的，而随着SystemC模型的广泛使用，SystemC通信机制TLM2.0也引起了UVM标准委员会的兴趣
- 与TLM1.0相比，TLM2.0提供了更丰富更强大的传输特性
	- 双向的阻塞或非阻塞接口
	- 时间标记
	- 统一的数据包

### 6.1.2 接口实现
- TLM2.0的传输是双向的，意味着在一次完整传输中有request和response类型
- TLM2.0支持blocking和nonblocking两种传输方式
	- blocking
		- 在一次传输中完成request和response的传输
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708195043.png)

	- nonblocking
		- 将request和response的传输分为两个独立的单向传输，而两次传输整体视为完成一次握手传输
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708195053.png)

			- fw代表把数据送出去，类似put
			- bw代表把数据拿回来
			- T代表着统一的数据类uvm_tlm_generic_payload，而P代表着在nonblocking传输中做状态同步的类型

- TLM2.0也有port、export、imp类型端口

- 为了区别于1.0，TLM2.0将端口类型称之为socket。他们是由port、export、imp组合而成的
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708195113.png)

### 6.1.3 传送数据
- 传送数据的统一格式是按照总线数据的内容来定义的
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708195618.png)

	- 如果需要传输其他的数据内容，有两种方法
		- 将其合并作为数据成员data数组中的一部分
		- 创建新的uvm_tlm_extension类，将额外的数据成员装入到该数据延伸对象中，通过uvm_tlm_generic_payload::set_extension(uvm_tlm_extension_base ext)来添加这一部分数据
			- uvm_tlm_extension类提供了do_copy()、do_compare()、do_print()等回调函数

### 6.1.4 时间标记
- 在TLM2传输中，由于可以标定延迟时间，这使得target端可以模拟延迟，并在准确的延迟时刻做出响应


## 6.2 同步通信原件
### 6.2.1 概述
- SV用来做线程同步的几种原件为semaphore、event、mailbox
- UVM中线程同步不再局限于在同一个对象中，而是扩大到组件和组件之间通信
- 为了解决封闭性问题，UVM定义了以下的类来满足组件之间的同步要求
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708200234.png)

- 回调函数在UVM中也被封装为一个类uvm_callback
- uvm_event解决了uvm_object和uvm_component之间的同步和通信，因为object无法例化端口进行TLM通信
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708200253.png)

### 6.2.2 uvm_event
- 与SV中event相比
	- event被->触发后，会触发使用@等待该事件的对象；uvm_event通过trigger()来触发，会触发使用wait_trigger()等待该事件的对象
	- 如果要再次等待事件触发，event只需要再次使用->来触发，而uvm_event需要先通过reset()方法重置初始状态，再使用trigger()来触发
	- event无法携带更多的信息，而uvm_event可以通过trigger(T data = null)的可选参数，将伴随触发的数据对象都写入到该触发事件中，而等待事件的对象可以通过方法wait_trigger_data(output T data)来获取数据
	- event触发时无法直接触发回调函数，而uvm_event可以通过add_callback(uvm_event_callback cb, bit append = 1)函数来添加回调函数
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709155638.png)

		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709155656.png)

			- pre_trigger()有返回值，返回为0时继续执行，返回为1时则不会被trigger,也不会执行后续的post_trigger()
			- 可以通过wait_ptrigger()和wait_ptrigger_data()来查找当前的状态，如果以经触发过，则不会阻塞，即电平触发

	- event无法直接获取等待他的进程数目，而uvm_event可以通过get_num_waiters()来获取等待他的进程数目

- 不同组件可以共享同一个uvm_event，这不需要跨层次传递句柄来实现共享，从而符合封闭的原则。这种共享是通过uvm_event_pool这一全局资源池来实现的。
	- 该资源池类是uvm_object_string_pool#(T)的子类，它可以生成和获取通过字符串来索引的uvm_event对象
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160059.png)

- 例化
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160114.png)
	- get_global会查找资源池中是否有事件e1,有则直接拿到，没有则会创建一个

### 6.2.3 uvm_barrier
- uvm提供了一个新的类uvm_barrier来进行同步和通信，也定义了uvm_barrier_pool来全局管理这些uvm_barrier对象，uvm_barrier_pool同样继承于uvm_object_string_pool
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160126.png)

- uvm_barrier可以设置一定的等待阈值（threshold），当有不少于该阈值的进程在等待对象时，才会触发该事件，同时激活所有等待的进程，使其可以继续进行
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160547.png)

- 例化
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160555.png)

- 等待
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160603.png)

### 6.2.4 uvm_callback
- uvm_callback的使用
	- 定义对应的callback和方法
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160903.png)

	- 要将component和callback关联起来时，需要在component里注册
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160912.png)

	- 在run_phase中插入callback
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160919.png)

	- 例化callback
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160925.png)

	- 在上层添加callback
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709160932.png)

			- c1为当前组件实例，m_cb1为callback的实例
			- 虽然comp1没有和cb2进行绑定，但是cb2继承于cb1，因此也可以插入

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709161304.png)

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709161312.png)

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709161321.png)
