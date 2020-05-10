# springboot笔记

## springboot自动配置

- ### springboot中自动配置的条件化注解

![springboot中自动配置的条件化注解](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200509234354028.png)

- ### springboot自动配置流程

  1）springboot启动类

  ![image-20200509235606712](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200509235606712.png)

  2）进入@springbootApplication注解

  ![image-20200509235644785](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200509235644785.png)

  3)  进入@EnableAutoConfiguration,发现@Import导入了AutoConfigurationImportSelector类

  ![image-20200510000136500](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510000136500.png)

  4)  找到 selectImports 和 getAutoConfigurationEntry方法

  ```java
  @Override
  	public String[] selectImports(AnnotationMetadata annotationMetadata) {
  		if (!isEnabled(annotationMetadata)) {
  			return NO_IMPORTS;
  		}
  		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
  				.loadMetadata(this.beanClassLoader);
  		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
  				annotationMetadata);
  		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
  	}
  
  	/**
  	 * Return the {@link AutoConfigurationEntry} based on the {@link AnnotationMetadata}
  	 * of the importing {@link Configuration @Configuration} class.
  	 * @param autoConfigurationMetadata the auto-configuration metadata
  	 * @param annotationMetadata the annotation metadata of the configuration class
  	 * @return the auto-configurations that should be imported
  	 */
  	protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
  			AnnotationMetadata annotationMetadata) {
  		if (!isEnabled(annotationMetadata)) {
  			return EMPTY_ENTRY;
  		}
  		AnnotationAttributes attributes = getAttributes(annotationMetadata);
  		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
  		configurations = removeDuplicates(configurations);
  		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
  		checkExcludedClasses(configurations, exclusions);
  		configurations.removeAll(exclusions);
  		configurations = filter(configurations, autoConfigurationMetadata);
  		fireAutoConfigurationImportEvents(configurations, exclusions);
  		return new AutoConfigurationEntry(configurations, exclusions);
  	}
  ```

   5) .debugger时直接进入了getAutoConfigurationEntry方法

  ![image-20200510002837866](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510002837866.png)

    6) .获取configurations,进入getCandidateConfigurations方法

  ![image-20200510003152026](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510003152026.png)

    7) .进入loadFactoryNames方法

  ![image-20200510004423890](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510004423890.png)                   

   8) .进入loadSpringFactories方法，通过类加载器获取META-INF/spring.factories路径,PropertiesLoaderUtils把对应路径下的文件转成properties文件，将factoryTypeName作为key，该写properties文件值转换成list作为vlue存入map，并且将map存入缓存中

  ![image-20200510004730017](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510004730017.png)

    9) .返回到loadFactoryNames，通过factoryTypeName(ps:值为org.springframework.boot.autoconfigure.EnableAutoConfiguration),读取需要加载的配置，配置为对应需要自动加载的全类名

  ![image-20200510011022700](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510011022700.png)

![image-20200510011149980](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510011149980.png)

- ### springboot starter配置原理

> 以spring-boot-starter-web为例

在springboot自动配置时,会加载spring.factories文件,文件中会有写出starter自动配置会加载的具体类

![image-20200510013324385](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510013324385.png)

spring-boot-starter-web会自动加载这几个类，比如配置org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration这个类。

![image-20200510013511240](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/image-20200510013511240.png)

头上的注解表明是该加载类所必须的条件(spring自动配置注解开头已经贴图自己对应比较),只有所有条件成立该类才会进行自动配置。@ConditionalOnProperty可以在配置文件中进行配置,该配置都有默认值，如果没有自己配置那就会加载默认配置

### 配置profile

- #### javabean配置

  在java bean中配置可以通过@Profile注解来指定该个bean为哪个profile

  ![image-20200510175543916](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/learnMD/img/image-20200510175543916.png)

  > 缺点是有些bean需要在特定版本下注入，如果忘记添加注解会导致所有环境都对该bean进入注入

* #### 外部配置文件配置

  ![image-20200510180953126](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/learnMD/img/image-20200510180953126.png)

  properties方式来配置版本

  ​	![image-20200510181256550](https://raw.githubusercontent.com/zhuzhixu/learnMD/master/img/learnMD/img/image-20200510181256550.png)

  yml方式来配置版本

  