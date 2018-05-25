---
title: 'java&guava常用用法'
date: "2018/05/23"
tags: [guava, util]
categories: ['java']
copyright: true
---
StringUtils：
```xml
<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-lang3</artifactId>
   <version>3.4</version>
</dependency>
```
CollectionUtils：
```xml
<dependency>
  <groupId>commons-collections</groupId>
  <artifactId>commons-collections</artifactId>
  <version>3.2.1</version>
</dependency>
```
1、`private static String LINE_SEPETOR = System.getProperty("line.separator");`   得到当前系统的换行符，防止因为系统不同造成不同结果。

## guava
2、Preconditions 判断参数是否正确/是否为null
3、`Strings.isNullOrEmpty`
3、`Splitter.on(LINE_SEPETOR).trimResults().omitEmptyStrings().withKeyValueSeparator(“=”).split(document); `  
     字符串按照“LINE_SEPETOR”分割后，去除每个分割结果前后的空格，去除空的分割结果，再把每个分割结果用“=”分割成map。
    （想象场景，想把字符串url的参数解析出来成map）
4、`Joiner.on(“&”).withKeyValueSeparator(“=”).join(map)`  把map的key和value之间用=连接，之后用&把所有的key=value对儿连接起来。（与上面方法相反，把一个map转成一个url的形式。）
5、`Maps.difference` 比较两个map，返回结果通过entriesOnlyOnLeft可以得到第一个map有第二个map没有的结果；entriesOnlyOnRight得到第二个map有，第一个map没有的结果；entriesDiffering得到key一样value不一样的结果。
6、ImmutableMap、ImmutableSet不可变。

## 集合过滤
1、过滤
```java
public class PostPredication implements Predicate<RequestLog>{

    @Override
    public boolean apply(RequestLog input) {
        if(...){
			return true;
		}
    }
}
```
对集合里的每条数据执行apply方法，符和条件的返回true。

2、使用
```java
List<RequestLog> logs = FileUtils.loadText(ACCESS_LOG, new OneLineProcess());
Collections2.filter(logs, new PostPredication());
```
把集合logs符和条件的返回。
还可以不实现Predicate方法，在filter的参数里直接放一个匿名类。

## 对map过滤
map可以通过key/value/entry等过滤，下面的例子是对value过滤。使用时用的是匿名类的方法，还可以使用上面的方法新建一个类实现Predicate类。
```java
private Map<City, Province> separateResult = new HashMap<City, Province>();
Map resultEntry = Maps.filterValues(separateResult, new Predicate<Province>() {
    @Override
    public boolean apply(Province province) {
        return province.getProvinceId() == provinceId ? true : false;
    }
});
```

## 读/处理文件，更简洁

1、对文件每行内容处理、实现LineProcessor。
```
public class OneLineProcess implements LineProcessor<List<RequestLog>> {

    private List<RequestLog> requestLogs = new ArrayList<RequestLog>();

    public OneLineProcess(){
        。。。
    }

    @Override
    public boolean processLine(String oneLine){
        Preconditions.checkNotNull(oneLine);
		。。。。对文件每行内容处理
        return true;
    }

    @Override
    public List<RequestLog> getResult(){
        return requestLogs;
    }
}
```
2.使用
```
File textFile = new File(filePath);
Files.readLines(textFile, Charsets.UTF_8, oneLineProcess)；
```
第二行代码的返回就是调用第一步中getResult方法的返回结果。


## spring拦截器

1、在mvc配置：
```
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <ref bean="loginInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```
标签`mvc:mapping`的参数`path`的值是要拦截的请求


2、代码实现loginInterceptor
```
@Component
public class LoginInterceptor extends HandlerInterceptorAdapter{

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Cookie[] cookies = request.getCookies();
        for(Cookie cookie: cookies){
            if(。。。){
                return true;
            }
        }
        response.sendRedirect("/tologin");
        return false;
    }
}
```
## spring切片

1、spring配置
```
<aop:aspectj-autoproxy proxy-target-class="true"/>
```
2、切片类
```
@Aspect
@Component
public class DaoAop {

    @Pointcut("execution(* com.qunar.fresh.dao.*.*(..))")
    public void daoPoint(){}

    @Around("daoPoint()")
    public Object timingDao(ProceedingJoinPoint proceedingJoinPoint){
        Object result = null;
        Stopwatch stopwatch = Stopwatch.createStarted();
        try {
            result = proceedingJoinPoint.proceed();
            long elapsed = stopwatch.elapsed(TimeUnit.MILLISECONDS);
            log.info("dao层{}执行耗时: {}", proceedingJoinPoint.getSignature(), elapsed);
        } catch (Throwable throwable) {
            log.error("aop记录，dao层执行时出错: {}", throwable);
        }
        return result;
    }
}
```
com.qunar.fresh.dao.\*.\*(..)  是包.类.方法(参数),上面的例子是统计dao层方法的执行时间。