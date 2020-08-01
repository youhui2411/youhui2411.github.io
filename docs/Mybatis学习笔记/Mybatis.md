# Mybatis学习笔记

## 一、Mybatis简介

1、开源免费框架，原名叫iBatis，2010在迁移google code，并改名为MyBatis，2013年迁移到Github

2、作用：数据访问层框架，底层是对JDBC的封装

3、MyBatis的优点之一：使用MyBatis时不需要编写实现类,只需要写需要执行的 sql 命令

## 二、设计数据库

1、创建表：

```mysql
create table book(
id int(10) primary key auto_increment comment '编号',
name varchar(30) not null comment '书名',
price float not null comment '价格',
press varchar(30) not null comment '出版社'
);

```

2、录入测试数据：

```mysql
insert into book values(default,'Java编程思想',100.30,'机械工业出版社');
insert into book values(default,'Head First设计模式',65.00,'中国电力出版社');
insert into book values(default,'MySQL必知必会',22.98,'人民邮电出版社');
insert into book values(default,'Java并发编程实战',54.40,'机械工业出版社');

```

## 三、传统方式

### 一、实体类代码

```java
public class Book {
	private int id;
	private String name;
	private double price;
	private String press;
	
	public Book() {
		super();
	}
	public Book(int id, String name, double price, String press) {
		super();
		this.id = id;
		this.name = name;
		this.price = price;
		this.press = press;
	}
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public double getPrice() {
		return price;
	}
	public void setPrice(double price) {
		this.price = price;
	}
	public String getPress() {
		return press;
	}
	public void setPress(String press) {
		this.press = press;
	}
}

```

### 二、数据访问层代码

```java
public interface BookDao {
	/**
	 * 查询全部
	 * @return
	 */
	List<Book> selAll();
	/**
	 * 新增
	 * @return
	 */
	int insBook(Book book);
}

```

### 三、数据访问层实现类代码

```java
public class BookDaoImpl implements BookDao{

	@Override
	public List<Book> selAll() {
		List<Book> list=new ArrayList<>();
		Connection conn=null;
		PreparedStatement ps=null;
		ResultSet rs=null;
		try {
			Class.forName("com.mysql.jdbc.Driver");
			conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/ssm","root","123456");
			ps = conn.prepareStatement("select * from book");
			rs = ps.executeQuery();
			while(rs.next()){
				list.add(new Book(rs.getInt(1), rs.getString(2), rs.getDouble(3), rs.getString(4)));
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		}finally{
			try {
				rs.close();
				ps.close();
				conn.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
		return list;
	}

	@Override
	public int insBook(Book book) {
		int  index= 0;
		Connection conn=null;
		PreparedStatement ps=null;
		try {
			Class.forName("com.mysql.jdbc.Driver");
			conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/ssm","root","123456");
			ps = conn.prepareStatement("insert into book values(default,?,?,?)");
			ps.setObject(1, book.getName());
			ps.setObject(2, book.getPrice());
			ps.setObject(3, book.getPress());
			index = ps.executeUpdate();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		}finally{
			try {
				ps.close();
				conn.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
		return index;
	}
}

```

### 四、业务层代码

```java
public interface BookService {
	/**
	 * 显示所有书籍信息
	 * @return
	 */
	List<Book> show();
	/**
	 * 新增
	 * @return
	 */
	int add(Book book);
}

```

### 五、业务层实现类代码

```java
public class BookServiceImpl implements BookService{
	private BookDao bookDao = new BookDaoImpl();
	@Override
	public List<Book> show() {
		return bookDao.selAll();
	}
	@Override
	public int add(Book book) {
		return bookDao.insBook(book);
	}

}

```

### 六、控制器代码

```java
@WebServlet("/insert")
public class InsertServlet extends HttpServlet{
	private BookService bookService =new BookServiceImpl();
	@Override
	protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		req.setCharacterEncoding("utf-8");
		String name = req.getParameter("name");
		String price = req.getParameter("price");
		String press = req.getParameter("press");
		Book book = new Book();
		book.setName(name);
		book.setPrice(Double.parseDouble(price));
		book.setPress(press);
		int index = bookService.add(book);
		if(index>0)
			resp.sendRedirect("show");
		else
			resp.sendRedirect("add.jsp");
	}
}

```

### 七、视图层代码

1、index.jsp中代码

```jsp
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>My JSP 'index.jsp' starting page</title>
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
	<meta http-equiv="description" content="This is my page">
	<!--
	<link rel="stylesheet" type="text/css" href="styles.css">
	-->
	<style type="text/css">
		a{
			color:black;
			text-decoration:none;
		}
		a:hover{
			color:red;
		}
	</style>
  </head>
  <body>
	  <table border='1'>
		<tr>
			<th>书籍编号</th>
			<th>书籍名称</th>
			<th>价格(元)</th>
			<th>出版社</th>
		</tr>
		<c:forEach items="${list }" var="book">
			<tr>
				<td>${book.id }</td>
				<td>${book.name }</td>
				<td>${book.price }</td>
				<td>${book.press }</td>
			</tr>
		</c:forEach>
	</table>
	<a href="add.jsp"> 添加书籍信息</a>
  </body>
</html>

```

2、add.jsp中代码

```jsp
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>My JSP 'index.jsp' starting page</title>
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
	<meta http-equiv="description" content="This is my page">
	<!--
	<link rel="stylesheet" type="text/css" href="styles.css">
	-->
	<script type="text/javascript" src="js/jquery-1.7.2.js"></script>
	<script type="text/javascript">
	$(function(){
		$("form").submit(function(){
	if($(":text:eq(0)").val()==""||$(":text:eq(1)").val()==""||$(":text:eq(2)").val()==""){
				alert("请填写完整信息");
				return false;
			}
		});
	});
	</script>
  </head>
  <body>
	  <form action="insert" method="post">
		<table border="1" align="center">
			<tr>
				<td colspan="2" style="text-align:center;font-size:30px;font-weight:bold;">书籍信息</td>
			</tr>
			<tr>
				<td><b>书籍名称:</b></td>
				<td><input type="text" name="name"/></td>
			</tr>
			<tr>
				<td><b>书籍价格:</b></td>
				<td><input type="text" name="price"/></td>
			</tr>
			<tr>
				<td><b>出版社:</b></td>
				<td><input type="text" name="press"/></td>
			</tr>
			<tr>
				<td colspan="2" align="center">
					<input type="submit" value="提交"/>
					<input type="reset" value="重置"/>
				</td>
			</tr>
		</table>
	</form>
  </body>
</html>

```

### 八、运行

1、在tomcat中启动项目，并在浏览器地址栏中输入***http://localhost:8080/book/show***，可以看到如下图：

![image-20200731160018376](\image\image-20200731160018376.png)

2、添加书籍信息：

![image-20200731160038204](\image\image-20200731160038204.png)

3、添加书籍信息后，如图：

![image-20200731160054196](\image\image-20200731160054196.png)