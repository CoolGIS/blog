---
title: ArcGIS API for JavaScript 安装和配置
date: 2022-03-30 15:54:44
categories:
  - ArcGIS API for JavaScript基础教程 
tags:
  - ArcGIS
  - API
  - 教程
  - 安装
  - 配置
---

ArcGIS API for JavaScript是将地理信息或者地理数据进行可视化表达或者地理分析处理的JS库，可用于浏览器环境和Node.js环境[^1]。要在项目中使用ArcGIS JS API，有下面几种方式。

## AMD与ESM

*选择API安装方式的前置条件，只做简单介绍。*

- **AMD**是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

  ```js
  require([module], callback);
  ```

  第一个参数`[module]`，是一个数组，里面的成员就是要加载的模块；第二个参数`callback`，则是加载成功之后的回调函数。

- **ES Module**把一个文件当作一个模块，每个模块有自己的独立作用域，核心点就是模块的导入（import）与导出（export）。

  - `export`后只能跟`function`、`class`、`var`、`let`、`const`、`default`、`{}`，export的作用就是给当前模块对象添加属性，方便后期导入到其他模块中，其中 `export default`方法最常用。

  - `import`命令用于导入其他模块提供的数据，格式：
  
    ```js
    import <module> from <url>
    ```

## API安装的四种方式

|                                         | **CDN (AMD)** | **ESM local build** | **AMD local build** |
| :-------------------------------------- | :-----------: | :-----------------: | :-----------------: |
| 不需要安装，配置和本地构建              |       X       |                     |                     |
| 通过CDN缓存快速下载                     |       X       |                     |                     |
| 可以通过npm快速安装                     |               |          X          |                     |
| 与大多数现代框架和构建工具无缝集成      |               |          X          |                     |
| 使用4.17版本或更早的API与框架或构建工具 |               |                     |          X          |
| 使用 Dojo 1 或者 RequireJS              |               |                     |          X          |

### 通过CDN提供的AMD模块

访问API的最常见方法是使用托管版本。

```html
<link rel="stylesheet" href="https://js.arcgis.com/4.20/esri/themes/light/main.css">
<script src="https://js.arcgis.com/4.20/"></script>
```

### 通过NPM提供的ES模块[^1]

API也可以通过npm作为ES模块使用。你可以在本地安装API，以便与React和Vue等JavaScript框架以及webpack等打包器一起使用。

1. 安装

   ```shell
   yarn add @arcgis/core
   ```

2. 配置CSS

   - 复制`/node_modules/@arcgis/core/assets`到`/public/assets`文件夹中

   - 配置ArcGIS资源路径为本地路径

     ```js
     import esriConfig from "@arcgis/core/config.js";
     esriConfig.assetsPath = "./assets";
     ```

   - 全局CSS文件中引入ArcGIS本地样式文件

     ```js
     @import "~@arcgis/core/assets/esri/themes/light/main.css";
     ```

3. 导入

   ```js
   import Map from "@arcgis/core/Map";
   ```

### 在本地托管的AMD模块

修改API中`init.js`文件和`dojo.js`文件，将修改好的API部署在自有服务器中，用于网络缺失及网络较差的环境中。

#### 较早版本

1. 下载

   [ArcGIS JS API 下载网站](https://developers.arcgis.com/downloads/#javascript)

   ![image-20210830174330752](https://cdn.jsdelivr.net/gh/CoolGIS/img-repo/img/image-20210830174330752.png)

   下载分为API和SDK，*API*包含开发所需的库文件，*SDK*为离线文档和实例[^2]。

2. 将压缩包解压，复制`\arcgis_js_v420_api\arcgis_js_api\javascript\4.20\`  及下面所有内容到托管服务器目录中。例如`C:\Inetpub\wwwroot\javascript\api\4.20\`。

3. 打开 `C:\Inetpub\wwwroot\javascript\api\4.20\init.js` 搜索 `[HOSTNAME_AND_PATH_TO_JSAPI]`, 并替换为以下字符串 `www.example.com/javascript/api/4.20/init.js`。


#### 较新版本

1. 下载同上
2. 将压缩包解压，复制`\javascript\4.22\`  及下面所有内容到托管服务器目录中。例如`C:\Inetpub\wwwroot\javascript\4.22\`。
3. 通过浏览器访问安装根目录中的`index.html`文件，如`http://localhost/javascript/4.22/`，根据输出信息判断是否安装成功。

### 通过CDN提供的ES模块[^1]

==注意：这种方法目前只推荐用于开发和原型阶段。==

```html
<link rel="stylesheet" href="https://js.arcgis.com/4.20/@arcgis/core/assets/esri/themes/light/main.css">
<script type="module">
  import Map from "https://js.arcgis.com/4.20/@arcgis/core/Map.js";

  // Use the Map class
</script>
```

## 服务器配置

托管 ArcGIS API for JavaScript 的 Web 服务器将需要注册以下 MIME/type（主要为IIS）。

| extension | MIME/type                  | Description                                                  |
| --------- | -------------------------- | ------------------------------------------------------------ |
| `.ttf`    | `application/octet-stream` | True Type Fonts                                              |
| `.wasm`   | `application/wasm`         | [WebAssembly](http://webassembly.org/)                       |
| `.woff`   | `application/font-woff`    | [Web Open Font Format](https://developer.mozilla.org/en-US/docs/Web/Guide/WOFF) |
| `.woff2`  | `application/font-woff2`   | [WOFF File Format 2.0](https://www.w3.org/TR/WOFF2/)         |
| `.wsv`    | `application/octet-stream` | Supports `SceneView`'s stars visualization                   |

[^1]:版本4.18开始提供。
[^2]:需要配置为离线访问。
