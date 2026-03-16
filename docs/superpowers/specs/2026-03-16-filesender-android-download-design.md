# FileSender Android 下载二维码 设计规范

**日期**: 2026-03-16
**状态**: 已批准
**仓库**: kabda/kabda.github.io

---

## 背景

FileSender 是一款跨设备文件传输工具，目前主页只展示 iOS 版本。需要在 GitHub Pages 主页的 FileSender 详情区域添加 Android APK 下载入口，通过扫描二维码让用户直接下载安装包。

APK 文件（54MB）已存放于 `packages/filesender-app-release.apk`，尚未提交到 git。

---

## 架构决策

### APK 托管：GitHub Releases（拒绝 Git LFS 和直接提交）

| 方案 | 决策 | 原因 |
|------|------|------|
| 直接提交到 repo | ❌ 拒绝 | 54MB 二进制进入 Git 历史，repo 持续膨胀，clone 变慢 |
| GitHub Pages 直接服务 | ❌ 同上 | 同上，且每次更新叠加历史体积 |
| GitHub Releases | ✅ 采用 | Release assets 独立于 Git 历史，单文件上限 2GB，CDN 分发 |
| 第三方对象存储 | ❌ 拒绝 | 引入额外依赖和费用，过度设计 |

---

## 实现方案

### Step 1：创建 GitHub Release

```bash
gh release create filesender-v1.0.0 \
  packages/filesender-app-release.apk \
  --title "FileSender v1.0.0" \
  --notes "Android APK 首发版"
```

APK 下载直链：
```
https://github.com/kabda/kabda.github.io/releases/download/filesender-v1.0.0/filesender-app-release.apk
```

### Step 2：生成 QR 码 PNG

使用 Python `qrcode` 库，指向上述下载直链：

```python
import qrcode

url = "https://github.com/kabda/kabda.github.io/releases/download/filesender-v1.0.0/filesender-app-release.apk"
qr = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_M, box_size=10, border=4)
qr.add_data(url)
qr.make(fit=True)
img = qr.make_image(fill_color="black", back_color="white")
img.save("docs/FileSender/qrcode/android_download.png")
```

输出路径：`docs/FileSender/qrcode/android_download.png`

### Step 3：修改 index.html

**3a. 在 FileSender badges 区域添加 Android badge**

在现有 4 个 badge 后追加：

```html
<span class="badge">
    <svg class="icon-svg" viewBox="0 0 24 24" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
        <path d="M17.523 15.341l-.91-1.582a5.223 5.223 0 001.997-4.091 5.223 5.223 0 00-2.007-4.1l.91-1.578a.25.25 0 00-.434-.25l-.92 1.595A5.196 5.196 0 0012 4.25a5.196 5.196 0 00-4.159 2.085l-.92-1.595a.25.25 0 00-.434.25l.91 1.578A5.223 5.223 0 005.39 9.668c0 1.638.757 3.1 1.997 4.091l-.91 1.582a.25.25 0 00.434.25l.924-1.603A5.19 5.19 0 0012 15.086a5.19 5.19 0 004.165-2.098l.924 1.603a.25.25 0 00.434-.25zM10 11a1 1 0 110-2 1 1 0 010 2zm4 0a1 1 0 110-2 1 1 0 010 2z"/>
    </svg>
    Android 5.0+
</span>
```

**3b. 在 `detail-header` 末尾添加 QR 码块**

紧接 `</div><!-- .detail-info -->` 之后，`</div><!-- .detail-header -->` 之前：

```html
<div class="detail-qrcode">
    <img src="docs/FileSender/qrcode/android_download.png" alt="扫码下载 FileSender Android 版">
    <span>Android 扫码下载</span>
</div>
```

无需新增 CSS，完全复用现有 `.detail-qrcode` 样式（与 PocketTutor 共享）。

### Step 4：提交并推送

```bash
git add docs/FileSender/qrcode/android_download.png index.html
git commit -m "feat: 在 FileSender 详情页添加 Android APK 下载二维码"
git push origin main
```

---

## 文件变更清单

| 文件 | 操作 |
|------|------|
| `docs/FileSender/qrcode/android_download.png` | 新增（生成的 QR 码） |
| `index.html` | 修改（添加 badge + qrcode 块） |
| `packages/filesender-app-release.apk` | 上传到 GitHub Release（不提交到 git） |

---

## 验证步骤

1. 浏览器访问主页，确认 FileSender 区域显示 QR 码和 Android badge
2. 手机扫描 QR 码，确认跳转并可下载 APK
3. 确认 `packages/filesender-app-release.apk` 未出现在 `git status` 已跟踪文件中
