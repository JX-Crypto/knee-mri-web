# 膝关节 MRI 骨龄智能评估系统（网页版）

基于深度学习的膝关节 MRI 骨骺发育自动评估网页：上传膝关节 MRI（NIfTI 格式），自动/手动分割骨骺 ROI 后，一键输出**预测骨龄（年龄回归）+ 股骨远端 Vieth 分级 + 胫骨近端 Vieth 分级**，并可下载 PDF 评估报告。全流程在浏览器本地运行（ONNX Runtime WebAssembly），**不上传任何患者数据到服务器，保护隐私**。

---

## 一、功能特点

- **6 个评估模型**可选：PD+T1 双模态融合 / PD 单模态 / T1 单模态，自定义 ResNet 与 ResNet50 两种骨干。
- **2 个 U-Net 自动分割模型**：只传原始图像即可自动分割股骨、胫骨骨骺 ROI；也支持上传 ITK-SNAP 等工具手工标注的掩码。
- **多任务输出**：年龄回归值、股骨/胫骨 Vieth 分级（2–6 级）、发育阶段判断。
- **PDF 报告下载**：包含性别、体位、模型、分割方式、预测骨龄、双侧 Vieth 分级、分级对照表与免责声明。
- **中英文双语**：右上角一键切换。
- **纯前端、离线可用**：所有模型与依赖均为本地文件，无需联网、无需后端、无需安装 Python。

---

## 二、目录结构

```
knee-mri-web/
├── index.html                 # 主网页（唯一入口，双击或经本地服务器打开）
├── nifti-reader-min.js        # NIfTI(.nii/.nii.gz) 医学影像读取库
├── 启动服务器.bat              # Windows 一键启动本地服务器
├── models/                    # 6 个骨龄评估模型（ONNX）
│   ├── model_pd_t1_resnet.onnx   ★ 推荐，PD+T1 双模态，精度最高（约47MB）
├── models/                    # 骨龄评估模型（ONNX）——仓库内只放 <100MB 的模型
│   ├── model_pd_t1_resnet.onnx   ★ 推荐，PD+T1 双模态，精度最高（约47MB）
│   ├── model_pd_resnet.onnx      PD 单模态（约21MB）
│   └── model_t1_resnet.onnx      T1 单模态（约21MB）
│   （以下 3 个超过 GitHub 100MB 上限，已移到独立文件夹、改由 Hugging Face 远程加载，见第五节）
│   （model_pd_t1_cnn.onnx≈113MB、model_pd_cnn.onnx≈103MB、model_t1_cnn.onnx≈103MB）
├── unet_models/               # 2 个 U-Net 自动分割模型（ONNX，各约51MB）
│   ├── knee_pd_unet.onnx
│   └── knee_t1_unet.onnx
├── ort-wasm/                  # ONNX Runtime WebAssembly 运行时（必需）
│   ├── ort.min.js
│   ├── ort-wasm.wasm
│   └── ort-wasm-simd.wasm
└── pdf-libs/                  # 生成 PDF 报告所需
    ├── jspdf.umd.min.js
    └── html2canvas.min.js
```

> 另有一个同级文件夹 **`knee-mri-large-models-for-hosting/`**，装着 3 个超过 100MB 的大模型，用于上传到 Hugging Face（见第五节），**不要**把它提交进 GitHub 仓库。

> 本文件夹是**纯发布版**，已剔除训练代码、Python 依赖、权重(.pth)、临时文件等与网页运行无关的内容。仓库内所有文件均 <100MB，可直接 `git push`。

---

## 三、本地运行（零基础分步）

模型推理依赖 WebAssembly，**不要直接双击 index.html 用 file:// 打开**（浏览器会因安全策略拦截 wasm/模型加载），请用本地服务器：

### 方法 A：Windows 一键脚本（最简单）
1. 进入本文件夹；
2. 双击 **`启动服务器.bat`**（需要电脑已安装 Python，安装时勾选 Add to PATH）；
3. 浏览器访问脚本窗口中提示的地址：**http://localhost:8000/** ；
4. 用完后在黑色命令行窗口按 `Ctrl + C` 停止。

### 方法 B：命令行手动启动
在本文件夹地址栏输入 `cmd` 回车，然后执行：
```bash
python -m http.server 8000
```
浏览器打开 http://localhost:8000/ 即可。

### 方法 C：VS Code
安装 “Live Server” 插件，右键 index.html → Open with Live Server。

---

## 四、上传到 GitHub，生成可分享链接（GitHub Pages）

### 1. 在 GitHub 新建仓库
登录 github.com → 右上角 `+` → `New repository`，仓库名例如 `knee-mri-web`，选 Public，**不要**勾选 Add README（本地已有），创建。

### 2. 本地初始化并推送
在本文件夹内打开 Git Bash 或命令行，依次执行（把下面的用户名换成你自己的）：
```bash
git init
git add .
git commit -m "膝关节MRI骨龄评估网页 发布版"
git branch -M main
git remote add origin https://github.com/你的用户名/knee-mri-web.git
git push -u origin main
```

### 3. 开启 GitHub Pages
仓库页面 → `Settings` → 左侧 `Pages` → `Build and deployment`：
- Source 选 `Deploy from a branch`；
- Branch 选 `main`、目录选 `/ (root)` → `Save`；
- 等待 1–2 分钟，页面顶部会出现链接，形如：
  **https://你的用户名.github.io/knee-mri-web/**
把这个链接发给别人即可直接使用，**上传后仍可随时修改**：本地改完后
```bash
git add .
git commit -m "更新说明"
git push
```
Pages 会在一两分钟内自动更新。

