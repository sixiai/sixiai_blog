# 四夕的博客小站

一个支持 Markdown 的现代化静态博客系统。

## 📁 目录结构

```
sixiai_blog/
├── css/
│   └── style.css          # 样式文件
├── js/
│   └── main.js            # 交互脚本
├── posts/                 # 📝 文章目录（你只需要关注这里！）
│   ├── posts.json         # 文章列表配置
│   ├── xxx.md             # Markdown 文章
│   └── ...
├── index.html             # 首页
├── post.html              # 文章渲染页
├── archives.html          # 归档页
├── tags.html              # 标签页
└── about.html             # 关于页
```

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

### 字段说明

| 字段 | 说明 | 示例 |
|------|------|------|
| `id` | 文章唯一标识（用于 URL） | `"my-new-post"` |
| `title` | 文章标题 | `"STM32 入门教程"` |
| `date` | 发布日期 | `"2025-01-15"` |
| `tags` | 标签数组 | `["stm32", "tutorial"]` |
| `file` | Markdown 文件名 | `"my-new-post.md"` |
| `excerpt` | 文章摘要 | `"这是一篇..."` |

### 可用标签

| 标签 ID | 显示名称 | 颜色 |
|---------|----------|------|
| `stm32` | STM32 | 红色 |
| `matlab` | Matlab | 蓝色 |
| `knowledge` | 知识 | 绿色 |
| `documentation` | 文档 | 紫色 |

如果需要添加新标签，在 `posts.json` 的 `tags` 对象中添加：

```json
"tags": {
  "newtag": {
    "name": "新标签",
    "color": "#ff6600",
    "count": 1
  }
}
```

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

## ❓ 常见问题

### Q: 文章不显示？
A: 检查 `posts.json` 格式是否正确，特别是逗号和引号。

### Q: 代码没有高亮？
A: 确保代码块指定了语言，如 ` ```c ` 或 ` ```python `。

### Q: 本地预览有问题？
A: 由于浏览器安全限制，本地直接打开可能无法加载 Markdown。建议：
- 使用 VS Code 的 Live Server 插件
- 或者直接部署到 GitHub Pages 查看

## 📞 联系方式

- GitHub: [sixiai](https://github.com/sixiai)

---

Made with ❤️ by 四夕
