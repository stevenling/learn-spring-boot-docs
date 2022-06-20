## 一、导入 Maven 依赖
```xml
<!--MySQL jdbc 驱动-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.17</version>
</dependency>
```
## 二、在 application.yml 中配置数据源
```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/csvswitch?serverTimezone=GMT%2B8&characterEncoding=utf8&useSSL=true
    username: root
    password: password # 你的数据库密码 
```

分析一下这个 `url` 的结构：
> url : jdbc:mysql://localhost:3306/csvswitch?serverTimezone=GMT%2B8&characterEncoding=utf8&useSSL=true


- 多个参数之间用 `&`符号连接
- `3306`是 `MySQL` 的端口号
- `csvswitch`是自己创建的数据库名称
- `serverTimezone=GMT%2B8` 设置时区为东八区，即北京时间，不设置会报错。
- `characterEncoding=utf8`覆盖客户端自动检测到的编码，使用指定的 `utf8`
- `useSSL=true`指定是否使用 `ssl` 连接。

