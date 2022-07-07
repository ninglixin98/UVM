# UVM —— 第一章 方法学、类库、工厂和覆盖
## 1.1 验证方法学概述
- 2010年发布UVM1.0版本
- 目前最新为UVM1.2版本

## 1.2 类库地图

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/20201220224130306.png)

1. 核心基类：提供底层支持
2. 工厂（factory）类：工厂（factory）类
3. 事务（transaction）和序列（sequence）类：用来发送激励
4. 结构创建（structure creation）类：构成验证环境
5. 环境组件（environment component）类：环境的各个组件
6. 通信管道（channel）类
7. 信息报告（message report）类
8. 寄存器模型（register model）类
9. 线程同步（thread synchronization）类
10. 事务接口（transaction interface）类

## 1.3 工厂机制
### 1.3.1 工厂的意义
- UVM工厂的存在就是为了更方便地替换验证环境中的实例或者注册了的类型，同时工厂的注册机制也带来了配置的灵活性
- 这里的实例或者类型替代，在UVM称作覆盖（override），而被用来替换的对象或者类型，应该满足注册和多态要求
- UVM的验证环境构成可以分为两部分，一部分构成了环境层次，这部分代码是通过uvm_component类完成，另一部分构成了环境的属性和数据传输，这部分通过uvm_object类完成
- 工厂可以帮我们创建对象，而要进行创建对象，需要先将类型进行注册
- 验证组件由uvm_component表示，而transaction数据包由uvm_object表示

### 1.3.2 创建（create）

- 运用工厂的步骤
- 将类注册到工厂，使用注册弘
	- `uvm_component_utils()
	- `uvm_object_utils()
- 在例化前设置覆盖对象类型（可选）
- 对象创建
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707104132.png)

### 1.3.3 uvm_coreservice_t 类
- 该类内置了UVM世界核心的组件和方法，他们主要包括
	- 唯一的uvm_factory，该组件用来注册、覆盖和例化
	- 全局的report_server，该组件用来做消息统筹和报告
	- 全局的tr_database，该组件用来记录transaction记录
	- get_root() 方法用来返回当前uvm环境的结构顶层对象

- 该类并不是uvm_component或者uvm_object，他也并没有例化在UVM环境中，而是独立于UVM环境之外的
- uvm_coreservice_t 只会被UVM系统在仿真开始时例化一次。用户无需，也不应该自行再额外例化该核心服务组件

### 1.3.4 工厂创建的方法
- 类型::type_id::create()
- create_compoent_by_name()
- create_compoent_by_type()
- create_object_by_name()
- create_object_by_type()

### 1.3.5 component/object创建方法
- create()
- create_component()
- get()
- get_type_name()
- set_inst_override()
- set_type_override()

## 1.4 覆盖（override）

- 覆盖机制可以将其原来所属的类型替换为另一个新的类型
- 在覆盖之后，原本用来创建原属类型的请求，将由工厂来创建新的替换类型
	- 无需修改原始代码，继而保证了原代码的封装性
	- 新的替换类型必须与被替换类型相兼容，否则稍后的句柄赋值会失败，因此要求新的类型必须继承原类型

- 做顶层修改时，非常方便
- 想要实现覆盖，原有类型和新类型都需要注册
- 使用工厂的create()来创建对象，工厂会检查是否原有类型被覆盖
- 确保正确覆盖代码的要求，只有使用范式创建的类才会发生覆盖
- 覆盖时，可以使用“类型覆盖”或“实例覆盖”
	- 类型覆盖：UVM层次结构下的所有原有类型都被覆盖类型所替换，即默认全部替换掉
	- 实例覆盖：在某些位置中的原有类型会被覆盖类型所替换，即可以指定具体需要替换的位置
- 覆盖方法
	- 类型覆盖：set_type_override()
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707110041.png)

		- overeide_type 为新的类型，是类在工厂注册时的句柄，可以使用 new_type::get_type() 找到
		- bit replace
			- 1：如果已经有覆盖存在，那么新的覆盖会替换旧的覆盖
			- 0：如果已经有覆盖存在，那么该覆盖将不会生效

		- set_type_override() 是一个静态函数，即可以在任何地方调用他
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707110052.png)

			- orig_type：原有类型

	- 实例覆盖：set_inst_override()
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707111012.png)

		- string inst_path 指向的是组件结构的路径字符串
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707111029.png)

		- set_inst_override()是一个静态函数，可以在其他地方调用
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707111038.png)

		- 路径
			- 最顶层为root，下面为test，下面为env，env中有checker
			- "root.test.env.checker"


	- 实例
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707111514.png)

- 覆盖时要注意
	- 父类的方法中添加virtual
	- 覆盖必须发生在类型创建之前
	- 覆盖采用parent wins模式，即层次越高，覆盖的优先级越高，当有两个地方同时对进行覆盖时，层次最高的起作用



