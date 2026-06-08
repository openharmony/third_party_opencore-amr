# 三方开源软件 opencore-amr

- [三方开源软件 opencore-amr](#三方开源软件-opencore-amr)
    - [1. opencore-amr 简介](#1-opencore-amr-简介)
    - [2. 引入背景简述](#2-引入背景简述)
    - [3. 使用场景](#3-使用场景)
    - [4. 为 OpenHarmony 带来的价值](#4-为-openharmony-带来的价值)
    - [5. 如何使用](#5-如何使用)
        - [5.1 系统架构（AVCodec Kit集成）](#51-系统架构avcodec-kit集成)
        - [5.2 编译构建](#52-编译构建)
        - [5.3 使用示例](#53-使用示例)
        - [5.4 仓目录结构](#54-仓目录结构)
        - [5.5 注意事项](#55-注意事项)

## 1. opencore-amr 简介

opencore-amr 是一个开源的 AMR（Adaptive Multi-Rate）音频编解码第三方库，支持 AMR-NB（AMR-Narrowband，窄带）编解码和 AMR-WB（AMR-Wideband，宽带）解码能力。在当前 OpenHarmony AVCodec Kit集成场景中，仅使用了 AMR-NB 编码功能，应用无需直接集成此库，通过 AVCodec Kit API 即可调用。

源代码参考资料可以访问：[opencore-amr 社区代码仓库](https://sourceforge.net/p/opencore-amr/code/ci/master/tree/)。

**框架集成**：AVCodec Kit通过 BUILD.gn deps 链接方式将 AMR-NB 编码库纳入依赖图，运行时由动态链接器在进程装载阶段自动完成库装载，无需运行期手动 `dlopen` / `dlsym`。

## 2. 引入背景简述

OpenHarmony 媒体编解码AVCodec Kit需要提供丰富的音频编解码能力，其中 AMR-NB 是电信及语音领域广泛使用的音频编码格式。opencore-amr 是业界成熟、经过广泛验证的开源 AMR 实现方案，引入该库可快速补齐 OpenHarmony 在 AMR-NB 编码方面的能力缺口。

## 3. 使用场景

- 语音通话、语音备忘录、录音机等应用场景中需要输出 AMR-NB 格式音频。
- 音频转码工具需要将其他编码格式转换为 AMR-NB 码流。
- 应用开发者通过 AVCodec Kit API 间接调用 AMR-NB 编码能力，无需关心底层编解码实现细节。

## 4. 为 OpenHarmony 带来的价值

1. 补充 AVCodec Kit的 AMR-NB 编码能力，完善 OpenHarmony 多媒体音频编码生态。
2. 满足生态伙伴产品对 AMR-NB 格式的兼容性需求，降低接入成本。

## 5. 如何使用

### 5.1 系统架构（AVCodec Kit集成）

AMR-NB 在 AVCodec Kit音频编码链路中的调用关系如下：

应用层 → AVCodec Kit接口层 → Framework 实现层 → 工厂/插件适配层 → opencore-amr → AVCodec Kit接口输出编码后的码流。

- **应用层**：录音、音频转码或其他音频处理业务。
- **AVCodec Kit接口层**：AVCodec API，对外提供统一音频编码能力入口。
- **Framework 实现层**：负责参数解析、实例创建、Buffer 管理及状态流转。
- **插件适配层**：根据 MIME（Multipurpose Internet Mail Extensions）、编码类型等信息选择 AMR-NB 编码路径。
- **第三方库层**：opencore-amr，提供 AMR-NB 编码核心实现。
- **AVCodec Kit接口输出**：输出 AMR-NB 编码码流。

### 5.2 编译构建

编译 64 位 ARM 目标：

```bash
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target third_party_opencore-amr
```

> `{product_name}` 为当前支持的平台名称，例如 `rk3568`。

### 5.3 使用示例

- **AVCodec Kit使用**：详见 [AVCodec 音频编码指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/media/avcodec/audio-encoding.md)。

### 5.4 仓目录结构

```text
/foundation/multimedia/third_party_opencore-amr
├── BUILD.gn             # OpenHarmony 编译配置
├── README.OpenSource    # OpenHarmony 开源合规说明
├── bundle.json          # OpenHarmony 部件描述文件
├── OAT.xml              # OpenHarmony OAT 扫描配置
├── src/                 # opencore-amr 源码
├── include/             # opencore-amr 头文件
└── test/                # 测试代码
```

### 5.5 注意事项

- 当前 AVCodec Kit集成场景中，仅使用了 opencore-amr 的 AMR-NB 编码功能，未使用其 AMR-NB 解码和 AMR-WB 解码功能。
- AVCodec Kit音频软编码链路为本地调用链路，不经过 IPC。
- **OpenHarmony 特有文件**：仓库中存在 `bundle.json`、`OAT.xml`、`README.OpenSource`、`BUILD.gn`，`README_zh.md`这些文件为 OpenHarmony 社区添加或修改，用于部件描述、OAT 扫描、开源合规说明和 GN 构建集成。
