在阅读Spring Boot源码时，看到Spring Boot中大量使用ImportBeanDefinitionRegistrar来实现Bean的动态注入。它是Spring中一个强大的扩展接口。本篇文章来讲讲它相关使用。

## Spring Boot中的使用

在Spring Boot 内置容器的相关自动配置中有一个ServletWebServerFactoryAutoConfiguration类。该类的部分代码如下：

```java
@Configuration(proxyBeanMethods = false)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {

	// ...
	
	/**
	 * Registers a {@link WebServerFactoryCustomizerBeanPostProcessor}. Registered via
	 * {@link ImportBeanDefinitionRegistrar} for early registration.
	 */
	public static class BeanPostProcessorsRegistrar implements ImportBeanDefinitionRegistrar, BeanFactoryAware {

		private ConfigurableListableBeanFactory beanFactory;

		// 实现BeanFactoryAware的方法，设置BeanFactory
		@Override
		public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
			if (beanFactory instanceof ConfigurableListableBeanFactory) {
				this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
			}
		}

		// 注册一个WebServerFactoryCustomizerBeanPostProcessor
		@Override
		public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
				BeanDefinitionRegistry registry) {
			if (this.beanFactory == null) {
				return;
			}
			registerSyntheticBeanIfMissing(registry, "webServerFactoryCustomizerBeanPostProcessor",
					WebServerFactoryCustomizerBeanPostProcessor.class);
			registerSyntheticBeanIfMissing(registry, "errorPageRegistrarBeanPostProcessor",
					ErrorPageRegistrarBeanPostProcessor.class);
		}

		// 检查并注册Bean
		private void registerSyntheticBeanIfMissing(BeanDefinitionRegistry registry, String name, Class<?> beanClass) {
			// 检查指定类型的Bean name数组是否存在，如果不存在则创建Bean并注入到容器中
			if (ObjectUtils.isEmpty(this.beanFactory.getBeanNamesForType(beanClass, true, false))) {
				RootBeanDefinition beanDefinition = new RootBeanDefinition(beanClass);
				beanDefinition.setSynthetic(true);
				registry.registerBeanDefinition(name, beanDefinition);
			}
		}
	}
}
```

在这个自动配置类中，基本上展示了ImportBeanDefinitionRegistrar最核心的用法。这里该接口主要用来注册BeanDefinition。

BeanPostProcessorsRegistrar实现了ImportBeanDefinitionRegistrar接口和BeanFactoryAware接口。其中BeanFactoryAware接口的实现是用来暴露Spring的ConfigurableListableBeanFactory对象。

而实现registerBeanDefinitions方法则是用来对Bean的动态注入，这里注入了WebServerFactoryCustomizerBeanPostProcessor和ErrorPageRegistrarBeanPostProcessor。

简单了解了Spring Boot中的一个使用实例，下面我们总结一下使用方法，并自己实现一个类似的功能。

## ImportBeanDefinitionRegistrar使用

Spring官方通过ImportBeanDefinitionRegistrar实现了@Component、@Service等注解的动态注入机制。

很多三方框架集成Spring的时候，都会通过该接口，实现扫描指定的类，然后注册到spring容器中。 比如Mybatis中的Mapper接口，springCloud中的FeignClient接口，都是通过该接口实现的自定义注册逻辑。

所有实现了该接口的类的都会被ConfigurationClassPostProcessor处理，ConfigurationClassPostProcessor实现了BeanFactoryPostProcessor接口，所以ImportBeanDefinitionRegistrar中动态注册的bean是优先于依赖其的bean初始化，也能被aop、validator等机制处理。

基本步骤：

- 实现ImportBeanDefinitionRegistrar接口；
- 通过registerBeanDefinitions实现具体的类初始化；
- 在@Configuration注解的配置类上使用@Import导入实现类；

## 简单示例

这里实现一个非常简单的操作，自定义一个@Mapper注解（并非Mybatis中的Mapper实现），实现类似@Component的功能，添加了@Mapper注解的类会被自动加载到spring容器中。

首先创建@Mapper注解。

```java
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
public @interface Mapper {
}
```

创建UserMapper类，用于使用@Mapper注。

```java
@Mapper
public class UserMapper {
}
```

定义ImportBeanDefinitionRegistrar的实现类MapperAutoConfigureRegistrar。如果需要获取Spring中的一些数据，可实现一些Aware接口，这实现了ResourceLoaderAware。

```java
public class MapperAutoConfigureRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware {
	
	private ResourceLoader resourceLoader;

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		MapperBeanDefinitionScanner scanner = new MapperBeanDefinitionScanner(registry, false);
		scanner.setResourceLoader(resourceLoader);
		scanner.registerFilters();
		scanner.addIncludeFilter(new AnnotationTypeFilter(Mapper.class));
		scanner.doScan("com.secbro2.learn.mapper");
	}

	@Override
	public void setResourceLoader(ResourceLoader resourceLoader) {
		this.resourceLoader = resourceLoader;
	}
}
```

在上面代码中，通过ResourceLoaderAware接口的setResourceLoader方法获得到了ResourceLoader对象。

在registerBeanDefinitions方法中，借助ClassPathBeanDefinitionScanner类的实现类来扫描获取需要注册的Bean。

MapperBeanDefinitionScanner的实现如下：

```java
public class MapperBeanDefinitionScanner extends ClassPathBeanDefinitionScanner {

	public MapperBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters) {
		super(registry, useDefaultFilters);
	}

	protected void registerFilters() {
		addIncludeFilter(new AnnotationTypeFilter(Mapper.class));
	}

	@Override
	protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
		return super.doScan(basePackages);
	}
}
```

MapperBeanDefinitionScanner继承子ClassPathBeanDefinitionScanner，扫描被@Mapper的注解的类。

在MapperBeanDefinitionScanner中指定了addIncludeFilter方法的参数为包含Mapper的AnnotationTypeFilter。

当然也可以通过excludeFilters指定不加载的类型。这两个方法由它们的父类ClassPathScanningCandidateComponentProvider提供的。

完成了上面的定义，则进行最后一步引入操作了。创建一个自动配置类MapperAutoConfig，并通过@Import引入自定义的Registrar。

```java
@Configuration
@Import(MapperAutoConfigureRegistrar.class)
public class MapperAutoConfig {
}
```

至此，整个代码的功能已经编写完成，下面写一个单元测试。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MapperAutoConfigureRegistrarTest {

	@Autowired
	UserMapper userMapper;

	@Test
	public void contextLoads() {
		System.out.println(userMapper.getClass());
	}
}
```

执行单元测试代码，会发现打印如下日志：

```java
class com.secbro2.learn.mapper.UserMapper
```

当然，这里的UserMapper并不接口，这里的实现也并不是Mybatis中的实现形式。只是为了演示该功能的简单示例。需要注意的是文中提到了两种实现的实例，第一种是Spring Boot中的实现，第二种是我们的Mapper实例。展现了两种不同方法的注册的操作，但整个使用流程是一致的，读者注意仔细品味，并在此基础上进行拓展更复杂的功能。