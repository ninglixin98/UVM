# UVM —— 第四章 结构、顶层方案、环境元素
## 4.1 UVM结构
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707181040.png)

- uvm_top
	- uvm_top是uvm_root的唯一实例
		- 他由uvm创建和管理
		- 他所在的域是uvm_pkg

	- uvm_top是所有test组件的顶层
		- 所有验证组件在创建时都需要指明他的父一级
		- 如果某些组件父一级为null，他将直接隶属于uvm_top

	- uvm_top提供了一系列的方法来控制仿真，如phase机制、objection防止仿真推出机制等

- uvm_test
	- test类是用户自定义的顶层结构
	- 所有test类都应该继承于uvm_test，否则uvm_top将不识别
	- test的目标包括
		- 提供不同的配置，包括环境结构配置，测试模式配置等，然后再创建验证环境
		- 例化测试序列，并且挂载到目标sequencer，使其命令driver发送激励

	- 验证策略：可以将需要配置的变量全部放在test里，从而方便管理

- 验证环境env
	- 没有额外的功能
	- 用来为验证环境提供一个容器

- 组件uvm_component
	- uvm_component是一个虚类，不能被例化，只能被继承
	- 提供：
		- 结构：get_full_name();
		- 阶段机制（phase）：build_phase();
		- 配置（configuration）：print_config()
		- 报告机制(report)：report_hook()
		- 事务记录：record()
		- 工厂机制（factoru）：set_inst_override()

## 4.2 MCDF顶层验证方案

### 4.2.1 模块验证方案
- reg_env
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707204800.png)

	- 写
		- 通过reg_master_agent中的driver对寄存器的addr、data、cmd进行数据写入，并且monitor监测写入的数据，送入scoreboard中
		- 通过reg_slave_agent中的monitor拿到寄存器的addr、data、cmd，送入scoreboard中进行对比
		- 写数据时，reg_master_agent主动输入寄存器的配置数据，reg_slave_agent被动监测寄存器的输出
	- 读
		- 通过reg_slave_agent中的driver对寄存器中fifo的余量进行写入，monitor监测到写入的数据，送入scoreboard中比较
		- 通过reg_master_agent中的monitor监测到寄存器的数据，送入scoreboard中进行比较

- chnl_env
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707205122.png)

- arb_env
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707205138.png)

- fmt_env
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707205148.png)

### 4.2.2 子系统验证方案
- 方案一
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707205343.png)

- 方案二
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707205355.png)

### 4.2.3 UVM相比SV的优点
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707205406.png)


## 4.3 模块验证环境的的内经
### 4.3.1 环境构建的四要素
- 1、单元组件的自闭性
	- 自闭性指的是单元组件（例如uvm_agent或者uvm_env）自身可以成为独立行为、不依赖其他并行组件
		- 如driver同sequencer之间，原先SV中无法独立使用，因为有连接两者的mailbox存在。而UVM中由于通过TLM连接，使得可以独立例化
	- 这种单元组件的自闭行为日后的组件复用提供了良好的基础

- 2、回归创建
	- 环境的框架建立主要就依靠这一技能了
	- 上一级的组件在例化后，会执行各个phase阶段，通过build_phase进一步创建子组件
- 3、通信端口连接
- 4、顶层配置
	- 由于单元组件的自闭性，UVM不建议引用子环境句柄，进而索引更深层次的变量进行顶层配置
	- 更好的方式是通过配置化对象，作为绑定于顶层环境的部分传递到子环境
	- 顶层配置对象可以在子环境还没创建时，就将配置对象提前存储起来，无需考虑顶层配置对象会先于子环境生成，这为UVM用户提供了安全的配置方式

### 4.3.2 环境元素分类
- 成员变量
	- 一般变量：用于对象内部的操作，或者为外部访问提供状态值
	- 结构变量：用来决定内部子组件是否需要创建和连接，例如顶层的is_active变量，一般由int或者enum来定义
	- 模式变量：用来控制组件行为，例如driver变量经过模式配置，可以在run phase做出不同的激励行为，一般由int或者enum来定义
- 子组件
	- 固件组件：环境必须创建的组件，称作固定组件，例如agent中的monitor
	- 条件组件：通过结构变量的配置来决定是否需要创建
	- 引用组件：是内部声明一个类型句柄，同时通过自定向下的传递，使得该句柄可以指向外部的一个对象
- 子对象
	- 自生对象：在某一层次中首先会创建一个对象，该对象可称为自生对象
	- 克隆对象：对象传递过程中，该对象经过克隆从而生成一个成员数据相同的对象
	- 引用对象：如果对象经过了端口传递，到达另一个组件，而该组件对其未经过克隆直接进行操作，可称为引用对象的操作
