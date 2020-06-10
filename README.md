# ComputerVisionOnAzure
使用计算机视觉服务处理图像，借助微软提供的计算机视觉 API 分析图像来获得见解、从图像中提取文本，以及生成优质的缩略图。
# 本文在[玩具盒](https://www.hacbox.studio/index.php/%e7%8e%a9%e5%85%b7%e7%9b%92/)同步更新！图片无法阅读请移步至玩具盒！
------------

## 前言
构建复杂的图像处理和分析引擎非常昂贵。 一种替代方法是使用 Microsoft 的计算机视觉 API。 在本手册中，我们将探讨此 API 提供的功能，并调用该功能处理一些图像。
## 准备工作
1. Azure账户——创建“计算机视觉”服务
3. Python/C#/Java/Go/Js 开发环境（非必须）

------------

### 作者在开始编写这个手册的时候并未深入学习python，只使用了Azure Cloud Shell对该API进行了调试和使用，如果你也还没掌握python又想体验微软视觉API的功能，请继续参阅本手册，否则建议在[微软官方手册](https://docs.microsoft.com/zh-cn/azure/cognitive-services/computer-vision/quickstarts-sdk/client-library?pivots=programming-language-python)，查找更多教程！

------------


## 配置
### 1. 启动Cloud Shell
打开Azure控制台，点击右上角的“Azure Cloud Shell”
![Azure控制台](/wp-content/uploads/2020/06/TIM截图20200609162333.png)
这里会出现选择配置环境，建议选择PowerShell

如果是第一次配置，会出现下面的提示，打开左边的“高级选项”
![创建存储](/wp-content/uploads/2020/06/TIM截图20200609162333-1.png)

这里的配置按照自己的喜好来即可，注意：
- 区域一定要选择你当前所在的区域；
- 存储账户名称的长度必须在 3 到 24 个字符之间，并且只能使用数字和小写字母；
- 用户设置文件共享名称只能包含小写字母、数字和连字符，并且必须以字母或数字开头。该名称不能包含两个连续的连字符；
![创建存储](/wp-content/uploads/2020/06/TIM截图20200609162333-2.png)
之后点击“创建存储”，稍等片刻
*有可能会报错，多半是命名不合法，按照提示修改即可*

### 2. 创建认知服务帐户
等待Cloud Shell加载完成，显示欢迎信息则说明成功创建。
![认知服务帐户](/wp-content/uploads/2020/06/TIM截图20200609162333-3.png)
*上方的四行数据建议另存至记事本方便查询*

我们需要 API 访问密钥来调用计算机视觉 API。 要获取访问密钥，我们需要一个计算机视觉 API 的认知服务帐户。 我们将使用 `az cognitiveservices create` 在订阅中创建帐户。

命令 `az cognitiveservices create` 用于在资源组中创建认知服务帐户。 调用此命令时，必须提供以下五个参数。

|  参数 |  说明 |
| :------------ | :------------ |
| resource-group  |  将拥有认知服务帐户的资源组，**和前面创建的存储账户信息一致。** |
|  kind |  认知服务帐户的 API 名称。 针对此练习为“计算机视觉”，但可以基于内容审查器、人脸 API 等。 |
| name  |  认知服务帐户名称 |
|  sku | 认知服务帐户的 Sku（库存单位）。 针对此练习为 F0（免费层），但可以是 F0、S1、S2、S3 或 S4。*（S层都是收费层）*  |
|  location |  要从中调用此 API 的位置或区域， **和前面创建的存储账户信息一致。** |

将下面这段代码输入控制台：
```shell
az cognitiveservices account create
--kind ComputerVision
--name ComputerVisionService
--sku F0
--resource-group Demo_ComputerVision
--location southeastasia
```
我们为 `ComputerVision API` 创建了认知服务帐户。 我们选择了 `F0 SKU` 并将帐户命名为`“ComputerVisionService”`。 我们的帐户属于资源组 `Demo_ComputerVision`，并且我们将从在 `location` 参数中设置的位置调用 API。

