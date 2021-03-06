# 个性化训练：演练

[Colab notebook](https://colab.research.google.com/github/tensorflow/models/blob/master/samples/core/tutorials/eager/custom_training_walkthrough.ipynb)


# To add a new cell, type '# %%'
# To add a new markdown cell, type '# %% [markdown]'
# %%
from IPython import get_ipython

# %% [markdown]
# ##### Copyright 2018 The TensorFlow Authors.

# %%
#@title Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# %% [markdown]
# # 自定义训练：实战
# %% [markdown]
# <table class="tfo-notebook-buttons" align="left">
#   <td>
#     <a target="_blank" href="https://www.tensorflow.org/tutorials/eager/custom_training_walkthrough"><img src="https://www.tensorflow.org/images/tf_logo_32px.png" />View on TensorFlow.org</a>
#   </td>
#   <td>
#     <a target="_blank" href="https://colab.research.google.com/github/tensorflow/docs/blob/master/site/en/tutorials/eager/custom_training_walkthrough.ipynb"><img src="https://www.tensorflow.org/images/colab_logo_32px.png" />Run in Google Colab</a>
#   </td>
#   <td>
#     <a target="_blank" href="https://github.com/tensorflow/docs/blob/master/site/en/tutorials/eager/custom_training_walkthrough.ipynb"><img src="https://www.tensorflow.org/images/GitHub-Mark-32px.png" />View source on GitHub</a>
#   </td>
# </table>
# %% [markdown]
# 本教程按照物种**分类**鸢尾花。本教程使用 TensorFlow [动态图机制](https://www.tensorflow.org/guide/eager)实现：
# 1. 构建模型，
# 2. 在样例数据集上训练模型，
# 3. 使用模型预测未知数据。
# 
# ## TensorFlow 编程
# 
# 本指南使用这些高级 TensorFlow 概念：
# 
# * 启用[动态图机制](https://www.tensorflow.org/guide/eager)开发环境，
# * 使用 [Datasets API](https://www.tensorflow.org/guide/datasets)导入数据，
# * 使用 TensorFlow [Keras API](https://keras.io/getting-started/sequential-model-guide/)构建模型和层。
# 
# 本教程的结构与许多 TensorFlow 程序类似：
# 
# 1. 导入并解析训练数据集。
# 2. 选择模型。
# 3. 训练模型。
# 4. 评估模型的效果。
# 5. 使用训练好的模型进行预测。
# %% [markdown]
# ## 程序设置
# %% [markdown]
# ### 导入配置和动态图计算
# 
# 导入所需的 Python 模块 — 包括 TensorFlow — 并启用动态图计算。动态图计算使得 TensorFlow 立即执行操作运算，返回具体值而不是创建稍后执行的[计算图](https://www.tensorflow.org/guide/graphs)。如果你习惯了 REPL 或者 `python` 交互式控制台，那就会感觉很熟悉。动态图计算需要 [Tensorlow >=1.8](https://www.tensorflow.org/install/)版本才能支持。
# 
# 一旦启用了动态图计算，就不能在同一程序中**禁用**。详情见[动态图计算指南](https://www.tensorflow.org/guide/eager)。

# %%
from __future__ import absolute_import, division, print_function

import os
import matplotlib.pyplot as plt

import tensorflow as tf
import tensorflow.contrib.eager as tfe

tf.enable_eager_execution()

print("TensorFlow version: {}".format(tf.VERSION))
print("Eager execution: {}".format(tf.executing_eagerly()))

# %% [markdown]
# ## 鸢尾花分类问题
# 
# 想象一下，你是一名植物学家，正在寻找一种自动化的方法来对你发现的每朵鸢尾花进行分类。机器学习提供了许多算法来对花进行统计分类。例如，复杂的机器学习程序可以根据照片对花进行分类。我们的方法相对简单 — 我们将根据鸢尾花的[萼片](https://en.wikipedia.org/wiki/Sepal)和[花瓣](https://en.wikipedia.org/wiki/Petal)的长度和宽度进行分类。
# 
# 鸢尾花累计有 300 多种，但是我们的程序将对以下三类进行分类：
# 
# * Iris setosa
# * Iris virginica
# * Iris versicolor
# 
# <table>
#   <tr><td>
#     <img src="https://www.tensorflow.org/images/iris_three_species.jpg"
#          alt="Petal geometry compared for three iris species: Iris setosa, Iris virginica, and Iris versicolor">
#   </td></tr>
#   <tr><td align="center">
#     <b>Figure 1.</b> <a href="https://commons.wikimedia.org/w/index.php?curid=170298">Iris setosa</a> (by <a href="https://commons.wikimedia.org/wiki/User:Radomil">Radomil</a>, CC BY-SA 3.0), <a href="https://commons.wikimedia.org/w/index.php?curid=248095">Iris versicolor</a>, (by <a href="https://commons.wikimedia.org/wiki/User:Dlanglois">Dlanglois</a>, CC BY-SA 3.0), and <a href="https://www.flickr.com/photos/33397993@N05/3352169862">Iris virginica</a> (by <a href="https://www.flickr.com/photos/33397993@N05">Frank Mayfield</a>, CC BY-SA 2.0).<br/>&nbsp;
#   </td></tr>
# </table>
# 
# 幸运的是，有人已经用萼片和花瓣的测量数据创建[包含 120 个鸢尾花数据集](https://en.wikipedia.org/wiki/Iris_flower_data_set)。这是机器学习初学者学习分类问题使用的经典数据集。
# %% [markdown]
# ## 导入并解析数据集
# 
# 下载数据集文件并将其转换为可供此 Python 程序使用的结构。
# 
# ### 下载数据集
# 
# 使用 [tf.keras.utils.get_file](https://www.tensorflow.org/api_docs/python/tf/keras/utils/get_file) 函数下载训练数据集文件。 这将返回下载文件的文件路径。

# %%
train_dataset_url = "http://download.tensorflow.org/data/iris_training.csv"

train_dataset_fp = tf.keras.utils.get_file(fname=os.path.basename(train_dataset_url),
                                           origin=train_dataset_url)

print("Local copy of the dataset file: {}".format(train_dataset_fp))

# %% [markdown]
# ### 检查数据
# 
# 该数据集 `iris_training.csv` 是一个纯文本文件，用于存储格式为逗号分隔值（CSV）的表格数据。使用 `head -n5` 命令查看前五个条目：

# %%
get_ipython().system('head -n5 {train_dataset_fp}')

# %% [markdown]
# 从数据集来看，请注意以下内容：
# 
# 1. 第一行是包含有关数据集的信息的标题：
#   * 总共有 120 个样本。每个样本都有四维特征和对应的标签。
# 2. 后面每行数据是样本数据，一个**[样本](https://developers.google.com/machine-learning/glossary/#example)**一行：
#   * 前四个字段是**[特征](https://developers.google.com/machine-learning/glossary/#feature)**：这些是样本的特征。这里，字段包含表示花卉测量值的浮点数。
#   * 最后一列是**[标签](https://developers.google.com/machine-learning/glossary/#label)**：这是我们想要预测的值。对于此数据集，它是与花名称对应的整数值 0，1 或 2。
# 
# 让我们编写代码：

# %%
# CSV 文件中的列排序
column_names = ['sepal_length', 'sepal_width', 'petal_length', 'petal_width', 'species']

feature_names = column_names[:-1]
label_name = column_names[-1]

print("Features: {}".format(feature_names))
print("Label: {}".format(label_name))

# %% [markdown]
# 每个标签都与字符串名称相关联（例如，“setosa”），但机器学习通常依赖于数值。数值标签映射到命名，例如：
# 
# * `0`: Iris setosa
# * `1`: Iris versicolor
# * `2`: Iris virginica
# 
# 有关特征和标签的详细信息，请参阅[机器学习速成课程的 ML 术语部分](https://developers.google.com/machine-learning/crash-course/framing/ml-terminology)。

# %%
class_names = ['Iris setosa', 'Iris versicolor', 'Iris virginica']

# %% [markdown]
# ### 创建 `tf.data.Dataset`
# 
# TensorFlow 的 [数据集 API](https://www.tensorflow.org/guide/datasets) 可以处理常见的加载数据方式。这是一个高级 API，用于读取数据并将其转换为用于训练的格式。有关详细信息，请参阅[数据集快速入门指南](https://www.tensorflow.org/get_started/datasets_quickstart)。
# 
# 
# 由于数据集是 CSV 格式的文本文件，因此请使用 [make_csv_dataset](https://www.tensorflow.org/api_docs/python/tf/contrib/data/make_csv_dataset) 函数将数据解析为合适的格式。由于此函数为训练模型生成数据，因此默认会对数据进行混洗（`shuffle = True，shuffle_buffer_size = 10000`），并永远重复数据集（`num_epochs = None`）。我们还设置了 [batch_size](https://developers.google.com/machine-learning/glossary/#batch_size) 参数。

# %%
batch_size = 32

train_dataset = tf.contrib.data.make_csv_dataset(
    train_dataset_fp,
    batch_size, 
    column_names=column_names,
    label_name=label_name,
    num_epochs=1)

# %% [markdown]
# `make_csv_dataset` 函数返回包含对应 `(features，label)` 的 `tf.data.Dataset`，其中 `features` 是一个字典：`{'feature_name'：value}`
# 
# 启用动态图计算后，这些“数据集”对象是可迭代的。我们来看看以下功能：

# %%
features, labels = next(iter(train_dataset))

features

# %% [markdown]
# 请注意，特征是组合在一起，或**分批**。每个样本的特征添加到相应的数组。更改 `batch_size` 以设置存储在数组中的样本数。
# 
# 你可以通过绘图查看一批样本集的特征分布：

# %%
plt.scatter(features['petal_length'],
            features['sepal_length'],
            c=labels,
            cmap='viridis')

plt.xlabel("Petal length")
plt.ylabel("Sepal length");

# %% [markdown]
# 要简化模型构建步骤，请创建一个函数将特征字典重新打包为大小为 `(batch_size，num_features)` 的数组。
# 
# 你可以使用 [tf.stack](https://www.tensorflow.org/api_docs/python/tf/stack) 方法，该方法从张量列表中获取值并在指定维度创建组合张量。

# %%
def pack_features_vector(features, labels):
  """Pack the features into a single array."""
  features = tf.stack(list(features.values()), axis=1)
  return features, labels

# %% [markdown]
# 然后使用 [tf.data.Dataset.map](https://www.tensorflow.org/api_docs/python/tf/data/dataset/map) 方法将每个`（特征，标签）`对中的`特征`映射到训练数据集：

# %%
train_dataset = train_dataset.map(pack_features_vector)

# %% [markdown]
# `Dataset` 的特征元素现在的大小为`(batch_size，num_features)`的数组。 我们来看几个例子：

# %%
features, labels = next(iter(train_dataset))

print(features[:5])

# %% [markdown]
# ## 模型选择
# 
# ### 为何选型？
# 
# 一个**[模型](https://developers.google.com/machine-learning/crash-course/glossary#model)**是特征和标签之间的映射关系。对于鸢尾花分类问题，该模型定义了萼片和花瓣测量值与预测的鸢尾花种类之间的关系。一些简单的模型可以用几个参数来描述，但是复杂的机器学习模型具有很多难以概括的参数。
# 
# 你能否在**不**使用机器学习的情况下确定四种特征与鸢尾花种类之间的关系？也就是说，你可以使用传统的编程技术（例如，很多条件语句）来创建模型吗？也许 - 如果你对数据集进行了足够长时间的分析，以确定物种与花瓣和萼片测量值之间的关系。在更复杂的数据集上，这变得很困难 — 也许是不可能的。好的机器学习方法**为你确定模型**。如果你将足够的代表性样本提供给正确的机器学习模型，程序将为你找出对应关系。
# 
# ### 模型选择
# 
# 我们需要选择要训练的模型。有很多类型的模型，你可以挑选一个好的进行试验。本教程使用神经网络来解决鸢尾花分类问题。**[神经网络](https://developers.google.com/machine-learning/glossary/#neural_network)**可以找到特征和标签之间的复杂关系。它是一个高度结构化的图模型，分为一个或多个**[隐藏层](https://developers.google.com/machine-learning/glossary/#hidden_​​layer)**。每个隐藏层由一个或多个**[神经元](https://developers.google.com/machine-learning/glossary/#neuron)**组成。有几类神经网络，这个程序使用密集或**[全连接神经网络](https://developers.google.com/machine-learning/glossary/#fully_connected_layer)**：一层中的神经元和前一层的**每一个**神经元相连接。例如，图 2 说明了一个全连接的神经网络，包括一个输入层，两个隐藏层和一个输出层：
# 
# <table>
#   <tr><td>
#     <img src="https://www.tensorflow.org/images/custom_estimators/full_network.png"
#          alt="A diagram of the network architecture: Inputs, 2 hidden layers, and outputs">
#   </td></tr>
#   <tr><td align="center">
#     <b>Figure 2.</b>包含特征，隐藏层和预测的神经网络。<br/>&nbsp;
#   </td></tr>
# </table>
# 
# 当对图 2 的模型进行训练并喂食未标记的样本时，会产生三个预测结果：该花是给定的 Iris 物种中每一种的可能性。此预测称为**[推理](https://developers.google.com/machine-learning/crash-course/glossary#inference)**。此例中，输出预测的总和为 1.0。在图 2 中，该预测分解为：**Iris setosa** 的概率为 `0.02`，**Iris versicolor** 的概率为 `0.95`，**Iris virginica** 的概率为 `0.03`。 这意味着模型以 95％ 的概率预测 — 未标记的花是 **Iris versicolor**。
# %% [markdown]
# ### 使用 Keras 创建模型
# 
# TensorFlow [tf.keras](https://www.tensorflow.org/api_docs/python/tf/keras) API 是创建模型和层的首选方式。用 Keras 处理所有错综复杂的内容，会使得构建模型和实验变得容易。
# 
# [tf.keras.Sequential](https://www.tensorflow.org/api_docs/python/tf/keras/Sequential) 模型是层的线性堆栈。它的构造函数实现了一个层实例列表，在本例中是两个[全连接](https://www.tensorflow.org/api_docs/python/tf/keras/layers/Dense)层，每层有 10 个节点，还有一个输出层有 3 个节点，分别代表我们的预测标签。第一层的 `input_shape` 参数对应于数据集特征的大小，并且是必需的。

# %%
model = tf.keras.Sequential([
  tf.keras.layers.Dense(10, activation=tf.nn.relu, input_shape=(4,)),  # 要求给定输入大小
  tf.keras.layers.Dense(10, activation=tf.nn.relu),
  tf.keras.layers.Dense(3)
])

# %% [markdown]
# **[激活函数](https://developers.google.com/machine-learning/crash-course/glossary#activation_function)**确定层中每个节点的输出形状。这些非线性函数很重要 — 没有它们，模型将等同于单个层。有许多[可用激活函数](https://www.tensorflow.org/api_docs/python/tf/keras/activations)，但 [ReLU](https://developers.google.com/machine-learning/crash-course/glossary#ReLU) 对于隐藏层很常见。
# 
# 
# 隐藏层和神经元的理想数量取决于问题和数据集。像机器学习的许多方面一样，选择神经网络的最佳参数需要知识和实践的结合。根据经验，增加隐藏层和神经元的数量通常会创建一个更强大的模型，这需要更多数据来有效训练。
# %% [markdown]
# ### 使用模型
# 
# 让我们快速浏览一下这个模型对给定特征数据的预测：

# %%
predictions = model(features)
predictions[:5]

# %% [markdown]
# 这里每个样本对应每个分类都返回一个[逻辑值](https://developers.google.com/machine-learning/crash-course/glossary#logit)。 
# 
# 要将这些逻辑值转化为每一个分类的概率，请使用 [softmax](https://developers.google.com/machine-learning/crash-course/glossary#softmax) 函数:

# %%
tf.nn.softmax(predictions[:5])

# %% [markdown]
# 在类之间取 `tf.argmax` 给出了预测的类索引。但是，该模型还没有被训练过，所以这些都不是很好的预测。

# %%
print("Prediction: {}".format(tf.argmax(predictions, axis=1)))
print("    Labels: {}".format(labels))

# %% [markdown]
# ## 训练模型
# 
# **[训练](https://developers.google.com/machine-learning/crash-course/glossary#training)**是机器学习中模型逐渐优化，或者模型**学习**数据集的阶段。目标是充分了解训练数据集的结构，以便对看不见的数据进行预测。如果对训练数据集学习**太多**，那么预测仅适用于它已经看到的数据，那么就不具有泛化能力。这个问题被称为**[过拟合](https://developers.google.com/machine-learning/crash-course/glossary#overfitting)**  — 就像记住答案而不是理解如何解决问题。
# 鸢尾花分类问题是**[监督机器学习](https://developers.google.com/machine-learning/glossary/#supervised_machine_learning)**的示例：模型是根据包含标签的样本进行训练的。在**[无监督机器学习](https://developers.google.com/machine-learning/glossary/#unsupervised_machine_learning)**中，样本不包含标签。相反，模型通常在特征中找到规律。
# %% [markdown]
# ### 定义损失和梯度函数
# 
# 训练和评估阶段都需要计算模型的**[损失](https://developers.google.com/machine-learning/crash-course/glossary#loss)**。这可以衡量模型预测结果与实际标签的关系，换句话说，模型的表现有多糟糕。我们希望最小化或优化此值。
# 
# 我们的模型将使用 [tf.keras.losses.categorical_crossentropy](https://www.tensorflow.org/api_docs/python/tf/losses/sparse_softmax_cross_entropy) 函数计算其损失，该函数输入是模型预测标签概率和真实标签，并返回样本的平均损失。

# %%
def loss(model, x, y):
  y_ = model(x)
  return tf.losses.sparse_softmax_cross_entropy(labels=y, logits=y_)


l = loss(model, features, labels)
print("Loss test: {}".format(l))

# %% [markdown]
# 使用 [tf.GradientTape](https://www.tensorflow.org/api_docs/python/tf/GradientTape) 计算**[梯度](https://developers.google.com/machine-learning/crash-course/glossary#gradient)**来优化我们的模型。想要了解更多，请参考[动态图计算指南](https://www.tensorflow.org/guide/eager)。

# %%
def grad(model, inputs, targets):
  with tf.GradientTape() as tape:
    loss_value = loss(model, inputs, targets)
  return loss_value, tape.gradient(loss_value, model.trainable_variables)

# %% [markdown]
# ### 创建优化器
# 
# **[优化器](https://developers.google.com/machine-learning/crash-course/glossary#optimizer)**将计算的梯度应用于模型的变量，以最大限度地减少 `loss` 函数值。你可以将损失函数视为曲面（参见图 3），我们希望通过四处走动找到它的最低点。梯度方向上最陡的是上升方向 — 所以我们将以相反的方向行进并向下移动。通过迭代计算每批的损失和梯度，我们将在训练期间调整模型。渐渐的，该模型将找到权重和偏差的最佳组合，从而最小化损失。损失越低，模型的预测越好。
# 
# <table>
#   <tr><td>
#     <img src="https://cs231n.github.io/assets/nn3/opt1.gif" width="70%"
#          alt="Optimization algorithms visualized over time in 3D space.">
#   </td></tr>
#   <tr><td align="center">
#     <b>Figure 3.</b> 在三维空间中可视化优化算法随着时间的变化。(来源：<a href="http://cs231n.github.io/neural-networks-3/">Stanford class CS231n</a>, MIT License)<br/>&nbsp;
#   </td></tr>
# </table>
# 
# TensorFlow 有许多[优化算法](https://www.tensorflow.org/api_guides/python/train)可用于训练。该模型使用 [tf.train.GradientDescentOptimizer](https://www.tensorflow.org/api_docs/python/tf/train/GradientDescentOptimizer) 实现**[随机梯度下降](https://developers.google.com/machine-learning/crash-course/glossary＃gradient_descent)**（SGD）算法。 `learning_rate` 设置每次迭代下降的步长。这是一个**超参数**，你通常会调整它以获得更好的结果。
# %% [markdown]
# 让我们设置优化器和 `global_step` 计数器：

# %%
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01)

