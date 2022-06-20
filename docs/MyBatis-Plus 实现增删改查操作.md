## 一、概述
之前已经使用 `Spring Boot` 集成 `MyBatis-Plus`，这次实现增删改查操作。
## 二、表的内容
| **id** | **name** | **age** | **email** |
| --- | --- | --- | --- |
| 1 | 孙悟空 | 1000 | sunwukong@gmail.com |
| 2 | 唐僧 | 20 | tangseng@gmail.com |
| 3 | 猪八戒 | 800 | zhubajie@gmail.com |
| 4 | 沙僧 | 900 | shaseng@gmail.com |
| 5 | 白龙马 | 500 | bailongma@gmail.com |

## 二、查询
### 2.1 通过 id 查询
```java
    /**
     * 查询通过 id
     */
    @GetMapping("/selectById")
    public User selectById(){
        User user = userMapper.selectById(3);
        return user;
    }
```
输出：
```json
{
	"id": 3,
	"name": "猪八戒",
	"age": 800,
	"email": "zhubajie@gmail.com"
}
```
### 2.2 通过条件构造器查询
```java
    /**
     * 查询通过条件构造器
     */
	@GetMapping("/selectByWrapper")
    public List<User> selectByWrapper(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                .eq("age", 1000);

        // 查询多条数据
        List<User> list = userMapper.selectList(queryWrapper);
        return list;
    }
```
输出：
```json
[
    {
        "id": 1,
        "name": "孙悟空",
        "age": 1000,
        "email": "sunwukong@gmail.com"
    }
]
```
## 三、插入记录
```java
	/**
     * 添加记录
     */
    @GetMapping("/insert")
    public List<User> insert(){
        User user = new User();
        user.setId(6L);
        user.setAge(1500);
        user.setName("二郎神");
        user.setEmail("erlangshen@gmail.com");
        userMapper.insert(user);
		
		// 查询添加后的表
        List<User> list = userMapper.selectList(null);
        return list;
    }
```
输出：
```json
[
    {
        "id": 1,
        "name": "孙悟空",
        "age": 1000,
        "email": "sunwukong@gmail.com"
    },
    {
        "id": 2,
        "name": "唐僧",
        "age": 20,
        "email": "tangseng@gmail.com"
    },
    {
        "id": 3,
        "name": "猪八戒",
        "age": 800,
        "email": "zhubajie@gmail.com"
    },
    {
        "id": 4,
        "name": "沙僧",
        "age": 900,
        "email": "shaseng@gmail.com"
    },
    {
        "id": 5,
        "name": "白龙马",
        "age": 500,
        "email": "bailongma@gmail.com"
    },
    {
        "id": 6,
        "name": "二郎神",
        "age": 1500,
        "email": "erlangshen@gmail.com"
    }
]
```
可以看到增加了**「二郎神」**这个用户。
## 四、修改记录
```java
   /**
     * 修改记录
     */
    @GetMapping("/update")
    public List<User> update(){
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
        updateWrapper.eq("name","唐僧")
                     .set("age", 1200);

        Integer rows = userMapper.update(null, updateWrapper);

        // 查询修改后的数据
        List<User> list = userMapper.selectList(null);
        return list;
    }
```
输出：
```json
[
    {
        "id": 1,
        "name": "孙悟空",
        "age": 1000,
        "email": "sunwukong@gmail.com"
    },
    {
        "id": 2,
        "name": "唐僧",
        "age": 1200,
        "email": "tangseng@gmail.com"
    },
    {
        "id": 3,
        "name": "猪八戒",
        "age": 800,
        "email": "zhubajie@gmail.com"
    },
    {
        "id": 4,
        "name": "沙僧",
        "age": 900,
        "email": "shaseng@gmail.com"
    },
    {
        "id": 5,
        "name": "白龙马",
        "age": 500,
        "email": "bailongma@gmail.com"
    },
    {
        "id": 6,
        "name": "二郎神",
        "age": 1500,
        "email": "erlangshen@gmail.com"
    }
]
```
可以看到**「唐僧」**这个用户的**「年龄」**从 `20`被修改为 `1200`。
## 五、删除记录
### 5.1 通过 id 删除记录
```java
	/**
     * 删除记录 通过 id
     */
    @GetMapping("/deleteById")
    public List<User> deleteById(){

        // 根据 ID 删除
        userMapper.deleteById(4);

        // 查询删除后的数据
        List<User> list = userMapper.selectList(null);
        return list;
    }
```
输出：
```json
[
    {
        "id": 1,
        "name": "孙悟空",
        "age": 1000,
        "email": "sunwukong@gmail.com"
    },
    {
        "id": 2,
        "name": "唐僧",
        "age": 1200,
        "email": "tangseng@gmail.com"
    },
    {
        "id": 3,
        "name": "猪八戒",
        "age": 800,
        "email": "zhubajie@gmail.com"
    },
    {
        "id": 5,
        "name": "白龙马",
        "age": 500,
        "email": "bailongma@gmail.com"
    },
    {
        "id": 6,
        "name": "二郎神",
        "age": 1500,
        "email": "erlangshen@gmail.com"
    }
]
```
可以看到**「沙僧」**这个用户的记录已经被删除掉。
### 5.2 通过条件构造器删除记录
```java
    /**
     * 删除记录，通过条件构造器
     */
    @GetMapping("/deleteByWrapper")
    public List<User> deleteByWrapper(){
        QueryWrapper query = new QueryWrapper();
        query.eq("name", "孙悟空");

        userMapper.delete(query);
		
        // 查询删除孙悟空后的记录
        List<User> list = userMapper.selectList(null);
        return list;
    }

```
输出：
```json
[
    {
        "id": 2,
        "name": "唐僧",
        "age": 1200,
        "email": "tangseng@gmail.com"
    },
    {
        "id": 3,
        "name": "猪八戒",
        "age": 800,
        "email": "zhubajie@gmail.com"
    },
    {
        "id": 5,
        "name": "白龙马",
        "age": 500,
        "email": "bailongma@gmail.com"
    },
    {
        "id": 6,
        "name": "二郎神",
        "age": 1500,
        "email": "erlangshen@gmail.com"
    }
]
```
可以看到，世间再无悟空了。
## 六、完整代码
```java
/**
 * MyBatis-Plus 增删改查练习
 * @author: 云胡
 * @date: 2022-4-19
 */
@CrossOrigin
@RestController
public class MybatisPlusController {
	
    @Autowired
    private UserMapper userMapper;

    /**
     * 查询通过 id
     */
    @GetMapping("/selectById")
    public User selectById(){
        User user = userMapper.selectById(3);
        return user;
    }

	/**
     * 查询通过条件构造器
     */
    @GetMapping("/selectByWrapper")
    public List<User>  selectByWrapper(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                .eq("age", 1000);

        // 查询多条数据
        List<User> list = userMapper.selectList(queryWrapper);
        return list;
    }


	/**
     * 添加记录
     */
    @GetMapping("/insert")
    public List<User> insert(){
        User user = new User();
        user.setId(6L);
        user.setAge(1500);
        user.setName("二郎神");
        user.setEmail("erlangshen@gmail.com");
        userMapper.insert(user);
		
		// 查询添加后的表
        List<User> list = userMapper.selectList(null);
        return list;
    }    

    /**
     * 修改记录
     */
    @GetMapping("/update")
    public List<User> update(){
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
        updateWrapper.eq("name","唐僧")
                     .set("age", 1200);

        Integer rows = userMapper.update(null, updateWrapper);

        // 查询修改后的数据
        List<User> list = userMapper.selectList(null);
        return list;
    }

    /**
     * 删除记录，通过条件构造器
     */
    @GetMapping("/deleteByWrapper")
    public List<User> deleteByWrapper(){
        QueryWrapper query = new QueryWrapper();
        query.eq("name", "孙悟空");

        userMapper.delete(query);
		
        // 查询删除孙悟空后的记录
        List<User> list = userMapper.selectList(null);
        return list;
    }

    /**
     * 删除记录 通过 id
     */
    @GetMapping("/deleteById")
    public List<User> deleteById(){

        // 根据 ID 删除
        userMapper.deleteById(4);

        // 查询
        List<User> list = userMapper.selectList(null);
        return list;
    }
}
```
## 七、参考资料
[MyBatis-Plus CRUD 接口](https://baomidou.com/pages/49cc81/#chain)
