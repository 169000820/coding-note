# Spring 建模对象 -- BeanDefinition

## 1、为什么要有 BeanDefinition ？

>  java 里面已经有了一个类的抽象 - Class 类， 为什么还会有 BeanDefinition 呢？-- 因为 Calss 无法完成 bean 的抽象，比如说 bean 的作用域，bean 的注入模型， bean 是否是懒加载等信息。所有需要用一个类来完成 bean 的抽象。即 BeanDefinition。

下面贴一张普通类和 spring bean 的创建过程：

> ![普通对象和spring bean 的创建过程](D:\学习笔记\Java\spring源码\img\普通对象和spring bean 的创建过程.png)

spring 创建一个 bean 大致可以分为 5 步：

* 1、当 spring 容器 启动的时候回去调用 ConfigurationClassPostProcessor 这个 bean 工厂的后置处理器完成扫描（扫描前 spring 还干了许多事情，比如实例化 BeanFactory、扫描器等等）， 所谓的扫描也就是把内的信息读取到， 但是这些信息应该要存到哪里呢？比如说类的类型， 类的名字，类的构造方法等。按理说这些信息直接存在 Class 对象就可以了，比如扫描的时候 Class clazz = X.class，那么这个 clazz 就会具备刚才说的哪些信息。<span style="color: red">但是 spring 实例化一个 bean 不仅仅只需要这些类的基本信息，还需要一些额外的信息，比如 scope， lazy，dependsOn等等需要进行储存的信息。所以 spring 通过创建一个BeanDefinition 对象来保存这些信息。</span>
* 2、实例化一个 BeanDefinition 对象，然后调用这个对象的各种 set 方法存储信息，每扫描到一个**符合规则**的类，spring 都会实例化一个 BeanDefinition 对象，如果没有自定义 bean 的名字话 spring 会根据类型**自动生成** bean 的名字**（即类名首字母小写）**。-- spring 里面有一套默认的 bean 名字生成规则，但是程序员可以提供自己的 bean 名字生成器覆盖掉 spring 内置的。

* 3、然后 spring 会把这个 BeanDefinition 对象和生成的 beanName 放到一个 map 中（key=beanName, value=beanDefinition），至此，上图的 1、2、3 完成。源代码如下：

  ```java
     /**
  	 * Register the given bean definition with the given bean factory.
  	 * @param definitionHolder the bean definition including name and aliases
  	 * @param registry the bean factory to register with
  	 * @throws BeanDefinitionStoreException if registration failed
  	 */
  public static void registerBeanDefinition(
  			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
  			throws BeanDefinitionStoreException {
  
  		// Register bean definition under primary name.
  		String beanName = definitionHolder.getBeanName();
      	// 这里调用方法注册
  		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
  
  		// Register aliases for bean name, if any.
  		String[] aliases = definitionHolder.getAliases();
  		if (aliases != null) {
  			for (String alias : aliases) {
  				registry.registerAlias(beanName, alias);
  			}
  		}
  	}
  
  
  	/**
  	  * DefaultListableBeanFactory#registerBeanDefinition
  	  */
  	@Override
  	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
  			throws BeanDefinitionStoreException {
  
  		Assert.hasText(beanName, "Bean name must not be empty");
  		Assert.notNull(beanDefinition, "BeanDefinition must not be null");
  
  		if (beanDefinition instanceof AbstractBeanDefinition) {
  			try {
  				((AbstractBeanDefinition) beanDefinition).validate();
  			}
  			catch (BeanDefinitionValidationException ex) {
  				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,"Validation of bean definition failed", ex);
  			}
  		}
  
  		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
  		if (existingDefinition != null) {
  			if (!isAllowBeanDefinitionOverriding()) {
  				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
  			}
  			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
  				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
  				if (logger.isInfoEnabled()) {
  					logger.info("Overriding user-defined bean definition for bean '" + beanName +
  							"' with a framework-generated bean definition: replacing [" +
  							existingDefinition + "] with [" + beanDefinition + "]");
  				}
  			}
  			else if (!beanDefinition.equals(existingDefinition)) {
  				if (logger.isDebugEnabled()) {
  					logger.debug("Overriding bean definition for bean '" + beanName +
  							"' with a different definition: replacing [" + existingDefinition +
  							"] with [" + beanDefinition + "]");
  				}
  			}
  			else {
  				if (logger.isTraceEnabled()) {
  					logger.trace("Overriding bean definition for bean '" + beanName +
  							"' with an equivalent definition: replacing [" + existingDefinition +
  							"] with [" + beanDefinition + "]");
  				}
  			}
              // 这里放入到 beanDefinitionMap 里面
              // beanDefinitionMap 定义如下:
              // 	/** Map of bean definition objects, keyed by bean name. */
  	//private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
  			this.beanDefinitionMap.put(beanName, beanDefinition);
  		}
  		else {
  			if (hasBeanCreationStarted()) {
  				// Cannot modify startup-time collection elements anymore (for stable iteration)
  				synchronized (this.beanDefinitionMap) {
  					this.beanDefinitionMap.put(beanName, beanDefinition);
  					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
  					updatedDefinitions.addAll(this.beanDefinitionNames);
  					updatedDefinitions.add(beanName);
  					this.beanDefinitionNames = updatedDefinitions;
  					removeManualSingletonName(beanName);
  				}
  			}
  			else {
  				// Still in startup registration phase
  				this.beanDefinitionMap.put(beanName, beanDefinition);
  				this.beanDefinitionNames.add(beanName);
  				removeManualSingletonName(beanName);
  			}
  			this.frozenBeanDefinitionNames = null;
  		}
  
  		if (existingDefinition != null || containsSingleton(beanName)) {
  			resetBeanDefinition(beanName);
  		}
  		else if (isConfigurationFrozen()) {
  			clearByTypeCache();
  		}
  	}
  ```

* 4、当 spring 把类所对应的 beanDefinition 对象存到 map 之后， spring 就会调用程序员提供的 bean 工厂后置处理器。什么叫 bean 工厂后置处理器？即在 spring 的代码级别是用一个接口来表示 BeanFactoryPostProcessor， 只要实现官方提供的这个接口便是一个 bean 工厂后置处理器了。 当然， BeanFactoryPostProcessor 接口在 spring 内部也有自己的实现，比如**第一步**中的扫描功能的类 <span style="color: red">ConfigurationClassPostProcessor </span>便是也给 spring 自己实现的 bean 工厂后置处理器， 这个类是 spring 中<span style="color: red">最重要</span>的类，它完成了非常多的功能。类结构图如下：

> ![ConfigurationClassPostProcessor结构图](D:\学习笔记\Java\spring源码\img\ConfigurationClassPostProcessor结构图.png)

​       ConfigurationClassPostProcessor类实现了许多接口，在 spring 中完成 1、2、3  步的功能主要是调用 BeanFactoryPostProcessor和