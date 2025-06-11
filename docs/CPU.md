###服务器CPU满载排查

线上运行的项目忽然有大批客户反馈系统卡顿，甚至有些项目无法打开，于是针对该问题逐步排查


首先使用 ping 查看网络状态

	ping  目标站点
发现网络返回数据并无异常

通过阿里云监控查看服务器状态，当然也可以使用 `top` 命令或者 `htop` 命令来检查 CPU 使用率
![](https://foruda.gitee.com/images/1685600886850566913/667522a8_5094274.png)

发现 CPU 使用率过高，可能会导致服务器性能严重下降。

###进入终端服务器使用Arthas
> （目前最快捷的方式）

安装


```
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar
```
查看堆栈信息

	dashboard
	
输出

```
ID    NAME                               GROUP            PRIORITY    STATE      %CPU        DELTA_TIME TIME        INTERRUPTE DAEMON      
170   PPA-15654608                       main             5           RUNNABLE   4.28        0.213      3:9.959     false      true        
-1    C2 CompilerThread0                 -                -1          -          3.95        0.197      5:4.039     false      true        
22039 ForkJoinPool.commonPool-worker-13  main             5           WAITING    0.69        0.034      2:44.081    false      true        
-1    VM Thread                          -                -1          -          0.37        0.018      1:1.155     false      true        
25729 ForkJoinPool.commonPool-worker-3   main             5           RUNNABLE   0.31        0.015      1:49.257    false      true        
47433 process reaper                     system           10          TIMED_WAIT 0.31        0.015      0:0.051     false      true        
47433 process reaper                     system           10          RUNNABLE   0.29        0.014      0:0.027     false      true        
47434 process reaper                     system           10          RUNNABLE   0.28        0.014      0:0.035     false      true        
21680 ForkJoinPool.commonPool-worker-1   main             5           WAITING    0.28        0.014      2:42.162    false      true        
26964 ForkJoinPool.commonPool-worker-7   main             5           WAITING    0.23        0.011      1:23.245    false      true        
29964 ForkJoinPool.commonPool-worker-19  main             5           WAITING    0.23        0.011      1:19.606    false      true        
Memory                        used      total     max       usage     GC                                                                   
heap                          4878M     8192M     8192M     59.55%    gc.g1_young_generation.count       5175                              
g1_eden_space                 1792M     3204M     -1        55.93%    gc.g1_young_generation.time(ms)    216104                            
g1_old_gen                    3062M     4964M     8192M     37.38%    gc.g1_old_generation.count         0                                 
g1_survivor_space             24M       24M       -1        100.00%   gc.g1_old_generation.time(ms)      0                                 
nonheap                       286M      349M      -1        81.95%                                                                         
codeheap_'non-nmethods'       2M        2M        5M        43.45%                                                                         
metaspace                     162M      212M      -1        76.72%                                                                         
codeheap_'profiled_nmethods'  51M       56M       117M      43.98%                                                                         
compressed_class_space        18M       26M       1024M     1.83%                                                                          
Runtime                                                                                                                                    
os.name                                                               Linux                                                                
os.version                                                            3.10.0-957.5.1.el7.x86_64                                            
java.version                                                          11.0.3                                                               
java.home                                                             /usr/lib/jvm/java-11-openjdk-11.0.3.7-0.el7_6.x86_64                 
systemload.average                                                    13.59                                                                
processors                                                            8                                                                    
timestamp/uptime                                                      Thu Feb 23 16:39:27 CST 2023/22652s                                  
ID    NAME                               GROUP            PRIORITY    STATE      %CPU        DELTA_TIME TIME        INTERRUPTE DAEMON      
172   Catalina-utility-2                 main             1           RUNNABLE   21.91       1.096      24:25.122   false      false       
170   PPA-15654608                       main             5           RUNNABLE   11.63       0.582      3:10.541    false      true        
-1    C2 CompilerThread0                 -                -1          -          7.38        0.369      5:4.408     false      true        
166   PPA-15654551                       main             5           RUNNABLE   4.7         0.235      3:1.917     false      true        
171   PPA-15654869                       main             5           WAITING    3.5         0.175      2:28.741    false      true        
22039 ForkJoinPool.commonPool-worker-13  main             5           WAITING    0.47        0.023      2:44.104    false      true        
21680 ForkJoinPool.commonPool-worker-1   main             5           WAITING    0.46        0.023      2:42.186    false      true        
-1    VM Thread                          -                -1          -          0.43        0.021      1:1.176     false      true        
25729 ForkJoinPool.commonPool-worker-3   main             5           WAITING    0.4         0.020      1:49.277    false      true        
47450 process reaper                     system           10          RUNNABLE   0.4         0.019      0:0.041     false      true        
30243 ForkJoinPool.commonPool-worker-9   main             5           WAITING    0.39        0.019      1:6.710     false      true        
Memory                        used      total     max       usage     GC                                                                   
heap                          5970M     8192M     8192M     72.88%    gc.g1_young_generation.count       5175                              
g1_eden_space                 2840M     3204M     -1        88.64%    gc.g1_young_generation.time(ms)    216104                            
g1_old_gen                    3106M     4964M     8192M     37.92%    gc.g1_old_generation.count         0                                 
g1_survivor_space             24M       24M       -1        100.00%   gc.g1_old_generation.time(ms)      0                                 
nonheap                       286M      349M      -1        81.99%                                                                         
codeheap_'non-nmethods'       2M        2M        5M        43.45%                                                                         
metaspace                     162M      212M      -1        76.72%                                                                         
codeheap_'profiled_nmethods'  51M       56M       117M      44.05%                                                                         
compressed_class_space        18M       26M       1024M     1.83%                                                                          
Runtime                                                                                                                                    
os.name                                                               Linux                                                                
os.version                                                            3.10.0-957.5.1.el7.x86_64                                            
java.version                                                          11.0.3                                                               
java.home                                                             /usr/lib/jvm/java-11-openjdk-11.0.3.7-0.el7_6.x86_64                 
systemload.average                                                    15.22                                                                
processors                                                            8                                                                    
timestamp/uptime                                                      Thu Feb 23 16:39:31 CST 2023/22657s          	
```

使用arthas命令，查找最耗时前5个线程堆栈信息

	thread -n 5
输出

```
"PVRC-10835394" Id=105 cpuUsage=27.63% deltaTime=68ms time=2962472ms RUNNABLE
    at java.base@11.0.3/java.lang.Integer.formatUnsignedInt(Integer.java:385)
    at java.base@11.0.3/java.lang.Integer.toUnsignedString0(Integer.java:344)
    at java.base@11.0.3/java.lang.Integer.toHexString(Integer.java:262)
    at org.apache.http.impl.conn.Wire.wire(Wire.java:77)
    at org.apache.http.impl.conn.Wire.input(Wire.java:117)
    at org.apache.http.impl.conn.LoggingInputStream.read(LoggingInputStream.java:88)
    at org.apache.http.impl.io.SessionInputBufferImpl.streamRead(SessionInputBufferImpl.java:139)
    at org.apache.http.impl.io.SessionInputBufferImpl.read(SessionInputBufferImpl.java:200)
    at org.apache.http.impl.io.ContentLengthInputStream.read(ContentLengthInputStream.java:178)
    at org.apache.http.conn.EofSensorInputStream.read(EofSensorInputStream.java:135)
    at java.base@11.0.3/java.util.zip.CheckedInputStream.read(CheckedInputStream.java:83)
    at java.base@11.0.3/java.io.FilterInputStream.read(FilterInputStream.java:133)
    at com.aliyun.oss.event.ProgressInputStream.read(ProgressInputStream.java:116)
    at java.base@11.0.3/java.util.zip.CheckedInputStream.read(CheckedInputStream.java:83)
    at java.base@11.0.3/java.io.FilterInputStream.read(FilterInputStream.java:107)
    at com.aliyun.oss.internal.OSSObjectOperation.getObject(OSSObjectOperation.java:308)
    at com.aliyun.oss.OSSClient.getObject(OSSClient.java:545)
    at com.soomey.util.OssUtils.persistFile(OssUtils.java:177)
    at com.soomey.util.OssUtils.get(OssUtils.java:109)
    at com.soomey.util.ResourceStorageUtils.retrieveIfNecessary(ResourceStorageUtils.java:107)
    at com.soomey.entity.core.ProductPicture.path(ProductPicture.java:389)
    at cn.deepdraw.masaimara.domain.productrenderer.product.Utilities.setPureColorBackgroundFeature(Utilities.java:77)
    at com.soomey.masaimara.bean.product.DefaultPictureFilter.lambda$setPureColorBackgroundFeature$1(DefaultPictureFilter.java:25)
    at com.soomey.masaimara.bean.product.DefaultPictureFilter$$Lambda$1518/0x0000000840b46840.accept(Unknown Source)
    at java.base@11.0.3/java.util.stream.ForEachOps$ForEachOp$OfRef.accept(ForEachOps.java:183)
    at java.base@11.0.3/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:177)
    at java.base@11.0.3/java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1654)
    at java.base@11.0.3/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:484)
    at java.base@11.0.3/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:474)
    at java.base@11.0.3/java.util.stream.ForEachOps$ForEachOp.evaluateSequential(ForEachOps.java:150)
    at java.base@11.0.3/java.util.stream.ForEachOps$ForEachOp$OfRef.evaluateSequential(ForEachOps.java:173)
    at java.base@11.0.3/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
    at java.base@11.0.3/java.util.stream.ReferencePipeline.forEach(ReferencePipeline.java:497)
    at com.soomey.masaimara.bean.product.DefaultPictureFilter.setPureColorBackgroundFeature(DefaultPictureFilter.java:25)
    at com.soomey.masaimara.bean.product.DefaultPictureFilter.filter(DefaultPictureFilter.java:19)
    at com.soomey.masaimara.bean.template.AbstractRenderer.preHandleProduct(AbstractRenderer.java:75)
    at com.soomey.masaimara.bean.template.AbstractRenderer.render(AbstractRenderer.java:61)
    at cn.deepdraw.masaimara.application.detailpage.DetailPageRenderingTask.renderAsHtml(DetailPageRenderingTask.java:105)
    at cn.deepdraw.masaimara.application.detailpage.DetailPageRenderingTask.render(DetailPageRenderingTask.java:77)
    at cn.deepdraw.masaimara.application.detailpage.DetailPageRenderingTask.call(DetailPageRenderingTask.java:63)
    at cn.deepdraw.masaimara.application.detailpage.ProductVisionResourceCreator.call(ProductVisionResourceCreator.java:13)
    at cn.deepdraw.masaimara.application.detailpage.DetailPageRenderingTask.call(DetailPageRenderingTask.java:30)
    at com.soomey.bean.task.CallableTask.run(CallableTask.java:43)
    at com.soomey.bean.task.WeighableTask.run(WeighableTask.java:49)
    at com.soomey.bean.task.WeighableTaskQueue$PoolWorker.run(WeighableTaskQueue.java:131)


"C2 CompilerThread0" [Internal] cpuUsage=25.92% deltaTime=64ms time=300015ms
```

找出对应方法进行优化


![](https://foruda.gitee.com/images/1685600942796443938/2975ba19_5094274.png)


根据以上栈堆信息找到发现导致接口卡顿主要为/masaimara/ppa改接口导致，该接口主要是接收masaimara服务图片分析数据进行db落库，没有采用分批批量插入的方式插入，导致占用过的池没有释放