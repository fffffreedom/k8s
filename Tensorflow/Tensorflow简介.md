# Tensorflow简介

[TensorFlow](https://www.tensorflow.org/)是谷歌开源的深度学习工具包，是一个使用数据流图(dataflow graph)进行数值计算的开源软件库。
图中的节点代表数学运算，而图中的边则代表在这些节点之间传递的多维数组（张量，即tensor）。这种灵活的架构可让您使用一个 API 将计算工作
部署到桌面设备、服务器或者移动设备中的一个或多个 CPU 或 GPU。  

## 标量、向量、矩阵和张量

张量（tensor）是N维矩阵抽象。零维张量是标量（scalar，就是一个单独的数），一维张量是向量，二维张量是矩阵，三维或以上称N维张量或N阶张量。  

> http://blog.csdn.net/howhigh/article/details/73929058

## [Tensor Values](https://www.tensorflow.org/programmers_guide/low_level_intro)

tensor，即张量，是Tensorflow数值计算的基本单位。一个tensor由一组形成任意数量dimensions的原始值组成（如下例子）。  
tensor的rank是它的dimensions（维数），tensor的shape是一个指定了各dimension长度的整数元组。
见如下例子说明：  
```
3. # a rank 0 tensor; a scalar with shape [],
[1., 2., 3.] # a rank 1 tensor; a vector with shape [3]
[[1., 2., 3.], [4., 5., 6.]] # a rank 2 tensor; a matrix with shape [2, 3]
[[[1., 2., 3.]], [[7., 8., 9.]]] # a rank 3 tensor with shape [2, 1, 3]
```
TensorFlow uses numpy arrays to represent tensor values.  

注：numpy是Python的一种开源的数值计算扩展。这种工具可用来存储和处理大型矩阵，
比Python自身的嵌套列表（nested list structure)结构要高效的多（该结构也可以用来表示矩阵（matrix））。  

## TensorFlow Core Walkthrough

你可以认为，TensorFlow的核心程序由两部分组成：  
- Building the computational graph (a tf.Graph).
- Running the computational graph (using a tf.Session).

### Graph

由一系列Tensorflow operations组成的graph叫作computational graph（计算图），它包含两种类型的对象：  
- Operations (or "ops")：graph中的节点（nodes）即为操作（ops），它描述消耗和产生tensor的计算。  
- Tensors：graph中的edges，它代表了将在graph中流动的数据。大多数Tensorflow函数返回的就是`tf.Tensors`。  

注：tf.Tensors没有值，它们只是计算图中元素的句柄。  

每个operation有一个唯一的名字，这个名字与在Python中分配给对象的名称无关。
tensor以产生它们的operation命名，后跟输出索引，比如"add:0"。  

### TensorBoard

TensorFlow提供了一个名为TensorBoard的实用程序。TensorBoard的主要功能之一是把计算图可视化。您可以使用几个简单的命令轻松完成此操作。

### Session

session封装了TensorFlow运行时的状态，并运行TensorFlow操作。或许下面这样说更清楚：  
```
If a tf.Graph is like a .py file, a tf.Session is like the python executable.
```

### Feeding

graph使用placeholder来接受外部的输入，就像函数参数一样。  
```
x = tf.placeholder(tf.float32)
y = tf.placeholder(tf.float32)
z = x + y
```

我们可以在运行session.run()时，使用`feed_dict`来给placeholder传递具体的数值。例如：  
```
print(sess.run(z, feed_dict={x: 3, y: 4.5}))
print(sess.run(z, feed_dict={x: [1, 3], y: [2, 4]}))
```
注意，feed_dict参数可用于覆盖graph中的任何tensor。placeholder和其他tf.Tensors之间的唯一区别是:  
**如果没有给它们赋值，placeholder会抛出错误。**  

## Datasets

在简单的实验中，我们使用placeholder来传递参数或数据，但Datasets是将数据流式传输到模型的首选方法！  

## Layers

可训练的模型必须修改graph中的值，以获得具有相同输入的新输出。Layers是将可训练参数添加到图形的首选方法。  
Layers将变量和作用于它们的操作打包在一起。

### Creating Layers

### Initializing Layers

The layer contains variables that must be initialized before they can be used.  

### Executing Layers

## Feature columns

## Training

到这里，我们已经对Tensorflow的基础知识有所熟悉。让我们手动训练一个小回归模型。  

### Define the data

我们先定义一些输入x, 以及每个输入对应的输出y_true:  
```
x = tf.constant([[1], [2], [3], [4]], dtype=tf.float32)
y_true = tf.constant([[0], [-1], [-2], [-3]], dtype=tf.float32)
```

### Define the model

构建一个简单的线性模型，它只有一个输出。
### loss
### Training

## Reference

Tensorflow官网  
https://www.tensorflow.org/programmers_guide/low_level_intro  
