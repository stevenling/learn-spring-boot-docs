## 一、Mybatis-Plus 概述
`Mybatis-Plus` 支持非常多的数据库，常规的有 `MySQL`，`H2`，`SQLite`，`SQLServer` 等等。

这边我将使用 `H2` 数据库做一次测试。
## 二、本地 H2 数据的配置
### 2.1 下载和配置 H2 数据库
`H2`数据库官网：[https://www.h2database.com/html/main.html](https://www.h2database.com/html/main.html)

在本地用户目录底下新建一个文件 `test.mv.d`，`test`表示数据库名称，你可以自定义名称。

在本地`H2` 目录下找到 `h2\bin\h2.bat`文件，这个是 `windows` 控制台的启动脚本，双击运行，会启动 `H2`的 `Web` 控制台。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/726118/1650258545343-933f6be7-44e1-4346-b043-a66407e28d20.png#clientId=u0ac601b2-7916-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=434&id=u06415b69&margin=%5Bobject%20Object%5D&name=image.png&originHeight=542&originWidth=710&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23333&status=done&style=none&taskId=u420f2c77-f168-417e-89ff-67dcdab917f&title=&width=568)
点击测试连接，如果测试成功，记下驱动类、`JDBC URL`、用户名和密码。
### 2.2 IDEA 连接 H2 数据库
先在 `pom.xml`中引入依赖
```xml
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.1.210</version>
            <scope>runtime</scope>
        </dependency>
```
在 `IDEA` 的数据库选项卡中新建「数据源」，选择 `H2` 数据源。

然后填入刚才的那些数据，最后测试一下连接。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/726118/1650258823407-9ae20b53-81ad-4a70-bce3-2295f0d24efa.png#clientId=u0ac601b2-7916-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=699&id=u2d35d456&margin=%5Bobject%20Object%5D&name=image.png&originHeight=874&originWidth=1307&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55465&status=done&style=none&taskId=ufcdf7ad2-8cda-4c5c-8f4b-421f9122752&title=&width=1045.6)
### 2.3 新建表和插入数据
#### 2.3.1 新建表
右键数据库点击「跳转到查询控制台」，输入创建表的命令：
```sql
create table user_test
(
	id int primary key not null,
	name varchar(30) null,
	age int null,
	email varchar(50) null
)
```
执行完毕，输入一下显示表的命令：
> show tables

![image.png](https://cdn.nlark.com/yuque/0/2022/png/726118/1650259197795-9152b430-4d62-4a70-bbd0-88a34045a6cf.png#clientId=u0ac601b2-7916-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=100&id=ucaf9c311&margin=%5Bobject%20Object%5D&name=image.png&originHeight=125&originWidth=608&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9911&status=done&style=none&taskId=u02079c3b-24aa-4b85-8331-c96c4dbf13e&title=&width=486.4)
发现已经新建了表。
#### 2.3.2 插入数据
输入插入的 `sql` 语句：
```sql
insert into user_test(id, name, age, email)
values
(1, 'Jone', 18, 'yunhu1@gmail.com'),
(2, 'Jack', 20, 'yunhu2@gmail.com'),
(3, 'Tom', 28, 'yunhu3@gmail.com'),
(4, 'Sandy', 21, 'yunhu4@gmail.com'),
(5, 'Billie', 24, 'yunhu5@gmail.com');
```
插入执行完毕后，执行查看表的命令：
> select * from user_test

![image.png](https://cdn.nlark.com/yuque/0/2022/png/726118/1650259466372-a5ec15e5-7017-4d07-83b6-a494b1e3d233.png#clientId=u0ac601b2-7916-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=202&id=u1bd8d81f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=252&originWidth=791&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35499&status=done&style=none&taskId=u9ac5351e-07e4-4052-985f-b2ee7230847&title=&width=632.8)
## 三、Mybatis-Plus 的配置
### 3.1 项目结构
```java
├─main
│  ├─java
│  │  └─com
│  │      └─example
│  │          └─mybatisplusdemo
│  │              │  MybatisplusdemoApplication.java
│  │              │  
│  │              ├─controller
│  │              │      UserController.java
│  │              │      
│  │              ├─entity
│  │              │      User.java
│  │              │      
│  │              └─mapper
│  │                      UserMapper.java
```
### 3.2 引入依赖
```xml
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
				
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```
### 3.3 在 application.yml 配置文件中添加 H2 的相关配置
```xml
server:
  port: 8080 # 端口号

spring:
  datasource:
    driver-class-name: org.h2.Driver # 驱动类
    url: jdbc:h2:tcp://localhost/~/test # JDBC URL
    username: sa # 用户名
    password:  # 密码
```
### 3.4 添加 user.java 数据实体类

在包目录下新建 `entity` 目录，用来存放数据实体类，然后在 `entity` 目录下新建 `User.java`。
```java
package com.example.mybatisplusdemo.entity;

import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

/**
 * User 数据实体类
 * @author 云胡
 * @data 2022-04-18
 */

@Data
@TableName("user_test")  //将对象和表进行关联
public class User {
    @TableId("id")
    private Long id;

    @TableField("name")
    private String name;

    @TableField("age")
    private Integer age;

    @TableField("email")
    private String email;
}

```
`@Data`是 `lombok.jar`包下的注解，之后就不需要写出 `set` 和 `get` 方法。

`@TableName`实现实体类和数据库中的表两者之间的映射。

`@TableId`是主键字段的注解。

`@TableField`是非主键字段注解。
### 3.5 编写 UserMapper.java 接口
在包目录底下新建一个 `mapper` 目录，用来存放数据访问类，然后新建 `UserMapper.java` 文件。
```java
package com.example.mybatisplusdemo.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.mybatisplusdemo.entity.User;
import org.apache.ibatis.annotations.Mapper;

/**
 * UserMapper 数据访问接口
 * @author 云胡
 * @data 2022-04-18
 */

@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```
`@Mapper`用在接口类上，后续才可以被 `controller`类使用。
### 3.6 编写 UserController.java 类
在包目录底下新建一个`controller` 目录，用来存放控制类，然后新建一个 `UserController.java`文件。
```java
package com.example.mybatisplusdemo.controller;

import com.example.mybatisplusdemo.entity.User;
import com.example.mybatisplusdemo.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * User 控制类
 * @author 云胡
 * @data 2022-04-18
 */

@RestController
public class UserController {
    @Autowired
    private UserMapper userMapper;

    @GetMapping("/showUser")
    public List<User> showUser(){
        List<User> list = userMapper.selectList(null);
        System.out.println(list);
        return list;
    }
}
```
### 3.7 结果
运行程序后，打开浏览器输入链接: `localhost:8080/showUser`可以看到控制台打印了以下数据：
> [User(id=1, name=Jone, age=18, email=yunhu1@gmail.com), User(id=2, name=Jack, age=20, email=yunhu2@gmail.com), User(id=3, name=Tom, age=28, email=yunhu3@gmail.com), User(id=4, name=Sandy, age=21, email=yunhu4@gmail.com), User(id=5, name=Billie, age=24, email=yunhu5@gmail.com)]

网页上显示了：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/726118/1650263103920-4723441d-d443-484a-9d79-10927d1fa546.png#clientId=u0ac601b2-7916-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=636&id=udbc13541&margin=%5Bobject%20Object%5D&name=image.png&originHeight=795&originWidth=751&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21410&status=done&style=none&taskId=u8bf24219-2838-439c-9bce-808e7c89252&title=&width=600.8)
 我这边装了 `Web` 前端助手 `FeHelper`，所以看到的表的数据被自动格式化了。

