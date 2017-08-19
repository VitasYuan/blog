## 配置Maven仓库的方法

1. 打开Maven配置文件Setting.xml,如下：  

        vim  /Users/yuanweipeng/.m2/settings.xml

2. 在配置文件中添加如下配置：


    <mirrors>
        <mirror>
          <id>alimaven</id>
          <name>aliyun maven</name>
          <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
          <mirrorOf>central</mirrorOf>        
        </mirror>
      </mirrors>
输入：wq保存即可