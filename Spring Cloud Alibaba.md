# Spring Cloud Alibaba

##  nacos

### 搭建nacos环境

> 第1步: 安装nacos

``` markdown
下载地址: https://github.com/alibaba/nacos/releases 下载zip格式的安装包，然后进行解压缩操作
```

> 第2步: 启动nacos

``` shell
#切换目录
cd nacos/bin
#命令启动
startup.cmd -m standalone
```

> 第3步: 访问nacos

打开浏览器输入http://localhost:xxxx/nacos，即可访问服务， 默认密码是nacos/nacos

###  将微服务注册到nacos

> 在pom.xml中添加nacos的依赖

```xml
<!--nacos客户端-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

> 在主类上添加@EnableDiscoveryClient注解

``` java
@SpringBootApplication
@EnableDiscoveryClient
public class ProductApplication
```

> 在application.yml中添加nacos服务的地址

``` yaml
spring:
	cloud:
		nacos:
			discovery:
				server-addr: 127.0.0.1:8848
```

> 启动服务， 观察nacos的控制面板中是否有注册上来的微服务

>  修改Controller， 实现微服务调用

```java
@Autowired
private DiscoveryClient discoveryClient;
```

```java
//从nacos中获取服务地址
// DiscoveryClient是专门负责服务注册和发现的，我们可以通过它获取到注册到注册中心的所有服务
ServiceInstance serviceInstance = discoveryClient.getInstances("service-product").get(0);
String url = serviceInstance.getHost() + ":" + serviceInstance.getPort();
```

##  基于Ribbon实现负载均衡

> 第1步：在Application的RestTemplate 的生成方法上添加@LoadBalanced注解

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
	return new RestTemplate();
}
```

> 第二步：修改服务调用的方法

```java
//直接使用微服务名字， 从nacos中获取服务地址
String url = "service-product";
```

> Ribbon 支持的负载均衡策略

| 策略名                    | 策略描述                                                     | 实现说明                                                     |
| :------------------------ | :----------------------------------------------------------- | ------------------------------------------------------------ |
| BestAvailableRule         | 选择一个最小的并发请求的server                               | 逐个考察 Server，如果 Server 被 tripped 了，则忽略，在选择其中 ActiveRequestsCount 最小的server |
| AvailabilityFilteringRule | 过滤掉那些因为一直连接失败的被标记为 circuit tripped的后 端server，并过滤掉那些高并发的的后端 server（active connections 超过配置的阈值） | 使用一个 AvailabilityPredicate 来包含过滤 server 的逻辑，其实就就是检查 status 里记录的各个 server 的运行状态 |
| WeightedResponseTimeRule  | 根据相应时间分配一 个weight，相应时间越长，weight越 小，被选中的可能性越低。 | 一个后台线程定期的从 status 里面读 取评价响应时间，为每个 server 计算一个 weight。Weight 的计算也比较简单responsetime 减去每个 server 自己 平均的 responsetime是 server 的权重。当刚开始运行，没有形成 statas 时，使用roubine 策略选择 server。 |
| RetryRule                 | 对选定的负载均衡策略机上重试机制。                           | 在一个配置时间段内当选择 server 不 成功，则一直尝试使用 subRule 的方 式选择一个可用的 server |
| RoundRobinRule            | 轮询方式轮询选择 server                                      | 轮询 index，选择 index 对应位置的 server                     |
| RandomRule                | 随机选择一个server                                           | 在 index 上随机，选择 index 对应位置 的server                |
| ZoneAvoidanceRule         | 复合判断server所在 区域的性能和server 的可用性选择server     | 使用 ZoneAvoidancePredicate 和 AvailabilityPredicate 来判断是否选择 某个 server，前一个判断判定一个 zone 的运行性能是否可用，剔除不可 用的 zone（的所有 server）， AvailabilityPredicate 用于过滤掉连接 数过多的 Server。 |



##  基于Feign实现服务调用

### 什么是Feign

		Feign是Spring Cloud提供的一个声明式的伪Http客户端， 它使得调用远程服务就像调用本地服务 一样简单， 只需要创建一个接口并添加一个注解即可。 Nacos很好的兼容了Feign， Feign默认集成了 Ribbon， 所以在Nacos下使用Fegin默认就实现了负 载均衡的效果。

### Feign的使用

> 第一步：加入Fegin的依赖

```xml
<!--fegin组件-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

> 第二步：在主类上添加Fegin的注解

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients//开启Fegin
public class OrderApplication {}
```

