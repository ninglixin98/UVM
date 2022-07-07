# UVM —— 第二章 核心基类、阶段、配置、消息机制
## 2.1 核心基类（uvm_object）
### 2.1.1 域的自动化（field automation）
- UVM通过域的自动化，使得用户在注册UVM类的同时也可以声明今后会参与到对象拷贝、克隆、打印等操作的成员变量
- 域的自动化使得使用方法时非常便捷，无需自定义方法
- 在注册时，使用\`uvm_{component, object}\_utils_begin和\`uvm_{component, object}_utils_end 包裹起来实现域的自动化
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707140415.png)
	
	- 其中的类型宏可以在红宝书275页查到
	- 括号里面最后为数据操作方法，小白默认使用UVM_ALL_ON和UVM_DEFAULT即可

### 2.1.2 核心方法
- 拷贝（copy）
	- UVM中区分拷贝和克隆，前者是自己已经创建新对象，只需要进行拷贝，后者会自动帮你创建新对象，并进行拷贝，最后返回target object句柄
	- do_copy()是copy()的回调函数，执行完copy()后会自动执行do_copy()
- 比较（compare）
	- 默认的比较强在出现比较不同时会立即结束，而不会比较完全
	- 返回0或1
- 打印（print）
	- 默认使用uvm_default_printer
	- 也可以通过uvm_default_print = uvm_default_line_printer来赋予不同的打印机
- 打包和解包（pack&unpack）
	- pack是为了将自动化声明后的域打包为比特流
	- unpack是将串行的数据包解包为原有的各自域

## 2.2 phase机制
- UVM在验证环境构建时，引入了phase机制，通过该机制可以将UVM仿真阶段层次化
- 只有component组件能够使用phase机制
- 9种phase及执行顺序为
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707151331.png)

	- 执行顺序由上至下
	- 其中只有run为任务类型，要消耗时间，run_phase可细分为
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707151355.png)

		- 与run_phase是并行执行的，只有run_phase和这些分支全部执行结束，才会进入下一阶段

- UVM编译和运行顺序
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707151755.png)

- UVM仿真开始
	- 通过全局 run_test() 传递参数来指定要运行哪一个uvm_test
	- 也可以不传参数给run_test()，用户可以在仿真时通过 +UVM_TESTNAME=<test_name>，来指定仿真时调用的


- UVM世界的诞生
	- 在uvm_pkg中，有且只有一个顶层类uvm_root所例化的对象，即uvm_top
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707152028.png)

	- 通过uvm_top调用方法run_test(test_name)，uvm_top做了以下初始化
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707152040.png)

- UVM仿真结束
	- uvm_objection 类提供了一种所有component和sequence共享的计数器。如果有组件来挂起（raise）objection，那么它还应该落下objection
	- 必须有挂起，并且放在run_phase的第一行，否则run_phase会立即退出
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707152541.png)

## 2.3 config机制
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707154741.png)
- 在UVM中提供了uvm_config_db配置类以及几种方便的变量设置方法来实现仿真时的环境控制，常见的uvm_config_db类的使用方式包括：
	- 传递virtual interface到环境中
	- 设置单一变量值，例如int、string、enum等
	- 传递配置对象（config object）到环境

- 传递virtual interface到环境中
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707153021.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707153034.png)

	- 注意：
		- 接口传递应该发生在run_test()之前。这保证了在进入build phase之前，virtual interface已经被传递到uvm_config_db
		- 用户应当把interface与virtual interface的声明区分开来，在传递过程中的类型应当为virtual interface，即实际接口的句柄
		- 传递时，必须保证地址值一致

- 设置单一变量值，例如int、string、enum等
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707153445.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707153455.png)

- 传递配置对象（config object）到环境
	- 如果配置的变量较多，可以将每个组件中的变量加以整合，放置到uvm_object中，再对中心化的配置对象进行传递，将会对整体环境的维护更有利
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707153805.png)

		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707153814.png)

		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707153825.png)

		- set和get的句柄类型必须严格一致

> 建议
> - 在使用set()/get()方法时，传递的参数上下应保持一致，对于uvm_object等实例的传递，如果get和set的类型不一致，应当先通过$cast()完成类型转换
> - set()/get()方法传递的参数可以通过通配符“*”来表示任意层次，类似于正则表达式的用法。同时用户需要懂得“*.comp1”与“*comp1”的区别，前者表示再目前层次以下所有名称为“comp1”的组件，而后者表示包括当前层次以及当前层次以下所有名为“comp1”的组件
> - 在module环境中如果使用uvm_config_db::set()，则传递的第一个参数uvm_component cntxt参数一般表示当前层次。如果当前层次为最高层，用户可以设置为null，也可以设置为uvm_root::get()来表示uvm_root的全局顶层实例
> - 在使用配置变量时，应当确保先进行uvm_config_db::get()操作，再获得正确的配置值后再使用
> - 应当确保uvm_config_db::set()方法在相关配置组件创建前调用
> - 先做set再做创建，先get再使用

## 2.4 消息管理
### 2.4.1 消息方法
- 在uvm环境中或环境外，只要有引入uvm_pkg，均可以通过下面的方法来按照消息的严重级别和冗余度来打印消息
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707155155.png)

	- 严重级别
		- info
		- warning
		- error
		- fatal

	- 冗余度（verbosity）
		- UVM_NONE：最重要
		- UVM_LOW
		- UVM_MEDIUM
		- UVM_HIGH
		- FULL, DEBUG：最不重要

	- 设置冗余度过滤等级
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707155733.png)

		- 只有low和跟高等级的none才能被打印出来

- 每一条消息对应的是如何处理这些消息,而不同严重级别的消息，用户可以使用默认的消息处理方方式
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707155935.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707155946.png)

### 2.4.2 消息宏
- 如果要进行自定义的消息处理方式，uvm_report_object类提供了一些方法，同时，UVM也提供了一些宏对应这些消息方法
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707155958.png)

### 2.4.3 消息机制
- 消息处理是由uvm_report_handle类完成的
- 用户还可通过 set_max_quit_count来修改UVM_COUNT的值（默认为1），从而修改允许UVM_ERROR出现的次数

### 2.4.4 回调函数
- uvm_report_object类提供了下面的一下回调函数
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707160528.png)

	- 设置只有ERROR才能调用回调函数
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707160537.png)

	- report_hook：report的回调函数，一般必须定义
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707160546.png)

	- report_error_hook：子级error的回调函数
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220707160555.png)
