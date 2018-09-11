# 立意之初

在GIS地图编码后期，考虑到要给客户演示GIS图素的效果，需要下载最新的瓦片地图。在搜索全网之后发现，有三种获取瓦片图的路径：

* 购买第三方地图下载器，通过下载器下载
* 高德地图授权，通过官方渠道获得
* 自己从Google Map或者高德地图官方API获得

# 确定思路

在权衡各种利弊之后，决定使用爬虫从Google Map和高德地图爬取瓦片图，一来这样可以获得最新的地图，二来可以借此机会了解一下爬虫，接触一下早就想学习的Python，三来这种方法是免费的。

# 接口调研

## Google Map接口

在使用爬虫尝试多次之后发现，一些过于老的URL已经不能获取到瓦片图png文件，下面找到了几个可以正常爬取瓦片图的URL。*【查找资料过程太过匆忙，来源已不可考。】*
> http://mt2.google.cn/vt/lyrs=p@167000000&hl=zh-CN&gl=cn&x={x}&y={y}&z={z}&s=Galil
> http://mt2.google.cn/vt/lyrs=h@167000000&hl=zh-CN&gl=cn&x={x}&y={y}&z={z}&s=Galil
> http://mt2.google.cn/vt/lyrs=y@167000000&hl=zh-CN&gl=cn&x={x}&y={y}&z={z}&s=Galil

下面的URL未经过验证
> https://mts1.google.com/vt/lyrs=m@186112443&hl=x-local&src=app&x=1325&y=3143&z=13&s=Galile

其中：
* h = roads only
* m = standard roadmap
* p = terrain
* r = somehow altered roadmap
* s = satellite only
* t = terrain only
* y = hybrid

## 高德地图接口

在查询多方资料，终于找到了一个能获取最新高德地图瓦片的URL，以及对高德地图URL的分析贴，先mark高手的分析贴：

