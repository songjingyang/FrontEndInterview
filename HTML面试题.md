# HTML 面试题

## 基础概念

### 1. HTML5 新特性
**问题**: HTML5 相比 HTML4 有哪些新特性？

**答案**:
```html
<!-- 新的语义化标签 -->
<header>页面头部</header>
<nav>导航</nav>
<main>主要内容</main>
<article>文章</article>
<section>章节</section>
<aside>侧边栏</aside>
<footer>页面底部</footer>

<!-- 新的表单元素 -->
<input type="email" placeholder="请输入邮箱">
<input type="tel" placeholder="请输入电话">
<input type="url" placeholder="请输入网址">
<input type="date" placeholder="请选择日期">
<input type="range" min="0" max="100">
<input type="color" placeholder="请选择颜色">

<!-- 多媒体标签 -->
<video controls>
  <source src="video.mp4" type="video/mp4">
</video>

<audio controls>
  <source src="audio.mp3" type="audio/mpeg">
</audio>

<!-- Canvas 和 SVG -->
<canvas id="myCanvas"></canvas>
<svg width="100" height="100">
  <circle cx="50" cy="50" r="40" fill="red" />
</svg>
```

**新特性总结**:
- 语义化标签
- 新的表单控件和属性
- 音频和视频支持
- Canvas 绘图
- SVG 支持
- 地理定位 API
- 拖放 API
- Web Storage
- Web Workers

### 2. 语义化标签
**问题**: 什么是 HTML 语义化？为什么要使用语义化标签？

**答案**:
```html
<!-- 不好的写法 -->
<div class="header">
  <div class="nav">
    <div class="nav-item">首页</div>
  </div>
</div>
<div class="content">
  <div class="article">
    <div class="title">文章标题</div>
    <div class="content">文章内容</div>
  </div>
</div>

<!-- 语义化写法 -->
<header>
  <nav>
    <a href="/">首页</a>
  </nav>
</header>
<main>
  <article>
    <h1>文章标题</h1>
    <p>文章内容</p>
  </article>
</main>
```

**语义化的好处**:
- 提高代码可读性和可维护性
- 有利于 SEO 优化
- 便于屏幕阅读器等辅助技术解析
- 提升无障碍访问性
- 更好的结构化数据

### 3. DOCTYPE 声明
**问题**: DOCTYPE 的作用是什么？

**答案**:
```html
<!-- HTML5 DOCTYPE -->
<!DOCTYPE html>

<!-- HTML4.01 严格模式 -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" 
  "http://www.w3.org/TR/html4/strict.dtd">

<!-- XHTML 1.0 严格模式 -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" 
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```

**作用**:
- 告诉浏览器使用哪种 HTML 版本规范
- 触发标准模式而非怪异模式
- 确保页面在不同浏览器中正确渲染

## 表单和交互

### 4. 表单验证
**问题**: HTML5 表单验证的使用方法

**答案**:
```html
<form novalidate>
  <!-- 必填项 -->
  <input type="text" required placeholder="用户名">
  
  <!-- 邮箱验证 -->
  <input type="email" required placeholder="邮箱">
  
  <!-- 长度限制 -->
  <input type="text" minlength="6" maxlength="20" placeholder="密码">
  
  <!-- 数值范围 -->
  <input type="number" min="18" max="65" placeholder="年龄">
  
  <!-- 正则验证 -->
  <input type="text" 
         pattern="[0-9]{11}" 
         placeholder="手机号"
         title="请输入11位手机号">
  
  <!-- 自定义验证 -->
  <input type="text" 
         oninput="setCustomValidity('')"
         oninvalid="setCustomValidity('请输入有效的用户名')">
  
  <button type="submit">提交</button>
</form>

<script>
// JavaScript 自定义验证
const input = document.querySelector('input[type="text"]');
input.addEventListener('input', function() {
  if (this.validity.patternMismatch) {
    this.setCustomValidity('格式不正确');
  } else {
    this.setCustomValidity('');
  }
});
</script>
```

### 5. 表单元素
**问题**: 常用的表单元素和属性

**答案**:
```html
<form action="/submit" method="POST" enctype="multipart/form-data">
  <!-- 文本输入 -->
  <input type="text" name="username" placeholder="用户名" autocomplete="username">
  
  <!-- 密码输入 -->
  <input type="password" name="password" placeholder="密码">
  
  <!-- 选择框 -->
  <select name="city">
    <option value="">请选择城市</option>
    <option value="beijing" selected>北京</option>
    <option value="shanghai">上海</option>
  </select>
  
  <!-- 多选框 -->
  <input type="checkbox" id="agree" name="agreement" required>
  <label for="agree">同意用户协议</label>
  
  <!-- 单选框 -->
  <input type="radio" id="male" name="gender" value="male">
  <label for="male">男</label>
  <input type="radio" id="female" name="gender" value="female">
  <label for="female">女</label>
  
  <!-- 文件上传 -->
  <input type="file" name="avatar" accept="image/*" multiple>
  
  <!-- 多行文本 -->
  <textarea name="description" rows="4" cols="50" placeholder="描述"></textarea>
  
  <!-- 隐藏字段 -->
  <input type="hidden" name="token" value="abc123">
  
  <!-- 提交按钮 -->
  <button type="submit">提交</button>
  <button type="reset">重置</button>
  <button type="button" onclick="customAction()">自定义操作</button>
</form>
```

