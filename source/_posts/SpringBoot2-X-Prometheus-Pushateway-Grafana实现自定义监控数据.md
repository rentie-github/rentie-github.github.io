title: SpringBoot2.X+Prometheus+Pushateway+Grafana实现自定义监控数据
author: Mario
date: 2020-12-10 16:18:24
tags:
---
**Prometheus** 是由SoundCloud开发的开源监控报警系统和时序列数据库(TSDB)。<br>
**Push Gateway** 支持临时性Job主动推送指标的中间网关。<br>
**Grafana** 是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示。<br>
监控流程是SpringBoot项目将指标往pushgateway推送，prometheus从pushgateway拉取指标，最终在Grafana中展示监控图表

<!--more-->

#### Docker安装Prometheus
###### 下载镜像
``` bash
docker pull prom/prometheus
```
###### 添加挂载目录并授权
``` bash
mkdir /opt/prometheus/data
chmod 777 -R /opt/prometheus
```
###### 维护配置信息
``` bash
mkdir /opt/prometheus    
cd /opt/prometheus/
vim prometheus.yml
```
###### 内容如下
``` bash
global:
  # 抓取目标实例的频率时间值，默认10s
  scrape_interval:     60s
  # 执行配置文件规则的频率时间值, 默认1m
  evaluation_interval: 60s
# 抓取配置的列表
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: prometheus
  # 抓取配置的列表
  # pushgateway路径配置
  - job_name: pushgateway
    static_configs:
      - targets: ['172.24.0.6:9091']
        labels:
          instance: pushgateway
```
###### 启动Docker
``` bash
docker run  -d -p 9090:9090 -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /opt/prometheus/data:/prometheus  prom/prometheusl
```
###### 检查是否启动
访问地址： http://ip:9090/graph
![upload successful](/images/pasted-25.png)

#### Docker安装Pushgateway
###### 下载镜像，启动Docker
``` bash
docker pull prom/pushgateway
docker run -d -p 9091:9091 prom/pushgateway
```
###### 检查是否启动
访问地址： http://ip:9091
![upload successful](/images/pasted-26.png)

#### Docker安装Grafana
###### 下载镜像，启动Docker
``` bash
docker pull grafana/grafana
mkdir /data/grafana
chmod 777 -R /data/grafana
docker run -d  -p 3000:3000 -v /data/grafana:/var/lib/grafana  grafana/grafana
```
###### 检查是否启动
访问地址： http://ip:3000
![upload successful](/images/pasted-28.png)

#### 编写Prometheus exporter
###### 4种常用Metrics
**Counter**
``` bash
连续增加不会减少的计数器，可以用于记录只增不减的类型，例如：网站访问人数，系统运行时间等。
对于Counter类型的指标，只包含一个inc()的方法，就是用于计数器+1.
一般而言，Counter类型的metric指标在冥冥中我们使用_total结束，如http_requests_total.
```
**Gauge**
``` bash
可增可减的仪表盘，曲线图
对于这类可增可减的指标，用于反应应用的当前状态。
例如在监控主机时，主机当前空闲的内存大小，可用内存大小等等。
对于Gauge指标的对象则包含两个主要的方法inc()和dec()，用于增加和减少计数
```
**Histogram** <br>
主要用来统计数据的分布情况，这是一种特殊的metrics数据类型，代表的是一种近似的百分比估算数值，统计所有离散的指标数据在各个取值区段内的次数。例如：我们想统计一段时间内http请求响应小于0.005秒、小于0.01秒、小于0.025秒的数据分布情况。那么使用Histogram采集每一次http请求的时间，同时设置bucket。
``` bash
Histogram会自动创建3个指标，分别为：
一、事件发生总次数： basename_count:
#实际含义： 当前一共发生了2次http请求
io_namespace_http_requests_latency_seconds_histogram_count{path="/",method="GET",code="200",} 2.0
二、所有事件产生值的大小的总和： basename_sum
#实际含义： 发生的2次http请求总的响应时间为13.107670803000001 秒
io_namespace_http_requests_latency_seconds_histogram_sum{path="/",method="GET",code="200",} 13.107670803000001
三、事件产生的值分布在bucket中的次数： basename_bucket{le="上包含"}
# 在总共2次请求当中。http请求响应时间 <=0.005 秒 的请求次数为0
io_namespace_http_requests_latency_seconds_histogram_bucket{path="/",method="GET",code="200",le="0.005",} 0.0
在总共2次请求当中。http请求响应时间 <=0.01 秒 的请求次数为0
io_namespace_http_requests_latency_seconds_histogram_bucket{path="/",method="GET",code="200",le="0.01",} 0.0
在总共2次请求当中。http请求响应时间 <=0.025 秒 的请求次数为0
io_namespace_http_requests_latency_seconds_histogram_bucket{path="/",method="GET",code="200",le="0.025",} 0.0
```
**Summary**
``` bash
Summary和Histogram非常相似，都可以统计事件发生的次数或者大小，以及其分布情况，他们都提供了对时间的计数_count以及值的汇总_sum，也都提供了可以计算统计样本分布情况的功能。
不同之处在于Histogram可以通过histogram_quantile函数在服务器计算分位数。而Sumamry的分位数则是直接在客户端进行定义的。
因此对于分位数的计算，Summary在通过PromQL进行查询的时候有更好的性能表现，而Histogram则会消耗更多的资源，但是相对于客户端而言Histogram消耗的资源就更少。
用哪个都行，根据实际场景自由调整即可。
```
##### 基于SpringBoot2.X写一个简单的exporter_demo
###### 项目结构如下

