# BBAPI 生图工作台 · 模型与依赖

BBAPI 的文生图、图生图、多图参考通过用户自行配置的接口执行，**基础生图无需下载本地图像模型，也不要求 NVIDIA 显卡**。接口支持的模型、图片数量、尺寸、质量与计费以用户自己的服务与密钥分组为准。

默认接口示例：[BBAPI](https://bbapi.ainivox.cn)。节点通过 `/v1/models` 读取该用户可用模型，不能假定所有用户都能访问 GPT Image 2/2.5 或相同模型目录。兼容服务需实际验证接口格式。

## 必需组件

| 组件 | 来源与用途 |
|---|---|
| ComfyUI | [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI)，Python 环境中应具备 requests、aiohttp、Pillow、numpy 等现有基础依赖 |
| BBAPI 节点 | [BBAIGC/ComfyUI-BBAIGC-BBAPI](https://github.com/BBAIGC/ComfyUI-BBAIGC-BBAPI) |
| 共用提示词/输出/修复节点 | [BBAIGC/ComfyUI-BBAIGC-MiniMax-H3](https://github.com/BBAIGC/ComfyUI-BBAIGC-MiniMax-H3)，基础 API 生图不需要下载 H3 视频模型 |
| 分组开关与对比 | [rgthree/rgthree-comfy](https://github.com/rgthree/rgthree-comfy) |

密钥在本机连接节点的密码框中填写并保存，工作流 JSON 不包含密钥。Windows 使用当前用户 DPAPI 加密；换电脑或换 Windows 用户后需重新填写。公开包不含配置文件、API Key、账号、生成记录或私人参考图。基础密钥持久化以 Windows 为已验证平台；其他系统先核对节点当前安全存储支持，不能宣称跨平台安全存储已验证。

## 可选本地能力

本页文件大小采用十进制 GB（1 GB = 10⁹ 字节），来自已下载文件或上游 Hugging Face 文件元数据；下载体积不等于显存占用。作者设备为 NVIDIA RTX 5060 Ti 16GB、Intel Core i7-12700KF、64GB 内存，Windows 11。其他设备的速度与最大尺寸取决于驱动、PyTorch/CUDA、分辨率、步数与内存卸载。无需一次下载所有可选路线。

ComfyUI 应使用支持这些模型的版本；作者现有核心为 0.43.0、前端 1.57.4。安装时先核对 `/object_info` 中所需节点，不能只凭版本号判断。RTX 50 系需要支持 Blackwell 的 NVIDIA 驱动及 PyTorch CUDA 构建；先执行 `python -c "import torch; print(torch.__version__,torch.cuda.is_available(),torch.cuda.get_device_name(0))"` 核验。保留现有可工作的 PyTorch，切勿盲目降级。

| 功能 | 需要下载 | 体积与目录 |
|---|---|---|
| 提示词优化 | [Ollama](https://ollama.com/download)，例如 `ollama pull qwen3.5:9b` | Ollama 自行管理；以模型卡及实际下载显示为准 |
| Qwen 图像修复 | [Comfy-Org/Qwen-Image-2.1](https://huggingface.co/Comfy-Org/Qwen-Image-2.1) 配套主模型/编码器/VAE | 官方INT8主模型 7.257 GB → `models/diffusion_models/`；编码器 9.351 GB → `models/text_encoders/`；VAE 0.676 GB → `models/vae/` |
| SeedVR2 超分 | [numz/SeedVR2_comfyUI](https://huggingface.co/numz/SeedVR2_comfyUI) | `seedvr2_ema_3b_fp8_e4m3fn.safetensors` 3.392 GB、`ema_vae_fp16.safetensors` 0.501 GB → `models/seedvr2/` |
| 语音输入 | [kautism/SenseVoiceSmall-onnx](https://huggingface.co/kautism/SenseVoiceSmall-onnx) | `model.onnx` 0.940 GB 及 `config.yaml`、`tokens.json`、`am.mvn` → `models/sensevoice-small/` |

SeedVR2 插件：[numz/ComfyUI-SeedVR2_VideoUpscaler](https://github.com/numz/ComfyUI-SeedVR2_VideoUpscaler)。其他超分可按需安装 [ComfyUI-VOSR2](https://github.com/ylchen333/ComfyUI-VOSR2) 或 [ComfyUI-RH-DLSS5](https://github.com/RH-RunningHub/ComfyUI-RH-DLSS5)，硬件要求遵循各自上游。

批量与并发有上限；每张请求只提交一次，网络超时不自动补发，避免重复计费。基础 API 生图的耗时受上游排队和模型影响，不以作者本地显卡耗时作承诺。

已有 SenseVoice 模型可通过 `BBAIGC_SENSEVOICE_MODEL_DIR` 指定目录；兼容旧变量 `CODEX_SENSEVOICE_MODEL_DIR`，新变量优先。该目录需包含 `model.onnx`、`config.yaml`、`tokens.json`、`am.mvn` 四个文件。

## 模型档位选择

云端模型由 BBAPI Key 实际授权的模型列表决定，不按本地显存选择量化。仅启用本地提示词优化、修复、语音或超分时，才需要对应本地模型。图片模型的作者同款 UC Q6_K、低显存量化和高配精度选择见 [生图工作台模型档位](https://github.com/BBAIGC/ComfyUI-BBAIGC-Image-Studio/blob/main/MODELS.md#选择作者同款或适配显存的量化版本)。安装智能体先查硬件再让使用者一次性选择尚未指定的方案；不要为了云端生图默认下载全部本地模型。
