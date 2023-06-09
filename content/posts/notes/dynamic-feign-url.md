---
title: "Feign 的动态 URL"
date: 2023-06-09T18:54:30+08:00
lastmod: 2023-06-09T18:54:30+08:00
draft: false
series: [Notes]
tags: [Feign]
summary: "将 @FeignClient 的 url 配置为动态 url，实现不重启服务，能切换请求地址"
---
## 继承 ***Feign.Builder***，重写 ***target*** 方法

```java
@Component
public class FeignBuilderHelper extends Feign.Builder {

    private final DynamicUrlHelper dynamicUrlHelper;

    @Autowired
    public FeignBuilderHelper(DynamicUrlHelper dynamicUrlHelper) {
        this.dynamicUrlHelper = dynamicUrlHelper;
    }

    @Override
    public <T> T target(Target<T> target) {
        return super.target(new Target.HardCodedTarget<T>(target.type(), target.name(), target.url()) {
            @Override
            public String url() {
                String url = dynamicUrlHelper.dynamicUrl(this.name());
                return StringUtils.isBlank(url) ? super.url() : url;
            }
        });
    }
}
```

## 从配置中心，获取动态的URL

```java
@Configuration
public class DynamicUrlHelper {

    private static final String BIG_DATA_SERVER_NAME = "BigDataServer";

    @Value("${big.data.virtual.host:}")
    private String bigDataServerUrl;

    public String dynamicUrl(String targetName) {
        if (Objects.equals(targetName, BIG_DATA_SERVER_NAME)) {
            return bigDataServerUrl;
        }
        return Strings.EMPTY;
    }
}
```

## 注意

可能会出现 `BeanCreationException` 异常：

```
Unsatisfied dependency expressed through constructor parammeter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creeating bean with name '......': FactoryBean threw exception object creation; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'feign.Feign$Builder' available: expected single matching bean but found 2: feignBuilderHelper,feignHystrixBuilder 
```

解决：

`FeignBuilderHelper` 添加 `@Primary` 注解

## 特别鸣谢

[Feign：实现动态URL](https://blog.csdn.net/suxingrui/article/details/113744589)