![upload successful](/images/pasted-27.png)
###### pom.xml配置如下
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.prometheus.exporter</groupId>
    <artifactId>exporter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>exporter</name>
    <description>Prometheus Exporter For Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--普罗米修斯依赖-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
        <!--普罗米修斯  client jar 依赖，java将指标往pushgateway推送，prometheus从pushgateway拉取指标-->
        <dependency>
            <groupId>io.prometheus</groupId>
            <artifactId>simpleclient_pushgateway</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
###### 项目配置如下
``` bash
server.port=8888
spring.application.name=PrometheusExporterForSpringBoot

# 普罗米修斯配置 pushGateWay地址
pushGateWay.ip = 172.24.0.6:9091
```
###### Pushgateway配置
``` bash
package com.prometheus.exporter.pushgateway;

import io.prometheus.client.exporter.PushGateway;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author Mario
 * @Date 2020/12/10
 * @Desc push网关配置
 */
@Configuration
public class PushGatewayConfig {

    @Value("${pushGateWay.ip}")
    String pushGateWayIp;

    @Bean
    public PushGateway getPushGateway() {
        return new PushGateway(pushGateWayIp);
    }
}

```
###### 监控指标注册
这里只写了Counter和Gauge举一个简单的例子
``` bash
package com.prometheus.exporter.metrics;

import io.prometheus.client.Counter;
import io.prometheus.client.Gauge;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author Mario
 * @Date 2020/12/10
 * @Desc Metrics 指标实例注册
 */

@Configuration
public class MetricsConfig {

    /**
     *连续增加不会减少的计数器，可以用于记录只增不减的类型，例如：网站访问人数，系统运行时间等。
     *对于Counter类型的指标，只包含一个inc()的方法，就是用于计数器+1.
     *一般而言，Counter类型的metric指标在冥冥中我们使用_total结束，如http_requests_total.
     * @return Counter实例
     */
    @Bean
    public Counter getCounter() {
        /*
         *  使用Counter.build()创建Counter类型的监控指标，并且通过name()方法定义监控指标的名称network_traffic_input
         * ，通过labelNames()定义该指标包含的标签。最后通过register()将该指标注册到Collector的defaultRegistry中
         */
        return Counter.build()
                .name("metrics_counter")
                .labelNames("metrics_id")
                .help("Counter类型的指标测试")
                .register();
    }

    /**
     *可增可减的仪表盘，曲线图
     *对于这类可增可减的指标，用于反应应用的当前状态。
     *例如在监控主机时，主机当前空闲的内存大小，可用内存大小等等。
     *对于Gauge指标的对象则包含两个主要的方法inc()和dec()，用于增加和减少计数。
     * @return Gauge实例
     */
    @Bean
    public Gauge getGauge() {
        /**
         * 指标注册
         * name设置指标名
         * labelNames设置各项指标名称
         * help设置指标描述
         */
        return Gauge.build()
                .name("metrics_gauge")
                .help("Gauge类型的指标测试")
                .register();
    }
}

```
###### 推送监控指标到Pushgateway
这里使用的是http请求的方式触发，其他触发方式可以自己定义
``` bash
package com.prometheus.exporter.controller;

import io.prometheus.client.Counter;
import io.prometheus.client.Gauge;
import io.prometheus.client.exporter.PushGateway;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.util.Random;

/**
 * @Author Mario
 * @Date 2020/12/10
 * @Desc 往pushgateway推送指标测试
 */
@RestController
@RequestMapping("/push")
public class PushMetricsController {

    @Autowired
    private PushGateway pushGateway;

    @Autowired
    private Counter counter;

    @Autowired
    private Gauge gauge;



    /**
     * 模拟测试推送counter类型指标数据
     */
    @GetMapping("/counter")
    public String counter() throws IOException {
        Random rnd = new Random();
        //随机生成1个随机数
        int metricsId = rnd.nextInt(100000);
        counter.labels(metricsId + "").inc();
        pushGateway.push(counter,"job-counter-test");
        return "finish!";
    }

    /**
     * 模拟测试推送gauge类型指标数据
     */
    @GetMapping("/gauge")
    public String gauge() throws IOException, InterruptedException {
        Random rnd = new Random();
        for (int i=0 ;i<20;i++){
            //随机生成1个随机数
            int metricsId = rnd.nextInt(100000);
            if (metricsId % 5 == 0) {
                gauge.inc();
            } else {
                gauge.dec();
            }
            pushGateway.push(gauge,"job-gauge-test");
            System.out.println("metricsId:" + metricsId);
            Thread.sleep(5000);
        }

        return "finish!";
    }

}
```
##### Grafana展示指标监控数据
前面prometheus的启动配置中已经写入了拉取pushgateway的配置，所以现在只需要配置Grafana就能将监控数据进行展示了
###### 基本配置
granafa默认端口为3000，可以在浏览器中输入http://ip:3000/<br>
granafa首次登录账户名和密码admin/admin，可以修改<br>
###### 配置数据源
配置数据源Data sources->Add data source -> Prometheus，输入prometheus数据源的信息，主要是输入name和url

![upload successful](/images/pasted-29.png)

![upload successful](/images/pasted-30.png)

###### 添加Dashboard

![upload successful](/images/pasted-31.png)
进入编辑选择数据源和指标，然后保存就能展示监控数据了
![upload successful](/images/pasted-33.png)

![upload successful](/images/pasted-32.png)

![upload successful](/images/pasted-34.png)