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
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new Myservlet(),"/myservlet");
        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        return new FilterRegistrationBean();
    }

    @Bean
    public ServletListenerRegistrationBean servletListenerRegistrationBean(){
        return new ServletListenerRegistrationBean();
    }
}
~~~

