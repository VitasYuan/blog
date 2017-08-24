## 1.HttpServlet的用法
提供了创建Http Servlet的抽象类，通过实现此类定义自己的Servlet

## 2.HttpServlet是否是线程安全
先说结论：HttpServlet不是线程安全

tomcat在只在第一次有请求的时候加载Servlet，加载之后调用init方法进行初始化，容器中只会保存一个Servlet对象，当有Http请求的时候会调用service方法对请求进行处理。所以Servlet不是线程安全的，当我们在Servlet中定义类变量或者处理共享资源时，要注意线程安全问题。简单的理解就是：单例多线程。  
以下是对Servlet的测试代码：  
    private static int staticCount = 0;

     private int count  = 0;

     @Override
     protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
         PrintWriter out = resp.getWriter();
         out.println("count = " + count++ + "staticCount = " + staticCount++);
     }

     @Override
     public void init() throws ServletException {
         super.init();
         System.out.println("init servlet!");
     }

运行tomcat，在浏览器中第一次发送http请求的时候控制台会输出   

      init servlet!

浏览器中多次发送请求后会输出：  

    count = 14staticCount = 14

由此验证以上结论：单例多线程，Servlet不是线程安全的。

## 3.关键方法分析：service()
当接收到Http请求的时候会有一个public的service方法，此方法中只是对参数类型进行判断和强制转换，然后调用此protected方法处理：  


    protected void service(HttpServletRequest req, HttpServletResponse resp)
      throws ServletException, IOException
    {
      String method = req.getMethod();

      if (method.equals(METHOD_GET)) {
          long lastModified = getLastModified(req);
          if (lastModified == -1) {
              // servlet doesn't support if-modified-since, no reason
              // to go through further expensive logic
              doGet(req, resp);
          } else {
              long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
              if (ifModifiedSince < lastModified) {
                  // If the servlet mod time is later, call doGet()
                  // Round down to the nearest second for a proper compare
                  // A ifModifiedSince of -1 will always be less
                  maybeSetLastModified(resp, lastModified);
                  doGet(req, resp);
              } else {
                  resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
              }
          }

      } else if (method.equals(METHOD_HEAD)) {
          long lastModified = getLastModified(req);
          maybeSetLastModified(resp, lastModified);
          doHead(req, resp);

      } else if (method.equals(METHOD_POST)) {
          doPost(req, resp);

      } else if (method.equals(METHOD_PUT)) {
          doPut(req, resp);

      } else if (method.equals(METHOD_DELETE)) {
          doDelete(req, resp);

      } else if (method.equals(METHOD_OPTIONS)) {
          doOptions(req,resp);

      } else if (method.equals(METHOD_TRACE)) {
          doTrace(req,resp);

      } else {
          //
          // Note that this means NO servlet supports whatever
          // method was requested, anywhere on this server.
          //

          String errMsg = lStrings.getString("http.method_not_implemented");
          Object[] errArgs = new Object[1];
          errArgs[0] = method;
          errMsg = MessageFormat.format(errMsg, errArgs);

          resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
      }
    }

此方法中比较关键的地方是对get请求的处理：在处理get请求的时候会对所请求中的资源上次修改时间和实际修改时间比较，根据比较结果返回状态码或者继续请求资源。此方式需要Servlet实现getLastModified方法。其他请求都是直接调用相应方法。
