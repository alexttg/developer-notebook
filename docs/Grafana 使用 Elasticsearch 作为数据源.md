#Grafana 数据可视化监控

> Grafana是用于可视化大型测量数据的开源程序，可以理解为一个可视化工具，只有配置了对应的数据源，通过简单的配置便可以达到精致的图标展示，Grafana专注于时序类图表分析，而且支持多种数据源，如Graphite、InfluxDB、Elasticsearch、Mysql、K8s、Zabbix等，可以配合Elastic Search 做日志埋点，进行用户行为分析。

###下载安装 Grafana
官网下载

https://grafana.com/grafana/download

yum 命令安装

	yum localinstall grafana-7.3.5-1.x86_64.rpm 
	
启动

	ystemctl start grafana-server.service 
	
查看端口（默认3000）

	netstat -lpnt|grep grafana
查看日志

	tail -f /var/log/grafana/grafana.log -n 100
	

### Grafana 使用 Elasticsearch 作为数据源


**选择新增数据源**
![](https://foruda.gitee.com/images/1685602357887576240/da671fdf_5094274.png)

**选择数据源为Elastic Search**
![](https://foruda.gitee.com/images/1685611550749710945/51b6d4d3_5094274.png)

** 配置数据源相关连接信息**
![](https://foruda.gitee.com/images/1685611577569144285/0f32b019_5094274.png)

创建一个新的图表
![](https://foruda.gitee.com/images/1685611615952920720/7fb06f05_5094274.png)
###配置图表参数

1. 选择数据源
2. 查询内容
3. 分组方式，根据Elastic Search 索引中的字段分组
4. 选择时间范围
5. 选择图标样式
![](https://foruda.gitee.com/images/1685611649314768200/5f844d1c_5094274.png)

相关调试，在Query inspector 中可以拿到对应Elastic Search语句放到Kibana 中去执行，检测自己是否配置错误，而导致图表无法呈现
![](https://foruda.gitee.com/images/1685611702777358837/ea636180_5094274.png)
