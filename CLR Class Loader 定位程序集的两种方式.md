###### 按文件路径

给出程序集路径，CLR根据路径定位找到程序集并加载。

###### 按命名空间

1. 查找GAC（全局应用程序缓存）；
2. 查找配置文件中codebase元素指定的URL查找；
3. 搜索应用程序目录及配置文件中probing指定的子目录，假设你的应用程序目录是C:\AppDir,<probing>元素中的privatePath指定了一个路径Path1，那么查找顺序如下：
  - C:\AppDir\AssemblyName.dll
  - C:\AppDir\AssemblyName\AssemblyName.dll
  - C:\AppDir\Path1\AssemblyName.dll
  - C:\AppDir\Path1\AssemblyName\AssemblyName.dll
