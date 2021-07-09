---
tags: java

title: Servlet规范
---

> Servlet规范，来自于JAVAEE规范中的一种。
>
> 后期使用springMVC框架。

<!--more-->

Servlet规范指定【动态资源文件】开发步骤；Http服务器调用动态资源文件规则；Http服务器管理动态资源文件实例对象规则。

Tomcat服务器下lib文件中servlet-api.jar存放Servlet接口（javax.servlet.Servlet接口）

Servlet规范任务中，Http服务器能调用的【动态资源文件】必须是一个**Servlet接口实现类**

## Servlet规范开发步骤

- 创建一个Java类继承HttpServlet父类，使之成为一个Servlet接口实现类


- 重写两个方法，doGet或则doPost
  
- 将Servlet接口实现类信息【注册】到Tomcat服务器

```xml
<!-- /web/WEB-INF/web.xml文件中-->
<!--将Servlet接口实现类类路径地址交给Tomcat-->
<servlet>
    <servlet-name>mm</servlet-name> <!--声明一个变量存储servlet接口实现类类路径-->
    <servlet-class>com.bjpowernode.controller.OneServlet</servlet-class><!--声明servlet接口实现类类路径-->
</servlet>
    Tomcat  String mm = "com.bjpowernode.controller.OneServlet"
    <!--为了降低用户访问Servlet接口实现类难度，需要设置简短请求别名-->
    <servlet-mapping> 
        <servlet-name>mm</servlet-name>
        <url-pattern>/one</url-pattern> <!--设置简短请求别名,别名在书写时必须以"/"为开头-->
    </servlet-mapping>
```

## Servlet对象生命周期:

- 网站中所有的Servlet接口实现类的实例对象，只能由Http服务器(如Tomcat，相当于servlet的容器)负责额创建。 开发人员不能手动创建。


- 在默认的情况下，Http服务器接收到对于当前Servlet接口实现类第一次请求时，自动创建这个Servlet接口实现类的实例对象；在手动配置情况下，要求Http服务器在启动时自动创建某个Servlet接口实现类的实例对象

```xml
<!--手动配置-->
<servlet>
    <servlet-name>mm</servlet-name> <!--声明一个变量存储servlet接口实现类类路径-->
    <servlet-class>com.bjpowernode.controller.OneServlet</servlet-class>
    <load-on-startup>30<load-on-startup><!--填写一个大于0的整数即可-->
</servlet>
```

- 在Http服务器运行期间，一个Servlet接口实现类只能被创建出一个实例对象

- 在Http服务器关闭时刻，自动将网站中所有的Servlet对象进行销毁


## HttpServletResponse接口

来自于Servlet规范，在Tomcat中存在servlet-api.jar，实现类由Http服务器负责提供。负责将doGet/doPost方法**执行结果**写入到【响应体】交给浏览器。惯于将HttpServletResponse接口修饰的对象称为【**响应对象**】

主要功能:

- 执行结果以二进制形式写入到【响应体】

- 设置响应头中[content-type]属性值，从而控制浏览器使用对应编译器将响应体二进制数据编译为【文字，图片，视频，命令】


```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) 
    throws ServletException, IOException {
         String result="Java<br/>Mysql<br/>HTML<br/>"; //既有文字信息又有HTML标签命令
        //设置响应头content-type
        response.setContentType("text/html;charset=utf-8");
        //向Tomcat索要输出流
        PrintWriter out = response.getWriter();
        //通过输出流将结果写入到响应体
        out.print(result);
    }//doGet执行完毕，Tomcat将响应包推送给浏览器
```

- 设置响应头中【location】属性，将一个请求地址赋值给location，从而控制浏览器向指定服务器发送请求


```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) 
		throws ServletException, IOException {
    String result ="http://www.baidu.com?userName=mike";
    //通过响应对象，将地址赋值给响应头中location属性
    response.sendRedirect(result);//[响应头  location="http://www.baidu.com"]
    }
    //浏览器在接收到响应包之后，如果发现响应头中存在location属性，自动通过地址栏向location指定网站发送请求。sendRedirect方法远程控制浏览器请求行为【请求地址，请求方式，请求参数】
```

## HttpServletRequest接口

来自于Servlet规范中，在Tomcat中存在servlet-api.jar，接口实现类由Http服务器负责提供。负责在doGet/doPost方法运行时读取Http请求协议包中信息，修饰的对象称为【请求对象】

