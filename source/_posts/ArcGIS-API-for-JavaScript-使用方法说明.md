---
title: ArcGIS API for JavaScript 使用方法说明
toc: true
date: 2022-03-30 16:56:53
categories:
  - ArcGIS API for JavaScript基础教程
tags:
  - ArcGIS
  - API
  - 教程
  - 使用方法
  - 技巧
---

## 构造函数

ArcGIS API for JavaScript中的所有类都有一个构造函数，所有属性都可以通过向构造函数传递参数来设置。

<!--more-->

```js
const map = new Map({
  basemap: "topo-vector"
});

const view = new MapView({
  map: map,
  container: "map-div",
  center: [ -122, 38 ],
  scale: 5
});
```

另外，对象的属性可以直接使用[setters](https://developers.arcgis.com/javascript/latest/programming-patterns/#setters)来指定。

```js
const map = new Map();
const view = new MapView();

map.basemap = "topo-vector";           // Set a property

const viewProps = {                    // Object with property data
  container: "map-div",
  map: map,
  scale: 5000,
  center: [ -122, 38 ]
};

view.set(viewProps);                   // Use a setter
```

## 属性

### Getters

`get`方法返回一个属性的值，这个方法是一个快捷方法，因为如果不使用`get()`，要返回嵌套属性的值，例如，要返回一个`Map`对象的属性`basemap`的标题，需要一个`if`语句来检查`basemap`是否未定义或为空。

```js
const basemapTitle = null;
if (map.basemap) {                     // Make sure `map.basemap` exists
  basemapTitle = map.basemap.title;
}
```

`get` 方法不需要 `if` 语句，如果 `map.basemap` 存在，则返回 `map.basemap.title` 的值，否则返回 `null`。

```js
const basemapTitle = map.get("basemap.title");
```

==注意：==

> 也可以采用可选链(`?.`)操作规避这类问题，但是需要考虑浏览器兼容性。
>
> ```js
> const basemapTitle = map?.basemap.title;
> ```

### Setters

可以直接设置属性的值。

```js
view.center = [ -100, 40 ];
view.zoom = 6;
map.basemap = "oceans";
```

当需要更改多个属性值时， `set()` 可以传递一个带有属性名和新值的 JavaScript 对象。

```js
const newViewProperties = {
  center: [ -100, 40 ],
  zoom: 6
};

view.set(newViewProperties);
```

### 监听属性变化

使用 `watch()` 来监听属性的变化，它带有两个参数：作为字符串的属性名称，以及当属性值更改时调用的回调函数。`watch()`返回一个`WatchHandle`实例。
代码示例监听一个`Map`对象的`basemap.title`属性，每当底图标题发生改变时，就调用`titleChangeCallback`函数。

```js
const map = new Map({
  basemap: "streets-vector"
});

function titleChangeCallback (newValue, oldValue, property, object) {
  console.log("New value: ", newValue,
              "<br>Old value: ", oldValue,
              "<br>Watched property: ", property,
              "<br>Watched object: ", object);
};

const handle = map.watch('basemap.title', titleChangeCallback);
```

可以在 `WatchHandle` 对象上调用 `remove` 方法来停止监听。

```js
handle.remove();
```

==注意：==

> 并非所有的属性都可以被监听，例如集合。注册一个事件处理程序，以便在一个集合发生[changes](https://developers.arcgis.com/javascript/latest/api-reference/esri-core-Collection.html#event-change)时可以被监听。

> 因为`FeatureLayer.source`和`GraphicsLayer.graphics`属性都是集合，使用`on()`而不是`watch()`来监听这些属性的变化。

## Autocasting

Autocasting将JavaScript对象作为ArcGIS API for JavaScript 类类型，而不需要开发人员显式导入这些类。

==注意：==

> 目前，由于TypeScript的限制，Autocasting在非TypeScript应用程序中效果最好。

代码示例中，需要五个API类来为一个`FeatureLayer`创建一个`SimpleRenderer`。

```js
require([
  "esri/Color",
  "esri/symbols/SimpleLineSymbol",
  "esri/symbols/SimpleMarkerSymbol",
  "esri/renderers/SimpleRenderer",
  "esri/layers/FeatureLayer",
], (
  Color, SimpleLineSymbol, SimpleMarkerSymbol, SimpleRenderer, FeatureLayer
) => {
  const layer = new FeatureLayer({
    url: "https://services.arcgis.com/V6ZHFr6zdgNZuVG0/arcgis/rest/services/WorldCities/FeatureServer/0",
    renderer: new SimpleRenderer({
      symbol: new SimpleMarkerSymbol({
        style: "diamond",
        color: new Color([255, 128, 45]),
        outline: new SimpleLineSymbol({
          style: "dash-dot",
          color: new Color([0, 0, 0])
        })
      })
    })
  });
});
```

通过Autocasting，不必导入渲染器和符号类；唯一需要导入的模块是`esri/layers/FeatureLayer`。

```js
require([ "esri/layers/FeatureLayer" ], (FeatureLayer) => {
  const layer = new FeatureLayer({
    url: "https://services.arcgis.com/V6ZHFr6zdgNZuVG0/arcgis/rest/services/WorldCities/FeatureServer/0",
    renderer: {                        // autocasts as new SimpleRenderer()
      symbol: {                        // autocasts as new SimpleMarkerSymbol()
        type: "simple-marker",
        style: "diamond",
        color: [ 255, 128, 45 ],       // autocasts as new Color()
        outline: {                     // autocasts as new SimpleLineSymbol()
          style: "dash-dot",
          color: [ 0, 0, 0 ]           // autocasts as new Color()
        }
      }
    }
  });
});
```

想知道一个类是否可以Autocasting，查看每个类的文档。如果一个属性可以被Autocasting，就会出现以下图片：

![image-20210903161646550](https://cdn.jsdelivr.net/gh/CoolGIS/img-repo/img/image-20210903161646550.png)

## 异步数据

### Promise

`Promise`通常与`then()`一起使用，它定义了回调函数，如果`resolved`，则调用该函数，如果`rejected`，则调用错误函数。第一个参数始终是成功回调，第二个可选参数是错误回调。

```js
someAsyncFunction().then(callback, errorCallback);
```

`catch()`方法可以用来指定一个`Promise`或`Promise`函数链的错误回调函数。

```js
someAsyncFunction()
  .then((resolvedVal) => {
    console.log(resolvedVal);
  }).catch((error) => {
    console.error(error);
  });
```

示例中，`GeometryService`被用来将几个点的几何图形投射到一个新的空间参考。在`GeometryService.project`的文档中，`project()`返回一个Promise，该Promise解析为一个投影的几何体数组。

```js
require([
  "esri/tasks/GeometryService",
  "esri/tasks/support/ProjectParameters",
  ], (GeometryService, ProjectParameters) => {
    const geoService = new GeometryService( "https://sampleserver6.arcgisonline.com/arcgis/rest/services/Utilities/Geometry/GeometryServer" );
    const projectParams = new ProjectParameters({
      geometries: [points],            
      outSR: outSR,
      transformation = transformation
    });
    geoService.project(projectParams)
      .then((projectedGeoms) => {
       console.log("projected points: ", projectedGeoms);
      }, (error) => {
        console.error(error);
      });
});
```

### Promise 函数链

使用`Promise` 的优点之一是可以利用`then()`将多个`Promise` 连接起来。
当一个以上的Promise 被链在一起时，一个Promise 的解析值将被被传递给下一个Promise 。这使得代码块可以按顺序执行，而不需要在彼此之间嵌套回调。

==注意：==

> 回调函数必须使用return关键字来返回一个值给下一个Promise 。

```js
  const bufferLayer = new GraphicsLayer();

  function bufferPoints(points) {
    return geometryEngine.geodesicBuffer(points, 1000, "feet");
  }

  function addGraphicsToBufferLayer(buffers) {
    buffers.forEach((buffer) => {
      bufferLayer.add(new Graphic(buffer));
    });
    return buffers;
  }

  function calculateArea(buffer) {
    return geometryEngine.geodesicArea(buffer, "square-feet");
  }

  function calculateAreas(buffers) {
    return buffers.map(calculateArea);
  }

  function sumArea(areas) {
    for (let i = 0, total = 0; i < areas.length; i++) {
      total += areas[i];
    }
    return total;
  }

  geoService.project(projectParams)
    .then(bufferPoints)
    .then(addGraphicsToBufferLayer)
    .then(calculateAreas)
    .then(sumArea)
    .catch((error) => {
      console.error("One of the promises in the chain was rejected! Message:", error);
    });
```

> 也可以使用`async`，`await`关键词，简单来说，它们是基于`Promise`的语法糖，使异步代码看起来更像是同步代码。具体请查阅[文档](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous/Async_await)。

### 手动加载

如果`View`是用`Map`实例构造的，那么`load`方法会自动执行，加载`View`和关联的`Map`中引用的所有资源。简单来讲，当实例化的图层并没有被map或view引用时，图层服务的所有资源并不会自动加载，需要手动调用`load`方法加载它所需的资源来完成初始化。

load状态包括`not-loaded`，`loading`，`failed`，`loaded`，分别代表尚未加载，加载中，加载失败和加载成功。

下面的状态转换代表了一个可加载资源所经历的阶段。

![loadable-pattern](https://cdn.jsdelivr.net/gh/CoolGIS/img-repo/img/loadable-pattern.png)

可以用以下代码验证load状态：

```js
require(["esri/Map", "esri/views/MapView", "esri/layers/FeatureLayer"], (Map, MapView, FeatureLayer) => {
        const map = new Map({
            basemap: "osm"
        })
        const view = new MapView({
            map,
            container: "view"
        })
        const layer = new FeatureLayer({
            url: "https://sampleserver6.arcgisonline.com/arcgis/rest/services/USA/MapServer/0"
        });
        console.log(layer.loadStatus)
        console.log(layer.fields)
        layer.load().then(() => {
            console.log(layer.loadStatus)
            console.log(layer.fields)
        }, (err) => {
            console.log(err)
        })
    });
```

这是一个没有与`Map`或者`View`关联的`FeatureLayer`，在实例化之后输出它的加载状态和包含的字段属性，并在手动调用`load()`之后再次输出加载状态和包含的字段。浏览器输出如下所示：

![image-20210903180058302](https://cdn.jsdelivr.net/gh/CoolGIS/img-repo/img/image-20210903180058302.png)

从结果可验证如上所说。

## fromJSON 方法

`fromJSON`方法从ArcGIS产品生成的`JSON`中创建一个指定类的实例。这种格式的`JSON`通常由`toJSON()`方法或通过`REST API`的查询创建。不能与一般的`JSON`或`GeoJSON`混用。

### jsonUtils 辅助方法

在使用`fromJSON()`实例化一个对象时，有几个`jsonUtils`类作为工具类提供。

- [`esri/geometry/support/jsonUtils`](https://developers.arcgis.com/javascript/latest/api-reference/esri-geometry-support-jsonUtils.html)
- [`esri/renderers/support/jsonUtils`](https://developers.arcgis.com/javascript/latest/api-reference/esri-renderers-support-jsonUtils.html)
- [`esri/symbols/support/jsonUtils`](https://developers.arcgis.com/javascript/latest/api-reference/esri-symbols-support-jsonUtils.html)

当一个`JSON`对象是来自REST API的几何体、渲染器或符号，但对象类型未知时，这些类可以快速处理这种情况。例如，当一个图层的渲染器来自于REST请求，并且不确定该渲染器是否是`UniqueValueRenderer`时，调用`esri/renderers/support/jsonUtils`类来帮助确定该渲染器的类型。

```js
require([ "esri/renderers/support/jsonUtils",
          "esri/layers/FeatureLayer"
], ( rendererJsonUtils, FeatureLayer ) => {

  const rendererJSON = {               // renderer object obtained via REST request
     "authoringInfo":null,
     "type":"uniqueValue",
     "field1":"CLASS",
     "field2":null,
     "field3":null,
     "expression":null,
     "fieldDelimiter":null,
     "defaultSymbol":{
        "color":[
           235,
           235,
           235,
           255
        ],
        "type":"esriSLS",
        "width":3,
        "style":"esriSLSShortDot"
     },
     "defaultLabel":"Other major roads",
     "uniqueValueInfos":[
        {
           "value":"I",
           "symbol":{
              "color":[
                 255,
                 170,
                 0,
                 255
              ],
              "type":"esriSLS",
              "width":10,
              "style":"esriSLSSolid"
           },
           "label":"Interstate"
        },
        {
           "value":"U",
           "symbol":{
              "color":[
                 223,
                 115,
                 255,
                 255
              ],
              "type":"esriSLS",
              "width":7,
              "style":"esriSLSSolid"
           },
           "label":"US Highway"
        }
     ]
  };

  // Create a renderer object from its JSON representation
  const flRenderer = rendererJsonUtils.fromJSON(rendererJSON);

  // Set the renderer on a layer
  const layer = new FeatureLayer({
    renderer: flRenderer
  });
});
```
