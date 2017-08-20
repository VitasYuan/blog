## 1.相关注解介绍
### 1.1 @Controller
@Controller是作为标识一个类为controller角色的注解，SpringMVC通过扫描所有带有此注解的类，并结合@RequestMapping来找到匹配的controller来处理相应的http请求  
定义controller由多种方式，比如通过标准的定义Spring bean来定义，但是一般会使用SpringMVC的自动扫描并注册controller的功能。此功能默认是关闭的，需要spring的配置文件中添加以下配置：

    <context:component-scan base-package="com.springmvc.study.*"></context:component-scan>

### 1.2 @RequestMapping
此注解用于指定controller的访问路径，用于类时表示此类中所有的方法都是相对与此路径。

    @Controller
    @RequestMapping("/appointments")
    public class AppointmentsController {

      @RequestMapping(method = RequestMethod.GET)
      public Map<String, Appointment> get() {
          return "";
      }

      @RequestMapping(path = "/{day}", method = RequestMethod.GET)
      public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
          return "";
      }
    }
其中需要注意的是：RequstMapping默认是支持所有的http方法访问的，使用method参数可以缩小访问的方式。

另外：Spring 4.3添加了以下注解简化配置  

      @GetMapping == Get @RequestMapping
      @PostMapping
      @PutMapping
      @DeleteMapping
      @PatchMapping

### 1.3URI模版模式
此功能主要是用于在http访问uri中添加参数，在实现restful接口中使用比较普遍。具体用法如以下代码：

    @Controller
    @RequestMapping("/owners/{ownerId}")
        public class RelativePathUriTemplateController {

        @RequestMapping("/pets/{petId}")
        public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
            // implementation omitted
        }
    }

使用@PathVariable来接收变量，其中变量类型SpringMvc会自动转换成匹配类型

### 1.4 接收和返回json字符串
接收json字符串：如下代码：

    @PostMapping(path = "/pets", consumes = "application/json")
    public void addPet(@RequestBody Pet pet, Model model) {
    // implementation omitted
    }

返回json字符串：如下所示：