## 多媒体和图像

### 6. img 标签优化
**问题**: img 标签的优化技巧

**答案**:
```html
<!-- 基本用法 -->
<img src="image.jpg" alt="描述文字" width="300" height="200">

<!-- 响应式图片 -->
<picture>
  <source media="(min-width: 800px)" srcset="large.jpg">
  <source media="(min-width: 400px)" srcset="medium.jpg">
  <img src="small.jpg" alt="响应式图片">
</picture>

<!-- srcset 属性 -->
<img src="image.jpg" 
     srcset="image-320w.jpg 320w,
             image-480w.jpg 480w,
             image-800w.jpg 800w"
     sizes="(max-width: 320px) 280px,
            (max-width: 480px) 440px,
            800px"
     alt="不同尺寸图片">

<!-- 懒加载 -->
<img src="placeholder.jpg" 
     data-src="actual-image.jpg" 
     loading="lazy"
     alt="懒加载图片">

<!-- WebP 格式支持 -->
<picture>
  <source srcset="image.webp" type="image/webp">
  <source srcset="image.jpg" type="image/jpeg">
  <img src="image.jpg" alt="WebP兼容">
</picture>
```

### 7. 视频和音频
**问题**: HTML5 音视频标签的使用

**答案**:
```html
<!-- 视频标签 -->
<video controls 
       width="800" 
       height="600" 
       poster="thumbnail.jpg"
       preload="metadata">
  <source src="video.mp4" type="video/mp4">
  <source src="video.webm" type="video/webm">
  <track kind="subtitles" 
         src="subtitles.vtt" 
         srclang="zh" 
         label="中文字幕">
  您的浏览器不支持 video 标签。
</video>

<!-- 音频标签 -->
<audio controls preload="none">
  <source src="audio.mp3" type="audio/mpeg">
  <source src="audio.ogg" type="audio/ogg">
  您的浏览器不支持 audio 标签。
</audio>

<!-- JavaScript 控制 -->
<script>
const video = document.querySelector('video');

// 播放控制
video.play();
video.pause();

// 事件监听
video.addEventListener('loadeddata', () => {
  console.log('视频数据加载完成');
});

video.addEventListener('ended', () => {
  console.log('视频播放结束');
});
</script>
```

## 性能优化

### 8. 页面加载优化
**问题**: HTML 页面加载优化的方法

**答案**:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <!-- DNS 预解析 -->
  <link rel="dns-prefetch" href="//cdn.example.com">
  
  <!-- 预连接 -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  
  <!-- 预加载关键资源 -->
  <link rel="preload" href="critical.css" as="style">
  <link rel="preload" href="hero-image.jpg" as="image">
  
  <!-- 预获取可能用到的资源 -->
  <link rel="prefetch" href="next-page.html">
  
  <!-- 关键 CSS 内联 -->
  <style>
    /* 首屏关键样式 */
    .header { display: block; }
  </style>
  
  <!-- 非关键 CSS 异步加载 -->
  <link rel="preload" href="non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
</head>
<body>
  <!-- 内容 -->
  
  <!-- 脚本放在底部 -->
  <script src="app.js" defer></script>
  <!-- 或使用 async -->
  <script src="analytics.js" async></script>
</body>
</html>
```

### 9. 资源加载策略
**问题**: script 和 link 标签的加载策略

**答案**:
```html
<!-- script 标签加载策略 -->

<!-- 默认：阻塞解析 -->
<script src="blocking.js"></script>

<!-- async：异步加载，加载完立即执行 -->
<script src="async.js" async></script>

<!-- defer：异步加载，文档解析完后执行 -->
<script src="defer.js" defer></script>

<!-- 模块化脚本 -->
<script type="module" src="module.js"></script>
<script nomodule src="fallback.js"></script>

<!-- link 标签预加载 -->

<!-- 预加载：立即获取资源 -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>

<!-- 预获取：空闲时获取资源 -->
<link rel="prefetch" href="next-page-resource.js">

<!-- 预连接：建立连接 -->
<link rel="preconnect" href="https://api.example.com">

