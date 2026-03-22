# FileSender Android 版下载功能 Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在主页 FileSender 详情区域添加 Android APK 下载二维码，与 PocketTutor App Store 扫码下载风格一致。

**Architecture:** APK 通过 GitHub Releases 托管，QR 码图片用 Python 生成并提交到仓库，index.html 复用现有 `.detail-qrcode` 样式块添加二维码展示区域。

**Tech Stack:** GitHub Releases（APK 托管）、Python `qrcode` 库（二维码生成）、静态 HTML/CSS（页面改动）

---

## Chunk 1: 安装工具并创建 GitHub Release

### Task 1: 安装 GitHub CLI 并登录

**Files:**
- 无文件改动（环境准备）

- [ ] **Step 1: 安装 GitHub CLI**

```bash
brew install gh
```

- [ ] **Step 2: 登录 GitHub**

```bash
gh auth login
```

按提示选择 `GitHub.com` → `HTTPS` → 浏览器登录。

- [ ] **Step 3: 验证登录状态**

```bash
gh auth status
```

期望输出：`Logged in to github.com as kabda`

---

### Task 2: 创建 GitHub Release 并上传 APK

**Files:**
- 无文件改动（Release 在 GitHub 远端创建）

- [ ] **Step 1: 确认 APK 文件存在**

```bash
ls -lh packages/app-release.apk
```

期望输出：约 54MB 的文件。

- [ ] **Step 2: 创建 Release 并上传 APK**

```bash
gh release create v1.0.0 \
  packages/app-release.apk \
  --repo kabda/kabda.github.io \
  --title "FileSender Android v1.0.0" \
  --notes "FileSender Android 首个正式版本。"
```

期望输出：`https://github.com/kabda/kabda.github.io/releases/tag/v1.0.0`

- [ ] **Step 3: 验证下载直链可访问**

```bash
curl -I "https://github.com/kabda/kabda.github.io/releases/download/v1.0.0/app-release.apk" 2>&1 | head -5
```

期望：HTTP 302 重定向（说明文件存在）。

---

## Chunk 2: 生成二维码图片

### Task 3: 生成 Android 下载二维码

**Files:**
- Create: `docs/FileSender/qrcode/android_download.png`

- [ ] **Step 1: 安装 qrcode 库**

```bash
pip3 install "qrcode[pil]"
```

- [ ] **Step 2: 生成二维码**

```bash
python3 -c "
import qrcode
url = 'https://github.com/kabda/kabda.github.io/releases/download/v1.0.0/app-release.apk'
qr = qrcode.QRCode(version=None, error_correction=qrcode.constants.ERROR_CORRECT_M, box_size=10, border=4)
qr.add_data(url)
qr.make(fit=True)
img = qr.make_image(fill_color='black', back_color='white')
img.save('docs/FileSender/qrcode/android_download.png')
print('二维码已生成')
"
```

期望输出：`二维码已生成`

- [ ] **Step 3: 验证图片存在且尺寸合理**

```bash
ls -lh docs/FileSender/qrcode/android_download.png
python3 -c "
from PIL import Image
img = Image.open('docs/FileSender/qrcode/android_download.png')
print('尺寸:', img.size)
"
```

期望：文件存在，尺寸约 290×290 像素以上。

- [ ] **Step 4: 提交二维码图片**

```bash
git add docs/FileSender/qrcode/android_download.png
git commit -m "feat: 添加 FileSender Android 版下载二维码"
```

---

## Chunk 3: 修改 index.html

### Task 4: 在 FileSender 详情区添加 Android badge 和二维码

**Files:**
- Modify: `index.html`（FileSender detail-header 区域，约第 1404~1422 行）

- [ ] **Step 1: 在 badges 区域添加 Android badge**

找到 FileSender badges 中最后一个 `<span class="badge">` 结束标签 `</span>` 后，插入：

```html
                        <span class="badge">
                            <svg class="icon-svg" viewBox="0 0 24 24" fill="currentColor" xmlns="http://www.w3.org/2000/svg"><path d="M17.523 15.341l-.002-.001c-.289-.162-.644-.066-.805.224-.162.288-.066.644.224.805l.002.001c.29.162.645.066.806-.224.162-.289.066-.644-.225-.805zM6.477 15.341c-.29.162-.386.517-.224.806.161.289.516.386.805.224l.002-.001c.29-.162.386-.517.224-.806-.162-.289-.517-.386-.807-.223zM17.45 9.01l1.573-2.723c.089-.153.036-.35-.117-.439-.153-.089-.35-.037-.439.116L16.88 8.7c-1.414-.647-3.001-1.01-4.882-1.01-1.879 0-3.466.363-4.879 1.009L5.531 5.965c-.088-.153-.285-.205-.439-.116-.153.089-.205.285-.116.439L6.55 9.01C3.892 10.468 2.098 13.12 2 16.237h20c-.098-3.117-1.892-5.769-4.55-7.227zm-9.951 4.228c-.553 0-1-.448-1-1s.447-1 1-1 1 .448 1 1-.447 1-1 1zm9 0c-.553 0-1-.448-1-1s.447-1 1-1 1 .448 1 1-.447 1-1 1z"/></svg>
                            Android 5.0+
                        </span>
```

- [ ] **Step 2: 在 detail-header 末尾（`</div>` 关闭 detail-info 之后）添加二维码块**

在 FileSender `detail-info` 的 `</div>` 闭合标签之后、`detail-header` 的 `</div>` 之前，插入：

```html
                <div class="detail-qrcode">
                    <img src="docs/FileSender/qrcode/android_download.png" alt="扫码下载 FileSender Android 版">
                    <span>扫码下载 Android 版</span>
                </div>
```

- [ ] **Step 3: 在浏览器中验证效果**

```bash
open index.html
```

检查项：
- FileSender 详情区 badge 行出现"Android 5.0+"标签
- 右侧出现二维码图片，标签为"扫码下载 Android 版"
- 布局与 PocketTutor 扫码区视觉一致

- [ ] **Step 4: 提交 index.html 改动**

```bash
git add index.html
git commit -m "feat: 在 FileSender 详情页添加 Android 版下载二维码"
```

---

## Chunk 4: 推送部署

### Task 5: 推送所有提交到 GitHub Pages

**Files:**
- 无文件改动（推送操作）

- [ ] **Step 1: 查看待推送的提交列表**

```bash
git log origin/main..HEAD --oneline
```

期望：看到共 5 个提交（3 个原有积压 + 二维码图片 + index.html 改动）。

- [ ] **Step 2: 推送到 origin main**

```bash
git push origin main
```

- [ ] **Step 3: 验证 GitHub Pages 部署成功**

等待约 1 分钟后，访问：

```
https://kabda.github.io
```

检查项：
- 滚动到 FileSender 区域，确认二维码图片正常显示
- 用手机扫描二维码，确认能正常跳转到 APK 下载页面
