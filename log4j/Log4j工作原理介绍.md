## 1.log4j工作流程图
log4j主要有Logger，Layout和Appender三个核心模块。Logger模块负责收集日志数据，然后通过Layout模块将数据格式化，最终由Appender模块将日志数据输出到不同的目的地，如数据库、控制台或者文件。工作流程图如下所示：
![]()

## 2.配置方式
log4j支持xml和properties文件两种配置方式，下面以properties文件的方式介绍log4j的配置。
由第一部分所说的，log4j的配置也包括三个部分，分别为logger，layout和appender。其中logger的配置格式如下：    

    log4j.rootLogger = [ level ] , appenderName, appenderName, …
其中[lever]为日志输出的级别，常用的日志级别优先级从高到低分别是ERROR、WARN、INFO、DEBUG。优先级后面为日志需要输出到的appernder名称，log4j支持同一个logger输出到多个目的地。
还可以配置以包为根据的logger，如下：  

    log4j.logger.com.zjut.dao
此配置为com.zjut.dao包下的文件日志默认输出。

## 3.配置文件的appender
appender决定了日志文件输出的位置，log4j支持以下四种：

    org.apache.log4j.ConsoleAppender（控制台），
    •org.apache.log4j.FileAppender（文件），
    •org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件），
    •org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件）
    •org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方
## 4.配置文件的layout

log4j支持以下几种layout：  

    •org.apache.log4j.HTMLLayout（以HTML表格形式布局），
    •org.apache.log4j.PatternLayout（可以灵活地指定布局模式），
    •org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）
    •org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）

## 5.一个完整的配置示例：Mybatis输出sql日志到指定文件

首选需要在mybatis配置文件中添加以下配置：

    <settings>
        <setting name="logImpl" value="SLF4J"/>
    </settings>
然后在log4j.properties文件中添加以下内容：

    MyBatis logging configuration...
    log4j.logger.com.zjut.dao=DEBUG file

    log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
    log4j.appender.file.DatePattern='-'yyyy-MM-dd
    log4j.appender.file.File=./logs/sql.log
    log4j.appender.file.Append=true
    log4j.appender.file.layout=org.apache.log4j.PatternLayout
    log4j.appender.file.layout.ConversionPattern=[%-d{HH:mm:ss}][%p][%c]- %m %n

完成此配置后，mybatis会打印sql操作信息，到sql.log文件
