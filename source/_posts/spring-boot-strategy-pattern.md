---
title: spring boot 中实现策略模式
date: 2021-06-16 10:58:12
tags: spring-boot design pattern strategy spring 设计模式
---


## 1. 简介

策略模式用于避免编写开发者在方法里使用大量的 `If-Else` 语句，将使用算法的责任和算法本身分割开来，委派给不同的对象管理，提升代码可读性以及可维护性。

![structure](/images/strategy-pattern/structure.png)

- 上下文 （Context） 维护指向具体策略的引用， 且仅通过策略接口与该对象进行交流。
- 策略 （Strategy） 接口是所有具体策略的通用接口， 它声明了一个上下文用于执行策略的方法。
- 客户端 （Client） 会创建一个特定策略对象并将其传递给上下文。 上下文则会提供一个设置器以便客户端在运行时替换相关联的策略。
- 当上下文需要运行算法时， 它会在其已连接的策略对象上调用执行方法。 上下文不清楚其所涉及的策略类型与算法的执行方式。

## Spring 框架中使用策略模式

spring 中对于策略模式模式的实现，可以利用 spring 框架关于 `Map` 类型的自动装配特性。
以 refactoringguru 网站上的 Navigator 为实例，进行思路分析。

![example](/images/strategy-pattern/example.png)

定义接口
```java
public interface RoutingStrategy {
  Route buildRoute();
}
```

RoadStrategy 实现类
```java
@Component("RoadStrategy")
public class RoadStrategy implements RoutingStrategy {
  Route buildRoute() {
    return new Route("road");
  };
}
```

PublicTransportStrategy 实现类
```java
@Component("PublicTransportStrategy")
public class PublicTransportStrategy implements RoutingStrategy {
  Route buildRoute() {
    return new Route("PublicTransport");
  };
}
```

WalkingStrategy 实现类
```java
@Component("WalkingStrategy")
public class WalkingStrategy implements RoutingStrategy {
  Route buildRoute() {
    return new Route("Walking");
  };
}
```

每个实现类分别实现 `RoutingStrategy` 接口，并使用 `@Component` 注解将实现类注入到 spring 容器中。

`RoutingController` 使用 `Map<String, RoutingStrategy>` 获得 spring 容器中全部实现 `RoutingStrategy` 接口的实例。
此外，Map 中 key 的值对应上文使用 `@Component` 注入是为实现类设置的名称。

```java
@RestController
@RequestMapping(value = "routing")
public class RoutingController {

  private Map<String, RoutingStrategy> routingStrategies;

  // 利用 spring 框架的 constructor 注入
  public OrderController(Map<String, RoutingStrategy> routingStrategies) {
    this.routingStrategies = routingStrategies;
  }

  @RequestMapping(value = "type")
  @RespondBody
  public Route getRouteByType(String type) {
    return this.routingStrategies.get(type).buildRoute();
  }
}
```

依据 Web API 传入的 `type`，在 `routingStrategies` 字典找到对应的策略实现类，并通过实现类中独立封装的算法逻辑，返回需要的 `Route` 对象。

## spring 集合类型注入知识补充

spring 框架处理集合类型自动装配的源码位于 `DefaultListableBeanFactory` 类的 `resolveMultipleBeans` 方法。

```java
@Nullable
private Object resolveMultipleBeans(DependencyDescriptor descriptor, @Nullable String beanName,
    @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) {

  Class<?> type = descriptor.getDependencyType();
  if (type.isArray()) {
    Class<?> componentType = type.getComponentType();
    ResolvableType resolvableType = descriptor.getResolvableType();
    Class<?> resolvedArrayType = resolvableType.resolve();
    if (resolvedArrayType != null && resolvedArrayType != type) {
      type = resolvedArrayType;
      componentType = resolvableType.getComponentType().resolve();
    }
    if (componentType == null) {
      return null;
    }
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, componentType,
        new MultiElementDescriptor(descriptor));
    if (matchingBeans.isEmpty()) {
      return null;
    }
    if (autowiredBeanNames != null) {
      autowiredBeanNames.addAll(matchingBeans.keySet());
    }
    TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
    Object result = converter.convertIfNecessary(matchingBeans.values(), type);
    if (getDependencyComparator() != null && result instanceof Object[]) {
      Arrays.sort((Object[]) result, adaptDependencyComparator(matchingBeans));
    }
    return result;
  }
  else if (Collection.class.isAssignableFrom(type) && type.isInterface()) {
    Class<?> elementType = descriptor.getResolvableType().asCollection().resolveGeneric();
    if (elementType == null) {
      return null;
    }
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, elementType,
        new MultiElementDescriptor(descriptor));
    if (matchingBeans.isEmpty()) {
      return null;
    }
    if (autowiredBeanNames != null) {
      autowiredBeanNames.addAll(matchingBeans.keySet());
    }
    TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
    Object result = converter.convertIfNecessary(matchingBeans.values(), type);
    if (getDependencyComparator() != null && result instanceof List) {
      Collections.sort((List<?>) result, adaptDependencyComparator(matchingBeans));
    }
    return result;
  }
  else if (Map.class == type) {
    ResolvableType mapType = descriptor.getResolvableType().asMap();
    Class<?> keyType = mapType.resolveGeneric(0);
    if (String.class != keyType) {
      return null;
    }
    Class<?> valueType = mapType.resolveGeneric(1);
    if (valueType == null) {
      return null;
    }
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, valueType,
        new MultiElementDescriptor(descriptor));
    if (matchingBeans.isEmpty()) {
      return null;
    }
    if (autowiredBeanNames != null) {
      autowiredBeanNames.addAll(matchingBeans.keySet());
    }
    return matchingBeans;
  }
  else {
    return null;
  }
}
```

结论
- 如果是数组，则获取数组元素类型，查找匹配该类型的所有 bean，返回一个这些 bean 的数组。
- 如果该类可赋给 Collection，并且是一个接口，则获取集合元素类型，查找匹配该类型的所有 bean，返回一个这些bean的集合。
- 如果该类型是 Map，且 key 是 String 类型，则获取 Map 的 value 的类型，查找匹配该类型的所有 bean，这是一个 key 为 bean name，value 为 bean 实例的一个Map。


Reference:
- https://refactoringguru.cn/design-patterns/strategy
- https://spring.io/
- https://www.baeldung.com/spring-injecting-collections

