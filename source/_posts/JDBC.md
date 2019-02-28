---
title: JDBC
date: 2018-10-17 17:29:06
tags:
comments: true
copyright: true
categories:
 - [Java, MySQL]
---

# JDBC

@(Java)

***
## ResultSetMetaData
[toc]
### 1. MetaData--元数据

* 是指结果集对象的相关其他数据,比如说总列数,每一列的名称,每一列的sql数据类型,每一列的java数据类型等
> 案例演示
```
  public static void main(String[] args) {
        Connection conn=null;
        String sql="select id,name,password  from dept where id<?";
        try {
            conn=DBUtiles.getConnection();
            PreparedStatement ps=conn.prepareStatement(sql);
            ps.setInt(1, 100);
            ResultSet rs=ps.executeQuery();
            //获取结果集的元数据
            ResultSetMetaData meta=rs.getMetaData();
            //遍历并显示结果集中所有列的名称       
            for (int i = 1; i <= meta.getColumnCount(); i++) {
                System.out.println(meta.getColumnName(i));
            }
            //获得结果集的所有列数量
//          int n=meta.getColumnCount();
//          System.out.println(n);
            //通过下标获得结果集的某一列的列名
//          String name1=meta.getColumnName(1);
//          String name2=meta.getColumnName(2);
//          System.out.println(name1);
//          System.out.println(name2);
//          while (rs.next()) {
//              System.out.println(rs.getString("name"));
//          }
            //关闭结果集
            rs.close();
            //关闭PS对象
            ps.close();
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }finally {
            DBUtiles.closeConnection(conn);
        }
    }

}
```
<!-- more -->

***
### 2.由于jdbc自动提交事务,所以需要手动关闭
> 案例演示
```
public class Demo2 {
/**
 * 由于JDBC自动事务提交
 * 1.关闭自动提交conn.setAutoCommit(false);
 * 2.try的最后部分手动提交conn.commit();
 * 3.由于一出现错误,就会运行catch块
 *   在catch中conn.rollback();
 * 4.封装回滚方法
 * @param args
 */
    static String sql1="update bal set"
            + " money=money+? where id=?";
    static String sql2="select money from bal where id=?";

    public static void main(String[] args) {
        pay(2, 4, 1000);
    }
    public static void pay(int from,int to,double money){
        Connection conn=null;
        try {
            conn=DBUtiles.getConnection();
            //关闭自动commit
            conn.setAutoCommit(false);
            PreparedStatement ps=conn.prepareStatement(sql1);
            /**
             * 业务模块---a-->b 打1000
             * create table bal(
             * id int,
             * name varchar(20),
             * money double(8,2)
             * )
             */
            //a-1000
            ps.setDouble(1, -money);
            ps.setInt(2, from);
            int n=ps.executeUpdate();
            //更新失败
            if (n!=1) {
                throw new Exception("减钱失败");
            }
            //b+1000
            ps.setDouble(1, money);
            ps.setInt(2, to);
            n=ps.executeUpdate();
            if (n!=1) {
                throw new Exception("加钱失败");
            }
            ps.close();
            //检查a有没有1000
            ps=conn.prepareStatement(sql2);
            ps.setInt(1, from);
            ResultSet rs=ps.executeQuery();
            while (rs.next()) {
                double bal=rs.getDouble(1);
                if (bal<0) {
                    throw new Exception("余额不足");
                }
            }
            //手动提交
            conn.commit();
        } catch (Exception e) {
            e.printStackTrace();
            DBUtiles.rollback(conn);
        }finally {
            DBUtiles.closeConnection(conn);
        }
    }
}
```
***
### 批量更新

-  把多条sql存入`Statement`对象的缓存,一次性发送给DB执行
```
Statement sta=conn.createStatement();
//          sta.addBatch(sql1);
//          sta.addBatch(sql2);         
//          int[] arr=sta.executeBatch();
```
- 创建一个执行计划,把多条参数存入`PreParedStatement`对象缓存中,一次性发送给DB执行


```
PreparedStatement ps=conn.prepareStatement(sql6);           
ps.setInt(1, 1);
ps.setString(2, "lili");
ps.addBatch();          
int[] arr=ps.executeBatch();
```
>内存溢出OutOfMemory

### 获取自动生成的主键


- ` String[] colNames={"id"};` 自动生成值的列的列名

- `PreparedStatement ps =conn.prepareStatement(sql1, colNames); `必须与上面的命令一起使用 `ps.getGeneratedKeys()`

