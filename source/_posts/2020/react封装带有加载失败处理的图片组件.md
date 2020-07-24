---
title: react封装带有加载失败处理的图片组件
date: 2020/07/13 23:10:00
updated: 2020/07/13 23:10:00
categories: 
- react
tags: 
- react
- typescript
- 组件
---

今天我们要来说一说一个很常用的东西：一个图片组件，不用写一堆繁琐的 `onError`，不用把代码搞的到处都是，当图片加载失败时，会用默认图片代替原来的图。

# 实现原理
`img` 标签有一个 `onerror` 事件，当图片加载失败时就会执行这个事件，我们只要在这个事件中，替换图片的 `src` 属性，就可以在加载失败时用默认图替换原有的图片。

```html
<!-- logo.png加载失败时，会显示logoError.png -->
<img src="logo.png" onerror="javascript:this.src='logoError.png';" />
```

<!-- more -->

# 封装组件
知道了原理就好办了，在react中，要使用 `onError` 事件：

```typescript
import React from 'react'

interface Props {
  src: string // img原本的的src
  style?: React.CSSProperties
  className: string
  defaultImg: string // 发生错误时需要使用的图片
}

const ImgWithDefault: React.FC<Props> = ({ src, style = {}, className = '', defaultImg }) => {
  const Img = React.createRef<HTMLImageElement>()
  return (
    <img
      ref={Img}
      style={style}
      className={className}
      src={src}
      onError={() => {
        if (Img.current) {
          Img.current.src = defaultImg
        }
      }}
      alt=""
    />
  )
}

export default ImgWithDefault
```

当网络状况很差时，`defaultImg`也可能加载失败，这样`onError`就会陷入死循环，因此这里需要做个改造：

```typescript
import React, { useState } from 'react'

interface Props {
  src: string
  style?: React.CSSProperties
  className: string
  defaultImg: string
}

const ImgWithDefault: React.FC<Props> = ({ src, style = {}, className = '', defaultImg }) => {
  // 新增一个状态，标记是否发生过错误
  const [imgError, setImgError] = useState(false)
  const Img = React.createRef<HTMLImageElement>()

  if (!imgError) {
    return (
      <img
        ref={Img}
        style={style}
        className={className}
        src={src}
        onError={() => {
          if (Img.current) setImgError(true)
        }}
        alt=""
      />
    )
  } else {
    // 重点在这里，如果发生错误的话，不再带有onerror处理，防止陷入僵局
    return <img style={style} className={className} src={defaultImg} alt="" />
  }
}
```

# 使用方法
使用起来很简单：

```javascript
import DefaultAvatar from '@/assets/avatar.png'
<ImgWithDefault src={user.avatar} className={styles.avatar} defaultImg={DefaultAvatar} />
```
