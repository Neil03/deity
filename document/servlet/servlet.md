# 1. web.xml

## 1.1 加载顺序

1. context-param
2. listener
3. filter
4. servlet

## 1.2 加载过程

* 当启动一个web项目时，容器首先会读取项目web.xml配置文件里的配置，当这一步没有出错并且完成之后，项目才能正常的被启动起来

1. 启动web项目的时候，容器首先会去他的配置文件web.xml中读取两个节点：<listener></listener>和<context-param></context-param>
2. 紧接着，容器创建一个ServletContext(application)，这个web项目所有部分都共享这个上下文
3. 容器以<context-param></context-param>的name为键，value为值，将其转化为键值对，存入ServletContext
4. 容器创建<listener></listener>中类的实例，根据配置的class类路径<listener-class>来创建监听，在监听中会有contextInitialized(ServletContextEvent args)初始化方法，启动web应用时，系统调用Listener的该方法，在这个方法中获得context-param的值，就是application.getInitParameter("context-param的键")。得到这个context-param的值之后，就可以做一些操作了
5. 举例：你可能想在项目启动之前就打开数据库，那么这里就可以在<context-param>中设置数据库的连接方式，在监听类中初始化数据库的连接。这个监听时自己写的一个类，除了初始化方法，他还有销毁方法，用于关闭应用前释放资源。比如数据库连接的关闭，此时调用contextDestroyed(ServletContextEvent args)方法，关闭web应用时，系统调用Listener的该方法
6. 接着，容器会读取<filter></filter>根据指定的类路径来实例化过滤器
7. 以上都是在web项目还没有完全启动起来的时候就已经完成了的工作。如果系统中有servlet，则servlet实在第一次发起请求时被实例化的，而且一般不会被容器销毁，它可以服务于多个用户的请求，所以servlet的初始化比上面几个要迟(servlet初始化过程中的两个map)
8. 如果web.xml中出现了相同的元素，则按照在配置文件中出现的先后顺序来加载
9. 对于某类元素而言，与他们出现的顺序是有关的。以<filter>为例，web.xml中可以定义多个<filter>，与<filter>相关的一个元素是<filter-mapping>，注意，对于拥有相同<filter-name>的<filter>和<filter-mapping>元素而言，<filter-mapping>必须出现在<filter>之后，否则当解析到<filter-mapping>时，它所对应的<filter-name>还未定义。web容器启动初始化每个<filter>时，按照<filter>出现的顺序来初始化的，当请求资源匹配多个<filter-mapping>时，<filter>拦截资源时按照<filter-mapping>元素出现的顺序来一次调用doFilter()方法的。<servlet>同<filter>类似

## 1.3 web.xml标签详解

### 1.3.1 xml文档有效性检查

### 1.3.2 <web-app></web-app>

### 1.3.3 <display-name></display-name>

### 1.3.4 <distributable/>

### 1.3.5 <context-param></context-param>

1. <context-param>解释

   * <context-param>元素包含有一对参数名和参数值，用作应用Servlet上下文初始化参数，参数名在整个web应用中必须唯一，在web应用的整个生命周期中上下文初始化参数都存在，任意的servlet和jsp都可以随时对地的访问它。<param-name>子元素包含有参数名，而<param-value>子元素包含的是参数值。作为选择，可用<description>子元素来描述参数

2. 什么情况下使用，为什么使用<context-param>

3. spring配置文件

   * 配置spring，必须要<listener>，而<context-param>可有可无，如果在web.xml中不写<context-param>配置信息，默认路径是/WEB-INF/applicationcontext.xml，在WEB-INF目录下创建的xml文件的名称必须是applicationContext.xml。若谷哦是需要自定义文件名可以子啊web.xml里加入contextConfigLoaction这个context参数，在<param-value></param-value>中指定相应的xml文件名，如果有多个xml文件，可以写在一起，并以“，”分隔。

   * 例如：

     ```xml
     <!--spring config-->
     <context-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>/WEB-INF/spring-configuration/*.xml</param-value>
     </context-param>
     
     <listener>
         <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
     </listener>
     ```

