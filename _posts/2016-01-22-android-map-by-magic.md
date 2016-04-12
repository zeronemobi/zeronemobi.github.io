
# 地图技术与Jingoal海外地图业务的实现探索



本人是今目标移动中心技术开发部的magic，这篇文章是我前段时间在组内进行技术分享的一个整理，通过这篇文章可以帮你了解我们常见的地图技术。
文章内容主要包括三个部分，一是我们常见的地图（百度地图、高德地图等）是如何从基本的底层地图数据展示到前端（浏览器、手机等）的。二是介绍了一整套的开源地图解决框架，并且通过框架模拟了一张魔兽世界地图jpg文件从切片到发布再到解析到浏览器上的一整套流程。第三部分则是回顾了今目标android端在解决海外地图相关业务中的一些尝试。

----- 
## 一、地图的显示

1)  **卫星，航拍生成原始地图**

* 地图显示的第一步就是获取地图数据了，我们一般通过卫星和航拍获得原始的地图数据，如图，我们得到了一整张的世界地图。 

  > ![一张世界地图]({{ site.url }}/assets/6598249542693895477.png) 

2) **原始地图切片（经纬度，投影）**

* 把一张世界地图显示到手机里是不可能的，所以就引入了金字塔模型的概念(也就是比例尺)，我们可以根据不同的缩放比例，显示不同的分辨率。在地图应用中，我们用手指缩放和放大地图，地图显示大小的变换，都是基于金字塔模型来组织瓦片图的。我们一般按照制定的经纬度和投影的规则对地图进行切片，切片和对应级别的组织形式是一整张切成4份，这就是一个级别，然后每一小份在切4份，依次类推。通过下图可以看到，在这种通用的切片规则下，非洲西海岸就是经纬度的（0，0）点。

>![按照规则对地图进行切片][2]

3) **地图切片发布成地图服务**

* 好了，现在我们瓦片数据也有了，现在就该把数据发布成服务了，发布地图数据的web应用有很多，国内外各大地图服务提供商都有自己的发布服务的应用，开源世界也有非常好用的免费应用Geoserver，通过这些应用发布后，生成的地图服务就会如下面链接所示。

