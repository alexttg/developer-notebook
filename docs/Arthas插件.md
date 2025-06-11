##Arthas idea 插件
> 由于Arthas 命令过于繁琐，使用起来稍微有些麻烦，有些时候只需要简单执行某些命令，那么可以借助idea 插件arthas idea 来快速生成需要最便捷的命令  

1.下载 Arthas  plugin 
![](https://foruda.gitee.com/images/1685602015956484167/81d2fa94_5094274.png)
2、选择需要监控的方法，
![](https://foruda.gitee.com/images/1685602073873956652/af6a5d86_5094274.png)
3.选择对该方法需要执行的命令
![](https://foruda.gitee.com/images/1685602117055349039/72be2f08_5094274.png)
4.复制命令稍做按自己需求稍做修改
![](https://foruda.gitee.com/images/1685602152819875551/92ea4c5a_5094274.png)
5.搬运到Arthas控制台
![](https://foruda.gitee.com/images/1685602181825952153/c3489cc7_5094274.png)

**线上时间情况使用**

> 线上请求过多，为了减轻排除范围，那么需要根据请求的入参、出参来缩小监控访问，从而来实现问题的定位
> 系统卡顿，在特定的时间与特定的用户，
> 比如某个用户反应系统卡顿，那么使用Arthas查看方法具体耗时
> 查询pictures()方法参数id为1的方法执行耗时

![](https://foruda.gitee.com/images/1685602216413575986/600304f0_5094274.png)

	trace com.soomey.controller.authorized.merchant.ProductController pictures 'params[2]==1'
![](https://foruda.gitee.com/images/1685602244676701983/5f61c0d0_5094274.png)