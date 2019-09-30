# Spring Cloud微服务实战

一、  Eureka注册中心搭建

1. @EnableEurekaServer

2. application.yml配置：

   ```properties
#提供服务端口
   server.port=1111
#提供服务的域名，本地可以使用localhost或者配置hosts测试
   eureka.instance.hostname=localhost
#关闭向注册中心注册自己
   eureka.client.register-with-eureka=false
#关闭发现注册服务，注册中心仅用于维护节点
   eureka.client.fetch-registry=false
#配置注册中心提供服务的url（这里引用上边的配置）
   eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
   ```
   
   application.yml配置：
   
   application.yml配置：
   
   application.yml配置：
   
   application.yml配置：
   
   application.yml配置：
   
   application.yml配置：
   
   application.yml配置：
   
   application.yml配置：
   
   application.yml配置：

二、  Eureka注册中心集群实现高可用

1. Peer1注册中心application.yml配置：

   #应用名
spring.application.name=eureka-server
   #提供服务端口1111
server.port=1111
   #提供服务的域名，这里在hosts文件中修改了
eureka.instance.hostname=peer1
   #向第二个注册中心注册自己
eureka.client.service-url.defaultZone=http://peer2:1112/eureka/

2. Peer2注册中心application.yml配置：

   #应用名称与第一个注册中心一样
spring.application.name=eureka-server
   #提供服务端口1112
server.port=1112
   #提供服务的域名，这里在hosts文件中修改了
eureka.instance.hostname=peer2
   #向第一个注册中心注册自己
eureka.client.service-url.defaultZone=http://peer1:1111/eureka/

三、  服务提供者搭建



1. ```java
   @EnableDiscoveryClient
   ```

2. ```java
   @Autowired
   private DiscoveryClient client; //注入发现客户端
   ```

3. ```properties
   server.port=8080
   
   spring.application.name=hello-service
   eureka.client.serviceUrl.defaultZone=http://localhSost:1111/eureka/, http://localhost:1112/eureka/
   ```

四、服务发现与消费：以Ribbon为例



1. ```java
   @EnableDiscoveryClient
   ```

2. ```java
   @Bean  //将此Bean交给spring容器
   @LoadBalanced  //通过此注解开启负载均衡
   RestTemplate restTemplate(){
        return new RestTemplate();
   }
   ```

3. ```java
@Autowired
   //注入restTemplate
private RestTemplate restTemplate;
   ```

4. ```java
   //使用restTemplate调用微服务接口
   restTemplate.getForEntity("**http://hello-service/hello**", String.class).getBody();
   ```

5. ```properties
application.yml配置：
   #为ribbon-customer指定服务端口
server.port=9000   
   #指定应用名 
spring.application.name=ribbon-customer
   #指定eureka注册中心地址
eureka.client.serviceUrl.defaultZone: http://peer1:1111/eureka/,http://peer2:1112/eureka/
   ```

备注：Ribbon作为服务消费者，可以在用户获取到服务提供者提供的服务的同时，不向用户暴露接口地址。可以看到，这里调用服务接口的时候使用的是服务提供者的服务名代替主机名，这在服务治理框架中，这种特性很重要。

五、使用Ribbon进行Restful请求

## Get请求

### getForEntity：此方法有三种重载形式，分别为：

·         `getForEntity(String url, Class<T> responseType)`

·         `getForEntity(String url, Class<T> responseType, Object... uriVariables)`

·         `getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables)`

·         `getForEntity(URI url, Class<T> responseType)`

###### 注意：此方法返回的是一个包装对象`ResponseEntity<T>`其中`T`为`responseType`传入类型，想拿到返回类型需要使用这个包装类对象的`getBody()`方法

### getForObject:此方法也有三种重载形式，这点与getForEntity方法相同：

·         `getForObject(String url, Class<T> responseType)`

·         `getForObject(String url, Class<T> responseType, Object... uriVariables)`

·         `getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables)`

·         `getForObject(URI url, Class<T> responseType)`

###### 注意：此方法返回的对象类型为`responseType`传入类型

## Post请求

post请求和get请求都有`*ForEntity`和`*ForObject`方法，其中参数列表有些不同，除了这两个方法外，还有一个postForLocation方法，其中postForLocation以post请求提交资源，并返回新资源的URI

### postForEntity：此方法有三种重载形式，分别为：

·         `postForEntity(String url, Object request, Class<T> responseType, Object... uriVariables)`

·         `postForEntity(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)`

· `postForEntity(URI url, Object request, Class<T> responseType)`

###### 注意：此方法返回的是一个包装对象`ResponseEntity<T>`其中`T`为`responseType`传入类型，想拿到返回类型需要使用这个包装类对象的`getBody()`方法

### postForObject:此方法也有三种重载形式，这点与postForEntity方法相同：

·         `postForObject(String url, Object request, Class<T> responseType, Object... uriVariables)`