命令完成认知服务帐户的创建后，你会获得 JSON 响应，其中包括已设置为“成功”的“provisioningState”属性。
![创建认知服务帐户](/wp-content/uploads/2020/06/TIM截图20200609162333-4.png)

### 3. 获取访问密钥
成功创建帐户后，我们可以检索此帐户的订阅密钥或访问密钥。
1. 在 Azure Cloud Shell 中执行以下命令

```shell
az cognitiveservices account keys list 
--name ComputerVisionService 
--resource-group Demo_ComputerVision 
```

上述命令将返回与名为“ComputerVisionService”的认知服务帐户相关联的密钥，该帐户属于给定的资源组。 它将返回两个密钥 - 其中一个是备用密钥。 密钥很难记住，因此我们将第一个密钥存储在变量中，以便将变量用于对 API 的所有调用。

![获取访问密钥](/wp-content/uploads/2020/06/TIM截图20200609162333-5.png)

2.简化密钥，在 Azure Cloud Shell 中执行以下命令

```shell
$key=$(az cognitiveservices account keys list
--name ComputerVisionService
--resource-group Demo_ComputerVision
--query key1 -o tsv)
```

Azure CLI 2.0 使用 --query 参数对命令的结果执行 JMESPath 查询。 JMESPath 是用于 JSON 的查询语言，提供从 CLI 输出中选择和显示数据的能力。 这些查询在任何显示格式化之前针对 JSON 输出执行。 Azure CLI 中的所有命令均支持 --query 参数。

在本示例中，我们查询名为“key1”条目的密钥列表，并将结果输出为“tsv”格式。 此格式删除字符串值周围的引号。 我们将结果分配给变量密钥。

3. 要查看密钥的值，请在 Azure Cloud Shell 中执行以下命令
```shell
echo $key
```
##### 现在，我们有帐户和密钥，可对 API 执行一些调用了！
## 使用API
### 1. 识别图像中的特征点
让我们首先找到图像中的特征点。 我们将在此示例中使用下面的图像，但你可以随意尝试对其他图像的 URL 使用相同的命令。
![素材](/wp-content/uploads/2020/06/illust_66933557_20200603_202939.jpg)
在 `Azure Cloud Shell` 中执行以下命令。 将命令中的 `<region>` 替换为认知服务帐户的区域。

```shell
curl "https://southeastasia.api.cognitive.microsoft.com/vision/v2.0/analyze?visualFeatures=Categories,Description&details=Landmarks"  
-H "Ocp-Apim-Subscription-Key: $key" 
-H "Content-Type: application/json" 
-d "{'url':'https://hacboxstudio.azurewebsites.net/wp-content/uploads/2020/06/illust_66933557_20200603_202939-300x169.jpg'}" 
| jq '.'
```

其中的`Ocp-Apim-Subscription-Key`后面可以直接输入我们的密钥key1的值，也可以使用声明的变量`key`
![输出结果](/wp-content/uploads/2020/06/TIM截图20200609162333-6.png) 

![输出结果](/wp-content/uploads/2020/06/TIM截图20200609162333-7.png)

- 此调用会查找图像 URL 指定的图像中的特征点。我们现在分析的图像存储在本网页的存储库中。
- 该调用还会要求服务返回图像的类别信息和说明。 返回的说明是一句英文，我们可以看到微软的API对这个图像分析的结果是

> “a group of people standing in front of a window”
可信度： 0.7655691527579038

- 如你所知，每次调用 API 都需要访问密钥。 在请求的 Ocp-Apim-Subscription-Key 标头上进行此设置。
### 2. 检查图像中是否具有不适宜内容
在此示例中，我们将分析图像中的成人内容。 置信度分数对图像包含成人或不雅内容的可能性评分。

