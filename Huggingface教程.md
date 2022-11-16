# Huggingface教程

## NLP

我们要学的是一个能力、学的是一个语言模型，而不是学一个具体的任务。

NLP模型是站在巨人的肩膀上前进的。

传统的NLP没必要学了，今天的NLP已经不再需要传统的方法，一些只需要交给transformer就够了。



## pytorch

### 基本用法

**python两大法宝函数**

dir()函数：知道工具箱，以及工具箱里有什么东西

```python
dir(pytorch)
```

help()函数：知道每个工具是如何使用，工具的使用用法

```python
help(pytorch)
```



**python属性**

张量是什么？可以简单地理解为跟ndarray很像的数组，不同的是张量可以在GPU上做加速运算。

跟ndarray对象相关的

```
dtype 张量存储的类型，可选类型torch.dtype
device 张量存储的设备类型，cpu/gpu
data 张量节点存储的值
```

跟自动微分相关的

```
requires_grad 表示autograd时是否需要计算tensor的梯度，默认False
grad 存储梯度的值，初始化为None。这个值只有每次反向传播时才会有，前向传播是没有的
grad_fn 反向传播时，用来计算梯度的函数
```

跟图相关的

```
is_leaf 张量节点在计算图中是否为叶子节点
```



正在访问非叶子节点张量的.grad属性，在autograd.backward()期间也不会填充.grad属性，因为pytorch会认为非叶子节点都是中间变量，一般情况下，用户是不会使用这些中间变量的导数，所以为了节省内存，他们在用完之后就会释放。如果确实需要使用非叶子节点的.grad属性，在非叶子节点的张量上使用.retain_grad()函数。



在调用backward()时，只有当requires_grad和is_leaf同时为真时，才会计算节点的梯度值。

也就是说只有requires_grad=True，不一定自动求导，还需要满足是否为叶子节点。



in-place操作：在不改变内存地址的情况下修改变量的值



**创建张量**

创建tensor时有两种：类构造函数和工厂函数，区别：

1.构造函数创建张量使用全局默认(Tensor)，工厂函数则根据输入推断数据类型(tensor)

2.Tensor和tensor方法都是深拷贝，在内存中创建一个额外的福本，不共享内存，所以不受数组的改变的影响。

from_numpy和as_tensor都是浅拷贝，在内存中共享数据。（针对cpu而言，gpu上不共享）



**pytorch加载数据集**

DataSet类：加载数据集以及对应的label分类

```python
from torch.utils.data import Dateset

# 查看Dataset用法
help(Dataset)
Dataset??

# 定义一个自己的Data类
import	torch
from torch.utils.data import Dataset
from torch.utils.data import DataLoader
class DiabetesDataset(Dataset):
    def _init_ (self):
    	pass
    def _getitem_ (self,index):
    	pass
    def _len_ (self):
    	pass
    
dataset = DiabetesDataset()
train_loader = DataLoader(dataset = dataset,
                            batch_size = 32,
                            shuffle = True,
                            num_workers = 2)
```



DateSet和DataLoader

epoch表示训练的周期。你的**所有**数据集都进行了一次前向传播和反向传播，这就叫一个epoch。

batch-size表示每次训练的时候所用的样本数量，完成一次前馈传播和反向传播一个更新所用的样本数量。

iteration表示batch分成了几份。

比如有10000个样本，假设batch-size是1000个，那么iteration就是10。



Dataset是抽象类，需要定义自己的类去继承，实现里面的方法（init、getitem、len）

DataLoader是一个类帮我们加载数据的





**深度学习训练过程可视化**

Visdom

画三维图时：matplotlib、np.meshgrid()

指数加权均值作平滑



**pytorch的张量tensor**

使用pytorch的tensor做运算其实就是在构建计算图，使用tensor.backward()会计算权重的梯度，并且会释放这次构建的计算图。

当你只是想获取tensor的数值时，但又不想构建计算图时，可以使用tensor.item()来获取

tensor.item()适用于tensor里只有一个元素时

如果有多个元素可以通过 tensor.numpy()[0]获得

w.grad.data.zero_()表示把权重里的数据都清零，不然在下一次运算中会进行累加。



tensor.detach()和tensor.data。detach()和data生成的都是无梯度的纯tensor，x.data和x.detach()新分离出来的tensor的requires_grad=False

tensor.detach()返回一个新的tensor，从当前计算图中分离下来。但是仍然指向原变量的存放位置，不同之处是requirse_grad为false，新得到的tensor永远不需要计算梯度，不具有grad。

```python
import torch
a = torch.tensor([1,2,3.], requires_grad = True)
print(a)	# tensor([1., 2., 3.], requires_grad=True)
b = a.data
print(b.requires_grad)	# False
print(b)	# tensor([1., 2., 3.])
c = a.detach()	# False
print(c.requires_grad)	# tensor([1., 2., 3.])
print(c)
```

tensor.data还是和之前的数据共享同一份内存，所以data的数据改变，之前的数据也改变

