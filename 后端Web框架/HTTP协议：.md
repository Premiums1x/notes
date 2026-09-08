规定浏览器、服务器间数据传输规则：
![[Pasted image 20260422201842.png]]

## 浏览器请求数据格式：
![[Pasted image 20260422201956.png]]

### 请求行与请求头：
![[Pasted image 20260422202936.png]]

#### 常见请求头及其含义：
![[Pasted image 20260422203225.png]]

**请求头、请求体间有一空行隔开**：
POST请求携带的请求参数在请求体中（通常是表单提交）
![[Pasted image 20260422203414.png]]

GET/POST请求方式区别：
![[Pasted image 20260422203518.png]]


### 获取请求数据：通过HttpServletRequest对象获取
![[Pasted image 20260422204115.png]]
```
@RestController  
public class requestController {  
  
    @RequestMapping("/request")  
    public String request(HttpServletRequest request){  
        //获取请求方法；  
        String method = request.getMethod();  
        System.out.println("method:"+method);  
  
        //获取请求头  
        String header = request.getHeader("Accept");  
        System.out.println("header:"+header);  
  
        //获取请求url地址  
        String requestURL = request.getRequestURL().toString();  
        System.out.println("requestURI:"+requestURL);  
  
        //获取请求uri（资源地址）  
        String requestURI = request.getRequestURI();  
        System.out.println("requestURI:"+requestURI);  
  
        //获取请求协议  
        String protocol = request.getProtocol();  
        System.out.println("protocol:"+protocol);  
  
        //获取请求参数(指明)  
        String parameter = request.getParameter("name");  
        System.out.println("parameter:"+parameter);  
          
        return "OK";  
    }  
}
```

- 浏览器发起请求：
![[Pasted image 20260422205310.png]]

- 服务端输出：
![[Pasted image 20260422205333.png]]








## 浏览器响应数据格式：响应给浏览器
![[Pasted image 20260422202058.png]]
**注：响应体在**：
![[Pasted image 20260422202126.png]]

- 各部分含义：![[Pasted image 20260423095947.png]]


重定向：
![[Pasted image 20260423100152.png]]
浏览器访问的资源在其他服务器上时，接收到请求的服务器返回3xx状态码，并携带Location：xxxx（需要重定向到的服务器地址）在响应头中。

常见五类状态码：
![[Pasted image 20260423100533.png]]

各类具体示例：
![[Pasted image 20260423100658.png]]

常见响应头key-value：
![[Pasted image 20260423100846.png]]

### Web服务器中设置响应数据：
- 方式一：基于HttpServletResponse对象：
![[Pasted image 20260423101630.png]]

因为返回给浏览器的数据由HttpServletResponse对象的方法去设置了，所以这里的返回值可以void。

- 实操：
![[Pasted image 20260423102821.png]]
效果：
![[Pasted image 20260423102836.png]]



- 方式二：（Spring所支持的方法）返回一个ResponseEntity对象
![[Pasted image 20260423110416.png]]


![[Pasted image 20260423103947.png]]
响应数据：通过链式构造并返回这个对象。

- 效果：
  ![[Pasted image 20260423104058.png]]

*注：响应头和响应状态码一般无需自己设置*