·         `postForObject(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)`

· `postForObject(URI url, Object request, Class<T> responseType)`

###### 注意：此方法返回的对象类型为`responseType`传入类型

### postForLocation:此方法中同样有三种重载形式，分别为：

·         `postForLocation(String url, Object request, Object... uriVariables)`

·         `postForLocation(String url, Object request, Map<String, ?> uriVariables)`

·         `postForLocation(URI url, Object request)`

###### 注意：此方法返回的是新资源的URI，相比`getForEntity`、`getForObject`、`postForEntity`、`postForObject`方法不同的是这个方法中无需指定返回类型，因为返回类型就是URI，通过`Object... uriVariables`、`Map<String, ?> uriVariables`进行传参依旧需要占位符，参看postForEntity部分代码

## Put请求&&Delete请求

 delete请求和put请求都是没有返回值的

### put请求方法如下：

·         `put(String url, Object request, Object... uriVariables)`

·         `put(String url, Object request, Map<String, ?> uriVariables)`

·         `put(URI url, Object request)`

### delete请求方法如下：

·         `delete(String url, Object... uriVariables)`

·         `delete(String url, Map<String, ?> uriVariables)`

·         `delete(URI url)`

 

六、Hystrix熔断器（读：high string s）

```java
@Service 
public class RibbonService { 
@Autowired 
private RestTemplate restTemplate; 
 
@HystrixCommand(fallbackMethod = "hystrixFallback") 
public String helloService(){ 
//调用服务提供者接口，正常则返回hello字符串 
return restTemplate.getForEntity("http://hello-service/hello", String.class).getBody(); 
} 

/** * 调用服务失败处理方法 * @return “error" */ 
public String hystrixFallback(){ 
    return "error"; 
  } 
}
```

七、自定义HystrixCommand

八、Hystrix请求缓存：同一请求多次访问中保证值调用一次服务提供者的接口

| **注解**     | **描述**                                                     | **属性**                     |
| ------------ | ------------------------------------------------------------ | ---------------------------- |
| @CacheResult | 该注解用来标记请求命令返回的结果应该被缓存，它必须与@HystrixCommand注解结合使用 | cacheKeyMethod               |
| @CacheRemove | 该注解用来让请求命令的缓存失效，失效的缓存根据commandKey进行查找。 | commandKey,   cacheKeyMethod |
| @CacheKey    | 该注解用来在请求命令的参数上标记，使其作为cacheKey，如果没有使用此注解则会使用所有参数列表中的参数作为cacheKey | value                        |

 

九、Hystrix请求合并：将单个请求合并成一个请求去调用服务提供者，从而降低服务提供者负载

​                                                  

问题：

1. 我们为这个请求人为的设置了延迟时间，这样在并发不高的接口上使用请求合并，会降低响应速度

2. 有可能会提高服务提供者的负载：返回List的方法并发比返回单个对象方法负载更高的情况

3. 实现请求合并比较复杂

 

十、Hystrix Dashboard单体监控、集群监控与消息代理结合

十一、          声明式服务调用：Feign的使用

1. 使用Feign当做Service来使用服务提供者

   

   ```java
/** * 服务提供者的Feign 
   \* 这个接口相当于把原来的服务提供者项目当成一个Service类， 
\* 我们只需在声明它的Feign-client的名字，会自动去调用注册中心的这个名字的服务 
   \* 更简单的理解是value相当于MVC中的Controller类的父路径，通过"父路径+子路径和参数来调用服务" 
*/ 
   @FeignClient(value = "eureka-service") //其中的value的值为要调用服务的名称 
public interface EurekaServiceFeign { 
   /** 
\* 第一个Feign代码 
   \* Feign中没有原生的@GetMapping/@PostMapping/@DeleteMapping/@PutMapping，要指定需要method进行 
*/ 
   @RequestMapping(value = "/hello", method=RequestMethod.GET) 
   String helloFeign(); 
   }
   ```
2. 不需要在每个接口加FeignClient注解，通过继承特性让这些接口可以直接使用Feign去调用服务提供者的接口方法

   

   ```java
   /** 
   * 继承服务提供者的HelloService的接口，从而拥有这个接口的所有方法 
   * 那么在这个Feign中就只需要使用HelloService定义的接口方法 
   */ 
   @FeignClient("eureka-service") 
   public interface RefactorHelloServiceFeign extends HelloService { }
   ```

十二、          Zuul：静态路由、静态过滤器与动态路由的实现

l  它作为系统的统一入口，屏蔽了系统内部各个微服务的细节

l  可以和服务治理框架结合，实现自动化服务实例维护和负载均衡

l  实现接口权限校验与微服务业务逻辑的解耦

l  通过网关中的过滤器，在各生命周期中去校验请求内容，将原本在对外的服务层做的校验前移，保证微服务的无状态性，降低测试难度，解放程序员专注业务的处理

十三、          Config配置中心与客户端的使用