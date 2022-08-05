# UVM —— 第七章 item、sequence、sequencer和driver
## 7.1 概述
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709164502.png)

- sequence会产生目标个数量的sequence item对象，并对其进行随机化
- 产生的sequence会经过sequencer，最终到达driver
- driver得到sequence item后，进行数据解析，然后将数据按照一定协议写入DUT
- 当有必要时，driver在解析并消化sequence item后，会将最后的状态写入sequence item并返回给sequencer，并最终到达sequence
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709164511.png)

- sequence item为互动的最小粒度
- sequence可以产生更小的sequence
- sequencer是组件，而sequence和sequence item是object
- 传输过程中，sequence item类型不能变化，必须保持一致
- driver是一直forever执行的，并且不能轻易修改item的值

- 继承关系
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709164526.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709164536.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709164546.png)

- 注意
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709165208.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709165217.png)

	> 数据传输采用get()模式

## 7.2 sequence和item
### 7.2.1 item

- item是基于uvm_object的类，这表明他具备uvm核心基类的数据操作方法，如copy()、clone()、compare()等
- item的数据成员类型
	- 控制类：如总线协议上的读写类型、数据长度、传送模式等
	- 负载类：一般指数据总线上的数据包
	- 配置类：用来控制driver的驱动行为，如driver的发送间隔和是否插入错误
	- 调试类：用来标记一些额外信息，方便调试，如对象的实例序号，创建时间，被driver解析的时间始末等
- item的使用特点
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709172211.png)

### 7.2.2 sequence
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709172226.png)

- 分类
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709172243.png)

	- flat sequence
		- 定义
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709173450.png)

		- 主体
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709173500.png)

		- 包含的信息
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709173508.png)

		- 可以使用弘来进行操作
			- `uvm_do
			- `uvm_do_with
			- `uvm_create

		- 建议
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709173518.png)

			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709173529.png)

	- hierarchical sequence
		- 可以嵌套sequence
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174007.png)

			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174016.png)

## 7.3 sequencer和driver

### 7.3.1 概述
- driver是一个永动机，一直执行
- 数据的传输使用get()模式，即driver作为Initiato，通过get()获取item

### 7.3.2 端口和方法

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174815.png)

- 为了便于item传输，uvm定于了匹配的TLM端口，以供sequencer和driver使用
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174826.png)

- driver一侧例化了以下端口
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174834.png)

		- 一般使用第一种
		- 是与下面sequencer的端口成对使用的

- 在sequencer一侧例化了
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174842.png)

- 连接
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174852.png)

- 这一对端口支持的方法
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709174904.png)

- req和rsp的类型必须保持一致，由于uvm_sequencer和uvm_driver都是参数类，可以传递具体的req类型，默认为uvm_sequence_item
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709175236.png)

	- 如果传递的item是子类，但是却使用默认的uvm_sequence_item类型，需要在driver进行动态类型转换

- driver在消化完item后，可以使用item_done(input RSP rsp_arg = null)来通知sequence传输已经结束，可以发送返回值rsp，也可以不发送
- driver可以通过另外的put_response()和put()来单独发送rsp

### 7.3.3 实例
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709175455.png)

	- create_item：创建item对象，并且指明item被挂载到具体的sequencer上，m_sequencer为uvm_sequencer的变量，当发生挂载后，m_sequencer就会变为具体挂载的句柄的名称
	- start_item：开始将item放入sequencer，以等待get_next_item()仲裁
	- finish_item：已经放入，等待仲裁
	- get_response：获取返回值rsp，拿到的为父类句柄
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709175508.png)

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709175523.png)

	- seq_item_port.get_next_item()：获取Item
	- req,clone()
		- 克隆不会复制sequence_id
		- 调用克隆返回的是uvm_object句柄，需要进行转换

	- rsp.set_sequence_id(req.get_sequence_id())：需要手动复制id
	- seq_item_port.item_done(rsp)：返回完成信号
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709175535.png)

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709180105.png)

	- 进行端口连接

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709180114.png)

	- seq.start(e.sqr)
		- 将sequence和sequencer进行挂载
		- 挂载后会自动执行body

### 7.3.4 通信时序
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220709180125.png)
