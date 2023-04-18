![image-20220706164708393](/Users/xujing/Library/Application Support/typora-user-images/image-20220706164708393.png)

![image-20220706165350020](/Users/xujing/Library/Application Support/typora-user-images/image-20220706165350020.png)

本地打包后报此错误，npm install安装完之后还是报错。

遇到问题不要慌，冷静分析Ωß≈

分析：报错提示这些依赖未安装，提供几个排查思路

1、查找node_modules下是否有core-js文件，如果没有npm i core-js安装

2、查找package.json 的dependencies是否有core-js依赖，如果没有npm i core-js安装

3、如果都有，查看是否是core-js的版本问题，所以找不到core-js下的文件了。看下版本和之前是否一致

4、实在不行，可以看下./src/.umi/core/polyfill.ts文件里面有啥内容，是否可以注释掉。

5、其他排错思路：看下tag CI成功了吗，你本地安装依赖有报错吗

6、查看是否是node版本问题

