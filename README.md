# 四夕的博客小站

一个支持 Markdown 的现代化静态博客系统。

## ✨ 特性

- 📝 **Markdown 支持** - 使用 Markdown 编写文章，自动渲染
- 🎨 **代码高亮** - 支持多种编程语言语法高亮
- 📐 **数学公式** - 支持 LaTeX 数学公式渲染
- 🏷️ **标签系统** - 文章分类管理
- 📅 **归档功能** - 按时间线浏览文章
- 🖼️ **图片支持** - 本地图片存储方案
- 📱 **响应式设计** - 适配各种设备
- ⚡ **统一数据源** - 只需维护一个 JSON 文件

## 📁 目录结构

```
sixiai_blog/
├── css/
│   └── style.css          # 样式文件
├── js/
│   └── main.js            # 交互脚本
├── posts/                 # 📝 文章目录
│   ├── posts.json         # ⭐ 文章列表配置（唯一数据源）
│   ├── xxx.md             # Markdown 文章
│   └── ...
├── images/                # 🖼️ 图片目录
│   ├── article-name/      # 按文章分类存放
│   └── common/            # 公共图片
├── index.html             # 首页
├── post.html              # 文章渲染页
├── archives.html          # 归档页
├── tags.html              # 标签页
└── about.html             # 关于页
```

## 🏗️ 架构说明

本博客采用**统一数据源**架构，所有页面都从 `posts/posts.json` 读取文章数据：

```
posts/posts.json (唯一数据源)
        │
        ├──→ index.html     (首页：显示文章列表)
        ├──→ post.html      (文章页：渲染文章内容)
        ├──→ archives.html  (归档页：按时间分组)
        └──→ tags.html      (标签页：按标签分类)
```

**优点：**
- ✅ 添加新文章只需修改一个文件
- ✅ 所有页面自动同步更新
- ✅ 避免数据不一致问题

---

## ✍️ 如何添加新文章

### 第一步：创建 Markdown 文件

在 `posts/` 文件夹中创建一个新的 `.md` 文件，例如 `my-new-post.md`：

```markdown
# 文章标题

> 📅 2025-01-15 | 🏷️ stm32

## 第一节

这是文章内容...

### 小节

- 列表项 1
- 列表项 2

## 代码示例

```c
int main() {
    return 0;
}
```

## 总结

文章结束。
```

### 第二步：更新 posts.json

打开 `posts/posts.json`，在 `posts` 数组的**最前面**添加新文章信息：

```json
{
  "posts": [
    {
      "id": "my-new-post",
      "title": "文章标题",
      "date": "2025-01-15",
      "tags": ["stm32"],
      "file": "my-new-post.md",
      "excerpt": "这是文章的简短描述，会显示在首页卡片上..."
    },
    // ... 其他文章
  ]
}
```

### 第三步：推送到 GitHub

```bash
git add .
git commit -m "添加新文章：文章标题"
git push
```

**就这样！** 首页、归档页、标签页都会自动更新！

### 字段说明

| 字段 | 说明 | 示例 |
|------|------|------|
| `id` | 文章唯一标识（用于 URL） | `"my-new-post"` |
| `title` | 文章标题 | `"STM32 入门教程"` |
| `date` | 发布日期 | `"2025-01-15"` |
| `tags` | 标签数组 | `["stm32", "tutorial"]` |
| `file` | Markdown 文件名 | `"my-new-post.md"` |
| `excerpt` | 文章摘要 | `"这是一篇..."` |

---

## 🖼️ 如何添加图片

### 方案：本地存储（推荐）

图片存放在 `images/` 文件夹中，建议按文章分类：

```
images/
├── stm32-uart/
│   ├── circuit.png
│   └── waveform.jpg
├── matlab-pid/
│   ├── block-diagram.png
│   └── result.png
└── common/
    └── logo.png
```

### 在 Markdown 中引用图片

**基本语法：**
```markdown
![图片描述](../images/文章名/图片名.png)
```

