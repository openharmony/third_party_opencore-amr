# Opencore-AMR 音频库（libopencore_amr_alg.z.so）

## 简介
`Opencore-AMR` 是一个开源 AMR（Adaptive Multi-Rate）音频编解码相关第三方库。在当前 OpenHarmony AVCodec 集成场景中，AMR 侧重点为**音频编码能力**，编译产物为 `libopencore_amr_alg.z.so`。

- **独立调用**：应用或服务可以直接调用 `libopencore_amr_alg.z.so` 提供的 AMR 编码接口。
- **框架集成**：AVCodec 通过 BUILD.gn deps 链接方式将 AMR 编码库纳入依赖图，进程装载相关模块时由动态链接器完成库装载。

## 1. Opencore-AMR 自身调用方式（独立使用）
Opencore-AMR 作为第三方库具备独立调用能力，不依赖 AVCodec 框架即可完成 AMR 编码处理。

### 调用流程
1. 准备 PCM 输入数据。
2. 初始化 AMR 编码器上下文。
3. 配置编码模式、采样率、声道等参数。
4. 调用 AMR 编码接口处理 PCM Buffer。
5. 输出 AMR 编码码流。
6. 释放编码器上下文。

## 2. 系统架构（AVCodec 集成）
AMR 在 AVCodec 音频编码链路中的调用关系如下：

应用层 → AVCodec 公共接口层 → Framework 实现层 → 工厂/插件适配层 → `libopencore_amr_alg.z.so` → AMR 码流输出层

- **应用层**：录音、通话、音频转码或其他音频处理业务。
- **AVCodec 公共接口层**：AVCodec API，对外提供统一音频编码能力入口。
- **Framework 实现层**：负责参数解析、实例创建、Buffer 管理及状态流转。
- **工厂/插件适配层**：根据 MIME、编码类型等信息选择 AMR 编码路径。
- **第三方库层**：`libopencore_amr_alg.z.so`，通过 BUILD.gn deps 纳入依赖图。
- **音频输出层**：输出 AMR 编码码流。

## 3. AVCodec 运行时调用流程
0. 进程装载 AVCodec 相关共享库时，动态链接器根据依赖关系装载 `libopencore_amr_alg.z.so`。
1. 应用通过 AVCodec API 创建音频编码实例，并设置 AMR 格式参数。
2. Framework 实现层接收配置并初始化编码请求。
3. `codec_factory` / `engine factory` 根据格式选择 AMR 编码路径。
4. 插件适配层识别目标格式为 AMR。
5. AMR 相关符号已可直接使用，无需再次显式装载库。
6. 创建 AMR Encoder 实例，并送入 PCM 输入 Buffer。
7. 执行 AMR 编码，输出 AMR 码流 Buffer 并返回上层。

**说明**：AMR 在 AVCodec 中为编码链路。由于 `libopencore_amr_alg.z.so` 通过 BUILD.gn deps 进入依赖图，库装载通常发生在进程装载相关共享库阶段，而不是首次编码时再通过 `dlopen` 装载。

## 4. 仓目录结构
```text
/foundation/multimedia/third_party_opencore-amr
├── BUILD.gn             # OpenHarmony 编译配置
├── README.OpenSource    # OpenHarmony 开源合规说明
├── bundle.json          # OpenHarmony 部件描述文件
├── OAT.xml              # OpenHarmony OAT 扫描配置
├── src/                 # Opencore-AMR 源码及 demo
├── include/             # Opencore-AMR 头文件
└── test/                # 测试代码
```

## 5. 编译构建
### 独立构建 Opencore-AMR
```bash
./build.sh --product-name {product_name} --ccache --build-target third_party_opencore-amr
```

### 编译 64 位 ARM 目标
```bash
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target third_party_opencore-amr
```

> `{product_name}` 为当前支持的平台名称，例如 `rk3568`。

### 编译产物
构建完成后生成 AMR 编码动态库：

```text
libopencore_amr_alg.z.so
```

AVCodec 使用 AMR 编码能力时，通过 BUILD.gn deps 建立链接依赖，运行时由动态链接器根据依赖关系完成装载。

## 6. 使用示例
- **独立使用**：详见仓内 AMR demo。
- **AVCodec 使用**：详见 `av_codec demo`。

## 7. 注意事项
- `libopencore_amr_alg.z.so` 为 Opencore-AMR 在 OpenHarmony 中的动态库产物名称。
- 当前 AVCodec 集成场景中，AMR 为编码能力，不是解码能力。
- AVCodec 音频软编码链路为本地调用链路，不经过 IPC。
- AMR 通过 BUILD.gn deps 纳入依赖图，运行期无需显式 `dlopen` / `dlsym`。
- 即使尚未进入 AMR 编码路径，相关依赖在进程装载阶段也可能已经被动态链接器装载或映射到进程地址空间。
- **OpenHarmony 特有文件**：仓库中存在 `bundle.json`、`OAT.xml`、`README.OpenSource`、`BUILD.gn`，这些文件为 OpenHarmony 社区添加或修改，用于部件描述、OAT 扫描、开源合规说明和 GN 构建集成。

## 8. 相关仓
- [multimedia_av_codec](https://gitcode.com/openharmony/multimedia_av_codec)
- [third_party_opencore-amr](https://gitcode.com/openharmony-sig/third_party_opencore-amr)
- [媒体子系统](https://gitcode.com/openharmony/docs/blob/master/zh-cn/readme/媒体子系统.md)