>  **[2017版高德地图瓦片分析](https://blog.csdn.net/fredricen/article/details/77189453)**  CSDN原创分析   
>  **[高德地图瓦片深探](http://cxlwill.cn/frontend/GaodeMap/)**    个人作者摘抄CSDN并总结   

防止两个帖子随着时间消失在互联网的茫茫大河中，下面摘录重要部分。

```
目前通过高德地图官方网站的影像切换，可以看到高德的瓦片地址有如下两种:
* http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=7
* http://webst0{1-4}.is.autonavi.com/appmaptile?style=7&x={x}&y={y}&z={z}

前者是高德的新版地址，后者是老版地址。
前者lang可以通过zh_cn设置中文，en设置英文，size基本无作用，scl设置标注还是底图，scl=1代表注记，scl=2代表底图（矢量或者影像），style设置影像和路网，style=6为影像图，style=7为矢量路网，style=8为影像路网
总结之：
http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=7 为矢量图（含路网、含注记）
http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=2&style=7 为矢量图（含路网，不含注记）
http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=6 为影像底图（不含路网，不含注记）
http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=2&style=6 为影像底图（不含路网、不含注记）
http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=8 为影像路图（含路网，含注记）
http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=2&style=8 为影像路网（含路网，不含注记）

后者可以通过style设置影像、矢量、路网。
总结之
http://webst0{1-4}.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z} 为影像底图（不含路网，不含注记）
http://webst0{1-4}.is.autonavi.com/appmaptile?style=7&x={x}&y={y}&z={z} 为矢量地图（含路网，含注记）
http://webst0{1-4}.is.autonavi.com/appmaptile?style=8&x={x}&y={y}&z={z} 为影像路网（含路网，含注记）
```

# Python爬取数据

本次使用Python爬取数据的源码并不是个人编写，从Github上搜索，果然找到了已经有前人造好的轮子，Github源码地址：
> https://github.com/brandonxiang/pyMap

该作者从文档，到源码，到使用说明，一应俱全，只是源码中的瓦片图地址并不是全部都好用，现在更新了以上两章所说的Google Map和高德地图URL，截止目前（2018-9-11 23:45:25），二者皆可可以正常抓取。

由于源码改变，现将整体抓取流程叙述一遍。

## 依赖

* Python 3.7
* requests 负责下载功能
* pillow 负责图片拼接
* tqdm 负责进度条

## 安装

* 安装Python 3.7[下载地址](https://www.python.org/ftp/python/3.7.0/python-3.7.0.exe)
* 安装第三方库
```
pip install -r requirement.txt
```

## 用法

### 配置文件

***配置文件格式***

**如果使用瓦片编码下载**

```
[config]
下载方式 = 瓦片编码
左上横轴 = 803
左上纵轴 = 984
右下横轴 = 857
右下纵轴 = 1061
级别 = 8
项目名 = test
地图地址 = default
```

**如果使用地理编码下载**

```
[config]
下载方式 = 地理编码
左上横轴 = 113.889962
左上纵轴 = 22.456671
右下横轴 = 114.212686
右下纵轴 = 22.345576
级别 = 13
项目名 = sample
地图地址 = gaode
```

### 运用命令行

```
python pyMap.py
```

## 依赖文件、配置文件、源代码

### 依赖文件

requirments.txt

```
requests == 2.10.0
tqdm == 4.7.1
Pillow == 3.2.0
```

### 配置文件

config.conf

```
[config]
下载方式 = 地理编码
左上横轴 = 116.352338790894
左上纵轴 = 39.9540301751639
右下横轴 = 116.448640823364
右下纵轴 = 39.869234540941
级别 = 12
项目名 = worknet_1122
地图地址 = gaodeshiliangditu
```

### 源代码

pyMap.py

``` python
"""
github: https://github.com/brandonxiang/pyMap
license: MIT
"""
import os
import sys
import math
import requests
from PIL import Image
from tqdm import trange
import configparser

URL = {
    "gaode": "http://webrd02.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=7&x={x}&y={y}&z={z}",
    "gaode.image": "http://webst02.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}",
    "tianditu": "http://t2.tianditu.cn/DataServer?T=vec_w&X={x}&Y={y}&L={z}",
    "googlesat": "http://khm0.googleapis.com/kh?v=203&hl=zh-CN&&x={x}&y={y}&z={z}",
    "tianditusat": "http://t2.tianditu.cn/DataServer?T=img_w&X={x}&Y={y}&L={z}",
    "esrisat": "http://server.arcgisonline.com/arcgis/rest/services/world_imagery/mapserver/tile/{z}/{y}/{x}",
    "gaode.road": "http://webst02.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scale=1&style=8",
    "default": "http://61.144.226.124:9001/map/GISDATA/WORKNET/{z}/{y}/{x}.png",
    "szbuilding": "http://61.144.226.124:9001/map/GISDATA/SZBUILDING/{z}/{y}/{x}.png",
    "szbase": "http://61.144.226.44:6080/arcgis/rest/services/basemap/szmap_basemap_201507_01/MapServer/tile/{z}/{y}/{x}",
    "google": "http://mt2.google.cn/vt/lyrs=p@167000000&hl=zh-CN&gl=cn&x={x}&y={y}&z={z}&s=Galil",
    "googleh": "http://mt2.google.cn/vt/lyrs=h@167000000&hl=zh-CN&gl=cn&x={x}&y={y}&z={z}&s=Galil",
    "googley": "http://mt2.google.cn/vt/lyrs=y@167000000&hl=zh-CN&gl=cn&x={x}&y={y}&z={z}&s=Galil",
    "gaodejiuban": "http://webst01.is.autonavi.com/appmaptile?style=7&x={x}&y={y}&z={z}",
    "gaodexinban": "https://wprd01.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=7",
    # "gaodeshiliangditu":"https://wprd01.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=2&style=7",
    "gaodeweixing": "https://wprd03.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=6",
    "gaodezhuji": "https://wprd02.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=8",
    "gaodeshiliangditu": "https://wprd01.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=2&style=7"
}


def process_latlng(north, west, south, east, zoom, output='mosaic', maptype="default"):
    """
    download and mosaic by latlng

    Keyword arguments:
    north -- north latitude
    west  -- west longitude
    south -- south latitude
    east  -- east longitude
    zoom  -- map scale (0-18)
    output -- output file name default mosaic

    """
    north = float(north)
    west = float(west)
    south = float(south)
    east = float(east)
    zoom = int(zoom)
    assert (east > -180 and east < 180)
    assert (west > -180 and west < 180)
    assert (north > -90 and north < 90)
    assert (south > -90 and south < 90)
    assert (west < east)
    assert (north > south)

    # for z in range(3, zoom):
    left, top = latlng2tilenum(north, west, zoom)
    right, bottom = latlng2tilenum(south, east, zoom)
    process_tilenum(left, right, top, bottom, zoom, output, maptype)


def process_tilenum(left, right, top, bottom, zoom, output='mosaic', maptype="default"):
    """
    download and mosaic by tile number

    Keyword arguments:
    left   -- left tile number
    right  -- right tile number
    top    -- top tile number
    bottom -- bottom tile number
    zoom   -- map scale (0-18)
    output -- output file name default mosaic

    """
    left = int(left)
    right = int(right)
    top = int(top)
    bottom = int(bottom)
    zoom = int(zoom)
    assert (right >= left)
    assert (bottom >= top)

    filename = getname(output, maptype)
    download(left, right, top, bottom, zoom, filename, maptype)
    _mosaic(left, right, top, bottom, zoom, output, filename)


def download(left, right, top, bottom, zoom, filename, maptype="default"):
    for x in trange(left, right + 1):
        for y in trange(top, bottom + 1):
            path = './tiles/%s/%i/%i/%i.png' % (filename, zoom, x, y)
            if not os.path.exists(path):
                _download(x, y, zoom, filename, maptype)


def _download(x, y, z, filename, maptype):
    url = URL.get(maptype, maptype)
    path = './tiles/%s/%i/%i' % (filename, z, x)
    map_url = url.format(x=x, y=y, z=z)
    print(map_url)
    r = requests.get(map_url)

    if not os.path.isdir(path):
        os.makedirs(path)
    with open('%s/%i.png' % (path, y), 'wb') as f:
        for chunk in r.iter_content(chunk_size=1024):
            if chunk:
                f.write(chunk)
                f.flush()


def _mosaic(left, right, top, bottom, zoom, output, filename):
    size_x = (right - left + 1) * 256
    size_y = (bottom - top + 1) * 256
    output_im = Image.new("RGBA", (size_x, size_y))

    for x in trange(left, right + 1):
        for y in trange(top, bottom + 1):
            path = './tiles/%s/%i/%i/%i.png' % (filename, zoom, x, y)
            if os.path.exists(path):
                target_im = Image.open(path)
                # if target_im.mode == 'P':
                output_im.paste(target_im, (256 * (x - left), 256 * (y - top)))
                target_im.close()
    output = "output/" + output + ".png"
    output_path = os.path.split(output)
    if len(output_path) > 1 and len(output_path) != 0:
        if not os.path.isdir(output_path[0]):
            os.makedirs(output_path[0])
    output_im.save(output)
    output_im.close()


def latlng2tilenum(lat_deg, lng_deg, zoom):
    """
    convert latitude, longitude and zoom into tile in x and y axis
    referencing http://www.cnblogs.com/Tangf/archive/2012/04/07/2435545.html

    Keyword arguments:
    lat_deg -- latitude in degree
    lng_deg -- longitude in degree
    zoom    -- map scale (0-18)

    Return two parameters as tile numbers in x axis and y axis
    """
    n = math.pow(2, int(zoom))
    xtile = ((lng_deg + 180) / 360) * n
    lat_rad = lat_deg / 180 * math.pi
    ytile = (1 - (math.log(math.tan(lat_rad) + 1 / math.cos(lat_rad)) / math.pi)) / 2 * n
    return math.floor(xtile), math.floor(ytile)


def getname(output, maptype):
    url = URL.get(maptype, maptype)
    return maptype if url != maptype else output


def config():
    cf = configparser.ConfigParser()
    cf.read("config.conf", encoding="utf-8-sig")
    download = cf.get("config", "下载方式")
    left = cf.get("config", "左上横轴")
    top = cf.get("config", "左上纵轴")
    right = cf.get("config", "右下横轴")
    bottom = cf.get("config", "右下纵轴")
    zoom = cf.get("config", "级别")
    name = cf.get("config", "项目名")
    maptype = cf.get("config", "地图地址")

    if download == "瓦片编码":
        process_tilenum(left, right, top, bottom, zoom, name, maptype)
    elif download == "地理编码":
        process_latlng(top, left, bottom, right, zoom, name, maptype)


def test():
    process_tilenum(803, 857, 984, 1061, 8, 'WORKNET')


def cml():
    if not len(sys.argv) in [7, 8]:
        print(
            'input 7 parameter northeast latitude,northeast longitude, southeast latitude, southeast longitude,zoom , output file, map type like gaode')
        return
    process_latlng(float(sys.argv[1]), float(sys.argv[2]), float(sys.argv[3]), float(sys.argv[4]), int(sys.argv[5]),
                   str(sys.argv[6]), str(sys.argv[7]))


if __name__ == '__main__':
    config()
    # test()
    # cml()
```

# 参考资料

```
https://blog.csdn.net/JairusChan/article/details/7497183
https://blog.csdn.net/qq_16064871/article/details/78869326?utm_source=gold_browser_extension
http://cxlwill.cn/frontend/GaodeMap/
https://blog.csdn.net/fredricen/article/details/77189453
https://www.jianshu.com/p/a3b2e01f602f
```