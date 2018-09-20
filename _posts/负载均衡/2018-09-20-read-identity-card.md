---
layout: blog
banana: true
category: 负载均衡
title:  使用JAVA获取国腾身份证阅读器身份证信息
date:   2018-09-20 11:22:42
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-8-1/24280498.jpg
tags:
- Redis
---


# 身份证阅读器简介

最近在做项目过程中，有使用到国腾实业的身份证阅读器读取用户身份证信息，所以在此处记录下使用过程，方便以后查找。

使用的设备类型：GTICR100，但是咨询过国腾技术部，以下代码适用于该公司所有系列产品。

![GTICR100-1]({{ site.url }}/assets/pic/GTICR100-1.jpeg)

![GTICR100-2]({{ site.url }}/assets/pic/GTICR100-2.jpeg)



# 以下是使用过程



## 安装驱动

身份证阅读器可以通过串口与电脑通信，也可以通过USB与电脑进行通信，串口的没试过，此处说明的是USB方式。

首先[下载USB驱动]({{ site.url }}/assets/others/GITCR100_USB_DRIVER.zip)，在下载的文档里有详细安装说明（win10可以使用win7驱动）。



##代码说明

SDK是一个dll动态库文件，java通过JNA调用dll方法获取身份证信息。

动态库文件下载：

+ [64位DLL]({{ site.url }}/assets/others/GITCR100_SDK/x64/termb.dll)

+ [32位DLL]({{ site.url }}/assets/others/GITCR100_SDK/x86/termb.dll)

###新建maven工程

在pom.xml中添加jna依赖

```xml
<!-- https://mvnrepository.com/artifact/net.java.dev.jna/jna -->
<dependency>
    <groupId>net.java.dev.jna</groupId>
    <artifactId>jna</artifactId>
    <version>4.5.2</version>
</dependency>
```



###使用JNA映射dll

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import com.sun.jna.*;
import com.sun.jna.win32.*;

public interface Termb extends StdCallLibrary
{
	public static class IdCardTxtInfo extends Structure
	{   
		public byte[] name=new byte[31];                
		public byte[] Sex=new byte[6];               
		public byte[] nation=new byte[11];          
		public byte[] borndate=new byte[9];            
		public byte[] address=new byte[71];         
		public byte[] idno=new byte[19];                
		public byte[] department=new byte[31];      
		public byte[] StartDate=new byte[9];           
		public byte[] EndDate=new byte[9];    
		public byte[] Reserve=new byte[37]; 
		public byte[] AppAddress1=new byte[71];
		@Override
		protected List<String> getFieldOrder() {
			 return Arrays.asList(new String [] {"name","Sex","nation", "borndate","address","idno","department","StartDate","EndDate","Reserve","AppAddress1"});

		}   
	}
	int SetJpgPath(String a);
	int InitComm(int iPort);//串口号
	int Authenticate();
	int Read_Content(int iActive);   
	int GetIdCardTxtInfo(IdCardTxtInfo result);
	int CloseComm();
}
```



###测试代码

```java
import java.io.*;
import com.sun.jna.*;

public class LoadTermb
{
	public static void main (String [] args)
	{
		Termb lib = (Termb) Native.loadLibrary ("termb", Termb.class);
		Termb.IdCardTxtInfo info = new Termb.IdCardTxtInfo();
		
		lib.SetJpgPath("c:\\1.jpg");
		if (lib.InitComm(1) != 1){//
			System.out.println ("InitComm error!");					
		}
		lib.Authenticate();
		if (lib.Read_Content(1) != 1){
			System.out.println ("Read_Content error!");	
		}
		lib.GetIdCardTxtInfo(info);
		System.out.println ("Card info is:");
		try{
			System.out.println(new String(info.name, "gb2312"));   
			System.out.println(new String(info.Sex, "gb2312")); 
			System.out.println(new String(info.nation, "gb2312"));
			System.out.println(new String(info.borndate, "gb2312"));
			System.out.println(new String(info.address, "gb2312"));
			System.out.println(new String(info.idno, "gb2312"));
			System.out.println(new String(info.department, "gb2312"));
			System.out.println(new String(info.StartDate, "gb2312"));
			System.out.println(new String(info.EndDate, "gb2312"));
			System.out.println(new String(info.Reserve, "gb2312"));
			System.out.println(new String(info.AppAddress1, "gb2312"));
		}catch(IOException e){   
			e.printStackTrace();   
		}   
	}
}

```

把身份证放到读卡器上， 运行代码，就可以获取身份证信息了。