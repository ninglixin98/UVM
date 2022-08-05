# UVM —— 第九章 寄存器模型概述与集成
## 9.1 寄存器模型概述
### 9.1.1 寄存器
- 硬件中各个功能模块与处理器的对话是通过寄存器进行的
- 寄存器的硬件实现是通过触发器，而每一个比特位的触发器都对应着寄存器的功能描述
- 一个寄存器由32个比特位构成，将单个寄存器拆开后，又可以分为多个域，不同的域代表着某一项独立功能
	- 单个域可以由一个比特位构成，也可以由多个比特位构成
	- 不同的域，对外部读写而言，大致可分为
		- WO（write-only，只写）
		- RO（read-only，只读）
		- RW（read and write，读写）
		- reserved，保留域
		- 特殊行为（quirky）寄存器
			- 读后擦除模式（clean-on-read, RC）
			- 只写一次模式（write-one-to-set, W1S）

- 将寄存器按照地址进行排列，即可构成寄存器列表，我们称之为寄存器块（register block）
- 一个功能模块的多个寄存器，可以组团构成一个寄存器模型
- 对于验证而言，可以将利用总线访问寄存器的方式抽象成为寄存器模型访问的方式，利于后续修改，提高复用性

- 寄存器模型优点
	- 对于sequence复用性，可读性更好
	- 对于regester的测试，具有预测的帮助
	- 寄存器覆盖率

### 9.1.2 中心化管理方式
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172330.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172340.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172348.png)

### 9.1.3 uvm_reg相关概念
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172356.png)

- 都是object

### 9.1.4 MCDF寄存器模型
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172730.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172741.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172750.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172802.png)

- map = create_map(名字，基地址，总线访问位宽，访问大小端提示);
	- 寄存器访问：基地址+偏移地址
- map.add_reg()添加偏移地址和访问属性
- configure(父类，位数，起点位数，读写属性，，复位值)，用来配置属性
- lock_model()不允许外部访问和修改属性
- 属性需要rand进行随机化

### 9.1.5 模型使用流程
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172828.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713172839.png)

## 9.2 寄存器模型集成
### 9.2.1 总线UVC的实现
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713173300.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713173309.png)

### 9.2.2 adapter集成
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713173342.png)

- adapter将不同的抽象级数据进行转化，例如将uvm_reg_item转化为bus_seq_item

#### 9.2.2.1 总线adapter需要实现
 ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713173751.png)

- 如果有返回值response，需要将provides_responses设置为1

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713173805.png)

- uvm_reg_bus_op
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713173815.png)

- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713173825.png)


#### 9.2.2.2 集成

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713174205.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713174215.png)

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713180206.png)


- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713174225.png)

	- 将map与adapter关联，adapter与bus关联，实现三者关联

- rgm.map.set_auto_predict()
	- 使用自动预测功能

#### 9.2.2.3 访问方式

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713175242.png)

- 前门访问
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713175314.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713175328.png)

		- 使用sequence定义的read()/write()访问
		- 使用uvm_reg_sequence预定义的 read_reg()/write_reg()访问

- 后门访问
	- 不是真实的访问，优点在于速度块
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713175341.png)

	- 路径映射
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713175353.png)


	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713175411.png)

	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713175425.png)

### 9.2.3 前门访问和后门访问对比

![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220713180051.png)