作用:

- 读取Http请求协议包中【请求行】信息
- 读取保存在Http请求协议包中【请求头】或则【请求体】中请求参数信息
- 代替浏览器向Http服务器申请资源文件调用

```java
public class OneServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
        throws ServletException, IOException {
        //1.通过请求对象，读取【请求行】中【url】信息
         String url = request.getRequestURL().toString();
        //2.通过请求对象，读取【请求行】中【method】信息
         String method = request.getMethod();
        //3.通过请求对象，读取【请求行】中uri信息
        /*
        * URI：资源文件精准定位地址，在请求行并没有URI这个属性。
        *      实际上URL中截取一个字符串，这个字符串格式"/网站名/资源文件名"
        *      URI用于让Http服务器对被访问的资源文件进行定位
        */
        String uri =  request.getRequestURI();// substring
        System.out.println("URL "+url);
        System.out.println("method "+method);
        System.out.println("URI "+uri);
        
        //1.通过请求对象获得【请求头】中【所有请求参数名】
        Enumeration paramNames =request.getParameterNames(); 
        //将所有请求参数名称保存到一个枚举对象进行返回
        while(paramNames.hasMoreElements()){
            String paramName = (String)paramNames.nextElement();
            //2.通过请求对象读取指定的请求参数的值
            String value = request.getParameter(paramName);
            System.out.println("请求参数名 "+paramName+" 请求参数值 "+value);
    }
        /*
        浏览器以GET方式发送请求,请求参数保存在【请求头】,在Http请求协议包到达Http服务器之后，第一件事就是进行解码，请求头二进制内容由Tomcat负责解码，Tomcat9.0默认使用【utf-8】字符集，可以解释一切国家文字
	浏览器以POST方式发送请求，请求参数保存在【请求体】,在Http请求协议包到达Http服务器之后，第一件事就是进行解码，请求体二进制内容由当前请求对象（request）负责解码。request默认使用[ISO-8859-1]字符集，一个东欧语系字符集，此时如果请求体参数内容是中文，将无法解码只能得到乱码
	解决方案:在Post请求方式下，在读取请求体内容之前，应该通知请求对象使用utf-8字符集对请求体内容进行一次重新解码*/
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //通知请求对象，使用utf-8字符集对请求体二进制内容进行一次重写解码
        request.setCharacterEncoding("utf-8");
        //通过请求对象，读取【请求体】参数信息
        String value = request.getParameter("userName");
        System.out.println("从请求体得到参数值 "+value);
    }
}
```

> 请求对象和响应对象生命周期
>
> - ​       在Http服务器接收到浏览器发送的【Http请求协议包】之后，自动为当前的【Http请求协议包】生成一个【请求对象】和一个【响应对象】
> - 在Http服务器调用doGet/doPost方法时，负责将【请求对象】和【响应对象】作为实参传递到方法，确保doGet/doPost正确执行
> - 在Http服务器准备推送Http响应协议包之前，负责将本次请求关联的【请求对象】和【响应对象】
>   ​      销毁
>
> 【请求对象】和【响应对象】生命周期贯穿一次请求的处理过程中

## Http状态码

由三位数字组成的一个符号。Http服务器在推送响应包之前，根据本次请求处理情况将Http状态码写入到响应包中【状态行】上。

如果Http服务器针对本次请求，返回了对应的资源文件，通过Http状态码通知浏览器应该如何处理这个结果

如果Http服务器针对本次请求，无法返回对应的资源文件，通过Http状态码向浏览器解释不能提供服务的原因

分类：
100---599组成；分为5个大类
100; 通知浏览器本次返回的资源文件并不是一个独立的资源文件，需要浏览器在接收响应包之后，继续向Http服务器所要依赖的其他资源文件

200，通知浏览器本次返回的资源文件是一个完整独立资源文件，浏览器在接收到之后不需要所要其他关联文件

302，通知浏览器本次返回的不是一个资源文件内容而是一个资源文件地址，需要浏览器根据这个地址自动发起请求来索要这个资源文件

```java
response.sendRedirect("资源文件地址") //写入到响应头中location
//而这个行为导致Tomcat将302状态码写入到状态行
```

404: 通知浏览器，由于在服务端没有定位到被访问的资源文件 因此无法提供帮助

405：通知浏览器，在服务端已经定位到被访问的资源文件（Servlet），但是这个Servlet对于浏览器采用的请求方式不能处理