<!-- DNS 预解析：解析域名 -->
<link rel="dns-prefetch" href="//cdn.example.com">
```

## SEO 和可访问性

### 10. Meta 标签
**问题**: 重要的 Meta 标签有哪些？

**答案**:
```html
<head>
  <!-- 字符编码 -->
  <meta charset="UTF-8">
  
  <!-- 视口设置 -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  
  <!-- SEO 相关 -->
  <title>页面标题 - 网站名称</title>
  <meta name="description" content="页面描述，用于搜索引擎显示摘要">
  <meta name="keywords" content="关键词1,关键词2,关键词3">
  <meta name="author" content="作者名称">
  
  <!-- Open Graph (社交分享) -->
  <meta property="og:title" content="页面标题">
  <meta property="og:description" content="页面描述">
  <meta property="og:image" content="https://example.com/image.jpg">
  <meta property="og:url" content="https://example.com/page">
  <meta property="og:type" content="website">
  
  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="页面标题">
  <meta name="twitter:description" content="页面描述">
  <meta name="twitter:image" content="https://example.com/image.jpg">
  
  <!-- 其他重要标签 -->
  <meta name="robots" content="index,follow">
  <meta name="googlebot" content="index,follow">
  <link rel="canonical" href="https://example.com/canonical-url">
  
  <!-- PWA 相关 -->
  <meta name="theme-color" content="#000000">
  <link rel="manifest" href="/manifest.json">
  
  <!-- 图标 -->
  <link rel="icon" href="/favicon.ico">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">
</head>
```

### 11. 无障碍访问
**问题**: HTML 无障碍访问的最佳实践

**答案**:
```html
<!-- 语义化标签 -->
<nav aria-label="主导航">
  <ul>
    <li><a href="/" aria-current="page">首页</a></li>
    <li><a href="/about">关于我们</a></li>
  </ul>
</nav>

<!-- 表单标签关联 -->
<label for="email">邮箱地址：</label>
<input type="email" id="email" required aria-describedby="email-help">
<div id="email-help">请输入有效的邮箱地址</div>

<!-- ARIA 属性 -->
<button aria-expanded="false" aria-controls="menu">菜单</button>
<ul id="menu" hidden>
  <li><a href="/page1">页面1</a></li>
  <li><a href="/page2">页面2</a></li>
</ul>

<!-- 跳转链接 -->
<a href="#main-content" class="skip-link">跳转到主要内容</a>

<!-- 焦点管理 -->
<div tabindex="-1" id="main-content">
  <h1>主要内容</h1>
</div>

<!-- 图片描述 -->
<img src="chart.png" alt="2023年销售额增长了25%">

<!-- 表格标题 -->
<table>
  <caption>2023年季度销售数据</caption>
  <thead>
    <tr>
      <th scope="col">季度</th>
      <th scope="col">销售额</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Q1</th>
      <td>100万</td>
    </tr>
  </tbody>
</table>
```

## 新技术

### 12. Web Components
**问题**: Web Components 的基本概念和使用

**答案**:
```html
<!-- 使用 Custom Elements -->
<my-custom-button>点击我</my-custom-button>

<script>
// 定义自定义元素
class MyCustomButton extends HTMLElement {
  constructor() {
    super();
    
    // 创建 Shadow DOM
    const shadow = this.attachShadow({mode: 'open'});
    
    // 创建样式
    const style = document.createElement('style');
    style.textContent = `
      button {
        background: blue;
        color: white;
        border: none;
        padding: 10px 20px;
        cursor: pointer;
      }
    `;
    
    // 创建按钮
    const button = document.createElement('button');
    button.textContent = this.textContent;
    
    // 添加到 Shadow DOM
    shadow.appendChild(style);
    shadow.appendChild(button);
  }
  
  connectedCallback() {
    console.log('元素已添加到页面');
  }
  
  disconnectedCallback() {
    console.log('元素已从页面移除');
  }
}

// 注册自定义元素
customElements.define('my-custom-button', MyCustomButton);
</script>

<!-- 使用 template 标签 -->
<template id="my-template">
  <style>
    p { color: red; }
  </style>
  <p>模板内容</p>
</template>

<script>
const template = document.getElementById('my-template');
const clone = template.content.cloneNode(true);
document.body.appendChild(clone);
</script>
```

### 13. Progressive Web App
**问题**: PWA 相关的 HTML 配置

**答案**:
```html
<!DOCTYPE html>
<html>
<head>
  <!-- PWA 基本配置 -->
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#2196F3">
  <link rel="manifest" href="/manifest.json">
  
  <!-- iOS 支持 -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="default">
  <meta name="apple-mobile-web-app-title" content="PWA App">
  <link rel="apple-touch-icon" href="/icons/icon-192x192.png">
  
  <!-- Microsoft 支持 -->
  <meta name="msapplication-TileColor" content="#2196F3">
  <meta name="msapplication-TileImage" content="/icons/icon-144x144.png">
</head>
<body>
  <!-- 应用内容 -->
  
  <script>
  // 注册 Service Worker
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js')
      .then(registration => {
        console.log('SW 注册成功');
      })
      .catch(error => {
        console.log('SW 注册失败');
      });
  }
  </script>
</body>
</html>
```

```json
// manifest.json
{
  "name": "PWA 应用",
  "short_name": "PWA App",
  "description": "一个渐进式 Web 应用",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2196F3",
  "icons": [
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```