---
tags: java
title: Java/JSP
---

> JSP规范来自于JAVAEE规范中一种，制定了如何开发JSP文件**代替响应对象将处理结果写入到响应体**的开发流程制，Http服务器应该如何调用管理JSP文件。放在`/web/*.jsp`下

<!-- more -->

## JSP规范

> 响应对象存在弊端

- 适合将数据量较少的处理结果写入到响应体。如果处理结果数量过多，使用响应对象增加开发难度

> JSP文件优势

- JSP文件在互联网通信过程，是响应对象替代品。降低将处理结果写入到响应体的开发工作量，降低处理结果维护难度。

- 在JSP文件开发时，可以直接将处理结果写入到JSP文件，不需要手写out.print命令。

- 在Http服务器调用JSP文件时，根据JSP规范要求自动将JSP文件书写的所有内容通过输出流写入到响应体。

> HTML文件与JSP文件区别

- 作为资源文件类型不同

HTML文件属于静态资源文件，其相关命令需要在浏览器编译并执行的.

JSP文件属于动态资源文件，其相关命令需要在服务端编译并执行的

- 调用形式不同

如果浏览器访问HTML文件，此时Http服务器直接通过一个输出流将HTML文件中所有的内容写入到响应体

如果浏览器访问JSP文件。此时Http服务器根据JSP规范来操作JSP文件编辑---->编译----->调用

```jsp
<!--在JSP文件中直接书写Java命令，不能被JSP文件识别，此时只会被当做字符串写入到响应体-->
<%
  //在只有书写在执行标记<%中内容才会被当做Java命令
  //1.声明Java变量
  int num1 = 100;
  int num2 = 200;
  //2.声明运行表达式：数学运算，关系运算，逻辑运算
  int num3 = num1 + num2; //数学运算
  int num4 = num2>=num1?num2:num1;//关系运算
  boolean num5 = num2>=200 && num1>=100;//逻辑运算
  //3.声明控制语句
   if(num2>=num1){
       //...
   }else{
       //...
   }

   for(int i=1;i<=10;i++){
   }
%>
```

```jsp
<%
   int num1 =100;
   int num2 =200;
%>

<!--在JSP文件，通过输出标记，通知JSP将Java变量的值写入到响应体-->
变量num1的值:<%=num1%><br/>
变量num2的值:<%=num2%><br/>
<!--执行标记还可以通知Jsp将运算结果写入到响应体-->
num1 + num2 = <%=num1+num2%>
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--制造数据-->
<%
       Student stu1 = new Student(10,"mike");
       Student stu2 = new Student(20,"allen");
       Student stu3 = new Student(30,"smith");
       List<Student> list = new ArrayList();
       list.add(stu1);
       list.add(stu2);
       list.add(stu3);
%>

<!--数据输出-->
<table border="2" align="center">
    <tr>
        <td>学员编号</td>
        <td>学员姓名</td>
    </tr>

    <%
       for(Student stu:list){
    %>
        <tr>
            <td><%=stu.getSid()%></td>
            <td><%=stu.getSname()%></td>
        </tr>
    <%
       }
    %>
</table>
```

## JSP文件内置对象

### request

```jsp
<!--
   JSP文件内置对象: request
             类型：HttpServletRequest
             作用: 在JSP文件运行时读取请求包信息
                  与Servlet在请求转发过程中实现数据共享

  浏览器： http://localhost:8080/myWeb/request.jsp?userName=allen&password=123
-->

<%
   //在JSP文件执行时，借助于内置request对象读取请求包参数信息
    String userName = request.getParameter("userName");
    String password =request.getParameter("password");
%>

来访用户姓名:<%=userName%><br/>
来访用户密码:<%=password%>
```

### session

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--
    JSP文件内置对象:session
              类型:HttpSession
              作用：JSP文件在运行时，可以session指向当前用户私人储物柜，添加共享数据，或则读取共享数据
-->

<!--session1.jsp,将共享数据添加到当前用户私人储物柜-->
<%
   // HttpSession session = request.getSession();
   session.setAttribute("key1", 200);
%>
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--
      session_1.jsp 与session_2.jsp为同一个用户/浏览器提供服务。
      因此可以使用当前用户在服务端的私人储物柜进行数据共享
 -->
<%
     Integer value=(Integer) session.getAttribute("key1");
