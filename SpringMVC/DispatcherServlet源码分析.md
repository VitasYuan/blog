## 1.DispatcherServlet的作用
  SpringMVC是以Http请求驱动的框架，DispatcherServlet的作用是作为一个前端控制器来对请求预处理，根据请求路径的不同，在hanler列表中找到匹配的handler来处理相应请求。其中SpringMVC处理请求的流程如下图所示：
  ![]()

## 2.DispatcherServlet类的继承结构和分析
DispatcherServlet类的继承结构图如下所示：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/dispatchersrc-1-1.png)  