> 案例演示
 
	    public static void main(String[] args) {
	        String sql1="create table "
	                + "log1 (id int ,msg varchar(20))";
	        String sql2="create table "
	                + "log2 (id int ,msg varchar(20))";
	        String sql3="create table "
	                + "log3 (id int ,msg varchar(20))";
	        String sql4="create table "
	                + "log4 (id int ,msg varchar(20))";
	        String sql5="create table "
	                + "log5 (id int ,msg varchar(20))";
	        String sql6="insert into log1 values "
	                + "(?,?)";
	        String sql7="insert into log2 values "
	                + "(?,?)";
	        //获得连接对象
	        Connection conn=null;
	        try {
	            conn=DBUtiles.getConnection();
	            //批量更新的第一种方法
	            //把多个sql语句存入sta对象的缓存中
	//          Statement sta=conn.createStatement();
	//          sta.addBatch(sql1);
	//          sta.addBatch(sql2);
	//          sta.addBatch(sql3);
	//          sta.addBatch(sql4);
	//          sta.addBatch(sql5);
	            //一次性发送给数据库执行
	            //返回值 >=0---成功,有结果
	            //返回值-2----成功,没有结果
	            //返回值-3----不成功
	//          int[] arr=sta.executeBatch();
	//          System.out.println(Arrays.toString(arr));
	
	            //第二中批量更新的方式
	            //使用ps固定一个执行计划
	            //把一堆参数存入ps的缓存
	            //一次性发送给DB,进行处理
	//          PreparedStatement ps=conn.prepareStatement(sql6);
	//          
	//          ps.setInt(1, 1);
	//          ps.setString(2, "lili");
	//          ps.addBatch();
	//          
	//          ps.setInt(1, 2);
	//          ps.setString(2, "lucy");
	//          ps.addBatch();
	//          
	//          int[] arr=ps.executeBatch();
	//          System.out.println(Arrays.toString(arr));
	        PreparedStatement ps =conn.prepareStatement(sql7);
	        for (int i = 1; i < 100; i++) {
	            ps.setInt(1, i);
	            ps.setString(2, "test"+i);
	            ps.addBatch();
	            if (i%8==0) {//97.98.99
	                ps.executeBatch();
	            }
	        }
	            ps.executeBatch();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }finally {
	            DBUtiles.closeConnection(conn);
	        }
	    }
	
## DAO

- DAO作为数据访问层,把业务逻辑层和数据库分割开来

- 业务逻辑层需要数据,就跟DAO要

- 业务逻辑层要保存数据,就交给DAO,让DAO去保存

- 业务逻辑层不关系数据如何获取,如何保存,全部都由DAO负责
> 案例演示

	      public class UserDAO1 implements UserDAO{
	        private static final String search_by_id
	            ="select * from user_1 where id=?";
	        private static final String search_all
	            ="select * from user_1";
	        private static final String update_user_password
	        ="update user_1 set name=?,password=? where id=?";
	        private static final String insert_user
	        ="insert into user_1 values(null,?,?,?)";
	        public User findUserById(int id) {
	            Connection conn=null;
	            try {
	                conn=DBUtiles.getConnection();
	                PreparedStatement ps=conn.prepareStatement(search_by_id);
	                ps.setInt(1, id);
	                ResultSet rs=ps.executeQuery();
	                int i=0;
	                String name=null;
	                String pwd=null;
	                int age=0;
	                while(rs.next()){
	                    i=rs.getInt(1);
	                    name=rs.getString(2);
	                    pwd=rs.getString(3);
	                    age=rs.getInt(4);
	                }
	                return new User(i, name, pwd, age);
	            } catch (Exception e) {
	                e.printStackTrace();
	                DBUtiles.rollback(conn);
	            }finally{
	                DBUtiles.closeConnection(conn);
	            }
	            return null;
	        }
	
	        public List<User> findAllUser() {
	            Connection conn=null;
	            try {
	                conn=DBUtiles.getConnection();
	                conn.setAutoCommit(false);
	                Statement sta=conn.createStatement();
	                ResultSet rs=sta.executeQuery(search_all);
	                List<User> list=new ArrayList<User>();
	                while (rs.next()) {
	                    int i=rs.getInt(1);
	                    String name=rs.getString(2);
	                    String pwd=rs.getString(3);
	                    int age=rs.getInt(4);
	                    User user=new User(i, name, pwd, age);
	                    list.add(user);
	                }
	                conn.commit();
	                return list;
	            } catch (Exception e) {
	                e.printStackTrace();
	                DBUtiles.rollback(conn);
	            }finally {
	                DBUtiles.closeConnection(conn);
	            }
	            return null;
	        }
	
	        public int updateUser(User user) {
	            Connection conn=null;
	            try {
	                conn=DBUtiles.getConnection();
	                PreparedStatement ps=conn.prepareStatement(update_user_password);
	                ps.setString(1, user.getName());
	                ps.setString(2, user.getPwd());
	                ps.setInt(3, user.getId());
	                int n=ps.executeUpdate();
	                if (n!=1) {
	                    throw new Exception("修改失败");
	                }
	                ps.close();
	            } catch (Exception e) {
	                e.printStackTrace();
	                DBUtiles.rollback(conn);
	            }finally {
	                DBUtiles.closeConnection(conn);
	            }
	            return 0;
	        }
	
	        public int saveUser(User user) {
	            Connection conn=null;
	            try {
	                conn=DBUtiles.getConnection();
	                PreparedStatement ps=conn.prepareStatement(insert_user);
	                ps.setString(1, user.getName());
	                ps.setString(2, user.getPwd());
	                ps.setInt(3, user.getAge());
	                int n=ps.executeUpdate();
	                if (n!=1) {
	                    throw new Exception("插入失败");
	                }
	                ps.close();
	            } catch (Exception e) {
	                e.printStackTrace();
	                DBUtiles.rollback(conn);
	            }finally {
	                DBUtiles.closeConnection(conn);
	            }
	            return 0;
	        }
	
	    }
      
      
     
 ***
>测试类

        public static void main(String[] args) {
            //数据--->数据库读取出来的
            //验证dao的findUserById方法
            UserDAO dao=new UserDAO1();
            //验证dao的findall
		    //List<User> list=dao.findAllUser();
		    //for (int i = 0; i < list.size(); i++) {
		    //System.out.println(list.get(i).getId());
		    //}

            //验证dao的修改方法
		    //User user=new User(2, "xiongda", "321", 18);
		    //dao.updateUser(user);

            //验证dao的插入方法
            User user=new User(1232, "xiongda", "321", 18);
            dao.saveUser(user);

	}
	 

 *** 

