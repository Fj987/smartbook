# Other Computer Vision Problems

# 其它计算机视觉问题

In the previous chapter you learned some important practical techniques for training models in practice. Considerations like selecting learning rates and the number of epochs are very important to getting good results.

在上一章节我们学习了一些在实践中训练模型的一些重要特定技术。考虑的因素像选择学习率和周期数对于获得良好结果是非常重要的。

In this chapter we are going to look at two other types of computer vision problems: multi-label classification and regression. The first one is when you want to predict more than one label per image (or sometimes none at all), and the second is when your labels are one or several numbers—a quantity instead of a category.

在本章节，我们将学习另外两个类型的计算机视觉问题：多标签分类和回归。第一个问题是当我们想要预测单张图像有多个标签时（或有时间根本什么都没有）遇到到，第二个问题是当你的标注是一个或多个时：数量而不是类别。

In the process will study more deeply the output activations, targets, and loss functions in deep learning models.

在这个过程中会学习在深度学习模型中更深层的输出激活、目标和损失函数。

## Multi-Label Classification

## 多标签分类

Multi-label classification refers to the problem of identifying the categories of objects in images that may not contain exactly one type of object. There may be more than one kind of object, or there may be no objects at all in the classes that you are looking for.

多标签分类适用于那种在图像中不可能只包含一个目标类型的识别问题。有可能会超过一个目标类型，或在分类中你根本没有找到目标。

For instance, this would have been a great approach for our bear classifier. One problem with the bear classifier that we rolled out in <chapter_production> was that if a user uploaded something that wasn't any kind of bear, the model would still say it was either a grizzly, black, or teddy bear—it had no ability to predict "not a bear at all." In fact, after we have completed this chapter, it would be a great exercise for you to go back to your image classifier application, and try to retrain it using the multi-label technique, then test it by passing in an image that is not of any of your recognized classes.

例如，我们的熊分类器这一方法也许已经是一个非常好的方法了。在<章节：产品>中我们遇到了一个熊分类器的问题，如果用户上传不包含任何种类熊的图像，模型依然会说它是灰熊、黑熊或泰迪熊的一种。它不具备预测“根本没有熊”的能力。实际上，当我们完成本章节后，你重回图像分类器应用会是很好的练习。用多标签技术尝试重训练模型，然后通过传递给模型一张不包含任何你的识别分类的图像，对它进行测试。

In practice, we have not seen many examples of people training multi-label classifiers for this purpose—but we very often see both users and developers complaining about this problem. It appears that this simple solution is not at all widely understood or appreciated! Because in practice it is probably more common to have some images with zero matches or more than one match, we should probably expect in practice that multi-label classifiers are more widely applicable than single-label classifiers.

在实践中，我们没有看到很多人们以此为目的训练多标签分类器的例子，但是我们常常看到用户和开发人员抱怨这个问题。表明这个简单的解决方案根本没有被广泛的理解或认可！由于在实践中可能更常见的是很多零匹配或超过一个匹配项的图片，相比单标签分类器，我们可能应该期望在实践中多标签分类器更更广泛的应用。

First, let's see what a multi-label dataset looks like, then we'll explain how to get it ready for our model. You'll see that the architecture of the model does not change from the last chapter; only the loss function does. Let's start with the data.

首先，我们看一下多标签数据集看起来是什么样子，然后我们会解释如何为我们的模型把它准备好。你会在看到从上一章节开始，模型的架构没有做变化，只是变了损失函数。让我们从数据开始。

### The Data

### 数据

For our example we are going to use the PASCAL dataset, which can have more than one kind of classified object per image.

We begin by downloading and extracting the dataset as per usual:

我们的例子会使用PASCAL数据集，这个数据集对每张图像有超过一个种类的分类对象。与平常一样我们开始下载并抽取数据集：

```
from fastai.vision.all import *
path = untar_data(URLs.PASCAL_2007)
```

This dataset is different from the ones we have seen before, in that it is not structured by filename or folder but instead comes with a CSV (comma-separated values) file telling us what labels to use for each image. We can inspect the CSV file by reading it into a Pandas DataFrame:

这个数据集与我们以前看到的那些是不同的，也就是说它没有通过文件名或文件夹结构化，而是用CSV（逗号分割值）文件告诉我们每一张图像所用的标签是什么。我们能够把它读取到Pandas DataFrame里，来查看这个CSV文件：

```
df = pd.read_csv(path/'train.csv')
df.head()
```

<table style="width: 370px;border-collapse: collapse;" >
  <tr>
    <td  style="width: 10px;" align="center"></td>
    <td  style="width: 100px;" align="center">fname</td>
    <td  style="width: 160px;" align="center">labels</td>
    <td  style="width: 100px;" align="center">is_valid</td>
  </tr>
    <td align="center" font-weight="bold">0</td>
    <td align="right">000005.jpg</td>
  	<td align="right">chair</td>
  	<td align="right">True</td>
  </tr>
  </tr>
    <td align="center">1</td>
    <td align="right">000007.jpg</td>
  	<td align="right">car</td>
  	<td align="right">True</td>
  </tr>
  </tr>
    <td align="center">2</td>
    <td align="right">000009.jpg</td>
  	<td align="right">horse person</td>
  	<td align="right">True</td>
  </tr>
  </tr>
    <td align="center">3</td>
    <td align="right">000012.jpg</td>
  	<td align="right">car</td>
  	<td align="right">False</td>
  </tr>
  </tr>
    <td align="center" >4</td>
    <td align="right">000016.jpg</td>
  	<td align="right">bicycle</td>
  	<td align="right">True</td>
  </tr>
</table>

As you can see, the list of categories in each image is shown as a space-delimited string.

正如你能看到的，每张图像的分类列表被显示为一个空格分割字符串。