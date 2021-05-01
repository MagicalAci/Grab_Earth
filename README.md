[![壁纸图片](http://47.117.3.188/wp-content/uploads/2021/05/HLLRU_3Q@E4WRRR7_M.png "壁纸图片")](http://47.117.3.188/wp-content/uploads/2021/05/HLLRU_3Q@E4WRRR7_M.png "壁纸图片")

## 应该导入的模块
```python
import os
import datetime
import requests
import win32gui, win32con, win32api
```
## 检查文件
```
def checkDir(dirname):
	if not os.path.exists(dirname):
		os.mkdir(dirname)
```
## 爬取壁纸
### 网址解析
*http://himawari8-dl.nict.go.jp/himawari8/img/D531106/1d/550/*

其中 
- **himawari8** 是 卫星名字"向日葵8号"
- **img** 是图片
- **500** 是像素

这并不是卫星的网址，这里是把后面的时间去掉，而拿到的前面包括涵盖：卫星名、图片、像素等信息，这样就不用再更改时间了

```
 date hour minute second 
```
直接在之后附上时间就好
```
 ext
```
设置图片格式

```
picture_url = url_base + date + hour + minute + second + ext
```
赋予抓取到的地球图片新的地址（URL）
接着保存到的文件夹里，然后输出结果

```
def crawlWallpaper(cache_dir='download'):
	checkDir(cache_dir)
	url_base = 'http://himawari8-dl.nict.go.jp/himawari8/img/D531106/1d/550/'
	date = datetime.datetime.utcnow().strftime('%Y/%m/%d/')
	hour = str(int(datetime.datetime.utcnow().strftime('%H')) - 1).zfill(2)
	minute = str(datetime.datetime.utcnow().strftime('%M'))[0] + '0'
	second = '00'
	ext = '_0_0.png'
	picture_url = url_base + date + hour + minute + second + ext
	res = requests.get(picture_url)
	with open(os.path.join(cache_dir, 'cache_wallpaper.png'), 'wb') as f:
		f.write(res.content)
```
## 更换壁纸

```
def setWallPaper(imagepath='download/cache_wallpaper.png'):
	keyex = win32api.RegOpenKeyEx(win32con.HKEY_CURRENT_USER, "Control Panel\\Desktop", 0, win32con.KEY_SET_VALUE)
	win32api.RegSetValueEx(keyex, "WallpaperStyle", 0, win32con.REG_SZ, "0")
	win32api.RegSetValueEx(keyex, "TileWallpaper", 0, win32con.REG_SZ, "0")
	win32gui.SystemParametersInfo(win32con.SPI_SETDESKWALLPAPER, imagepath, win32con.SPIF_SENDWININICHANGE)
```

### 主函数和运行
```
def main():
	crawlWallpaper('download')
	setWallPaper(os.path.join(os.getcwd(), 'download/cache_wallpaper.png'))

if __name__ == '__main__':
	main()
```

## 配置定时更换
### 打包
将.py文件打包为.exe

`win+R`打开控制面板输入`cmd`输入` pip install pyinstaller`安装模块
**pyinstaller -version查看是否安装成功**

打包方法：
- 进入.py文件目录打开终端
- 输入 ` pyinstaller -F 文件名.py`
- 新生成的**dist**里便有.exe

## 配置定时更换

* 控制面板-> 计划任务 -> 操作 -> 创建任务
- 常规
修改名称和索引位置

- 触发器
新建触发器->高级设置
->重复任务间隔改为10min（因为卫星图片更新为10min一次）
->持续时间：无限期

- 操作
设置里添加.exe路径

