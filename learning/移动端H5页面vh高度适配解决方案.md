## 背景

由于移动端真机上的浏览器底部存在操作栏，当使用 100vh 去适配高度时，底部操作栏的也会被统计在内，导致底部部分内容被操作栏盖住，但是使用 100% 去适配高度的话在一些复杂场景下存在问题，更难以兼容，基此目前通过动态兼容 vh 去解决

<br />

<br />

## 解决方案

 经过大量调试，发现我们可以使用动态计算浏览器可视区域去动态设置 root 根属性变量高度，然后再通过 scss 函数去获取 100vh高度

<br />

#### 步骤

* 在 App.vue 中动态获取可视区域高度，并设置根元素变量

  ```js
  mounted() {
    const vh = window.innerHeight
    document.documentElement.style.setProperty('--vh', `${vh}px`)
  
    window.addEventListener('resize', () => {
      const vh = window.innerHeight
      document.documentElement.style.setProperty('--vh', `${vh}px`)
    })
  },
  ```

* 使用 vh 的地方均需要做兼容处理

  ```css
  // 常规设置 100vh 高度
  height: 100vh;
  height: var(--vh, 1vh);
  
  // 计算设置 vh 高度
  margin-top: 5vh;
  margin-top: calc(var(--vh, 1vh) * 0.05);
  ---------------------------------------------------------------
  height: calc(100vh - 60px);
  height: calc(var(--vh, 1vh) - 60px);
  ```