**示例：**
```markdown
# STM32 串口通信

下面是电路连接图：

![电路图](../images/stm32-uart/circuit.png)

运行效果如下：

![运行效果](../images/stm32-uart/result.jpg)
```

**指定图片大小（HTML 方式）：**
```markdown
<img src="../images/stm32-uart/circuit.png" width="500" alt="电路图">
```

### 图片路径说明

因为 Markdown 文件在 `posts/` 目录下，引用 `images/` 中的图片需要用 `../`：

| 图片位置 | Markdown 中的路径 |
|----------|-------------------|
| `images/photo.png` | `../images/photo.png` |
| `images/stm32/demo.png` | `../images/stm32/demo.png` |

---

## 🏷️ 标签管理

### 可用标签

| 标签 ID | 显示名称 | 颜色 |
|---------|----------|------|
| `stm32` | STM32 | 红色 |
| `matlab` | Matlab | 蓝色 |
| `knowledge` | 知识 | 绿色 |
| `documentation` | 文档 | 紫色 |

### 添加新标签

在 `posts/posts.json` 的 `tags` 对象中添加：

```json
"tags": {
  "newtag": {
    "name": "新标签",
    "color": "#ff6600",
    "count": 1
  }
}
```

### 更新标签计数

添加新文章后，记得更新对应标签的 `count` 值。

---

## 🚀 部署到 GitHub Pages

### 方法一：直接上传

1. 登录 GitHub，创建新仓库 `sixiai.github.io`
2. 上传所有文件到仓库
3. 进入 Settings → Pages
4. Source 选择 `main` 分支
5. 等待几分钟后访问 `https://sixiai.github.io`

### 方法二：使用 Git 命令

```bash
cd C:\Users\luo\Desktop\sixiai_blog
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/sixiai/sixiai.github.io.git
git branch -M main
git push -u origin main
```

---

## 📝 Markdown 语法参考

### 标题
```markdown
# 一级标题
## 二级标题
### 三级标题
```

### 文本格式
```markdown
**粗体** *斜体* ~~删除线~~ `行内代码`
```

### 列表
```markdown
- 无序列表
- 无序列表

1. 有序列表
2. 有序列表
```

### 链接和图片
```markdown
[链接文字](https://example.com)
![图片描述](图片地址)
```

### 代码块
````markdown
```c
// 代码内容
```
````

### 表格
```markdown
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| A   | B   | C   |
```

### 引用
```markdown
> 这是引用内容
```

### 数学公式
```markdown
行内公式：$E = mc^2$

块级公式：
$$\int_{0}^{\infty} e^{-x} dx = 1$$
```

---

## ❓ 常见问题

### Q: 文章不显示？
A: 检查 `posts.json` 格式是否正确，特别是逗号和引号。可以用 [JSON 校验工具](https://jsonlint.com/) 检查。

### Q: 代码没有高亮？
A: 确保代码块指定了语言，如 ` ```c ` 或 ` ```python `。

### Q: 图片不显示？
A: 检查图片路径是否正确，记得使用 `../images/` 开头的相对路径。

### Q: 本地预览有问题？
A: 由于浏览器安全限制，本地直接打开可能无法加载 Markdown。建议：
- 使用 VS Code 的 Live Server 插件
- 或者直接部署到 GitHub Pages 查看

### Q: 首页文章数量不对？
A: 本博客已采用统一数据源架构，只需修改 `posts/posts.json`，所有页面会自动同步。

---

## 📋 快速检查清单

添加新文章时，确保完成以下步骤：

- [ ] 在 `posts/` 创建 `.md` 文件
- [ ] 在 `posts/posts.json` 的 `posts` 数组添加文章信息
- [ ] 如果使用新标签，在 `tags` 对象中添加标签定义
- [ ] 更新相关标签的 `count` 计数
- [ ] 如果有图片，放到 `images/` 对应文件夹
- [ ] `git add . && git commit -m "添加新文章" && git push`

---

## 📞 联系方式

- GitHub: [sixiai](https://github.com/sixiai)

---

Made with ❤️ by 四夕
