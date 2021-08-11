singleton（单例）、prototype（原型）、 request、session 和 global session


1.  singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的；
    
2.  prototype : 每次请求都会创建一个新的 bean 实例；
    
3.  request：每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效；
    
4.  session : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效；
    
5.  global-session：全局 session 作用域，仅仅在基于 portlet 的 web 应用中才有意义，Spring5 已经没有了。Portlet 是能够生成语义代码(例如：HTML)片段的小型 Java Web 插件。它们基于 portlet 容器，可以像 servlet 一样处理 HTTP 请求。但是，与 servlet 不同，每个 portlet 都有不同的会话。