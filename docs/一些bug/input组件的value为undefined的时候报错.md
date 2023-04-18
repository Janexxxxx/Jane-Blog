报错：

![image-20220608174832824](/Users/xujing/Library/Application Support/typora-user-images/image-20220608174832824.png)

[报错分析文章1](https://www.freesion.com/article/7725914124/)

[报错分析文章2](http://www.qiutianaimeili.com/html/page/2020/04/2020428uhq1diduf8c.html)

**组件从非受控状态到受控状态的切换不被推荐**

1. 加了onChange，input变成受控组件
2. react规定：当input的值变成undefined，input变成非受控组件
3. 当input有值时，又变成受控组件
4. **组件从非受控状态到受控状态的切换不被推荐**，react不喜欢这种做法，会报以上的错误