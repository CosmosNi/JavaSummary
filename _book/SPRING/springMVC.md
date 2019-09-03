# SpringMVC

###### 1.1 springMVC执行流程

- User向服务器发送request,前端控制Servelt DispatcherServlet捕获;
- DispatcherServlet对请求URL进行解析，调用HandlerMapping获得该Handler配置的所有相关的对象，最后以HandlerExecutionChain对象的形式返回.
- DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter.
- 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)
- Handler执行完成后，返回一个ModelAndView对象到DispatcherServlet
- 根据返回的ModelAndView，选择一个适合的ViewResolver
- ViewResolver 结合Model和View，来渲染视图
- 将渲染结果返回给客户端。