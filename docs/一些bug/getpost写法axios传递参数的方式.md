参考链接：

[Axios 各种请求方式传递参数格式](https://www.jianshu.com/p/53deecb09077)

[axios 传递参数的方式(data 与 params 的区别)](https://blog.csdn.net/qq_41499782/article/details/118916901?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default)

[axios-npm](https://www.npmjs.com/package/axios#axios-api)

[axios各种请求方式传递参数格式](https://www.1024sou.com/article/102951.html)

首先，post方法采用如下方式定义，

![image-20220309180916817](/Users/xujing/Library/Application Support/typora-user-images/image-20220309180916817.png)

出现错误如下：

![image-20220309181016539](/Users/xujing/Library/Application Support/typora-user-images/image-20220309181016539.png)

请求参数拼接如下：是采用拼接在url后面的方式

![image-20220309181048601](/Users/xujing/Library/Application Support/typora-user-images/image-20220309181048601.png)

错误原因：get/post传递参数方式混乱

正确方式：

```js
//get,参数会携带在url后面
//方法一：
axios({
  method: 'get',
  url: '/user/12345',
  params: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  },
 
});
//方法二：
axios.get('demo/url', {
    params: {
        id: 123,
        name: 'Henry',
    },
   timeout: 1000,
  ...//其他相关配置
})
```

```js
//post：参数放在data里面，会传递给body
//方法一：
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  },
  timeout: 1000,
  ...//其他相关配置
});

//方法二：
axios.post('demo/url', {
    id: 123,
    name: 'Henry',
},{
   timeout: 1000,
    ...//其他相关配置
})
```

