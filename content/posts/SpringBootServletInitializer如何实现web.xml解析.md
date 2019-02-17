# SpringBootServletInitializer

1. 源码分析如何实现SpringBootServletInitializer整个加载过程
2. 实现自定义WebApplicationInitializer配置加载
3. 实现自定义ServletContainerInitializer配置加载  

## 示例代码

1. 首先web.xml主要配置各种servlet,filter,listener等,如常见的Log4jConfigListener,OpenSessionInViewFilter,CharacterEncodingFilter,DispatcherServlet等,此大部分信息均是容器启动时加载.
2. 在SpringBoot中我们从SpringBootServletInitializer源码入手:

```java
public abstract class SpringBootServletInitializer implements WebApplicationInitializer {
    @Override
    	public void onStartup(ServletContext servletContext) throws ServletException {
    		// Logger initialization is deferred in case a ordered
    		// LogServletContextInitializer is being used
    		this.logger = LogFactory.getLog(getClass());
    		WebApplicationContext rootAppContext = createRootApplicationContext(
    				servletContext);
    		if (rootAppContext != null) {
    			servletContext.addListener(new ContextLoaderListener(rootAppContext) {
    				@Override
    				public void contextInitialized(ServletContextEvent event) {
    					// no-op because the application context is already initialized
    				}
    			});
    		}
    		else {
    			this.logger.debug("No ContextLoaderListener registered, as "
    					+ "createRootApplicationContext() did not "
    					+ "return an application context");
    		}
    	}
}
```

可以先关注此类先实现了WebApplicationInitializer,那么实现了该接口又如何呢?

3. 下面继续关注Spring的spring-web模块源码

```java
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    @Override
    	public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
    			throws ServletException {
    
    		List<WebApplicationInitializer> initializers = new LinkedList<WebApplicationInitializer>();
    
    		if (webAppInitializerClasses != null) {
    			for (Class<?> waiClass : webAppInitializerClasses) {
    				// Be defensive: Some servlet containers provide us with invalid classes,
    				// no matter what @HandlesTypes says...
    				if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
    						WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
    					try {
    						initializers.add((WebApplicationInitializer) waiClass.newInstance());
    					}
    					catch (Throwable ex) {
    						throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
    					}
    				}
    			}
    		}
    
    		if (initializers.isEmpty()) {
    			servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
    			return;
    		}
    
    		servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
    		AnnotationAwareOrderComparator.sort(initializers);
    		for (WebApplicationInitializer initializer : initializers) {
    			initializer.onStartup(servletContext);
    		}
    	}
    
}
```

4. 继续关注3中注解部分@HandlesTypes(WebApplicationInitializer.class),其主要就是在启动容器时负责加载相关配置:

```java
public interface ServletContainerInitializer {
    public void onStartup(Set<Class<?>> c, ServletContext ctx)
            throws ServletException; 
}
```

容器启动时会自动扫描当前服务中ServletContainerInitializer的实现类,并调用其OnStartup方法,其参数Set<Class<?>> c可通过在实现类上生命注解javax.servlet.annotation.HandlesTypes(WebApplicationInitializer.class)注解自动注入,@HandlesTypes会自动扫描项目中所有的WebApplicationinitializer.class的实现类,并将其全部注入Set.  

5. 通过4中的说明可以很清楚的理解其服务启动容器加载过程配置的装在过程,在SpringServletContainerInitializer中可以发现所有WebApplicationInitializer实现类在执行onStartup方法前需要根据器注解@Order值排序,下面自定义一个WebApplicationInitializer实现类:

```java
package org.ylf.springboot.config;  
  
import javax.servlet.ServletContext;  
import javax.servlet.ServletException;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.core.annotation.Order;  
import org.springframework.web.WebApplicationInitializer;  
import org.ylf.springboot.runner.MyStartupRunner1;  
@Order(1)  
public class MyWebApplicationInitializer implements WebApplicationInitializer {  
    private Logger logger=LoggerFactory.getLogger(MyStartupRunner1.class);   
      
    @Override  
    public void onStartup(ServletContext paramServletContext) throws ServletException {  
        logger.info("启动加载自定义的MyWebApplicationInitializer");  
        System.out.println("启动加载自定义的MyWebApplicationInitializer");  
    }  
  
}  
```

注：之前有专门讲解如何装载servlet、filter、listener的注解，且可以通过两种不同的方式。那么第三种方式可以通过WebApplicationInitializer的实现类来进行装载配置。但此方式仅限部署至常规容器下生效，采用jar方式应用内置容器启动服务不加载。

6、既然可以通过自定义的WebApplicationInitializer来实现常规容器启动加载，那么我们是否可以直接自定义ServletContainerInitializer来实现启动加载配置呢：
6.1、首先编写一个待注册的servlet：
```java
package org.ylf.springboot.servlet;  
  
import java.io.IOException;  
import javax.servlet.ServletContext;  
import javax.servlet.ServletException;  
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import org.springframework.boot.web.servlet.ServletContextInitializer;  
  
public class Servlet4 extends HttpServlet   {  
    private static final long serialVersionUID = -4186518845701003231L;  
  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        System.out.println("Servlet4");  
        resp.setContentType("text/html");    
        resp.getWriter().write("Servlet4");  
    }  
      
    @Override  
    public void init() throws ServletException {  
        super.init();  
        System.out.println("Servlet4 loadOnStart");  
    }  
  
}  
```

6.2、编写实现ServletContainerInitializer的自定义实现类：
```java
package org.ylf.springboot.config;  
  
import java.util.Set;  
import javax.servlet.ServletContainerInitializer;  
import javax.servlet.ServletContext;  
import javax.servlet.ServletException;  
import javax.servlet.ServletRegistration;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
  
public class MyServletContainerInitializer implements ServletContainerInitializer {  
    private Logger logger=LoggerFactory.getLogger(MyServletContainerInitializer.class);   
      
    @Override  
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {  
        logger.info("启动加载自定义的MyServletContainerInitializer");  
        System.out.println("启动加载自定义的MyServletContainerInitializer");  
        ServletRegistration.Dynamic testServlet=servletContext.addServlet("servlet4","org.ylf.springboot.servlet.Servlet4");  
        testServlet.setLoadOnStartup(1);  
        testServlet.addMapping("/servlet4");  
    }  
}  
```

6.3、对新增的servlet设置其请求路径，同时打成WAR包部署至tomcat启动服务，但请求http://localhost:8080/SpringBoot1/servlet4却失败，此时发现需要了解servlet3对于ServletContainerInitializer
 的加载机制是如何的，在官方有类似这样的描述“该接口的实现必须声明一个JAR资源放到程序中的META-INF/services下，并且记有该接口实现类的全路径，才会被运行时(server)的查找机制或是其它特定机制找到”。那么我们先参考spring-web-4.3.2.RELEASE.jar中


我们可以将自定义的类配置到一个jar包下部署至WEB-INF\lib目录下，
6.3.1、首先新建如下目录结构的文件并填写内容如下：


6.3.2、然后通过如下命令生成jar包


6.3.3、将myTest.jar放置WEB-INF\lib目录下重启服务，再次请求：



注：与5中实现WebApplicationInitializer一样，该方式仅限于部署常规容器生效。故jar通过内置容器启动的服务无法加载servlet4配置。