global_step = tf.train.get_or_create_global_step()

# %% [markdown]
# 我们将使用它来计算单个优化步骤：

# %%
loss_value, grads = grad(model, features, labels)

print("Step: {}, Initial Loss: {}".format(global_step.numpy(),
                                          loss_value.numpy()))

optimizer.apply_gradients(zip(grads, model.variables), global_step)

print("Step: {},         Loss: {}".format(global_step.numpy(),
                                          loss(model, features, labels).numpy()))

# %% [markdown]
# ### 循环训练
# 
# 随着所有模块的创建，模型已准备好进行训练！将数据集提供给模型循环训练，以帮助其做出更好的预测。以下代码块设置了训练步骤：
# 
# 1. 执行每个**迭代**。一个迭代将遍历一次数据集。
# 2. 在一个迭代内, 迭代训练 `Dataset` 中的每个样本，学习它们的**特征** (`x`) 和 **标签** (`y`)。
# 3. 使用样本的特征，进行预测并将其与真实标签进行比较。测量预测的不准确性，并使用它来计算模型的损失和梯度。
# 4. 使用 `optimizer` 来更新模型的参数。
# 5. 跟踪一些可视化统计数据。
# 6. 重复迭代训练。
# 
# `num_epochs` 变量是循环数据集的次数。与直觉相反，长时间训练模型并不能保证更好的模型。`num_epochs` 是**[超参数](https://developers.google.com/machine-learning/glossary/#hyperparameter)**，你可以调整。选择正确的数值通常需要经验和实验。

