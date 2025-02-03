+++
title = 'Short Code Hugo 短代码'
date = 2024-06-21T21:59:02+08:00

+++

本网站使用Hugo构建，因此可以使用[**短代码**](https://hugo.opendocs.io/content-management/shortcodes/)功能——通过在markdown文件中添加相应代码，嵌入相应的html内容，实现一定的能力。

本网站现在已经支持了两个外置短代码。（24/06/21）

**短代码 Plan：**

- [x] 友链卡片
- [ ] 时间轴（但好像主题已经内置了差不多的步骤条功能）

## Apple Music 卡片

标识符： `am`

这个代码可以快速添加一个 Apple Music 卡片。可以是一个专辑卡片或单曲卡片。

### 短代码

专辑卡片：

```
{{</* am url="red-taylors-version-a-message-from-taylor/1590368448" */>}}
```

单曲卡片：

```
{{</* am url="red-taylors-version-a-message-from-taylor/1590368448" i="1590368457" */>}}
```

### 预览

专辑卡片：

{{< am url="red-taylors-version-a-message-from-taylor/1590368448">}}

单曲卡片：

{{< am url="red-taylors-version-a-message-from-taylor/1590368448" i="1590368457">}}

### 源代码

```html
{{ if and (.Get "url") (not (.Get "i")) }}
<iframe id="embedPlayer" src="https://embed.music.apple.com/us/album/{{ .Get "url" }}?app=music&amp;itsct=music_box_player&amp;itscg=30200&amp;ls=1&amp;theme=auto" height="450px" frameborder="0" sandbox="allow-forms allow-popups allow-same-origin allow-scripts allow-top-navigation-by-user-activation" allow="autoplay *; encrypted-media *; clipboard-write" style="width: 100%; overflow: hidden; border-radius: 10px; transform: translateZ(0px); animation: 2s ease 0s 6 normal none running loading-indicator; background-color: rgb(228, 228, 228);"></iframe>

{{ else if and (.Get "url") (.Get "i") }}
<iframe id="embedPlayer" src="https://embed.music.apple.com/us/album/{{ .Get "url" }}?i={{ .Get "i" }}&amp;app=music&amp;itsct=music_box_player&amp;itscg=30200&amp;ls=1&amp;theme=auto" height="175px" frameborder="0" sandbox="allow-forms allow-popups allow-same-origin allow-scripts allow-top-navigation-by-user-activation" allow="autoplay *; encrypted-media *; clipboard-write" style="width: 100%; overflow: hidden; border-radius: 10px; transform: translateZ(0px); animation: 2s ease 0s 6 normal none running loading-indicator; background-color: rgb(228, 228, 228);"></iframe>

{{ else }}
Missing URL parameter in Apple Music shortcode
{{ end }}
```

## 折叠卡片

标识符：`fold-block`

### 短代码

```
{{</* fold-block "这是提示文字" */>}}

~~这是折叠的内容～~~

***这是折叠的内容～***

**同样是MD文档，不受影响**

{{</* /fold-block */>}}
```

### 预览

{{% fold-block "这是提示文字" %}}

~~这是折叠的内容～~~

***这是折叠的内容～***

**同样是MD文档，不受影响**

{{% /fold-block %}}

### 源代码

{{% fold-block "源代码太长折叠了，直接现学现用" %}}

```html
{{ $content := .Inner | markdownify }}

<div class="collapse-block">
    <div class="collapse-header" onclick="toggleCollapse(this)">
        <span class="collapse-arrow">►</span> <strong>{{ .Get 0 }}</strong>
    </div>
    <div class="collapse-content">
        {{ $content }}
</div>

<style>
.collapse-block {
    margin-top: 12px;
    margin-bottom: 24px;
    overflow: hidden;
}

.collapse-header {
    padding: 8px;
    cursor: pointer;
    transition: background-color 0.3s ease;
    color: var(--header-text-color);
    background-color: var(--header-bg-color);
    border: 1px solid #ccc; /* 灰色边框 */
    border-radius: 0.5rem; /* 圆角 */
    padding-top: 0.5rem;
    padding-bottom: 0.5rem;
    padding-left: 0.75rem;
    padding-right: 0.75rem;
}

.collapse-header:hover {
    background-color: var(--header-bg-hover);
    color: var(--header-text-hover);
}

.collapse-arrow {
    float: left;
    margin-right: 8px;
    transition: transform 0.3s ease;
}

.collapse-content {
    padding: 0 8px;
    height: 0;
    overflow: hidden;
    background-color: rgba(255, 255, 255, 0); /* 半透明浅色背景 */
    transition: height 0.5s ease-out, padding 0.3s ease-out;
}

.collapse-content.show {
    height: auto;
    padding: 8px;
    overflow: visible;
}

/* 明亮模式下的样式 */
@media (prefers-color-scheme: light) {
    .collapse-header {
        color: white;
        background-color: black;
    }
    
    .collapse-header:hover {
        background-color: #80808020; /* 更改为暗一点的颜色 */
    }
}

/* 暗黑模式下的样式 */
@media (prefers-color-scheme: dark) {
    /* .collapse-header {
        color: black;
        background-color: white;
    } */
    
    .collapse-header:hover {
        background-color: #80808020; /* 更改为浅一点的颜色 */
    }
}
</style>

<script>
function toggleCollapse(element) {
    var content = element.nextElementSibling;
    var arrow = element.querySelector('.collapse-arrow');

    if (content.classList.contains('show')) {
        content.style.height = '0';
        content.classList.remove('show');
        arrow.style.transform = 'rotate(0deg)';
    } else {
        content.style.height = content.scrollHeight + 'px';
        content.classList.add('show');
        arrow.style.transform = 'rotate(90deg)';
    }
}
</script>

```

{{% /fold-block %}}

## 分割线

标识符：`divider`

### 短代码

```
{{</* divider "END" */>}}
{{</* divider "End" */>}}
{{</* divider "end" */>}}
{{</* divider "描述文字" */>}}
{{</* divider "不是end则显示普通文字" */>}}
```

### 预览

当描述文字是 `END` （所有字母不区分大小写）时，将会显示花体字。

{{< divider "END" >}}

{{< divider "End" >}}

{{< divider "end" >}}

{{< divider "描述文字" >}}

{{< divider "不是end则显示普通文字" >}}

【对比】这是markdown语法`---`的分割线效果：

---

### 源代码

```html
<style>
    @import url('https://fonts.googleapis.com/css2?family=Playwrite+MX&display=swap');
    
    .divider {
        display: flex;
        align-items: center;
        margin: 20px 0;
        font-size: 12px;
    }
    .divider::before, .divider::after {
        content: '';
        flex: 1;
        border-bottom: 1px solid #333;
    }
    .divider::before {
        margin-right: 10px;
    }
    .divider::after {
        margin-left: 10px;
    }
    .divider span.end {
        font-family: 'Playwrite MX', cursive;
        font-size: 16px;
    }
    </style>
    <div class="divider">
        {{- if .Get 0 -}}
        <span class="{{ if eq (lower (.Get 0)) "end" }}end{{ end }}">{{ .Get 0 }}</span>
        {{- end -}}
    </div>
```

## 友链卡片

标识符：`friend`

### 短代码

```
{{</* friend name="Daaihang Wong Online" 
	url="https://wdh.hk/" description="一个信息安全学生的网站。" 
	reason="推荐原因：这就是我！" 
	icon="https://wdh.hk/android-chrome-192x192.png" */>}}
```

### 预览

{{< friend name="Daaihang Wong Online" url="https://wdh.hk/" 

description="一个信息安全学牲的网站。" 

reason="推荐原因：这就是我！" 

icon="https://wdh.hk/android-chrome-192x192.png" >}}

### 源代码

{{% fold-block "源代码太长折叠了" %}}

```html
<a href="{{ .Get "url" }}" target="_blank" rel="noopener noreferrer" class="friend-card">
    <div class="friend-card-header">
        <div class="friend-card-info">
            <h3 class="friend-card-title">{{ .Get "name" }}</h3>
            <p class="friend-card-url">{{ .Get "url" }}</p>
            <p class="friend-card-description">{{ .Get "description" }}</p>
        </div>
        <img src="{{ .Get "icon" }}" alt="{{ .Get "name" }} logo" class="friend-card-icon">
    </div>
    <p class="friend-card-reason">{{ .Get "reason" }}</p>
</a>

<style>
.friend-card {
    display: block;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    padding: 16px;
    width: 100%;
    box-sizing: border-box;
    margin: 16px 0;
    text-decoration: none;
    color: inherit;
    transition: box-shadow 0.3s;
    text-align: left; /* 确保内容居左 */
}
.friend-card:hover {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.friend-card-header {
    display: flex;
    align-items: center;
    justify-content: space-between; /* 确保图标在最右边 */
    margin-bottom: 8px;
}
.friend-card-icon {
    width: 72px;
    height: auto;
    /* border-radius: 50%; */
    border-radius: 0.5em;
    transition: transform 0.3s;
    margin-right: 16px;
}
.friend-card:hover .friend-card-icon {
    transform: scale(1.2);
}
.friend-card-info {
    display: flex;
    flex-direction: column;
    justify-content: center;
    flex-grow: 1; /* 确保信息部分占据剩余空间 */
}
.friend-card-title {
    margin: 0;
    font-size: 1.2em;
    color: #333;
}
.friend-card-url {
    font-size: 0.9em;
    color: #1e88e5;
    margin: 4px 0;
}
.friend-card-description {
    font-size: 0.9em;
    color: #666;
    margin-top: 0;
}
.friend-card-reason {
    font-size: 0.9em;
    color: #666;
    border-top: 1px solid #e0e0e0;
    padding-top: 8px;
    margin: 0;
}
</style>
```

{{% /fold-block  %}}