---

## 五、⚠️ 关于大文件：为什么用 Hugging Face 托管（已处理好）

### 1. GitHub 的三条硬限制（均已核实）
1. **单个文件超过 100MB 会被直接拒绝 `git push`**；
2. **GitHub Pages 不支持 Git LFS**（LFS 文件在 Pages 上只会返回指针文本，模型损坏）；
3. **GitHub Release 附件不能被网页跨域加载**：Release 单文件上限 2GB、适合“放上去让人下载”，但其实测最终响应不带 `Access-Control-Allow-Origin` 头，`*.github.io` 页面用 fetch 读取会被浏览器 CORS 拦截，因此**不能**把 ONNX 放 Release 让网页自动加载。

### 2. 超限的 3 个模型与本项目的处理方式
| 文件 | 大小 | 处理 |
|---|---|---|
| model_pd_t1_cnn.onnx | ≈113MB | 移出仓库，放 Hugging Face 远程加载 |
| model_pd_cnn.onnx | ≈103MB | 同上 |
| model_t1_cnn.onnx | ≈103MB | 同上 |

其余模型均 <100MB 直接放仓库。**精度最高的推荐模型 `model_pd_t1_resnet.onnx` 只有约 47MB，在仓库内、不受影响**。

网页 `index.html` 顶部已内置配置块 `REMOTE_MODELS`：这 3 个模型自动从远程地址加载，其余仍走本地；在你填好 HF 地址前会自动回退到 `./models/`（本机离线）路径。

### 3. Hugging Face 托管步骤（免费、已实测可被网页跨域加载）
1. 注册并登录 huggingface.co → 右上角 `+` → `New model` → 建一个**公开(Public)**模型仓库，名字例如 `knee-mri-models`；
2. 进入该仓库 → `Files` → `Add file` → `Upload files`，把同级文件夹 **`knee-mri-large-models-for-hosting/`** 里的 3 个 `.onnx` 拖进去上传并 `Commit`；
3. 上传后每个文件的直链形如
   `https://huggingface.co/你的用户名/knee-mri-models/resolve/main/model_pd_t1_cnn.onnx`；
4. 打开本项目 `index.html`，找到 `const REMOTE_MODELS`，把 `base` 改成
   `https://huggingface.co/你的用户名/knee-mri-models/resolve/main/`（末尾保留斜杠），保存；
5. 正常 `git add/commit/push` 部署 Pages 即可。访客打开网页时，3 个大模型会实时从 HF 拉取，其余从 GitHub 加载。

> 备选方案：① 只保留 3 个小模型（隐藏 3 张 CNN 卡片，最省事，且最佳模型仍在）；② 把大模型转 FP16 半精度压到 <100MB 后直接进仓库（需逐样本验证精度）。需要任一方案可再找我处理。

### 4. 附：GitHub Release 怎么用（仅用于“提供下载”，不用于网页加载）
仓库主页右侧 `Releases` → `Draft a new release` → `Choose a tag` 填如 `v1.0` → 把文件拖到 `Attach binaries`（单文件≤2GB）→ `Publish release`，即可得到固定下载链接分发给别人手动下载。它不适合本项目的“网页自动读取模型”，原因见上文第 1 条。

---

## 六、网页使用说明

### 输入要求
- 影像格式：`.nii` / `.nii.gz`（2D 切片，系统自动取第一个切片，与训练一致）；自动分割模式仅需原图，手动分割模式还需对应的 ROI 掩码。
- 双模态模型需要同时上传 **T1** 和 **PD** 两套图像；单模态模型只需一种。
- 临床信息：
  - **性别**：1 = 男，2 = 女（务必正确，对结果影响较大）；
  - **体位**：1 = 左侧（L），2 = 右侧（R）。多数文件名带 L/R 后缀可据此判断；若确实无法区分，任选其一即可，实测对年龄结果影响很小（平均约 0.08 岁）。

### 处理流程（与模型训练代码严格一致）
读取 NIfTI →（自动分割或读取手工掩码）→ 掩码二值化并与原图相乘提取 ROI → LANCZOS 缩放到模型输入尺寸（256 或 224）→ min-max 归一化到 [0,1] → ONNX 推理 → 输出结果。

### 模型选择建议
- 一般情况选默认的 **PD+T1 ResNet（双模态）**，综合精度最好；
- 只有一种序列时，选对应的单模态模型。

---

## 七、技术栈

- 前端：原生 HTML / CSS / JavaScript（单文件，无前端框架）
- 模型推理：ONNX Runtime Web（WebAssembly + SIMD）
- 影像解析：nifti-reader
- PDF：jsPDF + html2canvas
- 模型：PyTorch 训练并导出为 ONNX（opset 17），U-Net 分割 + 多任务（分类+回归）评估网络

---

## 八、常见问题

**Q：双击 index.html 打开后一直转圈 / 报模型加载失败？**
A：file:// 协议会被浏览器拦截，请按第三节用本地服务器打开。

**Q：推送时报 “exceeds GitHub's file size limit of 100 MB”？**
A：就是第五节的大文件限制，按方案一/二处理后再推送。

**Q：网页是英文的怎么办？**
A：右上角点 “中文 / EN” 切换，选择会被浏览器记住。

**Q：患者数据会上传吗？**
A：不会。所有计算都在访问者本机浏览器内完成，本站点不包含任何后端上传逻辑。

**Q：预测结果能替代医生诊断吗？**
A：不能。结果仅供科研与临床参考，最终诊断需由专业医师结合检查判断（PDF 报告内亦含免责声明）。