# %%
## 注意：重新运行此单元格使用相同的模型变量

# 保存结果用于绘图
train_loss_results = []
train_accuracy_results = []

num_epochs = 201

for epoch in range(num_epochs):
  epoch_loss_avg = tfe.metrics.Mean()
  epoch_accuracy = tfe.metrics.Accuracy()

  # 循环训练 — 批大小为 32
  for x, y in train_dataset:
    # 优化模型
    loss_value, grads = grad(model, x, y)
    optimizer.apply_gradients(zip(grads, model.variables),
                              global_step)

    # 跟踪进度
    epoch_loss_avg(loss_value)  # add current batch loss
    # compare predicted label to actual label
    epoch_accuracy(tf.argmax(model(x), axis=1, output_type=tf.int32), y)

  # epoch 结束
  train_loss_results.append(epoch_loss_avg.result())
  train_accuracy_results.append(epoch_accuracy.result())
  
  if epoch % 50 == 0:
    print("Epoch {:03d}: Loss: {:.3f}, Accuracy: {:.3%}".format(epoch,
                                                                epoch_loss_avg.result(),
                                                                epoch_accuracy.result()))

# %% [markdown]
# ### 随着时间的推移可视化损失函数
# %% [markdown]
# 虽然打印模型的训练进度很有帮助，但看到这一进展通常会**更**有帮助。[TensorBoard](https://www.tensorflow.org/guide/summaries_and_tensorboard) 是一个很好的可视化工具，与 TensorFlow 一起打包，但我们可以使用 `matplotlib` 模块创建基本图表。
# 
# 解释这些图表需要一些经验，但你真的希望看到**损失**下降，**准确度**上升。

