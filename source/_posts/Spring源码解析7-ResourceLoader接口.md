---
title: Spring源码解析7-ResourceLoader接口
date: 2019-09-03 09:35:00
tags: [java,spring]
categories: spring
---

#1. ResourceLoader 接口定义

ResourceLoader用于定义资源加载器，主要应用于根据给定的资源文件地址返回对应的Resource。

```java

public interface ResourceLoader {

	//用于从类路径加载的伪URL前缀: "classpath:"
	/** Pseudo URL prefix for loading from the class path: "classpath:". */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;


	/**
	 * 返回指定资源位置的Resource
	 */
	Resource getResource(String location);

	@Nullable
	ClassLoader getClassLoader();

}
```



#2. DefaultResourceLoader实现类

DefaultResourceLoader#getResource方法

```java
	@Override
	public Resource getResource(String location) {
		Assert.notNull(location, "Location must not be null");

		// 通过  用户自定义的指定协议解析器 来 解析
		for (ProtocolResolver protocolResolver : this.protocolResolvers) {
			Resource resource = protocolResolver.resolve(location, this);
			if (resource != null) {
				return resource;
			}
		}

		if (location.startsWith("/")) {
			// return new ClassPathContextResource(path, getClassLoader());
			return getResourceByPath(location);
		}
		else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
			// 类路径加载、通过ClassPathResource进行封装
			return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
		}
		else {
			try {
				// Try to parse the location as a URL...
				// 尝试将location当做一个URL来解析
				URL url = new URL(location);
				return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
			}
			catch (MalformedURLException ex) {
				// No URL -> resolve as resource path.
				return getResourceByPath(location);
			}
		}
	}
```

Spring的配置文件读取是通过ClassPathResource进行封装的。



## # 3. Java和Spring中对资源的抽象

在Java中，将不同来源的资源抽象成URL，通过注册不同的handler来处理不同来源的资源的读取逻辑，一般handler的类型使用不同前缀（协议，Protocol）来识别，例如：“file:”、“http:”、“jar:”等，然后URL没有默认定义相对Classpath或ServletContext等资源的handler。然后这需要了解URL的实现机制，而且URL也没有提供一些基本的方法，例如检查当前资源是否存在，检查当前资源是否可读等方法。

因为Spring对齐内部使用的资源实现了自己的抽象结构：Resource接口来封装底层资源。



InputStreamSource接口封装任何能返回InputStream的类。比如File、Classpath下的资源和Byte、Array等。

```java
public interface InputStreamSource {
	InputStream getInputStream() throws IOException;
}
```

Resource接口抽象了所有Spring内部使用的底层资源：File、URL、Classpath等

```java
public interface Resource extends InputStreamSource {

	boolean exists();

	default boolean isReadable() {
		return exists();
	}

	default boolean isOpen() {
		return false;
	}

	default boolean isFile() {
		return false;
	}

	URL getURL() throws IOException;

	URI getURI() throws IOException;

	File getFile() throws IOException;

	default ReadableByteChannel readableChannel() throws IOException {
		return Channels.newChannel(getInputStream());
	}

	long contentLength() throws IOException;

	long lastModified() throws IOException;

	Resource createRelative(String relativePath) throws IOException;

	@Nullable
	String getFilename();
  
	String getDescription();

}
```



# 4. 不同的Resource实现

对不同来源的资源文件都有相应的Resource实现

* 文件（FileSystemResource）
* Classpath资源（ClassPathResource）
* URL资源（URLResource）
* InputStream资源（InputStreamResource）
* Byte数组（ByteArrayResource）