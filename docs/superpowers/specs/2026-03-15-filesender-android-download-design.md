# FileSender Android 版下载功能设计文档

**日期**：2026-03-15
**状态**：已批准

## 需求

在主页 FileSender 详情区域添加 Android APK 下载功能，通过二维码形式呈现，与 PocketTutor 的 App Store 扫码下载保持视觉一致性。

## 架构

### APK 托管

使用 GitHub Releases 托管 APK 文件：

- Release 版本：`v1.0.0`
- APK 来源：`packages/app-release.apk`（54MB）
- 下载直链：`https://github.com/kabda/kabda.github.io/releases/download/v1.0.0/app-release.apk`

**不使用 Git LFS 原因**：Git LFS 文件无法通过 GitHub Pages 直接提供下载，Pages 只会返回 LFS 指针文本。

### 二维码图片

- 内容：指向 GitHub Release APK 下载直链的 QR 码
- 保存路径：`docs/FileSender/qrcode/android_download.png`
- 生成工具：Python `qrcode` 库

### 页面改动

在 `index.html` 的 FileSender `detail-header` 区域：

1. 添加 Android badge（与现有 `iOS 13.0+` 并列）：
   ```html
   <span class="badge">Android 5.0+</span>
   ```

2. 添加二维码块（复用现有 `.detail-qrcode` 样式）：
   ```html
   <div class="detail-qrcode">
       <img src="docs/FileSender/qrcode/android_download.png" alt="扫码下载 FileSender Android 版">
       <span>扫码下载 Android 版</span>
   </div>
   ```

## 实施步骤

1. 创建 GitHub Release `v1.0.0`，上传 APK
2. 生成二维码 PNG，保存到 `docs/FileSender/qrcode/`
3. 修改 `index.html`
4. 提交并推送 main 分支（含积压的 3 个本地提交）
