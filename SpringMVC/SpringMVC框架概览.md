## 1.SpringMVC的核心Servlet
  SpringMVC框架是以

    org.springframework.web.servlet.DispatcherServlet.DispatcherServlet
  为核心设计的，此servlet为所有需要SpringMVC请求处理请求的入口，通过此类来匹配已经注册的handler(默认的为@Controller和@RequestMappingb表识的类)来处理相应请求。具体此类的实现请参考：XXXXX

## 2.SpringMVC特性

1.完全的角色分离，每一个角色（控制器，视图解析器，表单对象，路由分发等）都由专门的Object来实现
2.强大而简单的框架和javabean配置

## 3.SpringMVC设计原则
对扩展开放，对修改关闭
