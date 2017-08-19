## 1.IDE说明和依赖管理工具说明

  开发工具：intellij idea  
  依赖管理使用：maven

## 2.创建maven工程

创建新的maven工程，并添加相应的文件夹，创建好的项目目录如下所示:  
![](https://github.com/VitasYuan/Blog/blob/master/pictures/springmvc-1-1.png)

## 3.添加SpringMVC相关依赖

在pom.xml中添加SpringMVC所需依赖。将以下代码添加到pom.xml中：

      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>4.3.10.RELEASE</version>
      </dependency>
在springmvc的依赖中内部依赖了Spring-bean，Spring-core等依赖，所以不需要额外添加其他Spring依赖包。具体依赖关系可以使用  

    mvn dependency:analyze  
    mvn dependency:tree

命令来分析依赖关系。

## 4.在web.xml中添加Context加载监听器和访问路由相关配置

在web.xml中添加以下配置代码：

    <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath*:applicationContext.xml</param-value>
    </context-param>
    //用于加载spring配置文件的监听器
    <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
       <servlet-name>applicationContext</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
       <servlet-name>applicationContext</servlet-name>
       <url-pattern>*.htm</url-pattern>
    </servlet-mapping>

第一个监听器用于加载spring配置文件，其中context-param中指定了配置文件的路径。
applicationContext中指定了所有.htm结尾的访问都使用springMVC来进行路由分发。

## 5.添加Spring和SpringMVC配置文件

1.在resource文件夹中添加applicationContext.xml配置文件，并添加以下配置：

    <context:annotation-config/>// 开启spring注解配置
    <mvc:annotation-driven></mvc:annotation-driven>//开启mvc注解配置，用于识别controller等注解
    <context:component-scan base-package="com.springmvc.study.*"></context:component-scan>// 配置扫描的包

2.在WEB_INF文件夹下添加applicationContext-servlet.xml配置文件，此文件名命名规则为：第三步骤中添加的applicationContext名称-servlet。
此文件中可以不添加内容，但文件是必须的，否则启动时会报file not found错误

# 6.添加Controller文件
在src/main/java文件下新建java类，如：HelloWorldController，并添加以下代码：

    @Controller
    @RequestMapping(value = "/first")
    public class HelloWorldController {

        @ResponseBody
        @RequestMapping(value = "/helloworld")
        public String helloWorld(){
            return "hello world!";
        }
    }

所有文件添加完成后项目目录截图如下所示：

![](https://github.com/VitasYuan/Blog/blob/master/pictures/springmvc-1-2.png)

# 7.测试
在tomcat中启动此项目后，在浏览器中输入：

    http://localhost:8083/first/helloworld.htm

可看到输出的hello world！；

第一个spring mvc项目完成！
