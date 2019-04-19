<p align="center">
<img src=".github/PARL-logo.png" alt="PARL" width="500"/>
</p>

[English](./README.md) | 简体中文

> PARL 是一个高性能、灵活的强化学习框架。
# 特点
**可复现性保证**。我们提供了高质量的主流强化学习算法实现，严格地复现了论文对应的指标。

**大规模并行支持**。框架最高可支持上万个CPU的同时并发计算，并且支持多GPU强化学习模型的训练。

**可复用性强**。用户无需自己重新实现算法，通过复用框架提供的算法可以轻松地把经典强化学习算法应用到具体的场景中。

**良好扩展性**。当用户想调研新的算法时，可以通过继承我们提供的基类可以快速实现自己的强化学习算法。


# 框架结构
<img src=".github/abstractions.png" alt="abstractions" width="400"/>  
PARL的目标是构建一个可以完整复杂任务的智能体。以下是用户在逐步构建一个智能体的过程中需要了解到的结构：

### Model
`Model` 用来定义前向(`Forward`)网络，这通常是一个策略网络(`Policy Network`)或者一个值函数网络(`Value Function`)，输入是当前环境状态(`State`)。

### Algorithm
`Algorithm` 定义了具体的算法来更新前向网络(`Model`)，也就是通过定义损失函数来更新`Model`。一个`Algorithm`包含至少一个`Model`。

### Agent
`Agent` 负责算法与环境的交互，在交互过程中把生成的数据提供给`Algorithm`来更新模型(`Model`)，数据的预处理流程也一般定义在这里。

以下是构建一个包含DQN算法的智能体(`Agent`)用来玩雅达利游戏(`Atari Games`)的示例：

```python
import parl
from parl.algorithms import DQN

class AtariModel(parl.Model):
	"""AtariModel
	This class defines the forward part for an algorithm,
	its input is state observed on environment.
	"""
	def __init__(self, img_shape, action_dim):
		# define your layers
		self.cnn1 = layers.conv_2d(num_filters=32, filter_size=5,
			 			stride=1, padding=2, act='relu')
		...
		self.fc1 = layers.fc(action_dim)
		
	def value(self, img):
		# define how to estimate the Q value based on the image of atari games.
		img = img / 255.0
		l = self.cnn1(img)
		...
		Q = self.fc1(l)
		return Q
"""
三步定义一个智能体：
   1.  定义前向模型，就是上面的值函数网络(Value)，定义了如何针对输入的游戏图像评估Q值。
   2.  通过DQN算法来更新模型(Model)，在这里我们直接import仓库中实现好的DQN算法即可。
   3.  在AtariAgent中定义数据交互部分，把交互过程中得到的数据用来传给DQN算法以更新模型。
"""

model = AtariModel(img_shape=(32, 32), action_dim=4)
algorithm = DQN(model)
agent = AtariAgent(algorithm)
```

# 简易高效的并行接口
在PARL中，一个**修饰符**(parl.remote_class)就可以帮助用户实现自己的并行算法。
以下我们通过`Hello World`的例子来说明如何简单地通过PARL来调度外部的计算资源实现并行计算。
```python
#============Agent.py=================
@parl.remote_class
class Agent(object):

	def say_hello(self):
		print("Hello World!")

	def sum(self, a, b):
		return a+b

# launch `Agent.py` at any computation platforms such as a CPU cluster.
if __main__ == '__main__':
    agent = Agent()
    agent.as_remote(server_address)


#============Server.py=================
remote_manager = parl.RemoteManager()
agent = remote_manager.get_remote()
agent.say_hello()
ans = agent.sum(1,5) # run remotely and not comsume any local computation resources 
```
两步调度外部的计算资源：
1. 使用`parl.remote_class`修饰一个类，之后这个类就被转化为可以运行在其他CPU或者机器上的类。
2. 通过`RemoteManager`获取远端的类实例，通过这种方式获取到的实例和原来的类是有同样的函数的。由于这些类是在别的计算资源上运行的，执行这些函数**不再消耗当前线程计算资源**。

<img src=".github/decorator.png" alt="PARL" width="450"/>

如上图所示，真实的actor（橙色圆圈）运行在CPU集群，learner（蓝色圆圈）和remote actor（黄色圆圈）运行在本地的GPU上。对于用户而言，完全可以像写多线程代码一样来实现并行算法，相当简单，但是这些多线程的运算利用了外部的计算资源。我们也提供了并行算法示例，更多细节请参考[IMPALA](examples/IMPALA), [A2C](examples/A2C) and [GA3C](examples/GA3C)。


# 安装:
### 依赖
- Python 2.7 or 3.5+. 
- PaddlePaddle >=1.2.1 (**非必须的**，如果你只用并行部分的接口不需要安装paddle) 


```
pip install parl
```

# 算法示例
- [QuickStart](examples/QuickStart/)
- [DQN](examples/DQN/)
- [DDPG](examples/DDPG/)
- [PPO](examples/PPO/)
- [IMPALA](examples/IMPALA/)
- [A2C](examples/A2C/)
- [GA3C](examples/GA3C/)
- [冠军解决方案：NIPS2018强化学习假肢挑战赛](examples/NeurIPS2018-AI-for-Prosthetics-Challenge/)

<img src=".github/NeurlIPS2018.gif" width = "300" height ="200" alt="NeurlIPS2018"/> <img src=".github/Half-Cheetah.gif" width = "300" height ="200" alt="Half-Cheetah"/> <img src=".github/Breakout.gif" width = "200" height ="200" alt="Breakout"/> 
<br>
<img src=".github/Aircraft.gif"  width = "808" height ="300"  alt="NeurlIPS2018"/>