500:通知浏览器，在服务端已经定位到被访问的资源文件（Servlet）这个Servlet可以接收浏览器采用请求方式，但是Servlet在处理请求期间，由于Java异常导致处理失败

## 多个Servlet之间调用规则：

多个Servlet:

- 前提条件：某些来自于浏览器发送请求，往往需要服务端中多个Servlet协同处理。但是浏览器一次只能访问一个Servlet，导致用户需要手动通过浏览器发起多次请求才能得到服务。这样增加用户获得服务难度，导致用户放弃访问当前网站


- 提高用户使用感受规则：无论本次请求涉及到多少个Servlet,用户只需要【手动】通知浏览器发起一次请求即可

> 重定向解决方案

用户第一次通过【手动方式】通知浏览器访问OneServlet。OneServlet工作完毕后，将TwoServlet地址写入到响应头location属性中，导致Tomcat将302状态码写入到状态行。浏览器接收到响应包之后，读取到302状态。此时浏览器自动根据响应头中location属性地址【自动】发起第二次请求，访问TwoServlet去完成请求中剩余任务。

```java
response.sendRedirect("请求地址") //将地址写入到响应包中响应头中location属性
```

- 请求地址：既可以把当前网站内部的资源文件地址发送给浏览器 （/网站名/资源文件名）也可以把其他网站资源文件地址发送给浏览器(http://ip地址:端口号/网站名/资源文件名)
- 请求次数：浏览器至少发送两次请求，只有第一次请求是用户手动发送。后续请求都是浏览器自动发送的。
- 请求方式：通过地址栏通知浏览器发起下一次请求，因此通过重定向解决方案调用的资源文件接收的请求方式一定是【GET】
- 缺点:重定向解决方案需要在浏览器与服务器之间进行多次往返，大量时间消耗在往返次数上，增加用户等待服务时间

> 请求转发解决方案

 用户第一次通过手动方式要求浏览器访问OneServlet。OneServlet工作完毕后，通过当前的请求对象代替浏览器向Tomcat发送请求，申请调用TwoServlet。Tomcat接收到这个请求之后，自动调用TwoServlet完成剩余任务。

```java
//请求对象代替浏览器向Tomcat发送请求
//1.通过当前请求对象生成资源文件申请报告对象
RequestDispatcher  report = request.getRequestDispatcher("/资源文件名"); //一定要以"/"为开头
//2.将报告对象发送给Tomcat
report.forward(request, response)
```

- 无论本次请求涉及到多少个Servlet,用户只需要手动通过浏览器发送一次请求


- Servlet之间调用发生在服务端计算机上，节省服务端与浏览器之间往返次数增加处理服务速度


请求次数：在请求转发过程中，浏览器只发送一次请求

请求地址：只能向Tomcat服务器申请调用当前网站下资源文件地址

请求方式：在请求转发过程中，浏览器只发送一个了个Http请求协议包。参与本次请求的所有Servlet共享同一个请求协议包，因此这些Servlet接收的请求方式与浏览器发送的请求方式保持一致

## 多个Servlet之间数据共享实现方案：

数据共享：OneServlet工作完毕后，将产生数据交给TwoServlet来使用

Servlet规范中提供四种数据共享方案

- **ServletContext接口**

来自于Servlet规范中一个接口。在Tomcat中存在servlet-api.jar，在Tomcat中负责提供这个接口实现类。

如果两个Servlet来自于同一个网站。彼此之间通过网站的ServletContext实例对象实现数据共享。

习惯于将ServletContext对象称为【全局作用域对象】。

> 每一个网站都存在一个全局作用域对象。 这个全局作用域对象【相当于】一个Map.在这个网站中OneServlet可以将一个数据存入到全局作用域对象，当前网站中其他Servlet此时都可以从全局作用域对象得到这个数据进行使用。

> 生命周期：全局作用域对象生命周期贯穿网站整个运行期间

在Http服务器启动过程中，自动为当前网站在内存中创建一个全局作用域对象

在Http服务器运行期间时，一个网站只有一个全局作用域对象，全局作用域对象一直处于存活状态

在Http服务器准备关闭时，负责将当前网站中全局作用域对象 进行销毁处理          

```java
// 命令实现： 【同一个网站】OneServlet将数据共享给TwoServlet
public class OneServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
        throws ServletException, IOException {
        //1.通过请求对象向Tomcat索要当前网站全局作用域对象
        ServletContext application = request.getServletContext();
        //2.将数据添加到全局作用域对象，作为共享数据
        application.setAttribute("key1", 100);// map: key-value
    }
}
```

```java
public class TwoServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.通过请求对象向Tomcat索要当前网站全局作用域对象
        ServletContext application = request.getServletContext();
        //2.从全局作用域对象得到指定关键字对应的值
        Integer money=(Integer)application.getAttribute("key1");
    }
}
```

- **Cookie类**

来自于Servlet规范中一个工具类，存在于Tomcat提供servlet-api.jar中。如果两个Servlet来自于同一个网站，并且为同一个浏览器/用户提供服务，此时借助于Cookie对象进行数据共享

Cookie存放当前用户的私人数据，在共享数据过程中提高服务质量。在现实生活场景中，Cookie相当于用户在服务端得到【会员卡】

原理: 用户通过浏览器第一次向MyWeb网站发送请求申请OneServlet。OneServlet在运行期间创建一个Cookie存储与当前用户相关数据。OneServlet工作完毕后，【**将Cookie写入到响应头**】交还给当前浏览器。浏览器收到响应响应包之后，将cookie存储在浏览器的缓存一段时间之后，用户通过【同一个浏览器】再次向【myWeb网站】发送请求申请TwoServlet时。【**浏览器需要无条件的将myWeb网站之前推送过来的Cookie，写入到请求头**】发送过去。此时TwoServlet在运行时，可以通过读取请求头中cookie中信息，得到OneServlet提供的共享数据。

```java
//实现命令:  同一个网站 OneServlet 与  TwoServlet 借助于Cookie实现数据共享
OneServlet{
    public void doGet(HttpServletRequest request,HttpServletResponse resp){
        //1.创建一个cookie对象，保存共享数据（当前用户数据）
        Cookie card = new Cookie("key1","abc");// Cookie(String name,String value)
        Cookie card1= new Cookie("key2","efg");
		//cookie相当于一个map,一个cookie中只能存放一个键值对
        //这个键值对的key与value只能是String,键值对中key不能是中文
        
        //2.【发卡】将cookie写入到响应头，交给浏览器
        resp.addCookie(card);
        resp.addCookie(card1);
    }
}

TwoServlet{			 
    public void doGet(HttpServletRequest request,HttpServletResponse resp){
        //1.调用请求对象从请求头得到浏览器返回的Cookie
        Cookie  cookieArray[] = request.getCookies();
        //2.循环遍历数据得到每一个cookie的key 与 value
        for(Cookie card:cookieArray){
            String key =   card.getName(); //读取key  "key1"
            String value = card.getValue();//读取value "abc"
        }
    }
}
```

> Cookie销毁时机

在默认情况下，Cookie对象存放在浏览器的缓存中。只要浏览器关闭，Cookie对象就被销毁

在手动设置情况下，可以要求浏览器将接收的Cookie 存放在客户端计算机上硬盘上，同时需要指定Cookie在硬盘上存活时间。在存活时间范围内，关闭浏览器关闭客户端计算机，关闭服务器，都不会导致Cookie被销毁。在存活时间到达时，Cookie自动从硬盘上被删除

```
cookie.setMaxAge(60); //cookie在硬盘上存活1分钟
```

Cookie域保存在自己浏览器内部，与别人互不干扰，但因为是客户端技术，所以安全性不高。

- **HttpSession接口**

来自于Servlet规范下一个接口。存在于Tomcat中servlet-api.jar，其实现类由Http服务器提供。Tomcat提供实现类存在于servlet-api.jar。如果两个Servlet来自于同一个网站，并且为同一个浏览器/用户提供服务，此时借助于HttpSession对象进行数据共享。习惯于将HttpSession接口修饰对象称为【会话作用域对象】

> HttpSession 与  Cookie 区别：
>
> 存储位置:Cookie：存放在客户端计算机（浏览器内存/硬盘）。HttpSession：存放在服务端计算机内存
>
> 数据类型：Cookie对象存储共享数据类型只能是String。HttpSession对象可以存储任意类型的共享数据Object
>
> 数据数量: 一个Cookie对象只能存储一个共享数据。HttpSession使用map集合，可以存储任意数量共享数据
>
> 参照物：Cookie相当于客户在服务端【会员卡】。HttpSession相当于客户在服务端【私人保险柜】

```java
//命令实现:   同一个网站（myWeb）下OneServlet将数据传递给TwoServlet
OneServlet{
    public void doGet(HttpServletRequest request,HttpServletResponse response){
        //1.调用请求对象向Tomcat索要当前用户在服务端的私人储物柜
        HttpSession session = request.getSession();
        //2.将数据添加到用户私人储物柜
        session.setAttribute("key1",共享数据)
    }
}
//浏览器访问/myWeb中TwoServlet
TwoServlet{
    public void doGet(HttpServletRequest request,HttpServletResponse response){
        //1.调用请求对象向Tomcat索要当前用户在服务端的私人储物柜
        HttpSession session = request.getSession();
        //2.从会话作用域对象得到OneServlet提供的共享数据
        Object 共享数据 = session.getAttribute("key1");
    }
}
```

Http服务器如何将用户与HttpSession关联：cookie

> getSession()  与  getSession(false)
>
> getSession(): 如果当前用户在服务端已经拥有了自己的私人储物。要求tomcat将这个私人储物柜进行返回。如果当前用户在服务端尚未拥有自己的私人储物柜。要求tocmat为当前用户创建一个全新的私人储物柜
>
> getSession(false):如果当前用户在服务端已经拥有了自己的私人储物柜.要求tomcat将这个私人储物柜进行返回。如果当前用户在服务端尚未拥有自己的私人储物柜。此时Tomcat将返回null

HttpSession销毁时机:

用户与HttpSession关联时使用的Cookie只能存放在浏览器缓存中，在浏览器关闭时，意味着用户与他的HttpSession关系被切断。由于Tomcat无法检测浏览器何时关闭，因此在浏览器关闭时并不会让Tomcat将浏览器关联的HttpSession进行销毁。为了解决这个问题，Tomcat为每一个HttpSession对象设置【空闲时间】，空闲时间默认30分钟，如果当前HttpSession对象空闲时间达到30分钟，此时Tomcat认为用户已经放弃了自己的HttpSession，Tomcat就会销毁这个HttpSession

- **HttpServletRequest接口**

在同一个网站中，如果两个Servlet之间通过【请求转发】方式进行调用，彼此之间共享同一个请求协议包。而一个请求协议包只对应一个请求对象。因此servlet之间共享同一个请求对象，此时可以利用这个**请求对象**在两个Servlet之间实现数据共享。

将请求对象称为【请求作用域对象】

```java
// OneServlet通过请求转发申请调用TwoServlet时，需要给TwoServlet提供共享数据
OneServlet{				 
    public void doGet(HttpServletRequest req,HttpServletResponse response){
        //1.将数据添加到【请求作用域对象】中attribute属性
        req.setAttribute("key1",数据); //数据类型可以任意类型Object
        //2.向Tomcat申请调用TwoServlet
        req.getRequestDispatcher("/two").forward(req,response)
    }
}
TwoServlet{
    public void doGet(HttpServletRequest req,HttpServletResponse response){
        //从当前请求对象得到OneServlet写入到共享数据
        Object 数据 = req.getAttribute("key1");
    }				 
}
```

## 监听器接口

一组来自于Servlet规范下接口，共有8个接口。在Tomcat存在servlet-api.jar包。监听器接口需要由开发人员亲自实现，Http服务器提供jar包并没有对应的实现类。监听器接口用于监控【**作用域对象**生命周期变化时刻】以及【作用域对象共享数据变化时刻】

> 作用域对象：在Servlet规范中，认为在**服务端内存**中可以在某些条件下为两个Servlet之间提供数据共享方案的对象，被称为【作用域对象】
>
> Servlet规范下作用域对象:
>
> - ServletContext：全局作用域对象
> - HttpSession:  会话作用域对象
> - HttpServletRequest: 请求作用域对象
>
> (cookie存放在客户端，故不属于作用域对象)

监听器接口实现类开发规范：根据监听的实际情况，选择对应监听器接口进行实现；重写监听器接口声明【监听事件处理方法】；在web.xml文件将监听器接口实现类注册到Http服务器

```java
public class OneListener implements ServletContextListener{
	@Override
    public void contextInitialized(ServletContextEvent sce) {
		//...
    }
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
		//...
    }
}
```

```xml
<!--将监听器接口实现类注册到Tomcat-->
<listener>
	<listener-class>com.bjpoewrnode.listener.OneListener</listener-class>
</listener>
```

- ServletContextListener接口

合法的检测全局作用域对象被初始化时刻以及被销毁时刻

```java
public void contextInitlized(ServletContextEvent sce)//在全局作用域对象被Http服务器初始化被调用
public void contextDestory(ServletContextEvent sce)//在全局作用域对象被Http服务器销毁时触发调用
```

- ServletContextAttributeListener接口


合法的检测全局作用域对象共享数据变化时刻

```java
public void contextAdd() //在全局作用域对象添加共享数据
public void contextReplaced() //在全局作用域对象更新共享数据
public void contextRemove() //在全局作用域对象删除共享数据
```

```java
// 全局作用域对象共享数据变化时刻
ServletContext application = request.getServletContext();
application.setAttribute("key1",100); //新增共享数据
application.setAttribute("key1",200); //更新共享数据
application.removeAttribute("key1");  //删除共享数据
```

### 过滤器接口

来自于Servlet规范下接口，在Tomcat中存在于servlet-api.jar包。Filter接口实现类由开发人员负责提供，Http服务器不负责提供。Filter接口在Http服务器调用资源文件之前，对Http服务器进行拦截。

> 拦截Http服务器，帮助Http服务器检测当前请求合法性；对当前请求进行增强操作

Filter接口实现类开发步骤：创建一个Java类实现Filter接口；重写Filter接口中doFilter方法；web.xml将过滤器接口实现类注册到Http服务器

```java
public class OneFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //合法请求
        //将拦截请求对象和响应对象交还给Tomcat,由Tomcat继续调用资源文件
        if(...){ 
        //增强功能，通知拦截的请求对象，使用UTF-8字符集对当前请求体信息进行一次重新编辑(POST())
        servletRequest.setCharacterEncoding("utf-8");
        filterChain.doFilter(servletRequest, servletResponse);//FilterChain的doFilter方法
        }
        // 否则过滤器代替Http服务器拒绝本次请求
    }
}
```

```xml
 <!--将过滤器类文件路径交给Tomcat-->
<filter>
    <filter-name>oneFilter</filter-name>
    <filter-class>com.bjpowernode.filter.OneFilter</filter-class>
</filter>
<!--通知Tomcat在调用何种资源文件时需要被当前过滤器拦截-->
<filter-mapping>
    <filter-name>oneFilter</filter-name>
    <url-pattern>拦截地址</url-pattern>
</filter-mapping>

<!--要求Tomcat在调用某一个具体文件之前，来调用OneFilter拦截--->
<url-pattern>/img/mm.jpg</url-pattern>
<!--要求Tomcat在调用某一个文件夹下所有的资源文件之前，来调用OneFilter拦截--->
<url-pattern>/img/*</url-pattern>
<!--要求Tomcat在调用任意文件夹下某种类型文件之前，来调用OneFilter拦截--->
<url-pattern>*.jpg</url-pattern>
<!--要求Tomcat在调用网站中任意文件时，来调用OneFilter拦截--->
<url-pattern>/*</url-pattern>
```

```java
// 使用过滤器避免恶意登录（避开登陆界面直接通过地址栏访问网站内资源文件）
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //...
        //调用Dao将查询验证信息推送到数据库服务器上
        result = dao.login(userName, password);
        if(result ==1){//用户存在
            //在判定来访用户身份合法后，通过请求对象向Tomcat申请为当前用户申请一个HttpSession令牌
            HttpSession session = request.getSession();
            response.sendRedirect("/myWeb/index.html");
        }else{
            response.sendRedirect("/myWeb/login_error.html");
        }
    }
}

public class OneFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        // 与login相关的不过滤
        String uri = request.getRequestURI();
        if(uri.indexOf("login") != -1 || "/myWeb/".equals(uri)){
            filterChain.doFilter(servletRequest, servletResponse);
        }
        //1.拦截后，通过请求对象向Tomcat索要当前用户的HttpSession。
        HttpSession session = request.getSession(false);
        //getSession(false):如果当前用户在服务端已经拥有了自己的私人储物柜.要求tomcat将这个私人储物柜进行返回。如果当前用户在服务端尚未拥有自己的私人储物柜。此时Tomcat将返回null
        //2.判断来访用户身份合法性
        if(session == null){
            request.getRequestDispatcher("/login_error.html")
                .forward(servletRequest, servletResponse);
            return;
        }
        //3.放行
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

```xml
<!--通知Tomcat在调用任意文件之前都要调用当前过滤器-->
<url-pattern>/*</url-pattern>
```