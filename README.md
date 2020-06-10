# ComputerVisionOnAzure
使用计算机视觉服务处理图像，借助微软提供的计算机视觉 API 分析图像来获得见解、从图像中提取文本，以及生成优质的缩略图。

## 前言
构建复杂的图像处理和分析引擎非常昂贵。 一种替代方法是使用 Microsoft 的计算机视觉 API。 在本手册中，我们将探讨此 API 提供的功能，并调用该功能处理一些图像。
## 准备工作
1. Azure账户——创建“计算机视觉”服务
2. Python/C#/Java/Go/Js 开发环境
> 本文主要以python为例讲解
----
### 作者在开始编写这个手册的时候并未深入学习python，只使用了Azure Cloud Shell对该API进行了调试和使用，如果你也还没掌握python又想体验微软视觉API的功能，请继续参阅本手册，否则建议在这个链接，查找更多教程！
----
## 配置
1. 获取试用订阅或资源的密钥（在Azure控制台获取）
2. 为该密钥和终结点 URL 创建环境变量，分别名为 COMPUTER_VISION_SUBSCRIPTION_KEY 和 COMPUTER_VISION_ENDPOINT
