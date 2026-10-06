# 影集 · 我的照片站

```bash
hugo server      # 本地预览 → http://localhost:1313/
hugo             # 构建到 public/
```

照片都在 `content/albums/` 下，一个目录就是一个分类。

---

## 一、分类怎么配

### 主分类 = 照片放在哪个目录

```bash
cp 新照片.jpg content/albums/gathering/     # 放进「欢聚时刻」
```

放进去就完事了，不用登记。目录名和分类名的对应关系：

| 目录 | 分类 |
|---|---|
| `gathering` | 欢聚时刻 |
| `journey` | 旅途足迹 |
| `moments` | 美好瞬间 |
| `daily` | 人间烟火 |
| `youth` | 青葱岁月 |

### 额外分类 = 在该目录的 `index.md` 里写 tags

一张照片可以同时属于多个分类。**不用复制文件**——在它所在的目录的
`index.md` 里声明一下就行：

```yaml
# content/albums/gathering/index.md
---
title: "欢聚时刻"
englishName: "GATHERING"
weight: 10

resources:
  - src: "20260509_195843-dinner-duo.jpg"
    params:
      tags: ["人间烟火"]          # 这张同时出现在「人间烟火」里
  - src: "20260801_212558-birthday-cake.jpg"
    params:
      tags: ["人间烟火", "美好瞬间"]   # 可以属于多个
---
```

`src` 写文件名（含扩展名），`tags` 里写**分类的中文名**（要和那个分类
`index.md` 里的 `title` 完全一致）。

分类页会自动收：`本分类 = 本目录的照片 ∪ 别处 tags 命中本分类的照片`。

> 改完 `index.md` 后跑一次 `hugo server` 就能看到效果。
> 归档页会自动去重，同一张照片不会出现两次。

---

## 二、年份（拍摄时间）怎么配

**默认全自动**，你不用管。推导顺序：

1. **文件名里的日期** —— 最高优先
   - `20260509_194424-qipao-lake.jpg` → 2026-05-09 19:44:24
   - `IMG_20260801_212545.jpg` → 2026-08-01 21:25:45
   - `mmexport1778327064654.jpg` → 从毫秒时间戳换算
2. **EXIF `DateTimeOriginal`**
3. **相册 `index.md` 里的 `photoDate`**（给整个相册兜底）

**所以只要文件名以 `YYYYMMDD_HHMMSS` 开头，时间就是对的**——这也是我把
照片重命名成这个格式的原因。

### 手动指定某一张的时间

如果某张照片的日期推错了（文件名没有日期、EXIF 又是坏的），
在 `index.md` 里显式写 `date`，它会压过所有自动推断：

```yaml
resources:
  - src: "某张照片.jpg"
    params:
      date: "2026-05-09 19:44:24"    # 优先级最高
      tags: ["人间烟火"]
```

### 给整个相册兜底

相册里有照片既没文件名日期、也没 EXIF 时，用 `photoDate` 兜底：

```yaml
---
title: "欢聚时刻"
photoDate: 2026-08-01        # 该相册里「没有任何时间信息」的照片用这个
---
```

什么都没配的照片会归入归档页末尾的「未标注时间」分组，不会丢。

---

## 三、完整示例

假设新来一张 `IMG_20260901_183000.jpg`，你想让它：

- 主分类是**人间烟火**
- 同时也出现在**美好瞬间**和**欢聚时刻**里

**第 1 步**：放进主分类目录

```bash
cp IMG_20260901_183000.jpg content/albums/daily/
```

**第 2 步**：在 `content/albums/daily/index.md` 里声明额外分类

```yaml
---
title: "人间烟火"
englishName: "DAILY"
weight: 40

resources:
  - src: "IMG_20260901_183000.jpg"
    params:
      tags: ["美好瞬间", "欢聚时刻"]
---
```

**完成。** 时间不用配（文件名里有），三个分类页都会收录它，归档里只出现一次。

---

## 四、其他

**重命名照片**：保持 `YYYYMMDD_HHMMSS-描述.jpg` 格式，日期解析就永远不用
依赖 EXIF。随便改描述部分，别动前面那两段数字。

**换首页背景**：`hugo.toml` 里

```toml
[params]
  homeBackground = "solid"   # 纯深色（当前）；改成 "photo" 用随机照片
```

**改图片质量**：`hugo.toml` 的 `[imaging]` 段。默认 JPEG 质量 95 +
Lanczos 重采样，基本视觉无损。灯箱看大图用的**永远是原图**，不受这里影响。

**加新分类**：建目录 + 写 `index.md`：

```bash
mkdir -p content/albums/新分类
```

```yaml
---
title: "新分类"
englishName: "NEW"
weight: 60          # 决定首页牌堆里从左到右的顺序
---
```

**更新主题**：

```bash
git submodule update --remote themes/photo-gallery
```

**`photos/` 目录**：最早放原始照片的中转目录，已在 `.gitignore` 里排除。
照片都整理进 `content/albums/` 之后就可以删了。
