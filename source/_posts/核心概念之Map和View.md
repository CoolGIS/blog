---
title: 核心概念之Map和View
date: 2022-03-30 16:15:55
toc: true
categories:
  - ArcGIS API for JavaScript基础教程 
tags:
  - ArcGIS
  - API
  - 教程
  - 核心概念
---

**Map**是用于管理图层和底图引用的容器。**View**用于显示地图图层并处理用户交互，弹出窗口，小部件和地图位置。通俗来说，可以理解为Map负责地图数据的管理，而View负责处理地图数据的显示及地图与用户之间的交互。

<!-- more -->

## Map介绍

地图是由`Map`类创建的。`Map`对象总是被传递给一个`View`对象。有两个`View`类用于显示地图：`MapView`类用于展示2D地图，`SceneView`类用于展示3D地图。

### 创建Map

#### 一般方法

创建map的一种方法是建立一个新的`Map`类的实例，同时指定一个底图和可选的图层集合。

> 底图与图层类型在后续的章节会详细说明。

```js
const myMap = new Map({                // Create a Map object
  basemap: "streets-vector",
  layers: additionalLayers             // Optionally, add additional layers collection
});

const mapView = new MapView({          // The View for the Map object
  map: myMap,
  container: "mapDiv"
});
```

#### 通过Web Map或Web Scene

创建map的第二种方式是加载 [web map](https://developers.arcgis.com/documentation/core-concepts/web-maps/)（用于2D地图）或[web scene](https://developers.arcgis.com/documentation/core-concepts/web-scenes/)（用于3D地图）。

> Web Map和Web Scene是包含地图或场景设置的JSON结构。这包括对底图、图层、图层样式、弹出窗口、图例、标签等的设置。它们通常是通过ArcGIS Online地图查看器或ArcGIS Online场景查看器创建的。ArcGIS Online或ArcGIS Enterprise会给它们分配一个唯一的ID，并将它们存储为[portal items](https://developers.arcgis.com/javascript/latest/api-reference/esri-portal-PortalItem.html)。

*示例：*
*`https://www.arcgis.com/home/item.html?id=41281c51f9de45edaf1c8ed44bb10e30`*

- 通过WebMap创建

  ```js
  const webMap = new WebMap({                        // Define the web map reference
    portalItem: {
      id: "41281c51f9de45edaf1c8ed44bb10e30",
      portal: "https://www.arcgis.com"               // Default: The ArcGIS Online Portal
    }
  });
  
  const view = new MapView({
    map: webMap,                                     // Load the web map
    container: "viewDiv"
  });
  ```

- 通过WebScene创建

  ```js
  const webScene = new WebScene({                    // Define the web scene reference
    portalItem: {
      id: "579f97b2f3b94d4a8e48a5f140a6639b",
      portal: "https://www.arcgis.com"               // Default: The ArcGIS Online Portal
    }
  });
  
  const view = new SceneView({                       // Load the web scene
    map: webScene,
    container: "viewDiv"
  });
  ```

## View介绍

有单独的类用于创建map和scene的视图：`MapView`和`SceneView`类。`MapView`显示的是`Map`对象的2D视图，`SceneView`显示的是3D视图。

![image-20210903131519148](https://cdn.jsdelivr.net/gh/CoolGIS/img-repo/img/image-20210903131519148.png)

### 创建View

为了使地图可见，`view`对象需要一个`map`对象和一个识别`div`元素或`div`元素引用的`id`属性的字符串。

- 创建MapView

  ```js
  const mapView = new MapView({          // Create MapView object
    map: myMap,
    container: "mapViewDiv"
  });
  ```

- 创建SceneView

  ```js
  const sceneView = new SceneView({      // Create SceneView object
    map: myMap,
    container: "sceneViewDiv"
  });
  ```

### 设置map的可视范围

`MapView`和`SceneView`的初始位置可以在创建视图时通过设置 `center` 和 `zoom` 或者 `scale`属性来设置。

```js
const view = new MapView({
  center: [ -112, 38 ],          // The center of the map as lon/lat
  zoom: 13                      // Sets the zoom level of detail (LOD) to 13
});
```

视图的位置在初始化后也可以通过更新属性来更新。

当使用`SceneView`（3D）时，可以通过定义 [`camera`](https://developers.arcgis.com/javascript/latest/api-reference/esri-views-SceneView.html#camera)属性来设置观察者的位置。

```js
const view = new SceneView({
  camera: {
    position: [
       -122, // lon
         38, // lat
      50000  // elevation in meters
    ],
    heading: 95, // direction the camera is looking
    tilt: 65 // tilt of the camera relative to the ground
  }
});
```

### 将视图通过动画改变位置
`MapView`的`goTo`方法也会改变视图的位置，但提供了额外的选项来平稳过渡。这种技术经常被用来从表面上的一个位置 "飞 "到另一个位置，或者放大到搜索结果。
`goTo`方法可以接受一个`Geometry`, `Graphic`, or [`Viewpoint`](https://developers.arcgis.com/javascript/latest/api-reference/esri-Viewpoint.html)对象。其他选项可以控制动画效果。

```js
view.goTo({                            // go to point with a custom animation duration
  target: {
      center: [ -114, 39 ]
    }, {
      duration: 5000
  });
});
```

*效果预览：*

![goTo3](https://cdn.jsdelivr.net/gh/CoolGIS/img-repo/img/goTo3.webp)

### 与视图交互

`View`还负责处理用户交互和显示弹出窗口。`View`为用户的交互提供了多个事件处理程序，如鼠标点击、键盘输入、触摸屏互动、摇杆和其他输入设备。

*示例：*

> 当用户点击地图时，默认行为是显示图层中已经预先配置好的弹出窗口。这种行为也可以用代码手动实现，即监听点击事件并使用`hitTest()`方法来寻找用户点击的要素。

```js
view.popup.autoOpenEnabled = false;  // Disable the default popup behavior

view.on("click", function(event) { // Listen for the click event
  view.hitTest(event).then(function (hitTestResults){ 
    // Search for features where the user clicked
    if(hitTestResults.results) {
      view.popup.open({ // open a popup to show some of the results
        location: event.mapPoint,
        title: "Hit Test Results",
        content: hitTestResults.results.length + "Features Found"
      });
    }
  })
});
```

### 添加小部件和 UI 组件

`view`也是一个添加小部件和HTML元素的容器。`view.ui`提供了一个`DefaultUI`容器，用来显示视图的默认widget。通过使用`view.ui.add`方法，也可以将额外的widget和HTML Elements添加到视图中。

演示添加搜索widget：

```js
  var searchWidget = new Search({
    view: view
  });

  // Add the search widget to the top right corner of the view
  view.ui.add(searchWidget, {
    position: "top-right"
  });
```
