## 1.SpringMVC在Controller中接收Date类型的参数
首先定义一个接收Date类型的handler，代码如下：  

    @GetMapping(value = "/hour", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public String getHourAverage(@RequestParam(value = "startTime") Date startTime, @RequestParam(value = "endTime") Date endTime){
    return "startTime:" + startTime + "endTime" + endTime;
    }
然后需要定义一个类型转换器，此类需要实现Converter<String, Date>接口，代码如下：

    public class DateConvert implements Converter<String, Date> {

        private Logger LOGGER = LoggerFactory.getLogger(DateConvert.class);

        @Override
        public Date convert(String s) {
            try {

                return DateUtil.parse(s);

            }catch (Exception e){
              LOGGER.error("Date convert exception, source :"+ s +",exception:", e);
            }

            return null;
        }

    }
最后需要将此转换器注入到FormattingConversionServiceFactoryBean中，Spring中的配置代码如下

<mvc:annotation-driven conversion-service="customConversionService"/>

    <bean id="customConversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.zjut.convert.DateConvert"></bean>
            </set>
        </property>
    </bean>
完成此配置后，即可在hanler中接收Date类型的参数。开启此服务，在浏览器中输入：

    http://localhost:8088/average/hour.ajax?startTime=2017-01-01%2012:12:12&&endTime=2017-09-09%2012:12:12
会得到以下输出：
![]()

## 2.SpringMVC返回json字符串时对Date类型的格式化
在使用SpringMVC实现返回json字符串的接口时候，当返回的数据类型由Date类型的时候，需要对这种类型做特殊的转换才能得到我们想要的格式化的时间字符串，实现此功能只需要在Model类中的get方法做简单的处理即可，首先我们需要添加jackson依赖，配置如下：

    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
          <dependency>
              <groupId>com.fasterxml.jackson.core</groupId>
              <artifactId>jackson-core</artifactId>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.codehaus.jackson/jackson-core-asl -->
          <dependency>
              <groupId>org.codehaus.jackson</groupId>
              <artifactId>jackson-core-asl</artifactId>
          </dependency>

          <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
          <dependency>
              <groupId>com.fasterxml.jackson.core</groupId>
              <artifactId>jackson-databind</artifactId>
          </dependency>

          <!-- https://mvnrepository.com/artifact/org.codehaus.jackson/jackson-mapper-asl -->
          <dependency>
              <groupId>org.codehaus.jackson</groupId>
              <artifactId>jackson-mapper-asl</artifactId>
          </dependency>

添加依赖后，只需要在Date类型的字段的get方法中添加如下注解即可，如下：

    @JsonFormat(pattern = DateUtil.DEFAULT_DATE_FORMAT)
    public Date getCreatedTime() {
        return createdTime;
    }
其中pattern参数为格式化参数。通过以上配置之后，访问接口后得到的数据如下所示，可以看到createTime格式为时间字符串：
![]()
