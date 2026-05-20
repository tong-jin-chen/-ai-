# 万物共形（3dapp）项目说明

本仓库是一个端到端的“手机端发起 → 服务器端 Gradio 重建 → 手机端预览/保存模型”的整合工程，包含：
- Android App（Jetpack Compose）：主应用，负责打开 Gradio 网页、从相册/眼镜照片选择并上传、保存模型、浏览模型与照片、社区占位等。
- 实体模型重建（Hunyuan3D Gradio）：在 WSL2/Windows 上运行的 Gradio Web 服务。
- 点云模型重建（VGGT Gradio）：在 Windows 上运行的 Gradio Web 服务（需要 CUDA）。
- （可选）server/：一个基于 Flask 的上传/任务/下载服务示例（当前 `server/config.py` 路径是机器相关的，需要你按自己环境修改）。

---

## 快速开始（最短流程）

目标：让 App 能分别打开“实体 Gradio 网页”和“点云 Gradio 网页”，并能在手机端从相册上传图片进行重建。

1) 启动实体 Gradio（WSL2，端口 7860）
- WSL2 进入目录：`/mnt/d/3dapp/2.1/Hunyuan3D-2.1-main`
- 启动（两种方式选一种）：
  - `python3 gradio_app.py --server_name 0.0.0.0 --server_port 7860`
  - `GRADIO_SERVER_NAME=0.0.0.0 GRADIO_SERVER_PORT=7860 python3 gradio_app.py`

2) 启动点云 Gradio（Windows，端口 7861）
- Windows 进入目录：`D:\3dapp\vggt-main`
- 启动：`python demo_gradio.py`

3) 让手机能访问到两个端口
- Android Studio 模拟器：直接用 `10.0.2.2` 即可访问宿主机端口
- 真机/雷电：用 Windows 局域网 IP（例如 `192.168.x.x`）
- 如果实体 Gradio 跑在 WSL2 里且手机访问不到：按“WSL2 端口转发”章节做 `portproxy`

4) App 里配置两个地址
- App → 资源库 → 设置
- 填：
  - 实体模型重建地址：`http://10.0.2.2:7860/`（模拟器）或 `http://192.168.x.x:7860/`（真机/雷电）
  - 点云模型重建地址：`http://10.0.2.2:7861/`（模拟器）或 `http://192.168.x.x:7861/`（真机/雷电）
- 点击“保存设置”

5) App 内打开重建
- 底部左侧“模型重建” → 选择“点云模型重建”或“实体模型重建”
- 选择“上传手机相册照片重建”进入网页 → 在网页中上传相册图片并开始重建

---

## 目录结构（关键部分）

- `app/`：Android App 源码（Gradle 工程）
- `2.1/Hunyuan3D-2.1-main/`：实体模型重建 Gradio（已中文化标题/文案）
- `vggt-main/`：点云模型重建 Gradio（`demo_gradio.py` 已中文化标题/文案）
- `server/`：可选的 Flask 服务（任务式上传 + 处理 + 下载）
- `gradle/`、`build.gradle.kts`、`settings.gradle.kts`：Android 工程配置

说明：仓库里同时存在 `d:\3dapp\Hunyuan3D-2.1-main` 与 `d:\3dapp\2.1\Hunyuan3D-2.1-main` 两份目录；当前 App 对接的“实体模型重建”以 `2.1/Hunyuan3D-2.1-main` 为准。

---

## 总体工作流（从用户视角）

1) 在电脑上分别启动：
- “实体模型重建” Gradio（Hunyuan3D）
- “点云模型重建” Gradio（VGGT）

2) 手机上打开 App「万物共形」：
- 底部：模型重建 / 链接蓝牙 / 资源库
- 首页上方：模型共享社区（当前为占位入口）

3) 在「设置」中分别配置两个 Gradio 地址：
- 实体模型重建地址（例如 `http://10.0.2.2:7860/` 或 `http://192.168.x.x:7860/`）
- 点云模型重建地址（例如 `http://10.0.2.2:7861/` 或 `http://192.168.x.x:7861/`）

4) 进入「模型重建」：
- 点云模型重建 / 实体模型重建
- 进入后提供三种方式：
  - 上传手机相册照片重建（打开对应 Gradio 网页，用系统相册选图上传）
  - 使用眼镜照片重建（在 App 内选取已接收的眼镜照片，回传给网页上传）
  - 连接眼镜并接收照片（蓝牙接收端）

