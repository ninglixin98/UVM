# UVM —— 第五章 TLM通信、单向双向多向、通信管道
## 5.1 TLM通信
### 5.1.1 概述
- TLM是一种基于事务（transaction）的通信方式，通常在高抽象级语言如SystemC或者SV/UVM中作为模块之间的通信方式
- TLM是从通信优化角度提出的一种抽象通信方式
- TLM通信需要两个通信的对象，这两个对象分别称为initiator和target
	- 谁先发起通信请求，谁就属于initiator
	- 谁作为发起通信的响应方，谁就属于target
- 注意通信的发起方并不代表transaction的流向起点，即不一定数据是从initiator流向target
- 按照transaction流向。我们可以将两个对象分为producer和consumer
	- 数据从哪产生，它就属于producer
	- 数据流向了哪里，它就属于consumer
- 有了两个参与通信的对象之后，用户需要将TLM通信方法在target一端实现，以便于initiator将来作为发起方可以调用target的通信方法，实现数据传输
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708100459.png)

	- monitor和scoreboard，monitor是发起端
	- generator和driver，driver是发起端

### 5.1.2 TLM通信

- TLM通信步骤可分为
	- 分辨出initiator和target，producer和consumer
	- 在target中实现TLM通信方法
	- 在两个对象中创建TLM端口
	- 在更高层次中将两个对象的端口进行连接

- 从数据流向来看，传输方向可以分为单向（unidirection）和双向（bidirection）
	- 单向传输：由initiator发起request transaction
	- 双向传输：由initiator发起request transaction，传送至target；而target在消化了request transaction后，会发起response transaction，继而返回给initiator
- 端口
	- port：经常作为initiator的发起端，initiator凭借port才可以访问target的TLM通信方法
	- export：作为initiator和target中间层次的端口
	- imp：只能作为target接收request的末端，它无法作为中间层次的端口，所以imp的连接无法再次延伸

- TLM端口分类
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708101813.png)
	
	- 端口不属于object类，不能使用工程机制

- 端口的定义
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708102000.png)

	- imp类型需要额外传递端口所在的类，用于查找相应的方法

- 端口的例化
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708102007.png)

	- 端口不是组件，不能使用create

- 端口的连接
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708102014.png)

	- 从initiator（左侧）连接到target（右侧）
- 端口的使用
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708102028.png)

		- port可以连接port,export,import，export可以连接export,import，import只能作为接收端，无法连接其他做扩展
		- 多个port可以连接同一个port，但port只能连接一个import
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708102838.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708102847.png)

## 5.2 单向通信
### 5.2.1 概述

- 从Initiato到target之间的数据流向是单一方向的，或者说initiator和target只能扮演produce和consumer中的一个角色

### 5.2.2 类型

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708103426.png)

- PORT代表了三种端口
	- port
	- export
	- imp
- 阻塞的方式
	- blocking：阻塞
		- 方法类型为 task，保证可以实现事件等待和延时
	- nonblocking：非阻塞
		- 方法类型为 function，保证方法调用可以立即返回
- 通信方法
	- 阻塞
		- put()
		- get()
		- peek()
	- 非阻塞
		- try_put()
		- can_put()
		- try_get()
		- can_get()
		- try_peek()
		- can_peek()

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708103439.png)


- initiator和target的端口成对，阻塞方式和通信方法必须保持一致

### 5.2.3 使用方法
- 实例
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708103449.png)

	- 端口不会自带put方法，而是调用port所连接的imp中实现的put方法

## 5.3 双向通信
### 5.3.1 概述
- 双向通信（bidirectional communication）的两端也分为initiator和target，但是数据流向在端对端之间是双向的
- 双向通信中的两端同时扮演着producer和consumer的角色，而initiator作为request发起方，在发起request之后，还会等待response返回

- 分类
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708104348.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708104404.png)

- transport双向通信方式，可以在同一方法调用过程中完成发送和返回，即一次传输既发送也返回
- master和slave双向通信方式，需要调用两个方法才能完成一次通信，即需要调用两次才能完成发送和返回
	- master为主动发送REQ数据，并获得返回值RSP
	- slave为接收REQ数据，并发送返回值RSP

### 5.3.2 transport通信方式
- 创建
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708104636.png)

- 例化
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708104647.png)

- run_phase中调用
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708104655.png)

- target组件中方法实现
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708104703.png)

- 高层次端口连接
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708104710.png)

## 5.4 多向通信
### 5.4.1 概述
- 指的是如果initiator和target之间的相同TLM端口数目超过一个时的处理解决办法，如当有多个put端口，方法命名会冲突
- 多向通信仍然是两个组件之间的通信
- UVM通过端口宏声明方式来解决这一问题
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111337.png)

		- SFX为后缀名

### 5.4.2 使用方法
- 使用宏
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111513.png)

- 定义端口
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111520.png)

		- port不需要区分，不加后缀

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111527.png)

		- 注意imp加后缀使用新定义的方法名

- 方法实现
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111534.png)

		- 方法名需要加后缀

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111541.png)

- 调用
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111547.png)

		- 注意port仍然调用的put，不需要加后缀

- 连接
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708111557.png)

## 5.5 通信管道
### 5.5.1 概述
- 在使用端口方法例如put和get时，能否使用预定义的方法，而不用自己去定义
- 能否解决一端连接到多端的问题
- 为了解决上述问题，可以使用几个TLM组件和端口
- TLM FIFO
	- analysis port
	- analysis TLM FIFO
	- request & response 通信管道

### 5.5.2 TLM FIFO
- uvm_tlm_fifo类是一个新组件，它继承于uvm_component类，而且预先内置了多个端口以及实现了多个对应方法供用户使用
- uvm_tlm_fifo自带fifo，可以缓存数据
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708112401.png)

- 创建
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708112412.png)

- 例化
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708112421.png)

- 存放数据
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708112427.png)

### 5.5.3 analysis port
- 有些情况需要满足一端到多端的需求
- 如果数据源端发生变化需要通知跟它关联的多个组件时，我们可以利用软件的设计模式之一观察者模式（observer pattern）来实现
	- observer pattern核心在于
		- 这是从一个initiator端到多个target端的方式
		- analysis port采取的是“push”模式，即从initiator端调用多个target端的write()函数来实现数据传输

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708112941.png)

	- analysis port可分为
		- uvm_analysis_port
		- uvm_analysis_export
		- uvm_analysis_imp

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708112948.png)

- 需要在target侧实现write()函数

### 5.5.4 analysis TLM FIFO
- uvm_tlm_analysis_fifo类继承于uvm_tlm_fifo，这表明它本身具有面向单一TLM端口的数据缓存特性，而同时该类又有一个uvm_analysis_imp端口analysis_export并实现了write()函数
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708113238.png)

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708113248.png)

### 5.5.5 request & response 通信管道
- uvm提供了两种简便的双向通信管道，他们作为数据缓存区域，既有TLM端口从外侧接收request和response，同时也有TLM端口供外侧获取request和response
- uvm_tlm_req_rsp_channel
	- 内部例化了两个mailbox分别用来存储request和response
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708113518.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708113534.png)

		- 利用两个端口

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708113543.png)

		- 利用一个端口

	- 上述两种连接方式都需要调用两次方法

- uvm_tlm_transport_channel
	- 继承于uvm_tlm_req_rsp_channel，并例化了transport端口
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708113850.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220708113900.png)

	- transport_export只调用一次，在一个通信周期内，initiator必须完成put和get才能结束