* [国测局提供在在线地图服务列表](http://www.tianditu.com/service/query.html)

* [天地图提供的全球矢量地图服务](http://t0.tianditu.com/vec_c/wmts)

* [newmap在线测试专用北京地图服务](http://www.newmapgis.com/newmap/ogc/beijing/beijing/wms?)

4) **前端框架解析地图服务展示地图（浏览器，手机view）**

* 大家可以看到，上面的地图服务打开后是以xml的形式提供的，所以，我们需要前端的框架来解析这种服务，展示地图。前端框架有很多，国内百度和高德都提供这种框架，开源世界也有OpenLayers这种免费好用的前端框架。我们打开百度地图的首页，通过浏览器看到的地图，就是通过百度的js框架解析地图服务所展示的。移动端的地图sdk也是一样的道理。

![高德地图][3]

* **一整套的开源地图解决方案**
_1.地图服务发布：Geoserver_
GeoServer是一款基于JAVA的地图发布web应用。
[Geoserver官网](http://geoserver.org/)
_2.地理解析数据库框架：gisgraphy_
gisgraphy是一款基于基于JAVA的地理解析/逆地理解析web应用，它的内置数据库是PostgreSQL+PostGIS扩展，有了它，你就可以制作基于空间的数据并进行存储，同时通过该框架生成地理解析/逆地理解析服务供其他人使用。
[gisgraphy官网](http://www.gisgraphy.com/)
_3.OpenLayers_
OpenLayers 是一个专为Web GIS 客户端开发提供的JavaScript 类库包，用于实现标准格式发布的地图数据访问，有了它，你就可以解析各大服务商的地图服务或者是你自己的，并展示到浏览器。
[OpenLayers官网](http://openlayers.org/)

## 二、制作魔兽地图
---- 
1)  **准备一张魔兽地图jpg的图片**

> ![一张魔兽地图的jpg格式文件][4]

2) **魔兽地图jpg文件切成瓦片**

* 通过地图切片工具对魔兽地图进行切片，我这里切了5-10级别。

![瓦片级别为5-10][5]

![10级别下瓦片文件][6]

3) **发布魔兽地图服务**

* 我这里用开源框架GeoServer发布。

> ![Geoserver地图服务列表下的魔兽地图服务][7]

4) **百度api解析瓦片并展示到浏览器**

* 我这里用了百度的js框架解析魔兽地图服务。重点是下图这段代码，我这里加载的是本地瓦片，把它换成地图服务的链接也是一样的。

> ![tileLayerlei类的getTileUrl方法加载魔兽地图瓦片][8]

#### 最终效果

> ![浏览器打开html文件，展示出了魔兽地图][9]

## 三、Jingoal海外地图，定位，逆地理解析的实现
---- 
>
Jingoal安卓端主要用到涉及地图的框架有高德地图SDK，高德定位SDK，和OpenStreetMap的逆地理解析服务。
### **1.地图实现**
#### _地图显示遇到的困难_
* 高德地图sdk只提供的国内的地图数据，没有国外数据。
#### _解决方案_
1. 最开始设想是维护两套地图sdk，国内用高德地图，国外用谷歌地图。但是最大的问题是要想用谷歌地图必须要手机安装谷歌service，很遗憾，国内的android手机是没有的，并且安装需要翻墙。
2. 解析海外地图的服务地址，通过高德SDK的TileOverLay类中的方法叠加海外地图图层 ，需要支出的是，找到的地图服务的规则必须是球面墨卡托投影的，否则和高德地图叠加的话位置会有偏移，最终找到了OpenStreetMap和GoogleMap的地图服务地址。
OpenStreetMap：
[OpenStreetMap地图服务瓦片地址](http://c.tile.openstreetmap.org/6/35/21.png)
GoogleMap：
[GoogleMap地图服务瓦片地址](http://mt2.google.cn/vt/lyrs=m@167000000&hl=zh-CN&gl=cn&x=420&y=193&z=9&s=Galil)
在高德sdk下调用其他地图服务的关键方法如下图，怎么样，是不是和叠加魔兽地图差不多？其实原理都是一样的，Constants.Google_URL就是地图服务的地址。

![TileOverLay类的getTileUrl方法加载谷歌地图服务][10]

![高德地图sdk显示谷歌地图][11]


### **2.定位**
#### _定位遇到的困难_
当时版本的定位sdk不支持海外定位，返回的坐标是“火星”坐标。
小tips：地球坐标系是GPS返回的WGS-84坐标系下的经纬度，“火星”坐标系是我国出于保密出发，讲WGS-84坐标系做了非线性加密处理。
定位遇到的问题
#### _解决方案_
地球<-->火星 坐标转换工具类
https://github.com/wandergis/coordtransform
### **3.逆地理解析**
#### _逆地理解析遇到的困难_
1. 如何判断所在的位置是在国内还是国外
2. 寻找靠谱的海外逆地理解析服务接口
#### _解决方案_
1. 关于判断坐标点是否在国内，github上有相关类库，通过下图方法判断，但是可以看到，边界区域会非常不准。

![通用解决方案，将中国切割成若干不规则矩形][12]

2. 海外位置的逆地理解析，基本上是找国外提供的逆地理解析服务来实现的，如下是找到的比较稳定可靠的解决方案
* 微信的解决方案Foursquar
[Foursquar](https://api.foursquare.com/v2/venues/search?client_id=PRU3F3TJWUYPZBN0LWN44PBRH35BIYBUQQNREMS0UN4GZV1V&client_secret=QFPGIII1VKYVFMMFHHVJMJ420BWP2GDOLBFHJ5H45DHJFG4G&v=20130815&ll=40.7246355,-73.9388155&)
* GoogleMap的googleplace
[GooglePlace](https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=40.7246355,-73.9388155&radius=50&key=AIzaSyCz5jwTX85gmEtE7KbfcKcjWQ1KJZ5Po_c)
* OpenStreeMap的逆地理解析服务
[OpenStreeMap](http://nominatim.openstreetmap.org/reverse?format=json&lat=40.7246355&lon=-73.9388155&zoom=18&addressdetails=1)
最终选择了OpenStreeMap的逆地理解析服务，因为返回的数据量小并且比较精确。

### 截止到目前的解决方案
上面的一些问题，其实通过高德不断的更新sdk已经很大程度上解决了，比如海外地图数据，3.2.1版本的高德sdk已经默认会在国外叠加谷歌地图的服务了。高德定位在2.0以后已经支持全球定位了，也就不用我们自己手动的进行火星和地球坐标的转换了。而判断是否在国外，高德定位2.1.0也已经提供的api，准确度也还可以。
基于以上，今目标android端国外地图相关业务基本完成。
下图是公司同事在美国出差帮忙测试的考勤应用，地图展示、定位和逆地理解析都比较准确。

![今目标西雅图分中心][13]


  [1]: http://img1.ph.126.net/FceWxtOGVNLdnU5nHHuh9w==/6598249542693895477.png
  [2]: http://img1.ph.126.net/0GZtf2AIW_VAi3mnJukkLg==/4912864243608184834.png
  [3]: http://img2.ph.126.net/eDA1q25X9TC_kuFWKMU4Ug==/1999035284699912954.jpg
  [4]: http://img1.ph.126.net/DaeniWzDJHRMWO8tytJGJw==/6598193467600879909.jpg
  [5]: http://img1.ph.126.net/CYBHxYNkjEyydKghgCv9rw==/4857695148173225560.jpg
  [6]: http://img0.ph.126.net/bN_SK_L0K8w0ao1C4MY9TA==/1994250210095805666.jpg
  [7]: http://img0.ph.126.net/f_gcGVL6Y5pc3fYaH-JxOA==/6598168178833445922.jpg
  [8]: http://img1.ph.126.net/2EVLPLJPTHO5CUucINYhjg==/4816318326596428672.jpg
  [9]: http://img1.ph.126.net/liTfdcwgl292uiwnScd8vw==/6598184671507861276.jpg
  [10]: http://img1.ph.126.net/lYOYuB1bRXRDUc5H_zEkHw==/6631201906123523422.jpg
  [11]: http://img2.ph.126.net/_CTRI8X8vTdDWjF_UnPywQ==/4858258098126730412.jpg
  [12]: http://img2.ph.126.net/i-F0oAkPqHdIufTIRsfdIw==/6598295722182268889.png
  [13]: http://img2.ph.126.net/DR20sk9KzT9RSj1ksjbZTQ==/6631301961681647123.jpg