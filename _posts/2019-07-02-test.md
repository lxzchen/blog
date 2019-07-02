---
layout: post
title: 记一次SpringBoot自定义多配置文件问题查找
date: 2019-07-02
Author: 态度大师
categories: 
tags: [SpringBoot, 自定义多配置]
comments: true
---

### 背景：
SpringBoot项目，由于项目过程中，引入的外部配置比较多，造成配置文件臃肿，管理混乱，简直就像把一辆车开在了烂泥潭里，简直无法自拔。于是决定将配置文件归类，并按业务拆分成几个，提高清晰度，方便维护。

### 过程：
SpringBoot允许自定义配置的方式将配置文件在项目启动的时候进行加载。方式有几种，主要包括通过@PropertySource、EnviromentPostProcessor继承，或者通过@Configuration等。每种方法虽然都是一个目的，但还是有很大差别，具体先占坑，以后来详细谈谈，今天来谈谈EnviromentPostProcessor这种方法（谁让他比较长呢）。

废话勿多，直接上代码。根据官方文档的使用方法，先继承EnviromentPostProcessor，重写方法postProcessEnviroment。下面是官方例子，稍作修改，增加多个配置文件加载支持。（大概这个意思）
```
public class EnvironmentPostProcessorExample implements EnvironmentPostProcessor {

	private final YamlPropertySourceLoader loader = new YamlPropertySourceLoader();

	@Override
	public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        // 资源文件A
		Resource path = new ClassPathResource("com/example/myapp/config-A.yml");
		PropertySource<?> propertySource = loadYaml(path);
		environment.getPropertySources().addLast(propertySource);
        // 资源文件B 
        path = new ClassPathResource("com/example/myapp/config-B.yml");
		propertySource = loadYaml(path);
		environment.getPropertySources().addLast(propertySource);
	}

	private PropertySource<?> loadYaml(Resource path) {
		if (!path.exists()) {
			throw new IllegalArgumentException("Resource " + path + " does not exist");
		}
		try {
			return this.loader.load("custom-resource", path).get(0);
		}
		catch (IOException ex) {
			throw new IllegalStateException("Failed to load yaml configuration from " + path, ex);
		}
	}
}
```
在META-INF下增加spring.factories文件，内容如下:
```
org.springframework.boot.env.EnvironmentPostProcessor=对应自定义EnviromentPostProcess类全路径
```
看来一切视乎很顺利，来个单元测试看是否可以加载相关配置项。但事实告诉我们，往往开头越顺利，后面就有越大的坑等你填。
启动项目，加载......果然，报错：Could not resolve placeholder .......。看错误日志，果然没有啥有用信息。

What！！！尽然没有找个配置项，第一直觉是哪里配置错误，反复检查，反复看官方例子，没错。
为啥没有加载到配置项内？再仔细看配置文件，感觉除最后一个配置文件加载到外，其他配置文件都似乎没加载。

# 这就很奇怪了呀！！！

为了验证自己的想法，我把几个拆分开的配置文件顺序颠倒，再次启动，果然，虽然还是报统一个错，但提示未加载的配置项换了。那估计就是这个问题没跑了。那问题来了，难道ConfigurableEnvironment对象，加载多个配置文件所用的方法并非官方的addLast()还是这个方法别有奥妙？还是看源码吧。果然，这一看，发真现了问题。源码如下：
```
public void addLast(PropertySource<?> propertySource) {
    if (logger.isDebugEnabled()) {
		logger.debug("Adding PropertySource '" + propertySource.getName() + "' with lowest search precedence");
	}
	removeIfPresent(propertySource);
	this.propertySourceList.add(propertySource);
}
```
removeIfPresent，根据传入的PropertySource对象，如果存在，先删除。那判断相同的依据是什么呢？继续往下翻，源码如下：
```
private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<>();

protected void removeIfPresent(PropertySource<?> propertySource) {
	this.propertySourceList.remove(propertySource);
}
```
到这里就明朗了，维护PropertySource的是一个List，删除使用的是自带remove方法。根据List接口的remove方法约定，需要比较两个对象内存地址是否一致、equals()方法是否相等、等等。但我要找的，就是equals()方法。

查找PropertySource对象的equals()方法，代码如下：
```
@Override
public boolean equals(Object other) {
	return (this == other || (other instanceof PropertySource &&
			ObjectUtils.nullSafeEquals(this.name, ((PropertySource<?>) other).name)));
}
```
看到这段代码，OK，托拉，问题应该找到了。原来在重写equals()方法的时候，有一项内容就是比较name属性。而name属性，恰巧就是YamlPropertySourceLoader 对象在load资源的时候，需要定义的名字。(Spring为什么这么做？这部分源码，留待大家自己去分析查看吧)

问题找到，修改代码，根据自定义资源名称的不同，加载相关的资源。
启动，一切顺利。

### 总结
虽然这是一个非常小的问题，但其实也反映了对接口文档的不熟悉，或者说想当然，甚至压根不会去考虑。

以此文章，共勉之。