5) 保存与查看：
- 实体模型在网页侧触发“保存模型”时，App 通过 JS Bridge 保存到本地文件并可在「我的模型」中查看。
- 「我的模型」支持“实体/点云”分类显示（按文件后缀区分），右上角“+”可快速跳转到点云/实体重建入口。
- 「上传到社区」目前为占位流程（未接服务器时会提示无法实际上传）。

---

## 环境准备（推荐）

### Windows（点云 VGGT）
- 建议使用独立 Python 环境（venv/conda 均可）
- 需要 CUDA + 对应的 PyTorch GPU 版本（否则点云推理会提示 CUDA 不可用）
- 端口建议固定为 `7861`

### WSL2（实体 Hunyuan3D）
- 建议在 WSL2 环境安装 Hunyuan3D 所需依赖
- 端口建议固定为 `7860`
- 若需要手机/雷电访问：通常要做端口转发（见“WSL2 端口转发”）

### Android App
- Android Studio
- JDK 11（工程目标 JVM 11）
- 真机测试建议打开 USB 调试或确保与电脑同 Wi-Fi

---

## 启动实体模型重建（Hunyuan3D Gradio）

推荐在 WSL2 里运行（也可 Windows 运行，取决于你的依赖与模型环境）。

WSL2 进入目录：
- `/mnt/d/3dapp/2.1/Hunyuan3D-2.1-main`

启动建议（绑定端口，避免冲突）：
- 端口建议：`7860`
- 如果脚本支持参数（以你实际脚本为准）：
  - `python3 gradio_app.py --server_name 0.0.0.0 --server_port 7860`
- 或使用环境变量（Gradio 常见支持方式）：
  - `GRADIO_SERVER_NAME=0.0.0.0 GRADIO_SERVER_PORT=7860 python3 gradio_app.py`

启动后会输出访问地址，形如：
- `http://127.0.0.1:7860/`
- `http://<你的机器IP>:7860/`

验证（WSL2 内）：
- `curl -I http://127.0.0.1:7860/`

---

## 启动点云模型重建（VGGT Gradio）

Windows 进入目录：
- `D:\3dapp\vggt-main`

启动：
- `python demo_gradio.py`

说明：
- 该脚本已固定 `server_name="0.0.0.0"`、`server_port=7861`（端口占用会导致启动失败）。
- VGGT 推理要求 CUDA；如果没有可用 GPU，会在运行时直接报错（脚本会检查 `torch.cuda.is_available()`）。

验证（Windows 本机）：
- 在浏览器打开 `http://127.0.0.1:7861/`

---

## 网络与地址选择（非常关键）

### Android Studio 模拟器
- 模拟器访问宿主机（Windows）的固定地址：`10.0.2.2`
- 所以常见配置：
  - 实体：`http://10.0.2.2:7860/`
  - 点云：`http://10.0.2.2:7861/`

### 真机 / 雷电等局域网设备
- 使用 Windows 的局域网 IP（例如 `192.168.31.157`）
  - 实体：`http://192.168.31.157:7860/`
  - 点云：`http://192.168.31.157:7861/`

### WSL2 端口转发（当手机访问不到 WSL2 的服务时）

WSL2 的服务常常只在 WSL2 内可见，需要将端口转发到 Windows：
1) 在 WSL2 获取 WSL IP：
   - `hostname -I`
2) Windows PowerShell（管理员）添加端口转发（示例把 7860 转发到 WSL）：
   - `netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=7860 connectaddress=<WSL_IP> connectport=7860`
3) 放行 Windows 防火墙端口（示例）：
   - `netsh advfirewall firewall add rule name="Gradio-7860" dir=in action=allow protocol=TCP localport=7860`
   - `netsh advfirewall firewall add rule name="Gradio-7861" dir=in action=allow protocol=TCP localport=7861`

常用排查命令（Windows PowerShell 管理员）：
- 查看当前转发规则：
  - `netsh interface portproxy show v4tov4`
- 删除转发规则（示例）：
  - `netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=7860`

验证 Windows 是否能访问到 WSL2 的 Gradio：
- `http://127.0.0.1:7860/` 能打开才算转发成功

