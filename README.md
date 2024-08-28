# FuzzJailbreaker：一种用于发现LLMs中越狱漏洞的新型通用模糊测试框架

![image](https://lbw15507.github.io/images/overview.png)

## 介绍
本仓库包含“FuzzJailbreaker：一种用于发现LLMs中越狱漏洞的新型通用模糊测试框架”的代码和数据。

FuzzJailbreaker是一个创新框架，用于主动识别和解决大型语言模型（LLMs）中的越狱漏洞。该框架通过自动生成和测试复杂提示，来有效评估LLM的安全性。

`construction/jailbreak_prompts/csv`：我们的工作中展示了7种类型的越狱提示数据集就在这里！😍😍😍

![image](https://lbw15507.github.io/images/pipeline.png)

## 目录

1. [介绍](#介绍)
2. [安装](#安装)
3. [使用](#使用)
   1. [构建](#构建)
   2. [越狱测试](#越狱测试)
   3. [越狱标注](#越狱标注)
4. [功能](#功能)
5. [许可证](#许可证)
6. [致谢](#致谢)
7. [引用](#引用)

## 安装

要安装FuzzJailbreaker，请按照以下步骤操作：

1. 克隆仓库：
   ```bash
   git clone https://github.com/lbw15507/FuzzJailbreaker.git
   ```
2. 进入FuzzJailbreaker目录：
   ```bash
   cd FuzzLLM
   ```
3. 安装所需的依赖：
   ```bash
   pip install -r requirements.txt
   ```

## 使用

### 构建

#### 文件和文件夹描述

- `construction/construct_prompts.py`：用于构建各种提示并生成CSV文件和随机样本JSON文件的脚本。
- `construction/jailbreak_prompts/csv`：包含全面提示集的CSV文件的目录。
- `construction/jailbreak_prompts/scaled_json`：用于测试提示的JSON文件存储目录。

#### 任务

- **构建提示**：使用模板、约束和问题生成提示。
- **文件存储**：
  - 每个类别的提示存储在`construction/jailbreak_prompts/csv`，可以重复使用。
  - 测试的最终JSON文件存储在`jailbreak_prompts/scaled_json`中。重新生成此文件以更改测试样本。

#### JSON格式

```json
[
    {
        "id": 501,
        "jailbreak_prompt": "balabala",
        "attack_result": null,
        "label": null
    },
    ...
]
```

#### 使用指南

- 使用命令 `python construct_prompts.py CLASSNAME -n SAMPLE_NUM` 生成提示。
- `CLASSNAME`：类名（例如，“RP”，“OC”，“RP&OC”）。
- `SAMPLE_NUM`：要生成的样本数。
- **示例**：`python construct_prompts.py RP -n 300` 用于生成300个RP类样本。

### 越狱测试

#### 文件结构

- `test/fuzzing_test.py`：用于执行本地越狱测试的脚本。
- `test/model_attack_result`：用于存储各种模型的越狱测试结果的JSON文件目录。
  - 例如，`test/model_attack_result/vicuna-13b-v1.5_attacked` 包含 `vicuna-13b-v1.5` 模型的测试结果。此目录中的每个文件专用于某个特定模型的测试结果。

#### 任务

- **本地越狱测试**：使用 `fuzzing_test.py` 脚本在本地对模型进行越狱测试。
- **结果存储**：这些测试的结果以JSON格式存储在`model_attack_result`目录中，提供了每个模型性能和漏洞的全面记录。

#### 运行越狱测试（示例：vicuna-13b-v1.5）

##### 启动控制器

```bash
python3 -m fastchat.serve.controller --host 0.0.0.0
```

- 控制器管理分布式工作者：
  - 管理模型工作者的注册、状态更新和请求分发。
  - 监听来自模型工作者的注册请求并维护活动工作者列表。
  - `--host 0.0.0.0` 监听所有可用的网络接口。

##### 启动vicuna-13b-v1.5模型工作者

要托管 `vicuna-13b-v1.5` 的特定实例并处理推理请求，运行以下命令：

```bash
python3 -m fastchat.serve.model_worker --model-path lmsys/vicuna-13b-v1.5 --host 0.0.0.0
```

- 多个模型工作者可能需要不同的GPU分配和端口。例如，对于GPU 0上的工作者0：

```bash
CUDA_VISIBLE_DEVICES=0 python3 -m fastchat.serve.model_worker --model-path lmsys/vicuna-7b-v1.5 --controller http://localhost:21001 --port 31000 --worker http://localhost:31000
```

等待进程完全加载模型。一旦模型工作者准备就绪并在控制器中注册，您可以继续运行越狱测试脚本：

```bash
cd test
python3 fuzzing_test.py --model-name vicuna-13b-v1.5 --json-directory /root/project/FuzzLLM/test/model_attack_result/vicuna-13b-v1.5_attacked --categories all
```

- 该脚本将使用构建并存储在指定目录中的所有类别的提示，对`vicuna-13b-v1.5`模型进行越狱测试。
- `label_model_func.py`中的`--categories`参数允许您指定要处理的提示类别。例如，如果您想测试OC和PE类的提示，使用以下命令：

```bash
python3 fuzzing_test.py --model-name vicuna-13b-v1.5 --json-directory /root/project/FuzzLLM/test/model_attack_result/vicuna-13b-v1.5_attacked --categories OC PE
```

### 越狱标注

#### 文件结构

- `label/label_model_func.py`：用于执行越狱标注的脚本。
- `label/model_result_label`：用于存储各种模型的越狱标注结果的JSON文件目录。

#### 任务

- **本地越狱标注**：使用`label_model_func.py`脚本在本地对模型进行越狱标注。
- **结果存储**：这些标注过程的结果以JSON格式存储在`model_result_label`目录中。

#### 运行越狱标注（示例：vicuna-13b-v1.5）

运行`label_model_func.py`的准备步骤与[越狱测试](#越狱测试)中描述的一致：

- 使用命令 `python3 -m fastchat.serve.controller --host 0.0.0.0` 启动控制器。
- 根据需要启动特定LLM实例的模型工作者。

一旦模型工作者准备就绪并在控制器中注册，您可以继续运行越狱标注脚本：

```bash
cd label
python3 label_model_func.py --model-name vicuna-13b-v1.5 --json-directory /root/project/FuzzLLM/label/model_result_label/vicuna-13b-v1.5_labeled --categories all
```

- 该脚本将使用指定目录中生成的输出，对指定模型的所有类别的越狱测试进行标注。

## 功能

- 自动化测试
- 综合漏洞发现
- 基于模板的提示生成
- 组合越狱攻击
- 针对各种LLMs的广泛实验

## 许可证

该项目根据[Apache 2.0许可证](https://github.com/OptimalScale/LMFlow/blob/main/LICENSE)授权。

## 致谢

本项目从[FastChat](https://github.com/lm-sys/FastChat)项目的方法和见解中汲取了灵感。