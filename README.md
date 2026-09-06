# Avatar Motion Studio

面向人物动作创作的 Windows 桌面工具：图生动作、视频换人、数字人，以及本地音频提取与转换。

**当前仅提供 Windows x64 版本（面向 Windows 10 / 11），尚未提供 macOS、Linux 或 Windows ARM64 安装包。** 本仓库仅发布 EXE 与使用说明，不包含产品源代码、API Key、个人素材或任务数据库。

[下载 Windows EXE](https://github.com/haozheyu/avatar-motion-studio/raw/refs/heads/main/avatar-motion-studio-windows-amd64.exe)

## 支持的功能

| 功能 | 输入 → 输出 | 执行位置 | 需要阿里云 Key |
| --- | --- | --- | --- |
| 图生动作 | 人物图片 + 动作参考视频 → 动作视频 | 百炼 `wan2.2-animate-move` | 是 |
| 视频换人 | 原视频 + 目标人物图片 → 替换人物的视频 | 百炼 `wan2.2-animate-mix` | 是 |
| 数字人 | 人物图片 + 人声音频 → 数字人视频 | 百炼 `wan2.2-s2v` | 是 |
| 视频提取 MP3 / WAV | 带音轨的视频 → 音频文件 | 本机 FFmpeg | 否 |
| 音频转 MP3 / WAV | 音频文件 → MP3 / WAV | 本机 FFmpeg | 否 |
| 素材导入、媒体信息检查 | 本地素材 → 本地素材库 | 本机 / FFprobe | 否 |
| 已下载结果预览、另存为 | 本地结果 → 预览或导出 | 本机 | 否 |

云端生成需要联网，所选素材会上传到百炼临时对象存储；提交、查询及结果下载也需要联网。本地音频功能不上传素材、不调用百炼、不产生百炼 API 费用。云端生成的费用、模型权限和额度以你自己的阿里云账户为准。

## 第一次使用 EXE

### 1. 下载并放入可写文件夹

下载上方的 `avatar-motion-studio-windows-amd64.exe`，放在例如 `D:\AvatarMotionStudio\`。不需要安装 Go、Node.js、Python 或 ComfyUI。

不要放在需要管理员权限才能写入的目录。程序会在 EXE 所在位置创建配置和数据目录；升级时保留这些目录，仅替换 EXE。

### 2. 准备 WebView2

界面依赖 Microsoft Edge WebView2 Runtime。如果打开时提示缺少运行时，请从 [Microsoft 官方 WebView2 页面](https://developer.microsoft.com/en-us/microsoft-edge/webview2/) 安装 Evergreen Runtime，再重新打开程序。

### 3. 准备 FFmpeg 与 FFprobe

**EXE 没有内置 FFmpeg。云端功能的媒体检查与结果验证也会用到 FFmpeg / FFprobe，建议在使用任何媒体功能前完成这一步。**

从 [FFmpeg 官方下载页](https://ffmpeg.org/download.html) 的 Windows EXE Files 链接取得 Windows 构建，解压后将 `bin` 中的 `ffmpeg.exe` 和 `ffprobe.exe` 放入下面的位置：

```text
AvatarMotionStudio/
├── avatar-motion-studio-windows-amd64.exe
├── README.md
└── tools/
    └── ffmpeg/
        └── windows/
            ├── ffmpeg.exe
            └── ffprobe.exe
```

FFmpeg / FFprobe 由用户单独下载，本仓库不分发这些第三方程序。如果已有可用版本，也可以在下一节的配置文件中填写其绝对路径。

### 4. 双击启动

双击 EXE。首次启动会自动生成：

```text
config/local.json          地域、接口地址、模型与媒体工具路径
 data/studio.db            任务与素材记录
 data/inputs/              导入的素材副本
 data/outputs/             已下载的视频和转换后的音频
 data/credentials/         保存密钥后生成的 Windows 加密凭据
```

路径相对于 EXE 所在文件夹，不依赖快捷方式的“起始位置”。同一时间只运行一个实例。

### 5. 配置阿里云 Key（仅云端生成需要）

进入 **设置 → API Key**，粘贴自己的阿里云百炼 Key，点击 **保存密钥**。

- 默认使用北京地域公共 DashScope 地址：`https://dashscope.aliyuncs.com/api/v1`。
- Key 由 Go 使用当前 Windows 用户的 DPAPI 加密保存在本机，不写入普通 JSON 配置，不回显已保存内容。
- 保存后生效，重启会自动读取。保存成功只代表本地保存成功，不代表已经通过阿里云鉴权或拥有模型权限。
- 有云端任务执行时不能更换 Key；等任务结束后再修改。
- 只使用音频工具时不需要填写 Key。

如果你的 Key 对应**专属 API Host**，关闭程序后，用记事本打开 `config/local.json`，将 `bailian.base_url` 改成你自己的 **DashScope 地址（以 `/api/v1` 结尾）**，同时核对 `region`。不要填写 OpenAI 兼容的 `/compatible-mode/v1` 地址。此发布包没有预置任何个人专属 Host。

`bailian.models` 中可以修改模型名称；当前 Provider 请求格式对应表中列出的模型，填写其他模型不保证兼容。修改 JSON 后需要重启。

媒体路径可以改成：

```json
"media": {
  "ffmpeg_path": "D:/MediaTools/bin/ffmpeg.exe",
  "ffprobe_path": "D:/MediaTools/bin/ffprobe.exe"
}
```

JSON 中的 Windows 路径建议使用 `/`。高级用户可以设置 `AVATAR_STUDIO_CONFIG` 指向已有配置文件；设置后不再使用 EXE 旁边的默认配置。默认也支持 `DASHSCOPE_API_KEY` 环境变量，界面保存的加密 Key 优先。

## 操作流程

### 图生动作 / 视频换人

1. 在侧栏选择功能。
2. 选择人物图片，再选择动作参考视频或原视频。
3. 选择标准或专业模式。
4. 点击 **上传素材并生成**。此操作会上传素材并提交可能产生费用的云端任务。
5. 在 **任务中心** 等待完成，预览结果并点击 **另存为**。

视频换人的目标是尽量保留原动作、背景与构图；实际效果取决于模型和素材，不保证完全一致。建议使用人物清晰、主体单一、遮挡少的连续镜头。

### 数字人

选择人物图片和人声音频，选择分辨率，再提交生成。界面会显示素材限制，提交后在任务中心查看进度。

### 音频工具

选择“视频提取音轨”或“音频格式转换”，导入文件，选择 MP3 / WAV，点击 **开始转换**。完成后可试听并另存为。视频必须带有音轨；没有音轨的视频不能提取音频。

## 任务与文件

云端任务异步执行：

```text
CREATED → UPLOADING → SUBMITTED → RUNNING → DOWNLOADING → COMPLETED
```

异常状态包括 `FAILED`、`CANCELLED`、`TIMEOUT`。生成成功后由 Go 自动下载至 `data/outputs/`，临时云端 URL 不作为最终文件保存。

请保持程序运行和网络连接直至下载完成。任务记录存储在 SQLite；程序有重启恢复机制，但不能保证所有中断都可恢复。取消本地任务不保证取消云端执行或停止计费，请以百炼控制台为准。

备份时先关闭程序，再复制整个工作文件夹。加密凭据绑定 Windows 用户，换电脑或换用户后需要重新配置自己的 Key。分享程序时只分享 EXE 与 README，不要分享 `data/` 或个人配置。

## 常见问题

| 问题 | 处理方法 |
| --- | --- |
| 双击后没有窗口 | 检查是否已有实例；确认 WebView2 已安装、EXE 所在目录可写；若编辑过配置，检查 JSON 格式与路径。 |
| 提示 FFmpeg / FFprobe 错误，无法读取媒体 | 检查两个工具是否放到指定目录，或在配置中设置正确的完整路径。 |
| API Key 未配置、鉴权失败或模型无权限 | 在设置中保存自己的 Key；核对地域和 DashScope 地址，以及阿里云账户的模型权限。 |
| `InvalidVideo.NoHuman` | 云端没有识别到符合要求的人体。换用清晰、单人、少遮挡的视频，避免无人画面、快速切镜或主体过小；原样重试不一定有效。 |
| 一直生成中 | 在任务中心检查状态与错误，同时查看网络和百炼控制台；不要连续重复提交。 |
| 不能另存到同名文件 | 当前版本避免覆盖已有文件，请换一个文件名。 |
| 能看到界面但离线不能生成视频 | 当前视频生成全部走百炼，没有本地 Wan / Avatar 推理；音频工具仍可本地使用。 |

## 版本说明

Windows x64 预览版。采用 Go + Wails + Vue3，支持上述三类百炼生成流程与四项本地音频操作。不同素材、账户权限和云端模型状态会影响实际结果；API 接入不代表每类素材都已经完成效果验证。

仓库仅含发布程序与 README。暂未声明开源许可证。问题反馈请使用 [GitHub Issues](https://github.com/haozheyu/avatar-motion-studio/issues)，描述操作步骤、版本与错误码，避免提交 API Key 或私人素材。

## 下载校验

Windows EXE 的 SHA-256：

ea52211c09d60fe9d0737916ddfc172bebae3ac751de17b60c7ae2b89e83cf6a