4. 部署在同一个容器中的多个web项目，要配置不同的webAppRootKey，web.xml中最好定义webAppRootKey参数，如果不定义，将会雀神为webapp.root
5. 多个配置文件交叉引用处理
   * 如果web.xml中有contextConfigLocation参数指向指定的spring配置文件，则会去加载响应的配置文件，而不会去加载/WEB-INF/下的applicationContext.xml。但是如果没有指定的话，默认会去加载applicationContext.xml

### 1.3.6 <session-config></session-config>

* <session-config>用来设置容器中session参数

### 1.3.7 <listener></listener>

1. aga 

#### 1.3.7.1 listener介绍

* <listener>为web应用程序定义监听器，监听器用来监听各种事件，比如：application和session事件，所有监听器按照相同的方式定义，功能取决于去他们各自实现的接口，常用的web事件接口有如下几个：
  1. ServletContextListener：用来监听web应用程序的启动和关闭
  2. ServletContextAttributeListener：用于监听ServletContext范围(application)内属性的改变
  3. ServletRequestListener：用于监听用户的请求
  4. ServletRequestAttributeListener：用于监听ServletRequest(request)内属性的改变
  5. HttpSessionListener：用于监听用户session的开始和结束
  6. HttpSessionAttributeListner：用于监听HttpSession范围(session)内属性的改变
* <listener>主要用于监听web应用事件，其中有两个个比较重要的web应用事件：应用的启动和停止，session的创建和失效。应用启动事件发生在应用第一次被servlet容器装载和启动的时候，停止事件发生在web应用停止的时候。session创建事件发生在每次一个新的session被创建的时候，session失效事件发生在每次一个session失效的时候。为了使用这些web应用事件做些有用的事情，我们必须创建和使用一些特殊的监听类。他们使实现以下两个接口中任何一个接口的简单java类，javax.servlet.ServletContextListener或javax.servlet.http.HttpSessionListener。如果想要监听类监听应用的启动和停止，就必须要实现ServletContextListener接口，如果想去监听session的创建和失效事件，那就必须得实现HttpSessionListener接口

#### 1.3.7.2 listener配置

* 配置listener只要向web应用注册listener实现类即可，无需配置参数之类的东西，因为Listener获取的使web应用ServletContext(application)的配置参数。为web应用配置listener的两种方式：

  1. 使用@WebListener修饰listener实现类

  2. 在web.xml中使用<listener>进行配置

     例如：

     ```xml
     <listener>
         <listener-class>org.springframework.web.context.ContextLoaderListenre</listener-class>
     </listener>
     ```

     

     * 这里的<listener>用于spring的加载，spring加载可以利用ServletContextListener实现，也可以采用 load-on-startup Servlet实现，但是当<filter>需要使用到bean时，由于加载顺序是先加载filter后加载servlet所以filter中初始化操作中的bean为null。所以如果过滤器中要使用到bean，此时就可以根据加载顺序listener-filter-servlet，将spring的加载改成listener的方式

       1. servlet加载

          ```XML
          <servlet>
              <servlet-name>context</servlet-name>
              <servlet-class>org.springframework.web.context.ContextLoaderServlet</servlet-class>
              <load-on-startup>1</load-on-startup>
          </servlet>
          ```

       2. listener加载

          ```xml
          <listener>
              <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
          </listener>
          ```

### 1.3.8 <filter></filter>

#### 1.3.8.1 filter介绍

#### 1.3.8.2 filter配置

### 1.3.9 <servlet></servlet>

#### 1.3.9.1 servlet介绍

#### 1.3.9.2 servlet配置

### 1.3.10 <welcome-file-list></welcome-file-list>

# 2 servlet内置对象与作用域

## 2.1 内置对象

### 2.1.1 request

### 2.1.2 response

### 2.1.3 session

### 2.1.4 application

### 2.1.5 out

### 2.1.6 pageContext

### 2.1.7 config

### 2.1.8 page

### 2.1.9 exception

## 2.2 作用域

### 2.2.1 request

### 2.2.2 session

### 2.2.3 application

### 2.2.4 page

# 3 spring父子容器

## 3.1 spring容器启动过程

## 3.2 springmvc容器启动过程

## 3.3 spring父子容器

