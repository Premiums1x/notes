![[Pasted image 20260423155152.png]]
单一职责原则：
![[Pasted image 20260423145359.png]]

Controller层承担规范“入参的功能”，将前端返回数据分装成DTO对象，交给Service层。
*注意区分DTO和Entity的职责*：
	DTO负责接收前端入参并转换为Service层需要的数据对象
	Entity是Service层处理完业务逻辑后要写入数据库表的时候，根据数据库表的结构所设置的类，结构和表结构一致，将要入库的数据“包成一个整体“，直接交由Mapper执行DDL语句写入数据库表中。


Service层是业务层：
	之所以采用面向接口编程，是为了“多态”，用接口规定规范，具体实现可有多种。若存在多种实现则最佳实现就是`Payment`接口类型，实现类去实现接口:`AliPay`、`BaiduPay`....
![[Pasted image 20260423151912.png]]


Mapper数据操作层：
	需要在Service层完成后拿到Entity数据对象，然后进行数据库表的写入操作
```
UserDao接口
public interface UserDao {  
    public List<String> findAll();  
}
```


```
UserDaoImpl实现类：
package com.lancer.dao.impl;  
  
import略 
 
  
/**  
 * 数据访问实现类  
 */  
  
public class UserDaoImpl implements UserDao {  
    @Override  
    public List<String> findAll() {  
        //1.加载读取user.txt文件，获取用户数据  
        InputStream in =  this.getClass().getClassLoader().getResourceAsStream("user.txt");  
        //按行读取，每一行是一个对象，封装到users集合类中  
        ArrayList<String> lines = IoUtil.readLines( in, StandardCharsets.UTF_8,new ArrayList<>());  
        return lines;  
    }  
}

```