%>
session_2.jsp从当前用户session中读取数据:<%=value%>
```

### application

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<!--
       JSP文件内置对象 ： application
       ServletContext application;全局作用域对象
       同一个网站中Servlet与JSP，都可以通过当前网站的全局作用域对象实现数据共享       
-->

<%
    application.setAttribute("key1", "hello world");
%>
```

## JSP文件原理

Tomcat根据JSP规范，将被访问的JSP文件[编辑]为一个java文件。这个Java文件是Servlet接口实现类；调用JVM（javac one_jsp.java）将这个java文件[编译]为class类型；生成这个class文件的实例对象（Servelt接口实例对象）；通过实例对象调用class文件中\_jspService方法，\_jspService方法在运行时负责将JSP文件中书写内容写入到响应体中

> \_jspService方法内部结构
>
> 判断当前请求方式。Jsp文件可以接收的请求方式有POST,GET,HEAD
>
> 声明局部变量。这些局部变量都可以在JSP文件开发时直接使用
>
> 输出部分。这部分执行时将JSP文件内容通过输出流写入到响应体

## Servlet 与 JSP

JSP文件被访问时，并不是JSP文件在执行，而是对应的Servlet在执行。自定义Serlvet接口实现类与JSP文件之间调用关系，等同于两个Servlet之间调用关系

- Servlet 与JSP 分工


Servlet：负责处理业务并得到【处理结果】

JSP：不负责业务处理，主要任务将Servlet中【处理结果】写入到响应体

- Servlet 与  JSP 之间调用关系


Servlet工作完毕后，一般通过请求转发方式 向Tomcat申请调用JSP

- Servlet  与 JSP 之间实现数据共享


Servlet将处理结果添加到【请求作用域对象】

JSP文件在运行时从【请求作用域对象】得到处理结果

```java
public class OneServlet extends HttpServlet {

    //处理业务，得到处理结果-----查询信息
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Student s1 = new Student(10,"mike");
        Student s2 = new Student(20,"allen");
        List stuList = new ArrayList();
        stuList.add(s1);
        stuList.add(s2);

        //将处理结果添加到请求作用域对象
        request.setAttribute("key", stuList);

        //通过请求转发方案，向Tomcat申请调用user_show.jsp
        //同时将request与response通过tomcat交给user_show.jsp使用
        request.getRequestDispatcher("/user_show.jsp").forward(request, response);
    }
}
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
   //从请求作用域对象得到OneServlet添加进去的集合
     List<Student> stuList = (List)request.getAttribute("key");
%>
<!--将处理结果写入到响应体-->
<table border="2" align="center">
    <tr>
        <td>用户编号</td>
        <td>用户姓名</td>
    </tr>
    <%
       for(Student stu:stuList){
    %>
        <tr>
            <td><%=stu.getSid()%></td>
            <td><%=stu.getSname()%></td>
        </tr>
    <%
       }
    %>
</table>
```

## EL表达式

Tomcat服务器本身自带了EL工具包（Tomcat安装地址/lib/el-api.jar）

格式：${作用域对象别名.共享数据}

作用：EL表达式是EL工具包提供一种特殊命令格式【表达式命令格式】，在JSP文件上使用，负责在JSP文件上从作用域对象读取指定的共享数据并输出到响应体。

`<%@ page isELIgnored="true" %>` 表示是否禁用EL语言,TRUE表示禁止.FALSE表示不禁止

### JSP文件可以使用的作用域对象

- ServletContext application:  全局作用域对象


- HttpSession session: 会话作用域对象


- HttpServletRequest request: 请求作用域对象


- PageContext  pageContext：当前页作用域对象，这是JSP文件独有的作用域对象。Servlet中不存在在当前页作用域对象存放的共享数据仅能在当前JSP文件中使用，不能共享给其他Servlet或则其他JSP文件真实开发过程，主要用于JSTL标签与JSP文件之间数据共享数据（JSTL------->pageContext---->JSP）

> EL表达式提供作用域对象别名
>
> ​          JSP                       	EL表达式
>
> ​      application                 ${applicationScope.共享数据名}
>
> ​      session                   	${sessionScope.共享数据名}
>
> ​      request                   	${requestScope.共享数据名}
>
> ​      pageContext              ${pageScope.共享数据名}

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
     Integer sid =(Integer)application.getAttribute("sid");
     String  home=  (String)request.getAttribute("home");
	 Student stu= (Student)request.getAttribute("key");
