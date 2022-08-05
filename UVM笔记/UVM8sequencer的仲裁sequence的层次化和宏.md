# UVM —— 第八章 sequencer的仲裁、sequence的层次化和发送宏
## 8.1 sequencer和sequence
### 8.1.1 seq和item发送方法和建议
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172133.png)

- 发送seq和item时挂载方法不同
	- 发送seq
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172155.png)

		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172205.png)

	- 发送item
		- start_item()
		- finsih_item()
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172451.png)

		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172459.png)

- 发送步骤
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172513.png)

	- 执行顺序
		- sequencer
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172526.png)

			- 不建议使用回调函数

		- item
			- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710172535.png)

### 8.1.2 发送相关的宏
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710173847.png)

- 只有sequence能调用这些宏

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710173857.png)

- on表示指定将uvm_sequence挂载到对应的sequencer上
- prio表示优先级
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710173905.png)

- 实例
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710173917.png)

- 建议
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710173926.png)

### 8.1.3 sequencer仲裁特性

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710174342.png)

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710174353.png)

- 一般不自定义仲裁方法


### 8.1.4 sequencer锁定机制

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710174405.png)

- lock和unlock
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710174417.png)

	- 其他seq已经获得授权时，lock和grab都无法锁定
	- lock需要关心优先级

- grab和ungrab
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710174425.png)

	- grab不关心优先级，仲裁时直接拿到权限

## 8.2 sequence的层次化
### 8.2.1 概述
- 水平复用是指，如何利用已有资源，完成高效的激励场景创建
- 垂直复用是指，可以完成结构复用和激励复用两个方面
### 8.2.2 hierarchical sequence
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710175417.png)

- hierarchical sequence可以创建底层的sequence，并把这些sequence挂载到同一个sequencer上

### 8.2.3 virtual sequence

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710175439.png)

- virtual sequence和virtual sequencer
	- virtual_sequencer起到一个全局统筹的作用，将各个组件的sequencer句柄放入进来，从而可以找到各个组件
	- virtual sequence可以创建底层的sequence，并把这些sequence挂载到不同的sequence上
	- 将virtual sequence挂载到virtual sequencer上，就可通过virtual sequencer把不同的sequence挂载到不同的sequencer上
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710175452.png)

- 实例
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710175504.png)

		- virtual sequence需使用`uvm_do_on()来指明挂载的sequencer
		- p_sequencer是子类句柄，mcdf_virtual_sequencer类型的，m_sequencer是父类句柄，是uvm_sequencer类型的
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180158.png)

			- 声明了p_sequencer
			- 将m_sequencer转换为p_sequencer

		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180207.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180218.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180226.png)

		- 将句柄进行连接，避免悬空

- 建议
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180235.png)

### 8.2.4 layering sequence

- 通过层次化的sequence可以分别构建trsnsaction layer、transport layer和physical layer等，从高抽象级到低抽象级的transaction转化，这种层次化的构建方式，称为layering sequence
- 实例
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180818.png)

	- 转化层
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180828.png)

	- 句柄连接和挂载
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180839.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220710180849.png)








