![[Pasted image 20260415095902.png]]


![[Pasted image 20260415095954.png]]


大模型有无function calling的区别：
![[Pasted image 20260415100153.png]]

有函数调用时：
![[Pasted image 20260415100531.png]]
1. 用户提出问题到达服务器，
2. 服务器根据用户的问题检索出相关的函数调用，
3. 服务器把用户问题和函数列表传给大模型，
4. 大模型根据用户意图等决定是否调用、调用哪些函数，（大模型本身不调用函数，只做决策、参数生成）
5. 然后提取用户问题中的关键词（用作函数参数）和相关函数，发送给服务器，让服务器根据关键词和函数来执行调用，
6. 调用得到结果后返回给大模型，
7. 大模型将调用结果与用户问题整合后发送给服务器，服务器再发送到用户端。


Dify对于Function calling的应用：
![[Pasted image 20260415101402.png]]



### 自定义插件：
![[Pasted image 20260415112555.png]]


1. 生成一个Python脚本以供天气调用，
2. 然后运行启动服务，
3. ！！！记住一定要先运行脚本启动flask服务，

4. 下载包实现内网穿透：
5. `npm install -g localtunnel `

6. 再执行内网穿透：
7. 选定一个端口监听
8. `lt -p 数字`
9. 然后返回一个url，这个就是要去测试接口的网址。


成功示例：
Python的Flask服务启动在5000端口
localtunnel也要暴露同一个端口，这样才能把服务暴露给外部
`token鉴权，city是具体城市`
![[Pasted image 20260415111133.png]]

配置在dify中，根据脚本代码生成一份openai schema：
```
openapi: 3.0.3
info:
  title: 天气查询工具
  version: 1.0.0
  description: 根据城市查询当前天气
servers:
  - url: https://你的localtunnel地址
paths:
  /weather:
    get:
      operationId: getWeather
      summary: 查询城市天气
      description: 根据城市名查询当前天气信息
      parameters:
        - name: token
          in: query
          required: true
          description: 身份验证 token
          schema:
            type: string
            example: lancer
        - name: city
          in: query
          required: true
          description: 城市名称，例如 Beijing、Shanghai
          schema:
            type: string
            example: Beijing
      responses:
        '200':
          description: 查询成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  code:
                    type: integer
                    example: 200
                  msg:
                    type: string
                    example: 查询成功
                  data:
                    type: object
                    properties:
                      city:
                        type: string
                        example: Beijing
                      country:
                        type: string
                        example: China
                      latitude:
                        type: number
                        example: 39.9075
                      longitude:
                        type: number
                        example: 116.3972
                      timezone:
                        type: string
                        example: Asia/Shanghai
                      weather:
                        type: object
                        properties:
                          time:
                            type: string
                            example: "2026-04-15T21:00"
                          temperature_2m:
                            type: number
                            example: 18.3
                          relative_humidity_2m:
                            type: number
                            example: 42
                          apparent_temperature:
                            type: number
                            example: 17.6
                          is_day:
                            type: integer
                            example: 0
                          precipitation:
                            type: number
                            example: 0
                          rain:
                            type: number
                            example: 0
                          weather_code:
                            type: integer
                            example: 1
                          wind_speed_10m:
                            type: number
                            example: 8.4
        '400':
          description: 缺少参数
        '401':
          description: token 无效
        '404':
          description: 城市未找到
        '500':
          description: 服务器错误
```

鉴权方式先选择无，这个接口现在其实**不是 Header 鉴权**，而是 **Query 参数鉴权**：
现在的 `token=lancer` 不是通过 Authorization Header 传的，而是普通 query 参数。
也就是说，`token` 已经在 OpenAPI 里定义成了请求参数，不用再单独配“鉴权方法”。

结果：
![[Pasted image 20260415112317.png]]
![[Pasted image 20260415112358.png]]

使用测试：
![[Pasted image 20260415112938.png]]
