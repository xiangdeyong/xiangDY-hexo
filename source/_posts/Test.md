title: Test
author: xiangDY
type: "tags"
tags:
  - code
categories:
  - SpringBoot
  - ''
date: 2021-09-28 14:15:00
---
# Condition

该功能是Spring新增的功能， 可以实现选择性的创建**Bean**。

## 1.定义Bean和Bean Config

```java
public class User {
}

@Configuration
public class UserConfig {

    @Bean
		//指定只有该Bean存在，user才被Spring容器加载	
    @MyConditionOnClass("org.springframework.data.redis.core.RedisTemplate")
    public User user(){
        return new User();
    }
}
```

## 2.自定义condition注解

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
//继承@Condition 内部使用MyUserCondition确定Bean是否加载的逻辑
@Conditional(MyUserCondition.class)
public @interface MyConditionOnClass {
    String[] value();
}
```

## 3.自定义Bean加载逻辑condition类

```java
public class MyUserCondition implements Condition {
    /**
     * 是否允许创建Bean true-是 false-否
     * @param context 上下文 可以获取属性文件、classLoader之类的
     * @param metadata 注解元信息:获取注解定义的属性值
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {

        Map<String, Object> map = metadata.getAnnotationAttributes(MyConditionOnClass.class.getName());
        String[] values = (String[]) map.get("value");

        for (String value : values) {
            Class<?> aClass = null;
            try {
                aClass = Class.forName(value);
								//指定任意一个类已存在，则该bean可被加载
                if(aClass != null){
                    System.out.println(aClass.getName());
                    return true;
                }
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
        return false;
    }
}
```