# %%
fig, axes = plt.subplots(2, sharex=True, figsize=(12, 8))
fig.suptitle('Training Metrics')

axes[0].set_ylabel("Loss", fontsize=14)
axes[0].plot(train_loss_results)

axes[1].set_ylabel("Accuracy", fontsize=14)
axes[1].set_xlabel("Epoch", fontsize=14)
axes[1].plot(train_accuracy_results);

# %% [markdown]
# ## 评估模型效果
# 
# 现在模型已经过训练，我们可以对其性能进行一些统计。
# 
# **评估**意味着确定模型如何有效地进行预测。为了确定模型在鸢尾花分类中的有效性，将一些萼片和花瓣测量值传递给模型，并要求模型预测它们代表的鸢尾花种类。然后将模型的预测与实际标签进行比较。例如，在输入样本的一半上预测正确则模型具有**[准确度](https://developers.google.com/machine-learning/glossary/#accuracy)**为 `0.5`。图 4 显示了一个稍微更有效的模型，命中 5 个样本中的 4 个，准确率为 80%：
# 
# <table cellpadding="8" border="0">
#   <colgroup>
#     <col span="4" >
#     <col span="1" bgcolor="lightblue">
#     <col span="1" bgcolor="lightgreen">
#   </colgroup>
#   <tr bgcolor="lightgray">
#     <th colspan="4">Example features</th>
#     <th colspan="1">Label</th>
#     <th colspan="1" >Model prediction</th>
#   </tr>
#   <tr>
#     <td>5.9</td><td>3.0</td><td>4.3</td><td>1.5</td><td align="center">1</td><td align="center">1</td>
#   </tr>
#   <tr>
#     <td>6.9</td><td>3.1</td><td>5.4</td><td>2.1</td><td align="center">2</td><td align="center">2</td>
#   </tr>
#   <tr>
#     <td>5.1</td><td>3.3</td><td>1.7</td><td>0.5</td><td align="center">0</td><td align="center">0</td>
#   </tr>
#   <tr>
#     <td>6.0</td> <td>3.4</td> <td>4.5</td> <td>1.6</td> <td align="center">1</td><td align="center" bgcolor="red">2</td>
#   </tr>
#   <tr>
#     <td>5.5</td><td>2.5</td><td>4.0</td><td>1.3</td><td align="center">1</td><td align="center">1</td>
#   </tr>
#   <tr><td align="center" colspan="6">
#     <b>Figure 4.</b> 其中一个分类器有 80% 准确率。<br/>&nbsp;
#   </td></tr>
# </table>
# %% [markdown]
# ### 设置测试集
# 
# 评估模型类似于训练模型。最大的区别是样本来自单独的**[测试集](https://developers.google.com/machine-learning/crash-course/glossary#test_set)**而不是训练集。为了公平地评估模型的有效性，用于评估模型的样本必须与用于训练模型的样本不同。
# 
# 测试 `Dataset` 的设置类似于训练 `Dataset` 的设置。下载 CSV 文本文件并解析，然后混洗：

