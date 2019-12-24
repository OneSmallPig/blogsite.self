

## SpringBoot对容器的定制和使用

### 1.容器的配置

直接在配置文件中配置

~~~properties
server.port=8080
#tomcat配置项等
server.tomcat.host-header=''
~~~

通过构造工厂类配置

~~~java
	@Bean
    public ConfigurableServletWebServerFactory configurableServletWebServerFactory(){
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.addConnectorCustomizers(new TomcatConnectorCustomizer() {
            @Override
            public void customize(Connector connector) {
                //配置容器参数
                connector.setPort(1080);
            }
        });
        return factory;
    }
~~~

SpringBoot中会有很多xxxCustomizers用来做定制化服务

### 2.对容器组件的注册

注册示例如下：

~~~java
@Configuration
public class MyServerConfig {

    @Bean
    public ServletRegistrationBean servletRegistrationBean(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new Myservlet(),"/myServlet");
        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(new MyFilter());
        registrationBean.setUrlPatterns(Arrays.asList("/server","/myServlet"));
        return registrationBean;
    }

    @Bean
    public ServletListenerRegistrationBean servletListenerRegistrationBean(){
        ServletListenerRegistrationBean<MyLinstener> listenerRegistrationBean = new ServletListenerRegistrationBean<>(new MyLinstener());
        return listenerRegistrationBean;
    }

}
~~~

SpringBoot注册SpringMVC的时候，自动注册SpringMVC前端控制器：dispatcherServlet

~~~java
@Bean(name = {"dispatcherServletRegistration"})
@ConditionalOnBean(value = {DispatcherServlet.class},name = {"dispatcherServlet"})
public DispatcherServletRegistrationBean dispatcherServletRegistration(DispatcherServlet dispatcherServlet, WebMvcProperties webMvcProperties, ObjectProvider<MultipartConfigElement> multipartConfig) {
    		//获取默认的拦截路径getPath()
            DispatcherServletRegistrationBean registration = new DispatcherServletRegistrationBean(dispatcherServlet, webMvcProperties.getServlet().getPath());
            registration.setName("dispatcherServlet");
    	 registration.setLoadOnStartup(webMvcProperties.getServlet().getLoadOnStartup());
            multipartConfig.ifAvailable(registration::setMultipartConfig);
            return registration;
        }


public static class Servlet {
    	//"/"指所有请求，包括静态资源，但是不拦截jsp请求
    	//"/*"包括jsp请求也会被拦截
        private String path = "/";
        private int loadOnStartup = -1;

        public Servlet() {
        }

        public String getPath() {
            return this.path;
        }
~~~

通过源码可以了解到

~~~properties
#可以通过修改配置文件来修改SpringMVC前端控制器的默认拦截请求路径
#Path of dispatcher servlet
spring.mvc.servlet.path='/*'
~~~

### 3.其他嵌入式容器

默认使用tomcat容器

~~~xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
~~~

#### （1）Jetty（长连接）

#### （2）Undertow（不支持JSP）

![tomcat源码中关于依赖剔除的部分](..\image\SpringMVC中关于tomcat的依赖树.png)

可以通过替换tomcat 的组件来修改容器的使用

~~~xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <groupId>org.springframework.boot</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <artifactId>spring-boot-starter-jetty</artifactId>
  <groupId>org.springframework.boot</groupId>
</dependency>
~~~

### 4.SpringMVC对其他嵌入式容器转换的实现

以对于Tomcat容器的注册为例

在EmbeddedWebServerFactoryCustomizerAutoConfiguration类中存在：

~~~java
@Configuration(
        proxyBeanMethods = false
    )
//判断当前是否引入了Tomcat依赖
    @ConditionalOnClass({Tomcat.class, UpgradeProtocol.class})
    public static class TomcatWebServerFactoryCustomizerConfiguration {
        public TomcatWebServerFactoryCustomizerConfiguration() {
        }

        @Bean
        public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
            return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
        }
    }
~~~

### 5.嵌入式容器启动原理

#### （1）SpringBoot启动

~~~java
    public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }

    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
~~~

#### （2）this.refreshContext(context)

this.refreshContext(IOC容器);

SpringBoot刷新IOC容器对象并进行初始化，创建容器中的每一个组件

~~~java
    public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
        this.configureHeadlessProperty();
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting();

        Collection exceptionReporters;
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            context = this.createApplicationContext();
            exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }
~~~

#### （3）refresh方法

~~~java
public void refresh() throws BeansException, IllegalStateException {
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
                this.registerListeners();
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var9) {
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
                }

                this.destroyBeans();
                this.cancelRefresh(var9);
                throw var9;
            } finally {
                this.resetCommonCaches();
            }

        }
    }
~~~