获取 Windows 局域网 IP：
- 在 PowerShell 执行：`ipconfig`
- 找到当前网卡的 IPv4 地址（例如 `192.168.31.157`）

---

## Android App 运行与打包

### 基础要求
- Android Studio
- JDK 11（本工程 `app/build.gradle.kts` 目标 Java/Kotlin JVM 为 11）

### 运行
- 用 Android Studio 打开 `D:\3dapp`
- 选择 `app` 作为运行模块
- Run 到模拟器/真机

### 配置 Gradio 地址
- App → 资源库 → 设置
- 填写：
  - 实体模型重建地址（Gradio）
  - 点云模型重建地址（Gradio）
- 保存后：
  - 实体入口 WebView 会打开实体地址
  - 点云入口 WebView 会打开点云地址

### App 功能入口（当前 UI）
- 首页：上方“万物共形 / 模型共享社区”，下方为底部胶囊导航
- 底部左侧（扳手）：进入“模型重建”汇总页 → 点云/实体 → 三种方式
- 底部中间（链接）：进入“链接蓝牙”（眼镜接收页）
- 底部右侧（房子）：进入“资源库”汇总页 → 我的模型 / 设置

### App 数据存储位置（本地）
- 眼镜照片（App 私有目录）：
  - `filesDir/glasses/photos/`
- 眼镜视频（用于抽帧缓存）：
  - `filesDir/glasses/videos/`
- 保存的模型（目前实体保存为 glb）：
  - `filesDir/model_<uuid>.glb`

---

## App 内部对接约定（开发视角）

### WebView（Gradio）打开规则
- 实体 Gradio：`type=entity`
- 点云 Gradio：`type=pointcloud`
- 普通方式：`mode=normal`（系统相册选择器）
- 眼镜方式：`mode=glasses`（弹出 App 内眼镜照片选择器，并用 FileProvider 回传 Uri 给网页上传）

### 实体模型保存（JS Bridge）
- WebView 注册桥接对象：`AndroidModelSaver`
- 网页侧调用：
  - `AndroidModelSaver.saveModelBase64(base64, name)`
- App 保存为 `.glb` 并进入“我的模型”


---

## 智能眼镜蓝牙接收（App 侧）

- 图片：保存到 App 私有目录
- 视频：按“每 20 帧抽 1 帧”抽帧并保存为图片

当前接收端按一个简单二进制协议解析数据：
- 头部 5 字节：`len(4 bytes, big-endian) + type(1 byte)`
- `type=1` 表示图片，`type=2` 表示视频
- payload 长度上限约 200MB

如果你的眼镜端协议不同，需要在 App 里按实际协议适配。

---

## 模型保存与“我的模型”

- 实体模型：网页侧通过 `AndroidModelSaver.saveModelBase64(base64, name)` 回传，App 保存为本地 `.glb` 并在「我的模型」里可见。
- 点云模型：当前主要以 Gradio 网页侧展示为主；如果需要“点云也能保存到本地并在我的模型里展示”，需要点云网页侧提供导出/回传逻辑（例如导出 `.ply` 或 `.glb`）。

---

## 模型共享社区（当前状态）

App 内已放置“模型共享社区”入口与占位页：
- 当前未接入服务器时：上传会提示“暂未接入服务器”
- 未来接入服务器后：可扩展为模型列表（点云/实体）、点赞/评论/下载等

---

## 常见问题排查

1) App 打不开 Gradio 网页
- 先在手机/模拟器浏览器里打开同一个 URL 验证网络是否通
- WSL2 运行的服务需要端口转发（见上文 WSL2 portproxy）

2) 点云 Gradio 启动后报 CUDA 不可用
- VGGT 推理需要 GPU；需要正确安装 CUDA/PyTorch GPU 版本并确保 `torch.cuda.is_available()` 为 true

3) 模拟器地址应该填什么
- Android Studio 模拟器：用 `10.0.2.2:<port>`

4) 图标/应用名没更新
- 卸载旧 App 后重新安装 APK（有时桌面会缓存旧图标）

5) WSL2 地址在浏览器能打开，但手机打不开
- 先确保 Windows 本机 `http://127.0.0.1:7860/` 能打开（否则 portproxy 没生效）
- 再用手机访问 Windows 局域网地址 `http://<Windows_IP>:7860/`