# %%
test_url = "http://download.tensorflow.org/data/iris_test.csv"

test_fp = tf.keras.utils.get_file(fname=os.path.basename(test_url),
                                  origin=test_url)


# %%
test_dataset = tf.contrib.data.make_csv_dataset(
    test_fp,
    batch_size, 
    column_names=column_names,
    label_name='species',
    num_epochs=1,
    shuffle=False)

test_dataset = test_dataset.map(pack_features_vector)

# %% [markdown]
# ### 测试集上评估模型
# 
# 与训练阶段不同，该模型仅评估测试数据的单个 [epoch](https://developers.google.com/machine-learning/glossary/#epoch)。在下面的代码单元格中，我们迭代测试集中的每个样本，并将模型的预测与实际标签进行比较。这用于测量整个测试集中模型的准确性。

# %%
test_accuracy = tfe.metrics.Accuracy()

for (x, y) in test_dataset:
  logits = model(x)
  prediction = tf.argmax(logits, axis=1, output_type=tf.int32)
  test_accuracy(prediction, y)

print("Test set accuracy: {:.3%}".format(test_accuracy.result()))

# %% [markdown]
# 我们在最后一批看到评估结果，例如，模型通常是正确的：

# %%
tf.stack([y,prediction],axis=1)

# %% [markdown]
# ## 模型预测
# 
# 我们已经训练了一个模型并且“证明”它对鸢尾花物种进行分类是好的 — 但不是完美的。现在让我们使用训练好的模型对[未标记的样本](https://developers.google.com/machine-learning/glossary/#unlabeled_example)进行预测;也就是说，包含特征但不包含标签的样本。
# 
# 在现实生活中，未标记的样本可能来自许多不同的来源，包括应用程序，CSV 文件和数据源。目前，我们将手动提供三个未标记的样本来预测其标签。回想一下，标签号被映射到命名表示，如下所示：
# 
# * `0`: Iris setosa
# * `1`: Iris versicolor
# * `2`: Iris virginica

# %%
predict_dataset = tf.convert_to_tensor([
    [5.1, 3.3, 1.7, 0.5,],
    [5.9, 3.0, 4.2, 1.5,],
    [6.9, 3.1, 5.4, 2.1]
])

predictions = model(predict_dataset)

for i, logits in enumerate(predictions):
  class_idx = tf.argmax(logits).numpy()
  p = tf.nn.softmax(logits)[class_idx]
  name = class_names[class_idx]
  print("Example {} prediction: {} ({:4.1f}%)".format(i, name, 100*p))