%>
学员ID:<%=sid%><br/>
学员地址:<%=home%><br/>
学员编号:<%=stu.getSid()%><br/>
学员姓名:<%=stu.getSname()%>
<!---EL表达式-->
<hr/>
学员ID:  ${applicationScope.sid}<br/>
学员地址：${requestScope.home}<br/>
<!---将引用对象属性写入到响应体-->
学员编号:${requestScope.key.sid}<br/>
学员姓名:${requestScope.key.sname}
```

**EL表达式没有提供遍历集合方法，因此无法从作用域对象读取集合内容输出**

### EL表达式简化版

EL表达式允许开发时省略作用域对象别名。命令格式： ${共享数据名}

EL表达式简化版由于没有指定作用域对象，所以在执行时采用【猜】算法

> 首先到【pageContext】定位共享数据，如果存在直接读取输出并结束执行
> 如果在【pageContext】没有定位成功，到【request】定位共享数据，如果存在直接读取输出并结束执行
> 如果在【request】没有定位成功，到【session】定位共享数据，如果存在直接读取输出并结束执行
> 如果在【session】没有定位成功，到【application】定位共享数据，如果存在直接读取输出并结束执行
> 如果在【application】没有定位成功，返回null
>
>  pageContext--->request--->session--->application

存在隐患：容易降低程序执行速度；容易导致数据定位错误

应用场景：设计目的就是简化从pageContext读取共享数据并输出的难度

EL表达式简化版尽管存在很多隐患，但是在实际开发过程中为了节省时间一都使用简化版。

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
标准版EL表达式输出session中key的值:${sessionScope.key}<br/>
简化版EL表达式输出session中key的值:${key}<br/>
```

### 支持运算表达式

在JSP文件有时需要将读取共享数据进行运算之后，将运算结果写入到响应体

> 运算表达式包括：
>
> 1) 数学运算
>
> 2) 关系运算:  >    >=   ==    <   <=  !=
>
> ​	                   gt   ge   eq    lt  le   !=
>
> 3)逻辑运算：  &&   ||    ！

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--EL表达式支持运算表达式-->
<!--将作用域对象中共享数据读取出来相加，将相加结果写入到响应体-->
<%
     String num1 = (String)request.getAttribute("key1");
     Integer num2 = (Integer)request.getAttribute("key2");
     int sum = Integer.valueOf(num1) + num2;
%>
传统的Java命令计算后的结果:<%=sum%>
EL表达式计算后的结果:${key1+key2}
EL表达式输出关系运算:${age ge 12?"欢迎光临":"谢绝入内"}
```

### EL表达式提供内置对象	

#### param

命令格式: ${param.请求参数名}

命令作用： 通过请求对象读取当前请求包中请求参数内容，并将请求参数内容写入到响应体

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %> 
<%
 String userName = request.getParameter("userName");
 String password = request.getParameter("password");
 %>
来访者姓名:<%=userName%>
来访者密码:<%=password%>

<!--使用EL表达式内置对象-->
<!--
   http://localhost:8080/myWeb/index_1.jsp?userName=mike&password=123
-->
来访者姓名:${param.userName}<br/>
来访者密码:${param.password}
```

#### paramValues

命令格式：${paramValues.请求参数名[下标]}

命令作用: 如果浏览器发送的请求参数是[一个请求参数关联多个值]，此时可以通过paramVaues读取请求参数下指定位置的值，并写入到响应体。此时pageNo请求参数在请求包以数组形式存在

```jsp
<%
	String  array[]= request.getParameterValues("pageNo");
%>
第一个值:<%=array[0]%>
第二个值:<%=array[1]%>

<!--使用EL表达式内置对象-->
<%@ page contentType="text/html;charset=UTF-8" language="java" %> 
<!--
http://localhost:8080/myWeb/index_2.jsp?deptNo=10&deptNo=20&deptNo=30
-->
第一个部门编号:${paramValues.deptNo[0]}<br/>
第二个部门编号:${paramValues.deptNo[1]}<br/>
第三个部门编号:${paramValues.deptNo[2]}<br/>
```

 header/headerValues/cookie

### EL表达式常见异常

`javax.el.PropertyNotFoundException`在对象中没有找到指定属性