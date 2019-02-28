---
title: 'java.io.File 一些基础'
copyright: true
comments: true
date: 2018-10-24 16:58:36
tags:categories:
 - [Java, File, IO流]
---

# File
----------
> java.io.File
> File表示操作系统中文件系统里的一个文件或目录
> 使用File可以：
> 1.访问一个文件或目录的属性信息
> 2.操作一个文件或目录（创建，删除）
> 3.访问一个目录的子项
> 但是不能使用File访问文件数据

<!-- more -->
### Flie方法
- 在创建File的时候书写路径尽量使用相对路径，避免平台差异化，目录层级别分割符应当使用File里面的一个常量`File.separator`，常见的相对路径：
- 1.`“.”`：表示当前目录，当前目录运行环境不同路径也不同，在eclipse中运行java程序时的当前目录类所在项目的根目录
- 2.类的加载路径
```
public static void main(String[] args) {
	File file = new File("."+File.separator+"test.txt");
	String name= file.getName();
	System.out.println(name);//读取文件名字

	long length = file.length();
	System.out.println(length);//读取文件大小
	
	boolean canRead = file.canRead();//是否可读
	boolean canWrite = file.canWrite();//是否可写
	System.out.println("可读:"+canRead);
	System.out.println("可写:"+canWrite);
	
	boolean isHidden = file.isHidden();//是否隐藏
	System.out.println("是否为隐藏："+isHidden);
	}
}
```
####  boolean	createNewFile() 

> 当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件。
	
```
public static void main(String[] args) throws IOException {
 //在当前文件下创建一个demo.txt的文件
File file = new File("."+File.separator+"demo1.txt");
```

>  boolean exists()

-  判断File表示的文件或目录是否真实存在
```
	if(!file.exists()) {
		file.createNewFile();//报错补错
		System.out.println("文件创建完毕！");	
	}else {
		System.out.println("文件已经存在！");
	}
  }
 }
```

----------


### 获取当前目录中的所有子项

#### boolean isFile()

> 判断当前File表示的是否为一个文件

#### boolean isDirectory()

> 判断当前File表示的是否为一个目录

```
	public static void main(String[] args) {
	File dir = new File(".");
	if(dir.isDirectory()) {
		File[] subs = dir.listFiles();
		System.out.println(subs.length);//有多少文件和目录
		for(int i = 0 ; i<subs.length; i++) {
			File sub = subs[i];
			if(sub.isDirectory()) {//判断是否为目录
				System.out.print("目录：");
			}
			if(sub.isFile()) {//判断是否为文件
				System.out.print("文件：");
			}System.out.println(sub.getName());//输出文件名
				}
			}
		}
	}
	```
----------

#### boolean accept(File pathname) 
> 测试指定抽象路径名是否应该包含在某个路径名列表中。
- `File`提供了一个重载的`listFire`方法
- 该方法允许我们传入一个文件过滤器`FileFilter`
-  该方法会将`File`表示的目录中所有满足过滤器要求的子项返回，而不满足的则被忽略。

```
	public class File_listFiles2 {
	public static void main(String[] args) {
	 //获取当前路径中所有名字以“.”开头的子项 
	File dir = new File(".");
	FileFilter FileFilter = new FileFilter() {
		// boolean	accept(File pathname) 
		//测试指定抽象路径名是否应该包含在某个路径名列表中。
		public boolean accept(File file) {
			String name = file.getName();//获取不符合“.”开头的文件
			System.out.println("正在过滤子项："+name);
			return name.startsWith(".");
		}
		};
		File[] subs = dir.listFiles(FileFilter);
		for(int i = 0; i<subs.length; i++) {
			System.out.println(subs[i].getName());
			}
		}
	}
```

----------



#### boolean mkdir() 

> 创建一个目录

```
public class File_mkdir {
	public static void main(String[] args) {
	//在当前目录中创建一个名为demo的目录
	File dir = new File("demo");
	if(!dir.exists()) {
		dir.mkdir();
		System.out.println("该目录已创建！");
	}else {
		System.out.println("该目录已存在！");
			}			
		}	
	}
```
	
#### boolean mkdirs()

> 创建一个多级目录

```
	public static void main(String[] args) {
	//在当前目录中创建 a/b/c/d/e/f
	File dirs  = new File("a"+File.separator+"b"+File.separator+"c"+File.separator+"d"+File.separator+"e"+File.separator+"f");
	if(!dirs.exists()) {
		//mkdirs是在创建当前File表示的目录同时将该目录其上的所有不存在的父目录一同创建出来
		dirs.mkdirs();
		System.out.println("文件目录创建完毕！");
	}else {
		System.out.println("文件目录已存在！");
			}
		}
	}
```
----------


#### boolean delete() 
    
> 删除一个文件

- 将当前目录中的demo.txt删除
 相对路径默认是从“当前目录”开始
 所以，`new File("demo.txt"）`等同`new File("./demo.txt)`

- 注：delete方法在删除目录的时候，需要注意，只能删除空目录；
	
> 删除一个文件：
	
```JAVA
	public static void main(String[] args) {
		File file = new File("demo.txt");
		if(file.exists()) {
			file.delete();
			System.out.println("文件已删除");
		}else {
			System.out.println("该文件不存在");
			}		
		}
	}
```
	
> 删除一个目录：
	
```
	public static void main(String[] args) {
	File del = new File("demo");//将当前目录中的demo目录删除
	if(del.exists()) {
		// delete方法在删除目录的时候，需要注意，只能删除空目录；
		del.delete();//必须是空目录才能删
		System.out.println("目录已删除！");
	}else {
		System.out.println("该目录不存在！");
			}		
		}
	}
```
----------