我们将在此示例中使用下面的图像，但你可以随意尝试对其他图像的 URL 使用相同的命令。
![测试2](https://docs.microsoft.com/zh-cn/learn/advocates/create-computer-vision-service-to-classify-images/media/3-people.png)

1. 在 Azure Cloud Shell 中执行以下命令，替换 URL 中的 `<region>`。

```shell
curl "https://<region>.api.cognitive.microsoft.com/vision/v2.0/analyze?visualFeatures=Adult,Description" \
-H "Ocp-Apim-Subscription-Key: $key" \
-H "Content-Type: application/json" \
-d "{'url' : 'https://raw.githubusercontent.com/MicrosoftDocs/mslearn-process-images-with-the-computer-vision-service/master/images/people.png'}" \
| jq '.'
```
![输出结果2](/wp-content/uploads/2020/06/TIM截图.png)

在此示例中，我们将 `visualFeatures` 设置为 `Adult,Description`。

可以看到输出结果中多了两个参数，响应提供了两个置信度得分，一个针对不雅内容，另一个针对成人内容。 通过使用这些分数以及图像说明和其他可视功能，可以开始标记发布到服务器的图像。

### 3. 调用计算机视觉 API 以生成缩略图

计算机视觉首先生成高质量缩略图，然后通过分析图像中的对象来确定感兴趣区域 (ROI)。 然后，计算机视觉会裁剪图像以满足感兴趣区域的要求。 可以用户需求使用与原始图像的纵横比不同的纵横比显示生成的缩略图。 让我们看看该项目的运行情况。

`generateThumbnail` 操作将创建具有用户指定的宽度和高度的缩略图。 默认情况下，服务会分析图像，标识感兴趣区 (ROI)，并根据 ROI 生成智能裁剪坐标。 当指定不同于输入图像的纵横比时，智能裁剪非常有用。 请求 URI 的格式如下：

`https://<region>.api.cognitive.microsoft.com/vision/v2.0/generateThumbnail?width=<...>&height=<...>&smartCropping=<...>`

可将不同的参数提供给该 API，以生成符合需要的适当缩略图。 `width` 和 `height` 参数是必需的。 它们将让 API 知道特定图像的所需大小。 `smartCropping` 参数通过分析图像中的所需区域来生成更智能的裁剪，使裁剪部分保留在缩略图中。 例如，启用智能裁剪后，经裁剪的个人资料图片会使相关人员的人脸部分保留在图片框内，即使此图像具有不同的纵横比。

#### 生成缩略图

我们将在此示例中使用下面的图像，但你可以随意尝试对其他图像的 URL 使用相同的命令。
![测试3](https://docs.microsoft.com/zh-cn/learn/advocates/create-computer-vision-service-to-classify-images/media/4-dog.png)
1. 在 Azure Cloud Shell 中执行以下命令。 将命令中的 `<region>` 替换为认知服务帐户的区域

```shell
curl "https://<region>.api.cognitive.microsoft.com/vision/v2.0/generateThumbnail?width=100&height=100&smartCropping=true" \
-H "Ocp-Apim-Subscription-Key: $key" \
-H "Content-Type: application/json" \
-d "{'url' : 'https://raw.githubusercontent.com/MicrosoftDocs/mslearn-process-images-with-the-computer-vision-service/master/images/dog.png'}" \
-o  thumbnail.jpg
```
在此示例中，我们将要求服务创建尺寸为 100x100 的缩略图。 智能裁剪已启用。 成功的响应包含缩略图二进制文件，我们会将其写入一个名为 thumbnail.jpg 的文件。

#### 查看生成的缩略图

可以在 Azure Cloud Shell 存储帐户中找到生成的缩略图。 我们已将文件命名为 thumbnail.jpg。
按照以下说明，通过 Azure 门户下载缩略图。
1. 在 Azure Cloud Shell 中执行以下命令，确认主文件夹中存在文件 thumbnail.jpg。
```shell
cd ~
ls -l
```
2. 执行以下命令，以将 `thumbnail.jpg` 移动到 `clouddrive` 文件夹。
```shell
mv ~/thumbnail.jpg ~/clouddrive
```
3. 登录[Azure 门户](https://portal.azure.com/)

4. 在 Azure 门户菜单上或在门户主页中，选择“所有资源”，然后选择名称以 `cloudshell` 开头的存储帐户。

5. 在存储帐户面板中，依次选择“存储资源管理器”、“文件共享”，再选择该集合中名称以 `cloudshellfiles` 开头的文件共享。

![下载缩略图](/wp-content/uploads/2020/06/TIM截图-1.png)

6. 选择`“thunbnail.jpg”`文件，然后从顶部菜单单击“下载”以查看图像。

生成的缩略图：![缩略图](/wp-content/uploads/2020/06/thumbnail.jpg)

### 4. 从图像中检测并提取手写文字
我们将在此示例中使用下面的图像，但你可以随意尝试对其他图像的 URL 使用相同的命令。

![手写文字](https://docs.microsoft.com/zh-cn/learn/advocates/create-computer-vision-service-to-classify-images/media/6-handwriting.jpg)

1. 在 Azure Cloud Shell 中执行以下命令。 将命令中的 `<region>` 替换为认知服务帐户的区域

```shell
curl "https://<region>.api.cognitive.microsoft.com/vision/v2.0/recognizeText?mode=Handwritten" \
-H "Ocp-Apim-Subscription-Key: $key" \
-H "Content-Type: application/json" \
-d "{'url' : 'https://raw.githubusercontent.com/MicrosoftDocs/mslearn-process-images-with-the-computer-vision-service/master/images/handwriting.jpg'}" \
-D -
```

上述命令可将此操作的标头转储到控制台。 例如：

```shell
HTTP/1.1 202 Accepted
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 0
Expires: -1
Operation-Location: https://westus2.api.cognitive.microsoft.com/vision/v2.0/textOperations/d0e9b397-4072-471c-ae61-7490bec8f077
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
apim-request-id: f5663487-03c6-4760-9be7-c9157fac10a1
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
Date: Wed, 12 Sep 2018 19:22:00 GMT
```

`Operation-Location` 标头是完成后将发布结果的位置。

2. 复制 `Operation-Location` 标头值。
3. 在 Azure Cloud Shell 中执行以下命令，将 ``"<Operation-Location>"`` 替换为上述步骤中复制的 `Operation-Location` 标头的值。

```shell
curl -H "Ocp-Apim-Subscription-Key: $key" "<Operation-Location>" | jq '.'
```
如果操作已完成，你将收到包含手写识别请求结果的 `JSON` 文件

输出展示：![输出OCR](/wp-content/uploads/2020/06/输出OCR.gif)

## 结语
计算机视觉 API 是用于处理图像的强大工具。使用此服务，而不是构建你自己的 API，有可能为你节省大量开发和维护成本。 专注于编写业务逻辑来作出业务相关决策，并让计算机视觉 API 从图像中提取丰富的信息，以便分类和处理可视数据。

在此手册中，我们分析了图像，提取了文本，并生成了缩略图。 为调用计算机视觉 API，我们在 Azure 订阅中创建了认知服务帐户。 此帐户为我们提供调用所需的访问密钥。

本文仅通过Azure Cloud Shell展示这个API的功能，如果需要在个人项目中调用该API，后续我会再单独写一篇手册，如果等不及的话，可以在[微软官方手册](https://docs.microsoft.com/zh-cn/azure/cognitive-services/computer-vision/quickstarts-sdk/client-library?pivots=programming-language-python)中查询有关资料。

本文部分内容引用自[微软官方手册](https://docs.microsoft.com/zh-cn/azure/cognitive-services/computer-vision/quickstarts-sdk/client-library?pivots=programming-language-python)和[Microsoft Learn教程](https://docs.microsoft.com/zh-cn/learn/modules/create-computer-vision-service-to-classify-images/1-introduction)


*写的好累...*
