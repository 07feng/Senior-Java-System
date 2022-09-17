| **指标** | **参数** |
| --- | --- |
| 机器 | 8C16G |
| 操作系统 | Linux |
| WEB服务器 | Tomcat |

首先我们根据Java Performance推荐公式配置最优GC参数
![performance.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663295471743-62651eb4-9427-4008-bd66-9192dcbe159e.png)
Xmx和Xms设置为老年代存活对象的3-4倍，即Full GC之后老年代内存占用的3-4倍
永久代（元空间）设置为老年代存活对象的1.2-1.5倍
Xmn设置为老年代存活对象的1-1.5倍
老年代内存大小设置为老年代存活对象的2-3倍
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663297221278-098044df-7dbe-4b37-b585-248732242d09.png)
经过测试发现Full GC后老年代内存占用为136m左右，所以设置Xms和Xmx为512m，Xmn设置为192m
# 吞吐量优先
## GC参数设置
```c
#初始堆大小/最大堆大小/新生代大小/元空间大小/元空间最大允许大小/线程栈大小
-Xms512m -Xmx512m -Xmn192m -XX:MetaspaceSize=192m -XX:MaxMetaspaceSize=320m -Xss512k
#垃圾回收器(ps和po)
-XX:+UseParallelGC  -XX:+UseParallelOldGC
#统计GC信息
-XX:PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-ps-po.log
```
查看JVM参数信息：jmap -heap pid
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663298777308-9a7e8136-092d-41d9-ad9c-be9e71dc71d2.png)
## 压测接口
线程组配置1600*5000
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663244661972-ca5f3f96-58bb-45ff-aef9-dd52d36058e7.png)
花费共29min13s
### 吞吐量
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663307852042-09f03f0e-b99e-4226-8a80-755b399d5c59.png)
平均吞吐量为4564.6
### RT
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663307958554-ea277749-8d92-4890-9ce9-373c0b2646d3.png)
平均响应时间为348ms
### TPS
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663308874534-1dbe9121-8439-4e3e-997b-7fbf52fb568d.png)
### GC统计信息
grafana+prometheus监控图
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663309198431-4273e8f9-7e5a-405d-a278-bc2983261d36.png)
GC Easy进行gc日志分析
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663309596749-55597147-676c-4ba4-a7f4-52ae952536fa.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663309644027-f3aa1ec0-4abc-4f57-8b88-2c0978e1477b.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663309760603-62c4eb12-42ee-43dd-b032-9e6f1d1316de.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663309836033-54d99a22-13e7-409e-8da1-106e65ca3eeb.png)
### 堆内存统计信息
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663309051765-f0ab4577-731b-489e-acf0-7e674a02f7e5.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663309084616-67ecfc05-4fbd-495b-81f3-2ecdf4d8413c.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663312127002-99855fa4-c2bc-47c4-bdaf-174f76aac3d9.png)
## 结果分析
GC总耗时：63.326s
Young GC总耗时：61.246s
Full GC总耗时：2.080s
最大GC停顿时间：210ms
# 响应时间优先
## GC参数设置
```c
#初始堆大小/最大堆大小/新生代大小/元空间大小/元空间最大允许大小/线程栈大小
-Xms512m -Xmx512m -Xmn192m -XX:MetaspaceSize=192m -XX:MaxMetaspaceSize=320m -Xss512k
#垃圾回收器(ParNew和CMS)
-XX:+UseParNewGC -XX:+UseConcMarkSweepGC
#统计GC信息
-XX:PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-parnew-cms.log
```
查看JVM参数信息：jmap -heap pid
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663310191152-c20b44ce-b73b-49c3-93f2-167a61403529.png)
## 压测接口
线程组配置1600*5000
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663244661972-ca5f3f96-58bb-45ff-aef9-dd52d36058e7.png)
花费共30min22s
### 吞吐量
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663312473300-8549c3ad-a305-4abb-aaa1-69969a8de9bb.png)
平均吞吐量为4392.0
### RT
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663312519137-4a80f2d8-b2bc-45e4-9ee2-fc06febe3ffe.png)
平均响应时间为361ms
### TPS
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663312583517-09aaab55-c957-430e-8e8c-97e448fd7a99.png)
### GC统计信息
grafana+prometheus监控图
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663312926717-90cbae2c-2e36-4f13-86cd-cfe1b1cc29e8.png)
GC Easy进行GC日志分析
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663313464252-6ba3971d-4a97-4c3e-b1b1-3f87560cb31c.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663313521630-34bd9719-e958-410b-8d9c-d01184d56a51.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663313641928-d2e4170d-56ec-4357-8506-37f2c70bce01.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663313735944-b7a88147-4294-4db1-8516-e2418cd8e591.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663313757594-f54157bd-bf33-446d-9e41-7c40eb68ac19.png)
### 堆内存统计信息
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663312856562-90507cb1-e612-4e98-9f22-71044ef9ea21.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663312877614-4df3fced-2af2-4494-91bd-01194a021a6d.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663314483898-409df9b6-17eb-412d-9206-6b222aa84803.png)
## 结果分析
GC总耗时：55.627s
Young GC总耗时：55.627s
Full GC总耗时：0s
最大GC停顿时间：90ms
# 全功能垃圾收集器
## GC参数设置
```c
#初始堆大小/最大堆大小/元空间大小/线程栈大小
-Xms2048m -Xmx2048m -XX:MetaspaceSize=128m -Xss512k
#垃圾回收器(G1)/期望最大停顿时间
-XX:+UseG1GC  -XX:MaxGCPauseMillis=80
#统计GC信息
-XX:PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-g1.log
```
查看JVM参数信息：jmap -heap pid
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663323441321-6121a4b5-9c83-4d1c-9b79-447c1f657fe6.png)
## 压测接口
线程组配置1600*5000
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663244661972-ca5f3f96-58bb-45ff-aef9-dd52d36058e7.png)
花费共27min37s
### 吞吐量
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325342716-a1c01c8b-a171-4f2d-8783-2c3959047023.png)
平均吞吐量为4828.7
### RT
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325391168-750e4ba7-2655-40fa-b268-68a739a60d67.png)
平均响应时间为329ms
### TPS
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325411677-762250ba-be3a-48f0-880e-f237e5245fb6.png)
### GC统计信息
grafana+prometheus监控图
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325656688-309f6339-34f7-4596-8f31-302ed4149dd4.png)
GC Easy进行GC日志分析
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325799751-e9310420-7ccf-45a3-84a5-55250882f08c.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325827069-e1bb96ff-bd53-40a7-b3de-2124205bdc60.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325849120-9e3b762d-6b46-41a6-8f46-84146053b6ca.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325881062-a701dad1-d463-4dc5-b8b0-779a8f97c44c.png)
### 堆内存统计信息
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325589191-9fb1912c-c75b-49ea-b99d-5063874a703f.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325623857-7f88fc3b-fec0-4f92-9af9-e2fcc4b693fd.png)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/26369422/1663325678142-7aed7cf9-cc44-4b16-bb0a-81b7a7e06db9.png)
## 结果分析
GC总耗时：14.630s
最大GC停顿时间：80ms
# 结论
| 指标 | 吞吐量 | TPS/s | 响应时间（ms） | GC总耗时（s） | 最大GC停顿时间（ms） |
| --- | --- | --- | --- | --- | --- |
| 吞吐量优先 | 98.134% | 4564.6 | 348 | 63.326 | 210 |
| 响应时间优先 | 97.112% | 4392.0 | 361 | 55.627 | 90 |
| 全功能 | 99.138% | 4828.7 | 329 | 14.630 | 80 |

在低延时接口下，三者对比性能其实没有很大差别，但相对来说，使用G1垃圾回收器性能明显会更优于其他两种策略，但是G1适用于堆内存较大的场景，我们还是需要具体场景具体分析
