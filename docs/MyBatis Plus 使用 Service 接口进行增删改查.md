## 一、概述
一般我们不在 `controller` 层直接使用 `mapper` 方法去操控数据库，而是通过 `service` 写业务逻辑，然后去操控数据库。
## 二、步骤
### 2.1 新建实体类
```java
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

/**
 * @author: yunhu
 * @date: 2022/6/14
 */

@Data
@TableName("sys_user")
public class UserTableEntity {
    @TableId("id")
    private Integer id;

    @TableField("username")
    private String username;

    @TableField("password")
    private String password;
}
```
`sys_user` 中的数据

| id | username | password |
| --- | --- | --- |
| 1 | admin	 | yunhu0 |
| 2 | test	 | 123456 |

### 2.2 新建 UserService 接口
新建 `service`目录，在 `service`目录下新建 `UserService` 接口继承  `IService<T>` 。
T 泛型在这边就是 `UserTableEntity`实体类。
```java
import com.baomidou.mybatisplus.extension.service.IService;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import java.util.List;

/**
 * @author: yunhu
 * @date: 2022/6/14
 */

@Service
@Component
public interface UserService extends IService<UserTableEntity> {

    List<UserTableEntity> listAllByUsername(String username);
}
```
### 2.3 新建 UserServiceImpl 类
在 `service`目录下新建 `impl`目录，在 `impl`目录中新建 `UserServiceImpl`类 。
`UserServiceImpl`去继承 `ServiceImpl<M extends BaseMapper<T>**, **T> implements IService<T>`。
`ServiceImpl` 有两个参数：

1. `M extends BaseMapper<T>`是一个继承 `BaseMapper<T>`的自定义 `mapper`，接下来会新建。
1. `T`实体类。
```java
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author: yunhu
 * @date: 2022/6/14
 */

@Service
public class UserServiceImpl extends ServiceImpl<UserTableMapper, UserTableEntity> implements UserService {
}
```
### 2.4 新建`UserTableMapper`接口
新建 `mapper` 目录，然后在 `mapper` 目录下新建 `UserTableMapper`接口。
```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Mapper;

/**
 * @author: yunhu
 * @date: 2022/6/14
 */

@Mapper
public interface UserTableMapper extends BaseMapper<UserTableEntity> {

}
```
## 三、测试
### 3.1 查询
```java
package com.example.library;

import com.example.library.entity.UserTableEntity;
import com.example.library.service.UserService;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

/**
 * @author: yunhu
 * @date: 2022/6/14
 */

@MapperScan("com.example.library.mapper")
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
class LibraryApplicationTests {

    @Autowired
    private UserService userService;

    @Test
    void contextLoads() {
        // 查找所有值
        List<UserTableEntity> list = userService.list();
        list.forEach(System.out::println);

    }
}
```
**output: **
```shell
UserTableEntity(id=1, username=admin, password=yunhu0)
UserTableEntity(id=2, username=test, password=123456)
```
### 3.2 插入
```java
void contextLoads() {
    // 插入操作
    UserTableEntity userTableEntity = new UserTableEntity();
    userTableEntity.setId(3);
    userTableEntity.setUsername("cheng");
    userTableEntity.setPassword("666666");
    Boolean res = userService.save(userTableEntity);
    if (res) {
        System.out.println("add success");
        // 查看插入后的表中数据
        List<UserTableEntity> list = userService.list();
        list.forEach(System.out::println);
    } 
}
```
**output：**
```shell
add success
UserTableEntity(id=1, username=admin, password=yunhu0)
UserTableEntity(id=2, username=test, password=123456)
UserTableEntity(id=3, username=cheng, password=666666)
```
### 3.3 删除
删除用户名为 `admin` 的记录。
```java
void contextLoads() {
        QueryWrapper<UserTableEntity> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username", "admin");
        boolean res = userService.remove(queryWrapper);
        if(res) {
            System.out.println("delete success");
            // 查看删除后的表中数据
            List<UserTableEntity> list = userService.list();
            list.forEach(System.out::println);
        }
}
```
**output: **
```java
delete success
UserTableEntity(id=2, username=test, password=123456)
UserTableEntity(id=3, username=cheng, password=666666)
```
### 3.4 更新
```java
void contextLoads() {
        UserTableEntity userTableEntity = new UserTableEntity();
        userTableEntity.setId(3);
        userTableEntity.setUsername("Tom Sawyer");
        userTableEntity.setPassword("777777");
        // 通过 id 更新
        boolean res = userService.updateById(userTableEntity);
        if(res) {
            System.out.println("update success");
            // 查看更新后的表中数据
            List<UserTableEntity> list = userService.list();
            list.forEach(System.out::println);
        }
}
```
**output:**
```shell
update success
UserTableEntity(id=2, username=test, password=123456)
UserTableEntity(id=3, username=Tom Sawyer, password=777777)
```
