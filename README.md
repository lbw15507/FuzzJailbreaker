# FuzzJailbreaker: 一种用于发现LLMs中越狱漏洞的创新通用模糊测试框架
> 可访问 [作品主页](https://mumuyeye.github.io/DtFormer/README.html)。
![image](https://lbw15507.github.io/images/overview.png)

## 介绍

本仓库提供了“FuzzJailbreaker: 一种用于发现LLMs中越狱漏洞的新型通用模糊测试框架”的完整代码和数据集。

FuzzJailbreaker 是一个前沿的研究框架，旨在主动检测和解决大型语言模型（LLMs）中的越狱漏洞。通过自动生成和测试多样化的提示，该框架能够全面评估LLM的安全性，并帮助研究人员和开发者预防潜在的安全风险。

`construction/jailbreak_prompts/csv`：该目录包含了我们研究中所展示的7种类型的越狱提示数据集，旨在提供广泛的测试覆盖。 😍😍😍

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

要安装 FuzzJailbreaker，请按照以下步骤操作：

1. 克隆仓库：
   ```bash
   git clone https://github.com/lbw15507/FuzzJailbreaker.git
   ```
2. 进入 FuzzJailbreaker 目录：
   ```bash
   cd FuzzJailbreaker
   ```
3. 安装所需的依赖：
   ```bash
   pip install -r requirements.txt
   ```

## 使用

### 构建

#### 文件和文件夹描述

- `construction/construct_prompts.py`：用于构建多种提示并生成CSV文件和随机样本JSON文件的脚本。
- `construction/jailbreak_prompts/csv`：存放各种提示集合的CSV文件目录。
- `construction/jailbreak_prompts/scaled_json`：用于存储用于测试的提示的JSON文件目录。

#### 任务

- **构建提示**：使用模板、约束和问题自动生成测试提示。
- **文件存储**：
  - 每种类型的提示被存储在 `construction/jailbreak_prompts/csv` 目录中，可多次重复使用。
  - 生成的最终测试JSON文件存储在 `jailbreak_prompts/scaled_json` 目录中。可重新生成此文件以改变测试样本。

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

- 使用命令 `python construct_prompts.py CLASSNAME -n SAMPLE_NUM` 来生成指定类型和数量的提示。
- `CLASSNAME`：类名（例如，“RP”，“OC”，“RP&OC”）。
- `SAMPLE_NUM`：要生成的样本数量。
- **示例**：`python construct_prompts.py RP -n 300` 将生成300个RP类别的样本。

### 越狱测试

#### 文件结构

- `test/fuzzing_test.py`：执行本地越狱测试的主要脚本。
- `test/model_attack_result`：用于存储各模型越狱测试结果的JSON文件目录。
  - 例如，`test/model_attack_result/vicuna-13b-v1.5_attacked` 存储了 `vicuna-13b-v1.5` 模型的测试结果，每个文件对应于特定模型的测试数据。

#### 任务

- **本地越狱测试**：利用 `fuzzing_test.py` 脚本在本地环境中对模型进行越狱测试。
- **结果存储**：测试结果以JSON格式存储在 `model_attack_result` 目录中，详细记录了每个模型的性能和潜在漏洞。

#### 运行越狱测试（示例：vicuna-13b-v1.5）

##### 启动控制器

```bash
python3 -m fastchat.serve.controller --host 0.0.0.0
```

- 控制器管理分布式工作者节点：
  - 负责模型工作者的注册、状态更新和请求分发。
  - 监听来自模型工作者的注册请求，并维护活跃工作者列表。
  - `--host 0.0.0.0` 表示监听所有可用网络接口。

##### 启动vicuna-13b-v1.5模型工作者

要托管 `vicuna-13b-v1.5` 模型实例并处理推理请求，请运行以下命令：

```bash
python3 -m fastchat.serve.model_worker --model-path lmsys/vicuna-13b-v1.5 --host 0.0.0.0
```

- 对于需要不同GPU资源和端口的多个模型工作者，可以使用如下命令。例如，GPU 0上的工作者0：

```bash
CUDA_VISIBLE_DEVICES=0 python3 -m fastchat.serve.model_worker --model-path lmsys/vicuna-7b-v1.5 --controller http://localhost:21001 --port 31000 --worker http://localhost:31000
```

等待进程完全加载模型。一旦模型工作者准备就绪并在控制器中注册，您可以运行越狱测试脚本：

```bash
cd test
python3 fuzzing_test.py --model-name vicuna-13b-v1.5 --json-directory /root/project/FuzzLLM/test/model_attack_result/vicuna-13b-v1.5_attacked --categories all
```

- 该脚本会对 `vicuna-13b-v1.5` 模型进行越狱测试，测试数据来自指定目录中的所有提示类别。
- `label_model_func.py` 中的 `--categories` 参数允许您指定要测试的提示类别。例如，如果只测试 OC 和 PE 类提示，请使用以下命令：

```bash
python3 fuzzing_test.py --model-name vicuna-13b-v1.5 --json-directory /root/project/FuzzLLM/test/model_attack_result/vicuna-13b-v1.5_attacked --categories OC PE
```

### 越狱标注

#### 文件结构

- `label/label_model_func.py`：用于执行越狱标注的脚本。
- `label/model_result_label`：用于存储各模型越狱标注结果的JSON文件目录。

#### 任务

- **本地越狱标注**：使用 `label_model_func.py` 脚本在本地标注越狱测试结果。
- **结果存储**：标注结果以JSON格式存储在 `model_result_label` 目录中。

#### 运行越狱标注（示例：vicuna-13b-v1.5）

运行 `label_model_func.py` 的准备步骤与 [越狱测试](#越狱测试) 中的步骤一致：

- 使用命令 `python3 -m fastchat.serve.controller --host 0.0.0.0` 启动控制器。
- 启动所需的LLM实例模型工作者。

一旦模型工作者就绪并在控制器中注册，您可以继续运行越狱标注脚本：

```bash
cd label
python3 label_model_func.py --model-name vicuna-13b-v1.5 --json-directory /root/project/FuzzLLM/label/model_result_label/vicuna-13b-v1.5_labeled --categories all
```

- 该脚本将标注所有类别的越狱测试结果，并将标注数据存储在指定目录中。

## 功能

- 自动化安全测试
- 全面的漏洞发现
- 基于模板的提示生成
- 组合型越狱攻击模拟
- 针对多种LLMs的广泛实验

## 许可证

该项目根据 [Apache 2.0 许可证](https://github.com/OptimalScale/LMFlow/blob/main/LICENSE) 进行授权。

## 致谢

本项目受到 [FastChat](https://github.com/lm-sys/FastChat) 项目方法和见解的启发。