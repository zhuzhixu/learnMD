# springboot笔记

## springboot自动配置

- ### springboot中自动配置的条件化注解

![springboot中自动配置的条件化注解](..\..\learnMD\img\image-20200509234354028.png)

- ### springboot自动配置流程

  1）springboot启动类

  ![image-20200509235606712](..\..\learnMD\img\image-20200509235606712.png)

  2）进入@springbootApplication注解

  ![image-20200509235644785](..\..\learnMD\img\image-20200509235644785.png)

  3)  进入@EnableAutoConfiguration,发现@Import导入了AutoConfigurationImportSelector类

  ![image-20200510000136500](..\..\learnMD\img\image-20200510000136500.png)

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

    1.debugger时直接进入了getAutoConfigurationEntry方法

  ![image-20200510002837866](..\..\learnMD\img\image-20200510002837866.png)

    2.获取configurations,进入getCandidateConfigurations方法

  ![image-20200510003152026](..\..\learnMD\img\image-20200510003152026.png)

    3.进入loadFactoryNames方法

  ![image-20200510004423890](..\..\learnMD\img\image-20200510004423890.png)                   

    4.进入loadSpringFactories方法，通过类加载器获取META-INF/spring.factories路径,PropertiesLoaderUtils把对应路径下的文件转成properties文件，将factoryTypeName作为key，该写properties文件值转换成list作为vlue存入map，并且将map存入缓存中

  ![image-20200510004730017](..\..\learnMD\img\image-20200510004730017.png)

    5.返回到loadFactoryNames，通过factoryTypeName(ps:值为org.springframework.boot.autoconfigure.EnableAutoConfiguration),读取需要加载的配置，配置为对应需要自动加载的全类名

  ![image-20200510011022700](..\..\learnMD\img\image-20200510011022700.png)

![image-20200510011149980](..\..\learnMD\img\image-20200510011149980.png)