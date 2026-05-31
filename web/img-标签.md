下面按 **前端面试常问方向** 给你整理。图片相关问题一般不会只问 `<img>` 标签，而是会串到 **性能优化、浏览器加载、缓存、事件、React 组件封装、用户体验、网络错误处理**。

------

## 一、图片基础考点

### 1. img 标签常用属性

常问属性：

```html
<img
  src="/a.png"
  alt="图片描述"
  width="300"
  height="200"
  loading="lazy"
/>
```

重点解释：

| 属性                   | 作用                                         |
| ---------------------- | -------------------------------------------- |
| `src`                  | 图片资源地址                                 |
| `alt`                  | 图片加载失败时的替代文本，也用于无障碍和 SEO |
| `width / height`       | 提前占位，减少页面抖动                       |
| `loading="lazy"`       | 原生懒加载                                   |
| `srcset`               | 响应式图片，根据设备选择不同图片             |
| `sizes`                | 配合 `srcset` 告诉浏览器图片显示尺寸         |
| `decoding="async"`     | 异步解码图片，减少阻塞                       |
| `fetchpriority="high"` | 提高关键图片加载优先级                       |

面试重点：
`alt` 不只是图片失败时显示文字，也和 SEO、无障碍有关。

------

## 二、图片优化考点

图片优化可以从这几个角度答：

```text
图片体积
图片格式
加载时机
加载优先级
缓存/CDN
布局稳定性
```

### 1. 图片体积优化

常见手段：

```text
压缩图片
裁剪图片尺寸
避免大图小用
使用合适格式
按需加载
```

比如展示区域只有 `200px × 200px`，就不要加载 `2000px × 2000px` 的原图。

### 2. 图片格式优化

常见格式：

| 格式       | 特点                                   |
| ---------- | -------------------------------------- |
| JPG / JPEG | 适合照片，体积较小，不支持透明         |
| PNG        | 支持透明，适合图标、截图，但体积较大   |
| WebP       | 体积更小，支持透明，现代项目常用       |
| AVIF       | 压缩率更高，但兼容性需要考虑           |
| SVG        | 矢量图，适合图标、logo                 |
| GIF        | 动图，但体积大，现在很多场景用视频替代 |

面试回答：

> 图片优化首先会选择合适格式，比如照片用 WebP/JPEG，图标用 SVG，透明图片用 PNG/WebP。对于大图要压缩和裁剪，避免加载远大于展示尺寸的图片。

------

## 三、图片懒加载考点

### 1. 原生懒加载

```html
<img src="/list.jpg" loading="lazy" alt="列表图片" />
```

回答：

> `loading="lazy"` 是浏览器原生懒加载，图片接近可视区域时才会加载，可以减少首屏请求数量和带宽压力。

### 2. IntersectionObserver 懒加载

核心思路：

```html
<img data-src="/real.jpg" src="/placeholder.jpg" class="lazy-img" />
const imgs = document.querySelectorAll('.lazy-img')

const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      const img = entry.target
      img.src = img.dataset.src
      observer.unobserve(img)
    }
  })
})

imgs.forEach((img) => observer.observe(img))
```

面试回答：

> 复杂场景可以用 `IntersectionObserver`。一开始把真实图片地址放到 `data-src`，等图片进入视口时，再把 `data-src` 赋值给 `src`，图片才开始加载，加载后取消监听。

### 3. scroll 懒加载

老方案：

```js
window.addEventListener('scroll', () => {
  // 判断图片是否进入视口
})
```

缺点：

```text
滚动频繁触发
需要手动计算位置
需要节流
性能不如 IntersectionObserver
```

------

## 四、图片加载成功/失败处理

### 1. 加载成功：onload

```js
const img = document.querySelector('img')

img.onload = function () {
  console.log('图片加载成功')
}
```

React：

```jsx
<img
  src={url}
  onLoad={() => console.log('加载成功')}
/>
```

### 2. 加载失败：onerror

```js
const img = document.querySelector('img')

img.onerror = function () {
  img.src = '/default.png'
}
```

React：

```jsx
function Avatar({ src }) {
  const handleError = (e) => {
    e.currentTarget.onerror = null
    e.currentTarget.src = '/default-avatar.png'
  }

  return <img src={src} alt="头像" onError={handleError} />
}
```

### 3. 重点坑：避免无限触发 error

如果默认图也加载失败，会继续触发 `onError`。