```python
a = torch.tensor([1,2,3.], requires_grad = True)
c = a.data  # 需要走注意的是，通过.data “分离”得到的的变量会和原来的变量共用同样的数据，而且新分离得到的张量是不可求导的，c发生了变化，原来的张量也会发生变化
print(a) # tensor([1., 2., 3.], requires_grad=True)
c.zero_() # c发生变化，a也会变化
print(a) # tensor([0., 0., 0.], requires_grad=True)
```

tensor.detach()还是和之前的数据共享同一份内存，所以data的数据改变，之前的数据也改变

```python
a = torch.tensor([1,2,3.], requires_grad = True)
c = a.detach()  # 需要走注意的是，通过.data “分离”得到的的变量会和原来的变量共用同样的数据，而且新分离得到的张量是不可求导的，c发生了变化，原来的张量也会发生变化
print(a) # tensor([1., 2., 3.], requires_grad=True)
c.zero_() # c发生变化
print(a) # tensor([0., 0., 0.], requires_grad=True)
```

tensor.data：由于我更改分离之后的变量值c,导致原来的张量out的值也跟着改变了，但是这种改变对于autograd是没有察觉的，它依然按照求导规则来求导，导致得出完全错误的导数值却浑然不知。它的风险性就是如果我再任意一个地方更改了某一个张量，求导的时候也没有通知我已经在某处更改了，导致得出的导数值完全不正确，故而风险大。

```python
import torch
 
a = torch.tensor([1,2,3.], requires_grad = True)
out = a.sigmoid()
c = out.data  # 需要走注意的是，通过.data “分离”得到的的变量会和原来的变量共用同样的数据，而且新分离得到的张量是不可求导的，c发生了变化，原来的张量也会发生变化
c.zero_()     # 改变c的值，原来的out也会改变
print(c.requires_grad)
print(c)
print(out.requires_grad)
print(out)
print("----------------------------------------------")
 
out.sum().backward() # 对原来的out求导，
print(a.grad)  # 不会报错，但是结果却并不正确
'''运行结果为：
False
tensor([0., 0., 0.])
True
tensor([0., 0., 0.], grad_fn=<SigmoidBackward>)
----------------------------------------------
tensor([0., 0., 0.])
'''
```

tensor.detach()：由于我更改分离之后的变量值c,导致原来的张量out的值也跟着改变了，这个时候如果依然按照求导规则来求导，由于out已经更改了，所以不会再继续求导了，而是报错，这样就避免了得出完全牛头不对马嘴的求导结果。

```python
import torch
 
a = torch.tensor([1,2,3.], requires_grad = True)
out = a.sigmoid()
c = out.detach()  # 需要走注意的是，通过.detach() “分离”得到的的变量会和原来的变量共用同样的数据，而且新分离得到的张量是不可求导的，c发生了变化，原来的张量也会发生变化
c.zero_()     # 改变c的值，原来的out也会改变
print(c.requires_grad)
print(c)
print(out.requires_grad)
print(out)
print("----------------------------------------------")
 
out.sum().backward() # 对原来的out求导，
print(a.grad)  # 此时会报错，错误结果参考下面,显示梯度计算所需要的张量已经被“原位操作inplace”所更改了。

'''
False
tensor([0., 0., 0.])
True
tensor([0., 0., 0.], grad_fn=<SigmoidBackward>)
----------------------------------------------
RuntimeError: one of the variables needed for gradient computation has been modified by an inplace operation
'''
```

总结：

1.x.data和x.detach()都是从原有计算中分离出来的一个tensor变量，生成的都是无梯度的纯tensor,并且通过同一个tensor数据操作，是共享一块数据内存。

2.tensor.data 是不安全的，tensor.detach()是安全的。体现在求导过程中



from torchvision import transform	是对图像做处理的类



### DataSet和DataLoader

**DataSet**

自己写的类要实现三个初始化方法，init、len、getitem



**DataLoader**

DataLoader将单个样本组织成批次，送入神经网络训练。

collate_fn对一个batch进行处理的函数，比如padding成一样的长度等。

### torch.nn.Module

所有自己写的神经网络都要继承nn.Module这个父类，需要实现两个方法，一个是init，一个是forward。

forward就是前向运算。

nn.Sequential():按照顺序传入一些层或者模块就可以了，他会自动的把他们给串联起来。

```python
class NeuralNetwork(nn.Module):
    def __init__(self):
        super(NeuralNetwork, self).__init__()
        self.flatten = nn.Flatten()
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(28*28, 512),
            nn.ReLU(),
            nn.Linear(512, 512),
            nn.ReLU(),
            nn.Linear(512, 10),
        )

    def forward(self, x):
        x = self.flatten(x)
        logits = self.linear_relu_stack(x)
        return logits
```

 

## 问题

### 1.训练时内存溢出，内存不够

比较有效的解决办法：

1.砍输入的max_length

2.砍 hidden_size

3.小 batch_size + 梯度累积





















