# FileSender Android 下载二维码 Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 GitHub Pages 主页 FileSender 详情区域添加 Android APK 下载二维码，用户扫码即可下载 APK。

**Architecture:** APK 通过 GitHub Releases 托管（不进 git 历史），生成指向 Release 直链的 QR 码 PNG，嵌入 index.html 的 FileSender 详情区域，复用现有 `.detail-qrcode` 样式。

**Tech Stack:** GitHub CLI (`gh`), Python `qrcode` 库, 静态 HTML

**Spec:** `docs/superpowers/specs/2026-03-16-filesender-android-download-design.md`

---

## Chunk 1: 环境检查与 GitHub Release 创建

### Task 1: 检查 GitHub CLI 并认证

**Files:**
- 无文件变更（环境准备）

- [ ] **Step 1: 检查 gh CLI 是否已安装**

```bash
gh --version
```

预期输出：`gh version 2.x.x ...`
若命令不存在，执行：`brew install gh`

- [ ] **Step 2: 检查认证状态**

```bash
gh auth status
```

预期输出：包含 `Logged in to github.com` 字样。
若未登录，执行 `gh auth login`，选择 GitHub.com → HTTPS → 浏览器认证。

---

### Task 2: 创建 GitHub Release 并上传 APK

**Files:**
- 无文件变更（上传到 GitHub Releases，不进 git）

- [ ] **Step 1: 检查 Release 是否已存在**

```bash
gh release view filesender-v1.0.0 --repo kabda/kabda.github.io
```

若已存在则跳过 Step 2，直接进行 Step 3 验证。

- [ ] **Step 2: 创建 Release 并上传 APK**

```bash
gh release create filesender-v1.0.0 \
  packages/filesender-app-release.apk \
  --repo kabda/kabda.github.io \
  --title "FileSender v1.0.0" \
  --notes "Android APK 首发版" \
  --latest=false
```

预期输出：`https://github.com/kabda/kabda.github.io/releases/tag/filesender-v1.0.0`
上传 54MB 文件需要约 1-2 分钟，请耐心等待。

- [ ] **Step 3: 验证下载直链可访问**

```bash
curl -sI "https://github.com/kabda/kabda.github.io/releases/download/filesender-v1.0.0/filesender-app-release.apk" | head -5
```

预期：HTTP 302 重定向（GitHub Release assets 会重定向到 CDN），或直接 200。
只要不是 404 即为成功。

---

## Chunk 2: 生成 QR 码 PNG

### Task 3: 安装 Python qrcode 库并生成 QR 码

**Files:**
- Create: `docs/FileSender/qrcode/android_download.png`

- [ ] **Step 1: 检查并安装 qrcode 库**

```bash
python3 -c "import qrcode" 2>/dev/null && echo "已安装" || pip3 install qrcode[pil]
```

- [ ] **Step 2: 确认 qrcode 目录存在**

```bash
mkdir -p docs/FileSender/qrcode
```

- [ ] **Step 3: 生成 QR 码 PNG**

```bash
python3 - <<'EOF'
import qrcode

url = "https://github.com/kabda/kabda.github.io/releases/download/filesender-v1.0.0/filesender-app-release.apk"
qr = qrcode.QRCode(
    error_correction=qrcode.constants.ERROR_CORRECT_M,
    box_size=10,
    border=4
)
qr.add_data(url)
qr.make(fit=True)
img = qr.make_image(fill_color="black", back_color="white")
img.save("docs/FileSender/qrcode/android_download.png")
print("QR 码已生成：docs/FileSender/qrcode/android_download.png")
EOF
```

预期输出：`QR 码已生成：docs/FileSender/qrcode/android_download.png`

- [ ] **Step 4: 验证文件已生成**

```bash
ls -lh docs/FileSender/qrcode/android_download.png
```

预期：文件存在，大小约 10-30KB。

---

## Chunk 3: 修改 index.html

### Task 4: 在 FileSender badges 区域添加 Android badge

**Files:**
- Modify: `index.html`（锚点：`照片视频` badge 的 `</span>` 之后、`</div>` 闭合 badges 之前）

- [ ] **Step 1: 在 `照片视频` badge 后追加 Android badge**

