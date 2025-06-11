#Arthas使用
>Arthas 是一款线上监控诊断产品，通过全局视角实时查看应用 load、内存、gc、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断，包括查看方法调用的出入参、异常，监测方法执行耗时，类加载信息等，大大提升线上问题排查效率。

[Arthas官方文档](https://arthas.aliyun.com/)
##Arthas下载与使用
	curl -O https://arthas.aliyun.com/arthas-boot.jar
	
启动Arthas

	java -jar arthas-boot.jar
	
**输入id序号选择想要监控的进程**
	
![](https://foruda.gitee.com/images/1685601690371919061/b83b719c_5094274.png)

本机安装的是openJDK没有jsp、jstack、jstat等相关命令，安装后便可正常运行

![](https://foruda.gitee.com/images/1685601720476567056/7838b9fc_5094274.png)

安装

	yum install java-1.8.0-openjdk-devel -y
###Arthas线上排查问题常用的几个命令	
	
* memory	查看JVM 内存信息
* sysprop	查看当前JVM的系统属性
* thread	获取当前线程执行情况
* trace	方法内部调用路径，并输出方法路径上的每个节点上耗时
* watch	函数执行数据观测
* tt	官方解释为时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

####1.memory 查看JVM 内存信息

	memory
![](https://foruda.gitee.com/images/1685601762515134734/b38e98fb_5094274.png)

####2.thread 获取当前线程执行情况

	thread
![](https://foruda.gitee.com/images/1685601794018102932/39f7c804_5094274.png)

####3.trace 查看方法耗时

参数

● -n: 命令捕捉次数，达到设置次数后自动退出该命令

● --skipJDKMethod：是否jdk里的函数调用

例：查看com.example.demo.service.AdminService类中getUsers各方法执行耗时
	
	trace com.example.demo.service.AdminService getUsers  -n 5 --skipJDKMethod false

![](https://foruda.gitee.com/images/1685601836189488130/8d3fcfc0_5094274.png)

####4 watch 方法出入参

● -n: 命令捕捉次数，达到设置次数后自动退出该命令

● express：默认值{params,returnObj,throwExp}，分别代表，入参、出参、异常

● -x：表示输出结果的属性遍历深度，默认为 1，最大值是4

● -b：在方法调用前执行监控

● -f：在方法调用后执行监控（默认）

● -e：在方法调用前执行监控

● -s： 在方法返回后执行

例：
	
	watch com.example.demo.service.AdminService getUsers '{params,returnObj,throwExp}'  -n 5  -x 3 
可添加cost参数进行方法耗时过滤

	watch com.example.demo.service.AdminService getUsers '{params,returnObj,throwExp}' '#cost>200' -x 3

```
watch com.example.demo.service.AdminService getUsers '{params,returnObj,throwExp}'  -n 5  -x 3 
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 70 ms, listenerId: 4
method=com.example.demo.service.AdminService.getUsers location=AtExit
ts=2022-07-07 23:01:34; [cost=77.637229ms] result=@ArrayList[
@Object[][
@Integer[1],
@Integer[10],
@String[],
null,
],
@PageResponseVo[
data=@ArrayList[
@User[User(id=159, username=刘11, sex=1, payType=6, jobType=0, idCard=2222, hiredate=Wed Jun 22 15:59:09 CST 2022, mobile=4444, departmentId=1, currentStatus=0, bankCode=, createTime=Wed Jun 22 15:59:09 CST 2022, updateTime=Wed Jun 22 15:59:09 CST 2022, dateStatus=0)],
@User[User(id=158, username=梁桂2, sex=1, payType=6, jobType=0, idCard=333, hiredate=Wed Jun 22 15:56:45 CST 2022, mobile=422, departmentId=1, currentStatus=0, bankCode=, createTime=Wed Jun 22 15:56:45 CST 2022, updateTime=Wed Jun 22 15:56:45 CST 2022, dateStatus=0)],
],
code=@Integer[1],
msg=@String[操作成功],
total=@Long[113],
],
null,
]
```
####5 tt 记录下指定方法每次调用的入参和返回信息

● -n: 命令捕捉次数，达到设置次数后自动退出该命令

● -t :这个参数的表明希望记录下类 *Test 的 print 方法的每次执行情况。

● -w: --watch-express 观察时空隧道使用ognl 表达式

● -p:指定调用次数

例：

	tt -t com.soomey.controller.authorized.merchant.setting.SettingController loadFormConfig -n 5
![](https://foruda.gitee.com/images/1685601888482740299/bd02c335_5094274.png)
再次输入观察的序号 `tt -p -i 1000 ` 则可以可以查看序号为1000记录的具体调用信息，出入参与方法耗时等等
![](https://foruda.gitee.com/images/1685601914044976010/69ebf034_5094274.png)
还可以使用 `-w`与时空隧道使用ognl 表达式执行其他方方法

例： 执行调用序号为1000的controller中的getDerivativePictureSettingData()方法

	tt -i 1000 -w 'target.getDerivativePictureSettingData()'
	
####6 sc+ognl表达式执行静态方法

`sc `查看 JVM 已加载的类信息

	sc -d com.soomey.util.MogujieUtils
![](https://foruda.gitee.com/images/1685601948722436518/5add6f8f_5094274.png)

获取到类加载器hashcode值，根据ognl表达式即可执行当前类中某个方法

	ognl -c 54ba4f78 '@com.soomey.util.MogujieUtils@getDecryptId("85C47D13A725FF56178934CE4038D38E","114q")'
或者根据ognl表达式查看某个类中成员变量

	ognl -c 54ba4f78 '@com.soomey.util.ResourceStorageUtils@ossBucketNames.toString()'