所以要么：

```js
e.currentTarget.onerror = null
```

要么用状态控制：

```jsx
import { useState } from 'react'

function Avatar({ src }) {
  const [imgSrc, setImgSrc] = useState(src)
  const [failed, setFailed] = useState(false)

  const handleError = () => {
    if (failed) return
    setFailed(true)
    setImgSrc('/default-avatar.png')
  }

  return <img src={imgSrc} alt="头像" onError={handleError} />
}
```

------

## 五、响应式图片考点

### 1. srcset

```html
<img
  src="image-800.jpg"
  srcset="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w
  "
  sizes="(max-width: 600px) 400px, 800px"
  alt="响应式图片"
/>
```

作用：

> 让浏览器根据屏幕宽度、设备像素比、图片展示尺寸，自动选择合适的图片资源。

### 2. picture 标签

```html
<picture>
  <source srcset="/image.avif" type="image/avif" />
  <source srcset="/image.webp" type="image/webp" />
  <img src="/image.jpg" alt="示例图片" />
</picture>
```

作用：

```text
优先使用 AVIF
不支持 AVIF 时使用 WebP
再不支持时降级为 JPG
```

面试回答：

> `picture` 更适合做格式降级和多端适配，`srcset` 更适合按分辨率选择不同尺寸图片。

------

## 六、图片和性能指标考点

### 1. LCP

LCP 经常和首屏大图有关。

如果首页 banner 是最大内容元素，它通常会影响 LCP。

优化方式：

```text
不要懒加载首屏大图
使用 preload 预加载
使用 fetchpriority="high"
压缩图片体积
使用 CDN
提前设置宽高
```

示例：

```html
<link rel="preload" as="image" href="/banner.webp" />

<img
  src="/banner.webp"
  fetchpriority="high"
  width="1200"
  height="600"
  alt="首页横幅"
/>
```

### 2. CLS

图片没有固定尺寸，加载完成后可能把下面内容挤下去，造成布局偏移。

解决：

```html
<img src="/cover.jpg" width="300" height="200" alt="封面" />
```

或者：

```css
.cover {
  width: 300px;
  aspect-ratio: 3 / 2;
  object-fit: cover;
}
```

------

## 七、图片和浏览器加载流程

这个可以和“从输入 URL 到页面展示”关联起来。

浏览器解析 HTML 时，遇到：

```html
<img src="/a.png" />
```

会发起图片资源请求。图片加载回来后，需要解码，然后参与绘制。

关键点：

```text
img 不会阻塞 DOM 解析
图片会影响页面渲染和 LCP
图片尺寸变化可能导致回流
图片资源可以被浏览器缓存
```

你的题库里“从输入 URL 到页面展示”里也提到了浏览器会加载 CSS、JS、图片等资源，然后构建 DOM、CSSOM、Render Tree，再进行 Layout、Paint、Composite 。图片如果尺寸变化，就可能影响 Layout，也就是回流；题库里也把回流、重绘作为浏览器性能优化重点 。

------

## 八、图片缓存考点

图片属于典型静态资源，可以走浏览器缓存。

常见缓存：

```text
强缓存：Cache-Control / Expires
协商缓存：ETag / Last-Modified
```

你的题库里也整理过强缓存和协商缓存：强缓存是不请求服务器直接读缓存，协商缓存是向服务器确认资源是否有效 。

面试回答：

> 图片一般作为静态资源，可以放 CDN，并设置较长的缓存时间。对于带 hash 的构建产物，可以设置强缓存；如果资源可能变化，可以用 ETag 或 Last-Modified 做协商缓存。

------

## 九、图片 CDN 考点

CDN 的作用：

```text
就近访问
减少源站压力
提升静态资源加载速度
```

你的题库里也提到 CDN 是内容分发网络，用于加速静态资源访问，访问流程一般是 DNS 到 CDN 节点，节点命中缓存则直接返回，未命中再回源 。

图片优化里可以说：

> 图片这类静态资源适合放 CDN，通过 CDN 节点缓存和就近访问，减少网络延迟。

------

## 十、图片跨域与防盗链考点

有些图片加载失败，不一定是地址错，也可能是：

```text
CDN 防盗链
跨域资源限制
服务端拒绝访问
Referer 校验失败
图片格式不支持
```

注意：

普通 `<img src="">` 加载跨域图片通常是允许的。