定位锚点：`index.html` 中 FileSender section（`id="filesender"`）内，包含 `照片视频` 的 `</span>` 之后、下一个 `</div>` 之前，插入：

```html
                        <span class="badge">
                            <svg class="icon-svg" viewBox="0 0 24 24" fill="currentColor" xmlns="http://www.w3.org/2000/svg"><path d="M17.523 15.341l-.91-1.582a5.223 5.223 0 001.997-4.091 5.223 5.223 0 00-2.007-4.1l.91-1.578a.25.25 0 00-.434-.25l-.92 1.595A5.196 5.196 0 0012 4.25a5.196 5.196 0 00-4.159 2.085l-.92-1.595a.25.25 0 00-.434.25l.91 1.578A5.223 5.223 0 005.39 9.668c0 1.638.757 3.1 1.997 4.091l-.91 1.582a.25.25 0 00.434.25l.924-1.603A5.19 5.19 0 0012 15.086a5.19 5.19 0 004.165-2.098l.924 1.603a.25.25 0 00.434-.25zM10 11a1 1 0 110-2 1 1 0 010 2zm4 0a1 1 0 110-2 1 1 0 010 2z"/></svg>
                            Android 5.0+
                        </span>
```

- [ ] **Step 2: 验证 badge 已添加**

```bash
grep -n "Android 5.0+" index.html
```

预期：输出包含行号和 `Android 5.0+` 的行。

---

### Task 5: 在 detail-header 末尾添加 QR 码块

**Files:**
- Modify: `index.html`（锚点：FileSender section 内 `</div>` 闭合 `.detail-info` 之后、`</div>` 闭合 `.detail-header` 之前）

- [ ] **Step 1: 在 `.detail-info` 结束后插入 QR 码块**

定位锚点：FileSender section（`id="filesender"`）中，`<div class="detail-info">` 的闭合 `</div>` 之后（该 div 包含 `<h2>FileSender</h2>`）、`detail-header` 的闭合 `</div>` 之前，插入：

```html
                <div class="detail-qrcode">
                    <img src="docs/FileSender/qrcode/android_download.png" alt="扫码下载 FileSender Android 版">
                    <span>Android 扫码下载</span>
                </div>
```

- [ ] **Step 2: 验证 QR 码块已添加**

```bash
grep -n "android_download.png" index.html
```

预期：输出包含 `android_download.png` 的行。

- [ ] **Step 3: 验证 HTML 结构正确**

```bash
grep -n "detail-qrcode\|detail-header\|detail-info" index.html
```

预期：输出中 `detail-qrcode` 对应行位于 `id="filesender"` 所在行之后，且早于下一个 app section 的起始行（即在 FileSender section 范围内）。

---

## Chunk 4: 提交并推送

### Task 6: 提交变更并推送

**Files:**
- Commit: `docs/FileSender/qrcode/android_download.png`, `index.html`

- [ ] **Step 1: 检查变更文件**

```bash
git status
git diff --stat index.html
```

预期：`index.html` 有修改，`docs/FileSender/qrcode/android_download.png` 为新文件。

- [ ] **Step 2: 暂存并提交**

```bash
git add docs/FileSender/qrcode/android_download.png index.html
git commit -m "feat: 在 FileSender 详情页添加 Android APK 下载二维码"
```

- [ ] **Step 3: 推送到远程**

```bash
git push origin main
```

预期：推送成功，无报错。

- [ ] **Step 4: 浏览器验证**

1. 访问 `https://kabda.github.io/#filesender`
2. 确认 FileSender 详情区域右侧显示 QR 码图片和"Android 扫码下载"文字
3. 确认 badges 区域显示 Android 5.0+ badge
4. 用手机扫描 QR 码，确认跳转并提示下载 APK

---

## 注意事项

- `packages/filesender-app-release.apk` **不应**出现在 git 提交中，验证：`git status` 中它应始终显示为 untracked
- 若 GitHub Release 上传失败，检查 gh 认证状态：`gh auth status`
- QR 码使用 ERROR_CORRECT_M 纠错级别，手机扫描成功率高，即使打印出来有轻微污损也可读取
