# AGENTS.md

面向 AI 编码代理的项目说明。**动手前先读「配置坑」** —— 有两处配置放错位置会静默失效，不报错但画质掉档。

## 项目定位

用自研 Hugo 主题 [`photo-gallery`](../photo-gallery)（git submodule）搭建的影集站点。没有后台、没有数据库，构建出来就是一堆静态文件。

| 项 | 值 |
| --- | --- |
| 仓库 | `git@github.com:huanglian666/my-photo.git`，分支 `main` |
| 相册 | 5 个：`gathering`（欢聚时刻）/ `journey`（旅途足迹）/ `moments`（美好瞬间）/ `daily`（人间烟火）/ `youth`（青葱岁月） |
| 站点照片 | 39 张 |

## 快速命令

```bash
hugo server     # http://localhost:1313/
hugo            # 构建到 public/
```

**环境要求**：Hugo **extended** ≥ 0.155（需要 `js.Build` 打包 ES 模块、`Image.Meta` 读 EXIF）。本地与 CI 统一用 0.167.0。不需要 Node.js。

## 目录结构

```
├── hugo.toml             # 站点配置（imaging / disableKinds / params 都在这）
├── content/
│   ├── albums/<分类>/    # 相册目录，照片放进去即自动归类
│   │   ├── index.md      # 相册元信息 + 单图 tags / date
│   │   └── *.jpg
│   └── archive/_index.md # 归档页
├── photos/               # ⚠️ 本地原始照片暂存区，见下
├── themes/photo-gallery/ # 主题（git submodule，勿直接改）
└── netlify.toml          # Netlify 部署配置
```

> **`photos/` 不是站点内容。** 它是本地整理原始照片的暂存区，已被 `.gitignore` 排除、不受版本管理。**站点实际使用的图片在 `content/albums/<分类>/` 下，那才是要提交的。** 新照片先放 `photos/` 整理，重命名后再放进对应相册目录。

## 内容维护

### 1. 归类：放进目录就完事

```bash
cp 新照片.jpg content/albums/gathering/     # 放进「欢聚时刻」
```

不需要在任何地方登记。

### 2. 一张照片属于多个分类：写 tags，不要复制文件

在**照片所在目录**的 `index.md` 里声明：

```yaml
resources:
  - src: "20260509_195843-dinner-duo.jpg"
    params:
      tags: ["人间烟火", "美好瞬间"]   # 写分类的中文名
```

`tags` 里写**分类的中文名**，必须和目标分类 `index.md` 的 `title` 完全一致。分类页会自动收「本目录的照片 ∪ 别处 tags 命中本分类的照片」，归档页会自动去重。

### 3. 拍摄时间：默认全自动，但**新照片必须重命名**

推导优先级（`params.dateSource = "auto"`）：

1. **文件名里的日期** —— 最高优先
   - `20260509_194424-qipao-lake.jpg` → 2026-05-09 19:44:24
   - `IMG_20260801_212545.jpg` → 2026-08-01 21:25:45
   - `mmexport1778327064654.jpg` → 从毫秒时间戳换算
2. EXIF `DateTimeOriginal`
3. 相册 `index.md` 的 `photoDate`（给整个相册兜底）

**所以新照片请重命名为 `YYYYMMDD_HHMMSS-描述.jpg` 再放进相册**，时间才是对的。文件名优先于 EXIF 是有意为之：EXIF 可能被传输工具改写成拷贝时间（整批时间戳一样），而文件名不会。

单张照片时间推错时，在该相册 `index.md` 里显式写 `date`，优先级最高：

```yaml
resources:
  - src: "某张照片.jpg"
    params:
      date: "2026-05-09 19:44:24"
```

### 4. 标签：两级叠加

```yaml
---
title: "旅行"
tags: ["旅行", "海边"]        # 相册级：整个相册都带上
resources:
  - src: "001.jpg"
    params:
      tags: ["日落", "长焦"]   # 单图级
---
```

每张照片最终标签 = **单图级 ∪ 相册级**。

## 配置坑

| 坑 | 说明 |
| --- | --- |
| **`[imaging]` 必须写在站点 `hugo.toml`** | Hugo 会合并主题的 `[params]`，但**不合并 `[imaging]`**。漏写会让 JPEG 质量掉回默认 75、重采样退回 `box` —— 对照片站是灾难 |
| **TOML 裸键要写在所有 `[table]` 之前** | 如 `theme`、`baseURL` 写到 `[imaging]` 后面会被解析成 `imaging.xxx.theme`，**静默失效** |
| `disableKinds` 别删 | 相册 front matter 用了 `tags` 键，Hugo 会顺带生成 `/tags/` 分类页。`disableKinds = ["taxonomy", "term", "RSS", "sitemap"]` 就是关掉它 |
| 时区 | EXIF 不存时区，只有「拍摄当时的墙上时间」。归档只用年/月，按墙上时间归并是对的，不要试图"修正" |

## 图片处理

| 用途 | 处理 |
| --- | --- |
| 网格缩略图 / 牌面封面 | 缩到 `thumbWidth`（默认 800px），保持原格式，质量 95 |
| **灯箱看大图** | **原图，完全不经处理** |

**这是刻意的设计取向：只缩尺寸，不压画质。** 灯箱直接指向原始文件，点开看到的永远是原图。

- 缩略图那一步不是压缩画质，而是不把 4000px 的原图塞进 300px 的格子。
- **不要建议"优化"图片质量或上 CDN。** 图床本身是支持的（见下），但默认取向仍是本地原图：不转码、不二次压缩。
- 原图要挪到自建图床/NAS 时**不用改模板**：主题已原生支持，在相册 `index.md` 里加 `remoteImages` 写图床 URL 即可，本地图和远程图可以混在同一个相册里，行为一致。写法见主题 README 的「远程图床」一节。

## CI / 部署

Netlify 与 GitHub Actions 是**两套独立 CI**，都在 push 到 `main` 时构建同一个站点，互不影响：

| 通道 | 配置 | 目标 |
| --- | --- | --- |
| Netlify | `netlify.toml` | Netlify 域名 |
| GitHub Actions | `.github/workflows/deploy.yml` | GitHub Pages |

- 两边都固定 `HUGO_VERSION = 0.167.0`。
- Netlify 检出时自动执行 `git submodule update --init --recursive` 拉主题；`.gitmodules` 已改成 HTTPS，公开仓库匿名可读，CI 里没有 SSH key 也能拉。
- Netlify 的 baseURL 取 `$URL`（正式）或预览地址。

## 交付前检查

```bash
hugo 2>&1 | tail -20
```

正常应看到 `Processed images` 数量与站点照片数一致（当前 39）。若 `Processed images` 为 0 或明显偏少，多半是图片没放进 `content/albums/`（而是留在了被 gitignore 的 `photos/`）。
