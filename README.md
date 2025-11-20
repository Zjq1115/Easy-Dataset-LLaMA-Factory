# Easy Dataset × LLaMA Factory: 让大模型高效学习领域知识

Easy Dataset 是一个专为创建大型语言模型（LLM）微调数据集而设计的应用程序。它提供了直观的界面，用于上传特定领域的文件，智能分割内容，生成问题，并为模型微调生成高质量的训练数据。支持使用 OpenAI、DeepSeek、火山引擎等大模型 API 和 Ollama 本地模型调用。LLaMA Factory 是一款开源低代码大模型微调框架，集成了业界最广泛使用的微调技术，支持通过 Web UI 界面零代码微调大模型，目前已经成为开源社区最热门的微调框架之一，GitHub 星标超过 4.6 万。支持全量微调、LoRA 微调、以及 SFT 和 DPO 等微调算法。
* 声明：数据及流程均可公开获得，不涉及任何涉密或隐私信息，本教程使用 Easy Dataset 从五家互联网公司的公开财报构建 SFT 微调数据，并使用 LLaMA Factory 微调 Qwen2.5-3B-Instruct 模型

* 前提准备：

  可在阿里云DSW平台创建以下实例：镜像：modelscope:1.25.0-paddle3.0.0-gpu-py310-cu123-ubuntu20.04；实例规格：ecs.gn7i-c8g1.2xlarge；

  前往 Node.js 和 pnpm 官网安装环境：https://nodejs.org/en/download | https://pnpm.io/；

  使用以下代码检查 Node.js 版本是否高于 18.0：

  ```shell
  node -v
  ```

  升级apt工具和安装前端工具依赖：

  ```shell
  sudo apt-get update
  sudo apt-get install -y \
      build-essential \
      libcairo2-dev \
      libpango1.0-dev \
      libjpeg-dev \
      libgif-dev \
      librsvg2-dev
  ```

## 一、使用 Easy Dataset 生成微调数据

从 GitHub 拉取 Easy Dataset 仓库：

```shell
git clone https://github.com/ConardLi/easy-dataset.git
```

安装软件依赖：

```shell
cd easy-dataset && npm install
```

启动 Easy Dataset 应用:

```shell
npm run build
npm run start
```

本流程使用开源的互联网公司财报作为示例数据，包含五篇国内互联网公司 2024 年二季度的财报，格式包括 txt 和 markdown。

```shell
git clone https://github.com/llm-factory/FinancialData-SecondQuarter-2024.git
```

在浏览器进入 Easy Dataset 主页后，点击创建项目。

![image-20251118183159824](/pics/image-20251118183159824.png)

首先填写**项目名称**（必填），其他两项可留空，点击确认**创建项目**。项目创建后会跳转到**项目设置**页面，打开**模型配置**，选择数据生成时需要调用的大模型 API 接口

![image-20251118183226229](/pics/image-20251118183226229.png)

修改模型**提供商**和**模型名称**，填写**API密钥**，点击保存后将数据**保存**到本地，在右上角选择配置好的模型。

![image-20251118183630757](/pics/image-20251118183630757.png)

打开**任务配置**页面，设置文本分割长度为最小 500 字符，最大 1000 字符。在问题生成设置中，修改为每 10 个字符生成一个问题，修改后在页面最下方**保存任务配置**。

![image-20251118183646070](/pics/image-20251118183646070.png)

打开**文献处理**页面，选择并上传示例数据文件，选择文件后点击**上传并处理文件**。

![image-20251118183702259](/pics/image-20251118183702259.png)

上传后会调用大模型解析文件内容并分块，耐心等待文件处理完成，示例数据通常需要 2 分钟左右。待文件处理结束后，可以看到文本分割后的文本段，选择全部文本段，点击**批量生成问题**。

![image-20251118183925072](/pics/image-20251118183925072.png)

点击后会调用大模型根据文本块来构建问题，耐心等待处理完成。视 API 速度，处理时间可能在 20-40 分钟不等。处理完成后，打开**问题管理**页面，选择全部问题，点击**批量构造数据集**，耐心等待数据生成。

![image-20251118183948804](/pics/image-20251118183948804.png)

答案全部生成结束后，打开数据集管理页面，点击导出数据集，导出数据集到 LLaMA Factory。

![image-20251118184017449](/pics/image-20251118184017449.png)

在导出配置中选择在**LLaMA Factory**中使用，点击更新**LLaMA Factory**配置，即可在对应文件夹下生成配置文件，点击复制按钮可以将配置路径复制到粘贴板。

![image-20251118184125339](/pics/image-20251118184125339.png)

在配置文件路径对应的文件夹中可以看到生成的数据文件，其中主要关注以下三个文件!

- ataset_info.json：LLaMA Factory 所需的数据集配置文件!
- alpaca.json：以 Alpaca 格式组织的数据集文件
- sharegpt.json：以 Sharegpt 格式组织的数据集文件!

其中**alpaca**和**sharegpt**格式均可以用来微调，两个文件内容相同。

![image-20251118184155724](/pics/image-20251118184155724.png)

## 二、使用LLaMA Factory微调LLM

确认 LLaMA Factory 安装完成后，运行以下指令启动 LLaMA Board：

```shell
export USE_MODELSCOPE_HUB=1 CUDA_VISIBLE_DEVICES=0 && llamafactory-cli webui
```

这里用到的环境变量解释如下：

- `USE_MODELSCOPE_HUB`设为1，表示模型来源是ModelScope。使用HuggingFace模型可能会有网络问题。
- `CUDA_VISIBLE_DEVICES`：指定使用的显卡序号，默认全部使用

点击返回的URL地址，进入Web UI页面。进入 Web UI 界面后，选择模型为 Qwen2.5-3B-Instruct，模型路径可填写本地绝对路径，不填则从互联网下载。

![image-20251118185607292](/pics/image-20251118185607292.png)

将数据路径改为使用 Easy Dataset 导出的配置路径，选择 Alpaca 格式数据集。

![image-20251118185614091](/pics/image-20251118185614091.png)

为了让模型更好地学习数据知识，将学习率改为 1e-4，训练轮数提高到 8 轮。批处理大小和梯度累计则根据设备显存大小调整，在显存允许的情况下提高批处理大小有助于加速训练，一般保持批处理大小×梯度累积×显卡数量等于 32 即可。

![image-20251118185651521](/pics/image-20251118185651521.png)

点击其他参数设置，将保存间隔设置为 50，保存更多的检查点，有助于观察模型效果随训练轮数的变化。

![image-20251118185713246](/pics/image-20251118185713246.png)

点击 LoRA 参数设置，将 LoRA 秩设置为 16，并把 LoRA 缩放系数设置为 32。

![image-20251118185719514](/pics/image-20251118185719514.png)

点击开始按钮，等待模型下载，一段时间后应能观察到训练过程的损失曲线。

![image-20251118185725729](/pics/image-20251118185725729.png)

训练完成后，可验证微调结果。

![image-20251118185732430](/pics/image-20251118185732430.png)

选择检查点路径为刚才的输出目录，打开 Chat 页面，点击加载模型，在下方的对话框中输入问题后，点击提交与模型进行对话，经与原始数据比对发现微调后的模型回答正确。
## 参考文档
https://pai.console.aliyun.com/?regionId=cn-shanghai&spm=5176.12818093_47.resourceCenter.7.47032cc9eSnr3q&workspaceId=949365#/dsw-gallery-workspace/preview/deepLearning/nlp/easy_dataset_llama_factory
