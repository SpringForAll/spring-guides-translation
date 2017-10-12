# 翻译指南


>  本指南的目的教您怎么实际操作,在申请翻译前请先阅读[申领流程](https://github.com/SpringForAll/spring-guides-translation/blob/master/translate-readme.md)

## 准备

在开始翻译前，您至少需要提前准备如下几样东西：

* Markdown 编辑器 ：[Typora](https://typora.io/#download)
* Git 客户端：`*nix` : teminal， `windows` : git cmd
* Github 账号



## 申请翻译

1.  找到源码仓库（https://github.com/SpringForAll/spring-guides-translation）

2.  点击`Fork` ：

   ![Fork仓库](http://oad0bexwv.bkt.clouddn.com/1.jpg)

3. 选择想要翻译的文章

   ![选择想要翻译的文章](http://oad0bexwv.bkt.clouddn.com/2.jpg)

4. 文章申请，

   a. 先在`source` 目录下找到您想翻译的文章，文件名和文章的名字相同；

   b. 回到`Readme` 中根据文件名找到原文路径(超链接形式，点击就行)；

   c.  选中文章的主要内容复制到`Typora` ，检查文章中是否有遗漏段落;

   d.  将翻译小队约定好的头文件，协议信息等填写好，之后将内容覆盖仓库中的文件；

   e.  提交PR，等待合并，**翻译小组承诺一定以最快的速度merge翻译申请**

   ![文章申请过程GIF](https://github.com/chenzhijun/spring-guides-translation/blob/master/static/2.gif)

   ​

## 翻译

在您的 PR 被 merge 之后，您就可以开始翻译了。您可以选择将仓库克隆到本地进行翻译（推荐方式），当然您也可以选择在远程 web 网页上直接翻译。

### 在本地翻译

首先您需要克隆您的仓库，并且保持仓库为最新状态：

#### 克隆仓库

选择一个本地文件夹，如果使用`*nix` 系统你可以使用像这样：`mkdir -p /home/translate` ；如果使用 windows ，你可以创建一个`tranlaste`文件夹。

进入到文件夹中，将我们的仓库克隆到本地：

``` shell
## 请将下面的仓库地址换成自己的地址 
git clone https://github.com/chenzhijun319/spring-guides-translation.git 

```

#### 保持和远端库同步

很多时候，翻译小组的远端库需要合并大量更新，所以您有必要将自己的仓库和远端库保持同步。

```shell
## 保持和远程仓库同步，即保持和 SpringForAll/spring-guides-translation 的仓库同步
git remote add upstream https://github.com/SpringForAll/spring-guides-translation.git

git fetch upstream

git merge upstream/master
```

>  如果不想敲命令，您可以直接删除自己的库，然后重新 Fork 仓库(https://github.com/SpringForAll/spring-guides-translation).

#### 开始翻译

同步最新的库之后，您就可以开始您的翻译了，用`Typora` 打开您申领的文章，之后就可以开始翻译了，**请注意您翻译之后的文章会在校对之后发布在`spring4all.com` 的官网上，请您尽量保持翻译的准确性，对此翻译小组表示衷心的感谢**。

#### 结束翻译

翻译完之后，你可以通过先将翻译后的文章`push`到您的仓库:

```shell
## 提交
git add .
git commit -m "将xx文章翻译完，待校对"

## 推送到github自己的远程库
git push origin master
```

完成`push`的时候之后，您只需要再提交一次`PR` 就可以了。



### 在 web 翻译

由于非常多的人同时提交PR，请您一定要保持和远程库(https://github.com/SpringForAll/spring-guides-translation)的同步，尽量只更改您申领的文章，以防出现冲突。

您在web端翻译，只需选择您申请的文章，然后点击编辑，完成翻译后点击下方的保存。之后跟申领翻译一样，提交`PR` 就行了。**请注意您翻译之后的文章会在校对之后发布在`spring4all.com` 的官网上，请您尽量保持翻译的准确性，对此翻译小组表示衷心的感谢**



## 结束语

SpringForAll 翻译小组成立时间不长，很多事情也都没有太多经验，存在太多不足，但是我们会慢慢改进，更好的为社区，为技术爱好者提供更多的资源。也希望您能积极参与我们，和我们一起将这件事情做得更好。如果您有什么好的建议或者意见，您可以在群里和我们沟通：
