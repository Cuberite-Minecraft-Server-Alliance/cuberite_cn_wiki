## 跨版本篇  
Cuberite原版是支持跨版本的，但版本很低且不完全  
在本篇中我们将通过使用ViaProxy实现跨版本  
## ViaProxy
**介绍**
ViaProxy是一种专用于跨版本的反代服务端。  
独立代理，允许玩家加入所有版本的 Minecraft 服务器（经典、Alpha、Beta、Release、Bedrock）  
要下载最新版本，请前往[发布版块](https://github.com/ViaVersion/ViaProxy#executable-jar-file)下载最新版本。  
使用方法非常简单，只需运行 jar 文件，它就会启动一个用户界面，在此可对一切进行配置。  
如需完整的用户指南，请访问 "[玩家使用方法](https://github.com/ViaVersion/ViaProxy#usage-for-players-gui) "部分或 "[服务器所有者使用方法](https://github.com/ViaVersion/ViaProxy#usage-for-server-owners-cli) "部分。  
**支持的服务器版本**  
发行版（1.0.0 - 1.20.5）  
测试版（b1.0 - b1.8.1）  
阿尔法（a1.0.15 - a1.2.6）  
经典 (c0.0.15 - c0.30 包括 [CPE](https://wiki.vg/Classic_Protocol_Extension))  
愚人节（3D Shareware，20w14infinite）  
战斗快照 (Combat Test 8c)  
基岩版 1.20.70（部分功能缺失）  
**支持的客户端版本**
发行版（1.7.2 - 1.20.5）
基岩版（需要 Geyser 插件）
经典版、阿尔法版、测试版、1.0 - 1.6.4 版（仅限直通版）
ViaProxy 支持从任何列出的客户端版本连接到任何列出的服务器版本。
**下载**
开始前请确保你已安装Java17  
首先，[下载ViaProxy](https://github.com/ViaVersion/ViaProxy/releases/download/v3.2.0/ViaProxy-3.2.0.jar)  
下载完后，就可以直接启动了
**启动**
以下是允许玩家使用您的服务器地址 25568 加入并连接到运行在端口 25565 上的 1.7.3 测试版服务器的命令示例： 
``` bash
java -jar ViaProxy-whateverversion.jar --bind_address 0.0.0.0:25568 --target_ip 127.0.0.1:25565 --version b1.7-b1.7.3
```
根据自己的需求修改命令并启动即可
