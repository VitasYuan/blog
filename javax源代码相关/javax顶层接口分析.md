## 1.Servlet接口分析

此接口是Servlet的最顶层接口，其中定义了Servlet生命周期相关的方法，所有Servlet都必须实现。此接口中的方法有以下几个：

    public void init(ServletConfig config) throws ServletException;
    public ServletConfig getServletConfig();
    public void service(ServletRequest req, ServletResponse res)
    public String getServletInfo();
    public void destroy();

  生命周期的调用顺序如下：  
  Servlet构建成功后调用init方法来初始化Servlet配置，此方法只调用一次，然后有一个http请求就调用service方法来处理相应请求，当所有请求结束，调用destroy方法来释放Servlet。  此三个方法都是由Servlet容器（比如tomcat，jetty）来调用。
  其中getServletConfig和getServletInfo用于获取Servlet相关信息。

## 2.ServletConfig接口分析

此接口为Servlet配置抽象接口，定义了获取Servlet信息的相关接口，接口列表如下：

    //获取Servlet名称，即是web.xml中配置的servlet-name
    public String getServletName()
    //返回此Servlet对应的上下文
    public ServletContext getServletContext()
    //获取初始化参数
    public String getInitParameter(String name)
    //获取初始化参数名称列表
    public Enumeration<String> getInitParameterNames()

## 3.GenericService抽象类分析
此类实现了上面两个接口，主要是实现了ServletConfig类中的接口。此类中维护了一个ServletConfig变量，定义方式如下：  

    private transient ServletConfig config;

添加transient修饰的作用：序列化的时候不包含此字段

  此局部变量在init的时候初始化，初始化方式如下：

    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        this.init();
    }

init方法是由Servletr容器调用，因此，web.xml中Servlet配置转化为ServletConfig的工作因该是由容器完成的。

其中ServletConfig接口的方法实现方式基本如下：

    public String getServletName() {
        ServletConfig sc = getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(
                lStrings.getString("err.servlet_config_not_initialized"));
        }

        return sc.getServletName();
    }
比较简单，不再分析。
