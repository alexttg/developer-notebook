## 使用 JProfiler 进行线上问题分析

记录JProfiler 的实时连接（Live Profiling）和离线内存快照分析两大流程，帮助后续开发高效定位性能瓶颈与内存泄漏。
JProfiler的监控模式主要有以下两种：


   - **Sampling（采样）**：性能开销小，可快速查看 CPU/内存热点，但不记录方法调用次数等详细信息。适合日常性能巡检。
   - **Instrumentation（埋点）**：记录完整执行信息，包括方法调用次数，但会显著增加运行开销。仅在需深入剖析时启用，切换模式需重启 JProfiler。
   
 常规使用选择sampling模式即可,我选择的也是Sampling

---

### 一、环境准备

1. **JProfiler 版本**
   本地客户端版本应与服务器端一致，避免协议或功能不兼容。

2. **激活**

	用户名：``` inkerbot```

	激活码: ``` inkerbot#568545139-27vxgmny6224a#5bbb5```


---

### 二、实时连接运行的 Java 应用

1. **查找目标进程**

   ```bash
   sudo ps -ef | grep "tomcat"
   ```

2. **启动 jpenable**

   ```bash
   cd ${JPROFILER_HOME}/bin
   sudo ./jpenable
   ```

3. **选择进程序号**

   在命令行界面中，输入对应进程前的序号，回车确认。

4. **选择监控模式** 

   默认 GUI 模式；可根据提示切换 Sampling 或 Instrumentation。

5. **输入通信端口** 

   确保端口（默认 8849）未被占用：

   ```bash
   sudo netstat -lnpt | grep 8849
   ```

6. **连接成功示例**
![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1695643106693-8bdea677-0b39-4ed8-8a71-a08cbebe69d0.png)

打开本地 JProfiler 客户端，选择 Attach to Remote JVM，输入服务器 IP 与端口，即可建立实时连接

连接成功后，在 JProfiler GUI 中即可实时查看 CPU、内存、线程、方法调用等各类性能数据。

找到耗时最长调用最多的redis通信连接
![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1701243004803-1686fc22-ccac-4614-a0da-e7a4a783be85.png)
将本次排查出来的问题优化后，服务器的相应时间得到明显的缩短
![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1701243276474-3ba7ece0-e45a-4c3c-a928-39715309ad0c.png)


---

### 三、导出 `.hprof` 内存快照

1. 在 JProfiler 界面顶部，选择 **Session → Save Snapshot...**。
2. 选择快照类型为 **Heap Dump (.hprof)**，指定文件路径并保存。

---

### 四、离线 `.hprof` 快照分析

以下步骤适用于已生成 `.hprof` 文件的场景。

1.  **打开快照**

   - 启动 JProfiler，选择 `Session → Open Snapshot`，加载内存快照文件。

2. **定位内存占用较大的对象**

  ![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1701242197057-6eaf9b13-c27a-496c-a115-0e0f61fa83a8.png)


3. **隔离感兴趣的对象**

   - 在目标对象行上 **右键 → List Objects → Use Selected Objects**，将其单独列出便于深入查阅。
![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1701242357805-d9475ad8-06ad-40d3-a786-79b4754c1758.png)


4. **查看对象的引用链**

   - 切换到 **Incoming References** 视图，展示从 GC Roots 到该实例的引用路径。
   - 双击引用链条可直接跳转到对应方法或类的代码位置。

![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1701242396442-cf827920-69ab-4b6b-9912-3fec8c91ee70.png)


5.**定位与优化触发代码**

![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1701242424895-842971fa-c149-4c50-9a4b-e107d2aa5327.png)

![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1701242460273-61ed4e2f-e91f-457d-a893-37059a0d9b8a.png)

排查发现 `selectByParms() `方法中频繁创建了大的对象
   - 根据引用链查找根源代码位置，一般都是：
     - 长生命周期集合或缓存未及时清理
     - 注册后未注销的监听器/回调
     - 循环中频繁创建的大对象