> 第三步： 创建一个service， 并使用Fegin实现微服务调用

```java
@FeignClient("service-product")//声明调用的提供者的name
public interface ProductService {
    //指定调用提供者的哪个方法
    //@FeignClient+@GetMapping 就是一个完整的请求路径 http://serviceproduct/product/{pid}
    @GetMapping(value = "/product/{pid}")
    Product findByPid(@PathVariable("pid") Integer pid);
}
```

> 第四步： 修改controller代码，并启动验证

```java
//通过fegin调用商品微服务
Product product = productService.findByPid(pid);
```

##  Sentinel-服务容错

### 微服务集成Sentinel

>  第一步：在pom.xml中加入下面依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

> 第二步： 安装Sentinel控制台

1. 下载[jar包](https://github.com/alibaba/Sentinel/releases),解压到文件夹

2.  启动控制台

``` shell
# 直接使用jar命令启动项目(控制台本身是一个SpringBoot项目)
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -
Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.7.0.jar
```

3.  修改 `shop-order `,在里面加入有关控制台的配置

``` yaml
spring:
cloud:
sentinel:
transport:
port: 9999 #跟控制台交流的端口,随意指定一个未使用的端口即可
dashboard: localhost:8080 # 指定控制台服务的地址
```

4.  通过浏览器访问localhost:8080 进入控制台 ( 默认用户名密码是 sentinel/sentinel )

### Sentinel规则

#### 流控规则

![/picture/admin/2021/10/17/P002.jpg](http://oss.jankinwu.com/img/P002.jpg)

**资源名**：唯一名称，默认是请求路径，可自定义 

**针对来源**：指定对哪个微服务进行限流，默认指default，意思是不区分来源，全部限制 

**阈值类型/单机阈值**： 

+ QPS（每秒请求数量）: 当调用该接口的QPS达到阈值的时候，进行限流 
+ 线程数：当调用该接口的线程数达到阈值的时候，进行限流 

**是否集群**：暂不需要集群

**sentinel共有三种流控模式，分别是**： 

+ 直接（默认）：接口达到限流条件时，开启限流 
+ 关联：当关联的资源达到限流条件时，开启限流 (适合做应用让步)
+ 链路：当从某个接口过来的资源达到限流条件时，开启限流



> 禁止收敛URL的入口 context

​		从1.6.3 版本开始，Sentinel Web filter默认收敛所有URL的入口context，因此链路限流不生效。

​		 1.7.0 版本开始（对应SCA的2.1.1.RELEASE)，官方在CommonFilter 引入了 `WEB_CONTEXT_UNIFY` 参数，用于控制是否收敛context。将其配置为 `false` 即可根据不同的 URL 进行链路限流。 

​		SCA 2.1.1.RELEASE之后的版本,可以通过配置spring.cloud.sentinel.web-context-unify=false即 可关闭收敛 

​		我们当前使用的版本是SpringCloud Alibaba 2.1.0.RELEASE，无法实现链路限流。 

​		目前官方还未发布SCA 2.1.2.RELEASE，所以我们只能使用2.1.1.RELEASE，需要写代码的形式实 现

+  暂时将SpringCloud Alibaba的版本调整为2.1.1.RELEASE

``` pom
<spring-cloud-alibaba.version>2.1.1.RELEASE</spring-cloud-alibaba.version>
```

+  配置文件中关闭sentinel的CommonFilter实例化

```yaml
spring:
	cloud:
		sentinel:
			filter:
				enabled: false
```

+  添加一个配置类，自己构建CommonFilter实例

``` java
package com.itheima.config;

import com.alibaba.csp.sentinel.adapter.servlet.CommonFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterContextConfig {
    
    @Bean
    public FilterRegistrationBean sentinelFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new CommonFilter());
        registration.addUrlPatterns("/*");
        // 入口资源关闭聚合
        registration.addInitParameter(CommonFilter.WEB_CONTEXT_UNIFY, "false");
        registration.setName("sentinelFilter");
        registration.setOrder(1);
        return registration;
    }
}
```

>  配置流控效果

+ 快速失败（默认）: 直接失败，抛出异常，不做任何额外的处理，是最简单的效果 
+ Warm Up：它从开始阈值到最大QPS阈值会有一个缓冲阶段，一开始的阈值是最大QPS阈值的 1/3，然后慢慢增长，直到最大阈值，适用于将突然增大的流量转换为缓步增长的场景。 
+ 排队等待：让请求以均匀的速度通过，单机阈值为每秒通过数量，其余的排队等待； 它还会让设 置一个超时时间，当请求超过超时间时间还未处理，则会被丢弃。

####  降级规则

降级规则就是设置当满足什么条件的时候，对服务进行降级。Sentinel提供了三个衡量条件：

- 平均响应时间 ：当资源的平均响应时间超过阈值（以 ms 为单位）之后，资源进入准降级状态。 如果接下来 1s 内持续进入 5 个请求，它们的 RT都持续超过这个阈值，那么在接下的时间窗口 （以 s 为单位）之内，就会对这个方法进行服务降级。

> 注意 Sentinel 默认统计的 RT 上限是 4900 ms，超出此阈值的都会算作 4900 ms，若需要 变更此上限可以通过启动配置项 -Dcsp.sentinel.statistic.max.rt=xxx 来配置。



- 异常比例：当资源的每秒异常总数占通过量的比值超过阈值之后，资源进入降级状态，即在接下的 时间窗口（以 s 为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是 [0.0, 1.0]。
- 异常数 ：当资源近 1 分钟的异常数目超过阈值之后会进行服务降级。注意由于统计时间窗口是分 钟级别的，若时间窗口小于 60s，则结束熔断状态后仍可能再进入熔断状态。

####  热点规则

热点参数流控规则是一种更细粒度的流控规则, 它允许将规则具体到参数上。

**热点规则使用**

> 第一步：添加注解

```java
/**
* 方法必须有参数
*/
@RequestMapping("/order/message3")
@SentinelResource("message3")//注意这里必须使用这个注解标识,热点规则不生效
public String message3(String name, Integer age) {
	return name + age;
}
```

> 第二步：配置热点规则

![/picture/admin/2021/10/17/P003.png](http://tbed.jankinwu.cn/picture/admin/2021/10/17/P003.png)

![/picture/admin/2021/10/17/P004.png](http://oss.jankinwu.com/img/P004.png)

**参数索引**：方法传入参数从左往右数的位置（从0开始）

![/picture/admin/2021/10/17/P005.png](http://tbed.jankinwu.cn/picture/admin/2021/10/17/P005.png)

**参数例外项**：对参数中传的值进行限流

#### 授权规则

![/picture/admin/2021/10/17/P006.png](http://oss.jankinwu.com/img/P006.png)

**流控应用**：要填写的是来源标识，Sentinel提供了 RequestOriginParser 接口来处理来源。 只要Sentinel保护的接口资源被访问，Sentinel就会调用 RequestOriginParser 的实现类去解析访问来源

> 第一步：自定义来源处理规则

```java
@Component
public class RequestOriginParserDefinition implements RequestOriginParser{
    @Override
    public String parseOrigin(HttpServletRequest request) {
        String serviceName = request.getParameter("serviceName");
        return serviceName;
    }
}
```

> 第二步：授权规则配置

上图配置的意思是只有serviceName=pc能访问(白名单)

> 第三步：访问 http://localhost:8091/order/message1?serviceName=pc观察结果

####  系统规则

​		系统保护规则是从应用级别的入口流量进行控制，从单台机器的总体 Load、RT、入口 QPS 、CPU 使用率和线程数五个维度监控应用数据，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。 系统保护规则是应用整体维度的，而不是资源维度的，并且仅对入口流量 (进入应用的流量) 生效。 

- Load（仅对 Linux/Unix-like 机器生效）：当系统 load1 超过阈值，且系统当前的并发线程数超过 系统容量时才会触发系统保护。系统容量由系统的 maxQps * minRt 计算得出。设定参考值一般 是 CPU cores * 2.5。 
- RT：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。 
- 线程数：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。 
- 入口 QPS：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。 
- CPU使用率：当单台机器上所有入口流量的 CPU使用率达到阈值即触发系统保护。

**自定义异常返回**

```java
 //异常处理页面
@Component
public class ExceptionHandlerPage implements UrlBlockHandler {
    
    @Override
    public void blocked(HttpServletRequest request, HttpServletResponse response, BlockException e) throws IOException {
        response.setContentType("application/json;charset=utf-8");
        ResponseData data = null;
        //BlockException 异常接口,包含Sentinel的五个异常
        // FlowException 限流异常
        // DegradeException 降级异常
        // ParamFlowException 参数限流异常
        // AuthorityException 授权异常
        // SystemBlockException 系统负载异常
        if (e instanceof FlowException) {
        	data = new ResponseData(-1, "接口被限流了...");
        } else if (e instanceof DegradeException) {
        	data = new ResponseData(-2, "接口被降级了...");
        }
        	response.getWriter().write(JSON.toJSONString(data));
	}
}
@Data
@AllArgsConstructor//全参构造
@NoArgsConstructor//无参构造
class ResponseData {
    private int code;
    private String message;
}
```

###  @SentinelResource的使用

@SentinelResource 用于定义资源，并提供可选的异常处理和 fallback 配置项。其主要参数如下:

| 属性               | 作用                                                         |
| ------------------ | :----------------------------------------------------------- |
| value              | 资源名称                                                     |
| entryType          | entry类型，标记流量的方向，取值IN/OUT，默认是OUT             |
| blockHandler       | 处理BlockException的函数名称,函数要求：1.  必须是 public 2.返回类型 参数与原方法一致 3. 默认需和原方法在同一个类中。若希望使用其他类的函数，可配置 blockHandlerClass ，并指定blockHandlerClass里面的方法。 |
| blockHandlerClass  | 存放blockHandler的类,对应的处理函数必须static修饰。          |
| fallback           | 用于在抛出异常的时候提供fallback处理逻辑。fallback函数可以针对所 有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进 行处理。函数要求： 1. 返回类型与原方法一致 2. 参数类型需要和原方法相匹配 3. 默认需和原方法在同一个类中。若希望使用其他类的函数，可配置 fallbackClass ，并指定fallbackClass里面的方法。 |
| fallbackClass      | 存放fallback的类。对应的处理函数必须static修饰。             |
| defaultFallback    | 用于通用的 fallback 逻辑。默认fallback函数可以针对所有类型的异常进 行处理。若同时配置了 fallback 和 defaultFallback，以fallback为准。函数要求： 1. 返回类型与原方法一致 2. 方法参数列表为空，或者有一个 Throwable 类型的参数。 3. 默认需要和原方法在同一个类中。若希望使用其他类的函数，可配置 fallbackClass ，并指定 fallbackClass 里面的方法。 |
| exceptionsToIgnore | 指定排除掉哪些异常。排除的异常不会计入异常统计，也不会进入 fallback逻辑，而是原样抛出。 |
| exceptionsToTrace  | 需要trace的异常                                              |

定义限流和降级后的处理方法

> 方式一：直接将限流和降级方法定义在方法中

```java
@Service
@Slf4j
public class OrderServiceImpl3 {
    
    int i = 0;
    @SentinelResource(
        value = "message",
        blockHandler = "blockHandler",// 指定发生BlockException时进入的方法
        fallback = "fallback"// 指定发生Throwable时进入的方法
    )
    public String message() {
        i++;
        if (i % 3 == 0) {
        	throw new RuntimeException();
    	}
    	return "message";
    }
    
    // BlockException时进入的方法
    // 1.当前方法返回值类型和参数必须和原方法一致
    // 2.但是允许在参数列表的最后加入一个参数BlockException，用来接收原方法中的异常
    public String blockHandler(BlockException ex) {
        log.error("{}", ex);
        return "接口被限流或者降级了...";
    }
    
    //Throwable时进入的方法
    public String fallback(Throwable throwable) {
        log.error("{}", throwable);
        return "接口发生异常了...";
    }
}
```

> 方式二: 将限流和降级方法外置到单独的类中

```java
@Service
@Slf4j
public class OrderServiceImpl3 {
    
    int i = 0;
    @SentinelResource(
        value = "message",
        blockHandlerClass = OrderServiceImpl3BlockHandlerClass.class,
        blockHandler = "blockHandler",
        fallbackClass = OrderServiceImpl3FallbackClass.class,
        fallback = "fallback"
    )
    public String message() {
        i++;
        if (i % 3 == 0) {
        	throw new RuntimeException();
        }
    	return "message4";
    }
}

@Slf4j
public class OrderServiceImpl3BlockHandlerClass {
    //注意这里必须使用static修饰方法
    public static String blockHandler(BlockException ex) {
        log.error("{}", ex);
        return "接口被限流或者降级了...";
    }
}

@Slf4j
public class OrderServiceImpl3FallbackClass {
    //注意这里必须使用static修饰方法
    public static String fallback(Throwable throwable) {
        log.error("{}", throwable);
        return "接口发生异常了...";
    }
}
```

### Sentinel规则持久化

​		首先 Sentinel 控制台通过 API 将规则推送至客户端并更新到内存中，接着注册的写数据源会将新的 规则保存到本地的文件中。

![Sentinel 推送过程](http://oss.jankinwu.com/img/P007.jpg)

<center><font size=3>Sentinel 推送过程</font></center>

> 第一步; 编写处理类

```java
//规则持久化
public class FilePersistence implements InitFunc {
    
    @Value("spring.application:name")
    private String appcationName;
    
    @Override
    public void init() throws Exception {
        String ruleDir = System.getProperty("user.home") + "/sentinelrules/"+appcationName;
        String flowRulePath = ruleDir + "/flow-rule.json";
        String degradeRulePath = ruleDir + "/degrade-rule.json";
        String systemRulePath = ruleDir + "/system-rule.json";
        String authorityRulePath = ruleDir + "/authority-rule.json";
        String paramFlowRulePath = ruleDir + "/param-flow-rule.json";
        
        this.mkdirIfNotExits(ruleDir);
        this.createFileIfNotExits(flowRulePath);
        this.createFileIfNotExits(degradeRulePath);
        this.createFileIfNotExits(systemRulePath);
        this.createFileIfNotExits(authorityRulePath);
        this.createFileIfNotExits(paramFlowRulePath);
        
        // 流控规则
        ReadableDataSource<String, List<FlowRule>> flowRuleRDS = new FileRefreshableDataSource<>(
            flowRulePath,
            flowRuleListParser
        );
        FlowRuleManager.register2Property(flowRuleRDS.getProperty());
        WritableDataSource<List<FlowRule>> flowRuleWDS = new FileWritableDataSource<>( 
            flowRulePath, 					
            this::encodeJson
        );
		WritableDataSourceRegistry.registerFlowDataSource(flowRuleWDS);
        
        // 降级规则
        ReadableDataSource<String, List<DegradeRule>> degradeRuleRDS = new FileRefreshableDataSource<>(
            degradeRulePath,
            degradeRuleListParser
        );
        DegradeRuleManager.register2Property(degradeRuleRDS.getProperty());
        WritableDataSource<List<DegradeRule>> degradeRuleWDS = new FileWritableDataSource<>(
            degradeRulePath,
            this::encodeJson
    	);
        WritableDataSourceRegistry.registerDegradeDataSource(degradeRuleWDS);
        
        // 系统规则
        ReadableDataSource<String, List<SystemRule>> systemRuleRDS = new FileRefreshableDataSource<>(
            systemRulePath,
            systemRuleListParser
        );
        SystemRuleManager.register2Property(systemRuleRDS.getProperty());
        WritableDataSource<List<SystemRule>> systemRuleWDS = new FileWritableDataSource<>(
            systemRulePath,
            this::encodeJson
        );
        WritableDataSourceRegistry.registerSystemDataSource(systemRuleWDS);
        
        // 授权规则
        ReadableDataSource<String, List<AuthorityRule>> authorityRuleRDS = new FileRefreshableDataSource<>(
            authorityRulePath,
            authorityRuleListParser
        );
        AuthorityRuleManager.register2Property(authorityRuleRDS.getProperty());
        WritableDataSource<List<AuthorityRule>> authorityRuleWDS = new FileWritableDataSource<>(
            authorityRulePath,
            this::encodeJson
        );
        WritableDataSourceRegistry.registerAuthorityDataSource(authorityRuleWDS);
        
        // 热点参数规则
        ReadableDataSource<String, List<ParamFlowRule>> paramFlowRuleRDS = new FileRefreshableDataSource<>(
            paramFlowRulePath,
            paramFlowRuleListParser
        );
        ParamFlowRuleManager.register2Property(paramFlowRuleRDS.getProperty());
        WritableDataSource<List<ParamFlowRule>> paramFlowRuleWDS = new FileWritableDataSource<>(
            paramFlowRulePath,
            this::encodeJson
        );
        ModifyParamFlowRulesCommandHandler.setWritableDataSource(paramFlowRuleWDS);
	}
    
    private Converter<String, List<FlowRule>> flowRuleListParser = source -> JSON.parseObject(
        source,
        new TypeReference<List<FlowRule>>() {
    	}
    );
    
    private Converter<String, List<DegradeRule>> degradeRuleListParser = source -> JSON.parseObject(
        source,
        new TypeReference<List<DegradeRule>>() {
    	}
    );
    
    private Converter<String, List<SystemRule>> systemRuleListParser = source -> JSON.parseObject(
        source,
        new TypeReference<List<SystemRule>>() {
        }
    );
    
    private Converter<String, List<AuthorityRule>> authorityRuleListParser = source -> JSON.parseObject(
        source,
        new TypeReference<List<AuthorityRule>>() {
        }
    );
    
    private Converter<String, List<ParamFlowRule>> paramFlowRuleListParser = source -> JSON.parseObject(
        source,
        new TypeReference<List<ParamFlowRule>>() {
        }
    );
    
    private void mkdirIfNotExits(String filePath) throws IOException {
        File file = new File(filePath);
        if (!file.exists()) {
        	file.mkdirs();
        }
    }
    
	private void createFileIfNotExits(String filePath) throws IOException {
        File file = new File(filePath);
        if (!file.exists()) {
        	file.createNewFile();
        }
	}
    
    private <T> String encodeJson(T t) {
    	return JSON.toJSONString(t);
    }
}
```

> 第二步：添加配置  

在`resources`下创建配置目录 `META-INF/services` ,然后添加文件`com.alibaba.csp.sentinel.init.InitFunc`

在文件中添加配置类的全路径  

```
com.itheima.config.FilePersistence
```

### Feign整合Sentinel  

> 第一步：引入sentinel的依赖  

```xml
<!--sentinel客户端-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

> 第二步：在配置文件中开启Feign对Sentinel的支持  

```yaml
feign:
    sentinel:
    	enabled: true
```

> 第三步：创建容错类  

```java
//容错类要求必须实现被容错的接口,并为每个方法实现容错方案
@Component
@Slf4j
public class ProductServiceFallBack implements ProductService {
    
    @Override
    public Product findByPid(Integer pid) {
        Product product = new Product();
        product.setPid(-1);
        return product;
    }
}
```

> 第四步：为被容器的接口指定容错类  

```java
//value用于指定调用nacos下哪个微服务
//fallback用于指定容错类
@FeignClient(value = "service-product", fallback = ProductServiceFallBack.class)
public interface ProductService {
    
    @RequestMapping("/product/{pid}")//指定请求的URI部分
    Product findByPid(@PathVariable Integer pid);
}
```

## Spring Cloud Gateway

### 基础使用

> 第一步：创建一个 api-gateway 的模块,导入相关依赖

```xml
<!--gateway网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

> 第二步：创建主类

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
    	SpringApplication.run(GatewayApplication.class, args);
    }
}
```

> 第三步：添加配置文件

```yaml
server:
	port: 7000
spring:
	application:
		name: api-gateway
	cloud:
		gateway:
			routes: # 路由数组[路由 就是指定当请求满足什么条件的时候转到哪个微服务]
                - id: product_route # 当前路由的标识, 要求唯一
                    uri: http://localhost:8081 # 请求要转发到的地址
                    order: 1 # 路由的优先级,数字越小级别越高
                    predicates: # 断言(就是路由转发要满足的条件)
                        - Path=/product-serv/** # 当请求路径满足Path指定的规则时,才进行路由转发
                    filters: # 过滤器,请求在传递过程中可以通过过滤器对其进行一定的修改
                        - StripPrefix=1 # 转发之前去掉1层路径
```

### 增强版

> 第一步：加入nacos依赖  

```xml
<!--nacos客户端-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

> 第二步：在主类上添加注解  

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ApiGatewayApplication {
    public static void main(String[] args) {
    	SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

> 第三步：修改配置文件

```yaml
server:
	port: 7000
spring:
    application:
    	name: api-gateway
    cloud:
        nacos:
            discovery:
            	server-addr: 127.0.0.1:8848
        gateway:
            discovery:
                locator:
                	enabled: true # 让gateway可以发现nacos中的微服务
            routes:
                - id: product_route
                    uri: lb://service-product # lb指的是从nacos中按照名称获取微服务,并遵循负载均衡策略
                    predicates:
                    	- Path=/product-serv/**
                    filters:
                    	- StripPrefix=1
```

### 简写版  

> 第一步: 去掉关于路由的配置  

```yaml
# uri 默认为“网关服务地址/微服务名称/接口地址”，按照这个格式访问，就可以成功响应
server:
	port: 7000
spring:
    application:
    	name: gateway
    cloud:
        nacos:
            discovery:
            	server-addr: 127.0.0.1:8848
        gateway:
        	discovery:
        		locator:
        			enabled: true `
```

### 断言  

Predicate(断言, 谓词) 用于进行条件判断，只有断言都返回真，才会真正的执行路由。
断言就是说: 在什么条件下才能进行路由转发  

#### 内置路由断言工厂
