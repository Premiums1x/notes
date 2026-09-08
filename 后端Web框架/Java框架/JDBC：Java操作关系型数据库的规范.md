![[Pasted image 20260423190736.png]]

JDBC不提供实现，仅提供API接口规范，具体实现类在各个关系型数据库厂商根据JDBC规范制作的驱动

第一个JDBC程序：
```
public void testJDBC() throws ClassNotFoundException, SQLException {  
    //将驱动加载到内存：  
    Class.forName("com.mysql.cj.jdbc.Driver");  
  
    //获取数据库连接：  
    String url = "jdbc:mysql://localhost:3306/web01";  
    String username = "root";  
    String password = "123456";  
  
    Connection connection = DriverManager.getConnection(url, username, password);  
  
    //创建执行sql语句的Statement对象：  
    Statement statement = connection.createStatement();  
  
    //执行sql语句  
    int i = statement.executeUpdate("update user set age = 18 where id = 1 ");  
    System.out.println("受影响行数 "+i);  
  
    //释放资源：  
    statement.close();  
    connection.close();  
}
```


查询需求：
**（新！）** 引入预编译执行sql语句对象PrepareStatement、结果集ResultSet
![[Pasted image 20260423203901.png]]
代码：
```
    @Test  
    public void testSelect() throws ClassNotFoundException, SQLException {  
        Class.forName("com.mysql.cj.jdbc.Driver");  
  
        // 2. 获取数据库连接  
        String url = "jdbc:mysql://localhost:3306/web01";  
        String user = "root";  
        String password = "123456";  
  
        Connection conn = DriverManager.getConnection(url, user, password);  
  
        // 3. 定义SQL  
        String sql = "select id, username, password, name, age from user where username = ? and password = ?";  
  
        // 4. 获取PreparedStatement对象  
        PreparedStatement pstmt = conn.prepareStatement(sql);  
  
        // 5. 设置参数  
        pstmt.setString(1, "daqiao");  
        pstmt.setString(2, "123456");  
  
        // 6. 执行SQL，得到结果集 ResultSet        ResultSet rs = pstmt.executeQuery();  
  
        // 7. 遍历结果集，封装User对象  
        while (rs.next()) {  
            User u = new User();  
  
//           每个字段的类型是什么就get对应的数据类型，然后参数指定字段名  
            u.setId(rs.getInt("id"));  
            u.setUsername(rs.getString("username"));  
            u.setPassword(rs.getString("password"));  
            u.setName(rs.getString("name"));  
            u.setAge(rs.getInt("age"));  
  
            System.out.println(u);  
        }  
  
        // 8. 释放资源  
        rs.close();  
        pstmt.close();  
        conn.close();  
  
    }
```

结果：
![[Pasted image 20260423204742.png]]

### 预编译sql与静态sql：
![[Pasted image 20260424104904.png]]

![[Pasted image 20260424105758.png]]

静态sql的注入问题：用户可以在输入查询条件（如表单时）输入一些特殊语句而改变查询逻辑，从而入侵系统。比如输入密码时输入`''or '1' = '1'`,那么这段被拼接到查询语句中会变成`xxxx or 恒为真表达式`，尤其是在登录鉴权时，这样走了or的判断就能直接进入入侵系统。