但是如果你要把图片画到 canvas，再读取像素，就会涉及跨域污染：

```js
const canvas = document.querySelector('canvas')
const ctx = canvas.getContext('2d')

ctx.drawImage(img, 0, 0)
const data = ctx.getImageData(0, 0, 100, 100)
```

如果图片没有正确设置 CORS，canvas 会被污染，读取像素会报错。

需要：

```html
<img crossorigin="anonymous" src="https://cdn.example.com/a.png" />
```

同时服务端要设置：

```http
Access-Control-Allow-Origin: *
```

------

## 十一、React 中图片封装考点

面试可能让你封装一个图片组件。

可以考虑这些能力：

```text
懒加载
加载中占位
加载失败兜底图
固定宽高
onLoad / onError 回调
防止重复 error
```

示例：

```jsx
import { useState } from 'react'

function SmartImage({
  src,
  fallback = '/default.png',
  alt = '',
  width,
  height,
  lazy = true
}) {
  const [currentSrc, setCurrentSrc] = useState(src)
  const [hasError, setHasError] = useState(false)

  const handleError = () => {
    if (hasError) return
    setHasError(true)
    setCurrentSrc(fallback)
  }

  return (
    <img
      src={currentSrc}
      alt={alt}
      width={width}
      height={height}
      loading={lazy ? 'lazy' : 'eager'}
      onError={handleError}
    />
  )
}
```

------

## 十二、图片上传预览考点

这也可能被问到。

### 1. FileReader 预览

```js
const file = input.files[0]
const reader = new FileReader()

reader.onload = function () {
  img.src = reader.result
}

reader.readAsDataURL(file)
```

特点：

```text
会把图片转成 base64
体积会变大
适合小图片预览
```

### 2. URL.createObjectURL 预览

```js
const file = input.files[0]
const url = URL.createObjectURL(file)

img.src = url
```

用完释放：

```js
URL.revokeObjectURL(url)
```

特点：

```text
性能更好
不需要转 base64
适合大图片预览
注意释放 URL，避免内存泄漏
```

------

## 十三、图片相关高频面试题清单

### 基础类

```text
1. img 的 alt 有什么作用？
2. img 加载失败怎么处理？
3. img 加载成功怎么监听？
4. img 标签会阻塞页面渲染吗？
5. img 的 width 和 height 为什么建议写？
```

### 优化类

```text
1. 图片有哪些优化方式？
2. 如何实现图片懒加载？
3. loading="lazy" 的原理是什么？
4. 首屏图片可以懒加载吗？
5. 如何优化 LCP 图片？
6. 如何减少图片导致的 CLS？
7. WebP、AVIF、PNG、JPG 有什么区别？
8. 图片为什么要放 CDN？
9. 图片缓存怎么做？
```

### 进阶类

```text
1. srcset 和 sizes 是什么？
2. picture 标签有什么作用？
3. 图片跨域会有什么问题？
4. canvas 绘制跨域图片为什么会报错？
5. 如何封装一个 React 图片组件？
6. 图片上传后如何预览？
7. FileReader 和 URL.createObjectURL 有什么区别？
```

------

## 十四、面试统一回答模板

你可以背这个版本：

> 图片相关优化我一般会从体积、格式、加载时机、缓存和展示稳定性几个方面考虑。体积上会压缩图片，避免大图小用，并根据不同设备提供不同尺寸图片；格式上会优先使用 WebP、AVIF，图标类资源可以用 SVG。加载时机上，非首屏图片可以使用 `loading="lazy"` 或 `IntersectionObserver` 做懒加载，首屏关键图片不建议懒加载，可以使用 `preload` 或 `fetchpriority="high"` 提高优先级。缓存上，图片适合放 CDN，并配合强缓存或协商缓存。展示上会设置 `width`、`height` 或 `aspect-ratio`，避免图片加载后造成布局偏移。加载失败时可以监听 `error` 事件，在 React 中用 `onError` 切换默认图，同时要防止默认图失败导致无限触发。

重点背这几个关键词：

```text
压缩
WebP / AVIF
srcset / picture
loading="lazy"
IntersectionObserver
onLoad / onError
fallback 默认图
CDN
Cache-Control
LCP
CLS
width / height
aspect-ratio
```

这类题你回答时不要只说“懒加载、压缩图片”，要主动补到 **LCP、CLS、CDN、缓存、失败兜底**，面试官会觉得你不是只会背标签属性。





