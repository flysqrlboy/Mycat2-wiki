# 使用codestyle统一java编码规范

## 1. Eclipse中安装code style配置

- 第零步：下载 eclipse-java-google-style.xml 

- 第一步：往Eclipse导入下载的xml文件

Window->Preferences->Java->Code Style->Formatter

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/eclipse-codestyle-config-1.png)


- 第二步：点击Edit，修改配置

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/eclipse-codestyle-config-2.png)

Tab Policy选择 Spaces Only，Indentation size 为 4 ，即换行时自动向右缩进4个空格。点击OK确认修改。

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/eclipse-codestyle-config-3.png)

- 第三步： 配置每次修改时自动格式化

Window->Preferences->Java->Editor-Save Actions,如下图选中Format edited lines，即是当你编辑保存时自动对修改行进行格式化。

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/eclipse-codestyle-config-4.png)



## 2. Intellij IDEA 中安装code style 配置

- 第零步：下载 intellij-java-google-style.xml 

- 第一步：往intellij导入下载的xml文件

依次点击 File -> Settings -> Editor -> Code Style -> Java， 然后点击 Manage... -> Import... ， 选择 "Intellij IDEA code style XML"，点击 OK。

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/intellij-codestyle-config-1.png)

然后选择 intellij-java-google-style.xml 的保存位置，再确认。

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/intellij-codestyle-config-2.png)

点击OK。成功导入GoogleStyle。

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/intellij-codestyle-config-3.png)

- 第二步：在Scheme中选择 GoogleStyle。设置Tab size为4，Indent为4。

![](https://raw.githubusercontent.com/MyCATApache/Mycat2/master/doc/images/codestyle/intellij-codestyle-config-4.png)

- 第三步：格式化

intellij没有保存文件的时候自动应用code style格式化代码。需要点击 Code -> Reformat code，或者使用快捷键Ctrl+Alt+L