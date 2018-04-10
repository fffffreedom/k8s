# Tensors

> https://www.tensorflow.org/programmers_guide/tensors  

## 术语对照
| 英文     | 描述        |
| :----- | :-------- |
| graph  | 图         |
| tensor | 张量，表示N维数组 |
| shape  | 张量的维度     |

## introduce

TensorFlow是一个定义和运行张量（Tensors）相关计算的框架。TensorFlow将张量表示为基本数据类型的n维数组。  

当编写一个TensorFlow程序时，你操纵和传递的主要对象是tf.Tensor。  
A tf.Tensor object represents a [partially defined computation](???) that will eventually produce a value.  

Tensorflow程序工作流程：  
首先建立一个tf.Tensor对象的图形，细说明如何根据其他可用张量计算每个张量，然后通过运行该图的一部分来实现期望的结果。  

一个tf.Tensor有以下属性：  
- a data type(float32, int32, or string, for example)  
- a shape  

Tensor中的每个元素具有相同的已知的数据类型。shape（也就是说，Tensor具有的维度数量和每个维度的大小）可以只有部分已知。  
如果其输入的shape是完全已知的，则大多数操作产生完全已知shape的张量，但在某些情况下，只能在graph执行时找到张量的shape。  

某些类型的tensors比较特殊，将在其它单元描述。主要的的


## Reference

TensorFlow编程指南: Tensor(张量) -- 官网的翻译  
https://vimsky.com/article/3649.html  

谷歌开源深度学习工具—TensorFlow初探    
https://www.jianshu.com/p/1edde870eefe  
