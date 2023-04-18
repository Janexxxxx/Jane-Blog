![shouqianba-osp-portal项目启动报错](/Users/xujing/Library/Application Support/typora-user-images/image-20211104182924194.png)

问题：安装完依赖，`npm run dev`启动后报错

原因：项目老，目前所使用的node版本过高

解决：使用低版本的node

扩展：[安装多个node.js且多个node.js版本之间切换](https://www.jianshu.com/p/7204af51fa01)

1. `sudo npm install -g n`
2. `sudo n latest`  安装最新版node（最新非稳定版，奇数版本）
3. `sudo n lts`   安装最新稳定版（一般偶数为稳定版）
4. `sudo n 8.17.0`  安装指定版本
5. `sudo n`  查看所有的node版本 ,之后再切换已经安装的版本node版本，回车就好了。
5. node -v 查看当前node版本