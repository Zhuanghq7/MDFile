记录Spring求学道路上的坎坷
==

----
#2017/4/21 13:09:42  
###工程:TEEEEXT
####遇到问题：<br/>

 进行AOP尝试的时候，使用@Aspect注解创建了一个切面,将切面作为一个bean进行测试。发现自动装备的实例与实际应用的切面不一致。

 通过打印类.rostring()方法发现有两个实例被创建。

 在构造函数中添加输出语句，发现构造函数被调用两次。


####解决问题:<br/>

>就发现是即便使用了aspect注解创建的切面，依旧调用的是aspectJ的模式。该切面在spring之前就已经被aspectJ进行了实例化，由Spring进行bean创建的时候，由于spring不知道已经实例化了这个类，再次实例化了一次该切面。

	@Bean
	public TrackCounter trackCounter(){
		return TrackCounter.aspectOf();//我特么就是个天才
	}
	
干脆不使用aspect注解而是采用public aspect 构造aspect类，调用其aspectOf()这一工厂方法返回由aspectJ构造的实例，这样这个Bean的构造也使用了已经产生的实例了。
	
	输出正确结果:
	Play With a Little Help from My Friends
	Play With a Little Help from My Friends
	Play With a Little Help from My Friends
	Play Lucy in the Sky with Diamonds
	{0=2, 1=3, 2=1}
<br/>
<br/>

---- 
#2017/4/21 15:09:23   
###工程：SpringMVCStudy

maven简直是个神器。  

在项目上右->Maven->update project  
就可以自动添加jar包依赖。 
**简直无敌**

<br/>
<br/>


----
#2017/4/22 17:08:29   
###工程:spittrr

####遇到问题：
简直爆炸，终于能跑起来java config 的web页面了。  
原来tomcat一直是有error的。  
当服务器启动之后，第一次对spittrr的请求到来，服务器通过Init类创建其上下文。  
但是上下文创建失败了  
####解决：
**在Concole会输出error**  
解读error发现少一个log包  
加上就好……


----
#2017/4/25 14:52:28  
###工程:Zhuangh7

想用spring重新搞一波站来着。  
然后从头撸了一下，发现css文件跟png文件一直加载不出来。  
生气=- =  
仔细debug发现console提示说**找不到对应的requestmapping **  
这不对啊，静态资源怎么能用控制器解析呢= =   
各方搜索，突然意识到应该是静态资源的url链接被dispatchservlet拦截下来了。  
尴尬= =遂寻找解决方案  
**发现有三种**  
>http://blog.csdn.net/u012730299/article/details/51872704  
> SpringMVC访问静态资源的三种方式

看完百度前两面，所述相差无几，基本可以确定就这些方法了。其中由springmvc主动提供的resources标签显然是最正规的。   

然鹅，所有（*所有对就是所有）博客均只描述了xml的解决方案。  
老夫那个不信啊，有xml怎么可能会没有java的？？？  
遂翻墙。  
**Google大法好**   
>http://www.jianshu.com/p/6dfc48f46290   
>简书博客上提供了非常奈斯的解决方案

汗。。我都一度想要看源码算了的。心累_(:з」∠)_

百度累觉不爱。

####解决

	@Configuration
	@EnableWebMvc
	public class WebConfig extends WebMvcConfigurerAdapter {
	  @Override
	  public void addResourceHandlers(ResourceHandlerRegistry registry) {
	     registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/");
	 }
	}

先继承一下这个WebMvcConfigurerAdapter
重写增加资源句柄方法。  
在该方法中对参数调用添加资源句柄方法，把不拦截的静态地址丢进去。（这里他用了/**  然而我用了/*也一样（似乎））  
再调用增加资源位置，把对应静态资源的目录丢进去。搞定！




----
#2017/5/2 13:15:48 
###工程:Zhuangh7

####遇到问题  
爆炸爆炸、上传文件设置最大限制之后，客户端直接返回101连接被重置页面。  
百度说可以用ExceptionHandle捕获这个error然后返回新视图  
**然而并不行**  
能重复捕获到一万次error  返回的视图也不显示，chrome端出现重复提交form的情况  
使用IE浏览器，成功返回页面，认为是chrome的锅  
遂睡觉  
</br>
哪知道，第二天IE也GG了。 
多方搜寻发现，Apache有问题，自己的代码也有问题.  
		
	//位于rootinit
	@Override
	protected void customizeRegistration(Dynamic registration){
		registration.setMultipartConfig(
				new MultipartConfigElement("/tmp/uploads",
						-1,-1,0)
				//设置Multipart的相关参数 默认路径、文件最大大小、单次请求大小、超过多大存储
				);
	}
	

	//位于webConfig
	@Bean(name = "multipartResolver")
	public MultipartResolver multipartResolver() throws IOException{
		StandardServletMultipartResolver temp = new StandardServletMultipartResolver();
		temp.setResolveLazily(true);
		return temp;
	}


这段代码已经修改为不限制上传的大小了。  
原本是限制大小的，然后一旦上传文件太大就直接崩溃。  
问题在于。这是框架级别的exception 。  
Exception handler 可以捕获到exception 但是框架不能有效的继续返回视图了。  
多方搜索，反正是没有什么好的解决办法。  

###注释  
一说配置解析器后解析可以将文件大小解析放到controller里面进行，这样就不会有系统error现象的发生，然而出现了很多莫名其妙的错误。应该有一些是浏览器缓存的问题，我等下再试一下。
>实验结果：  
>并没有什么卵用，但是设置该属性之后的表现各有不同：  
>**延迟解析的表现**
> 
	GET
	intercepter success
	POST
	come to intercepter
	1967884006;104857600
	ExceptionProcess4Multipart  -> Maximum upload size of 104857600 bytes exceeded
	POST
	come to intercepter
	1967884006;104857600
	ExceptionProcess4Multipart  -> Maximum upload size of 104857600 bytes exceeded


>**直接解析的表现**  
>
	GET
	intercepter success
	ExceptionProcess4Multipart  -> Could not parse multipart servlet request; nested exception is java.lang.IllegalStateException: org.apache.tomcat.util.http.fileupload.FileUploadBase$FileSizeLimitExceededException: The field profilePicture exceeds its maximum permitted size of 104857600 bytes.
	ExceptionProcess4Multipart  -> Could not parse multipart servlet request; nested exception is java.lang.IllegalStateException: org.apache.tomcat.util.http.fileupload.FileUploadBase$FileSizeLimitExceededException: The field profilePicture exceeds its maximum permitted size of 104857600 bytes.


>**推测**  
>不管是先解析还是后解析。Spring对该次请求都会整个abort掉，这样的话，return返回的视图解析就不会有机会被传回客户端，客户端得到的就一定是连接被重置。因此，想要返回视图的话，完整的走完整个后台流程非常的重要，也就是说，这个文件不管多大。。还是得接受的_(:з」∠)_
>
>  
>发现了多种sevlet容器对请求的处理不一样。sts自带的server更明显的看到，拦截器在整个文件上传完成之后才被触发。也就是说，文件的上传是包含在http请求中的，不为整个后台程序所操纵。即便是http请求的拦截器，也是在整个请求上传到服务器之后才触发。

	temp.setResolveLazily(true);

###退而求其次

将上传限制取消（都设置为-1）改用FileUpdateIntercepter，为全局创建拦截器


	webconfig 中添加代码
	@Override
	public void addInterceptors(InterceptorRegistry registry){
		registry.addInterceptor(fileintecepter()).addPathPatterns("/**");
	}//哇，有效果了。。你敢信


尼玛试了一万年试出来的配置方法，独此一家好吧。  
创建全局拦截器之后，拦截请求，计算请求大小，这实际上算的应该是整个请求的大小而非单个文件的大小了，然而并没有什么办法2333。

        	System.out.println("come to intercepter");
            ServletRequestContext ctx = new ServletRequestContext(request);
            long requestSize = ctx.contentLength();
            System.out.println(""+ requestSize+";"+maxSize);
            if (requestSize > maxSize) {
					.......
			}

###功能上可行  
然而还是有问题（**比如经常性失效，这时候重启sevlet容器清空浏览器缓存。。**）。不管文件大小是否合格，文件都会上传到服务器，结束之后才会返回错误视图。那么这里就有一些问题:  

1. 文件上传并不会保存在服务器，那么去哪里了呢，持续上传对服务器会不会有巨大能耗呢
2. 服务器的内存是否会因为上传而增加
3. 上传完再有结果对客户端肯定是不友好的，只能在前端判断了(这样有孩子犯贱删除js的话让他自作自受去吧23333)。
4. 服务器功耗问题还是得测试一下，然而并没有那么多电脑，生气= =，(其实有啊_(:з」∠)_但是部署起来好麻烦233又没人给工资)


###有个结论  
别人说的 果然没几句可以听= =   
</br>



----
#2017/5/9 14:03:18 
###Zhuangh7

####遇到问题  
spring不能返回html视图

>- First the DispatcherServlet is invoked by the Servlet Container.  
- The DispatcherServlet finds a mapping which maps to the home method of your Controller and the home method returns a view name "HelloWorld"
- Now the DispatcherServlet uses a View Resolver (your InternalResourceViewResolver) to find the View to render the model through, since the name is "HelloWorld", this maps to the /WEB-INF/view/HelloWorld.html view.
- Now essentially a call is made to RequestDispatcher.forward("/WEB-INF/views/HelloWorld.html",....
- The Servlet container at this point tries to find the servlet which can handle /WEB-INF/views/HellowWorld.html uri - if it had been a .jsp there is a JSPServlet registered which can handle rendering the jsp, however for *.html there is no servlet registered, so the call ends up with the "default servlet", which is registered with a servlet-mapping of / which probably your DispatcherServlet is.
- Now the Dispatcher servlet does not find a controller to handle request for /WEB-INF/views/HelloWorld.html and hence the message that you are seeing  


**显而易见的解决办法** 把jsp的解析器定向到可以解析html= =  
采用webconfig提供的设置默认sevlethandle的方法，把叫做jsp的sevlet设置为默认的sevlet，解决html不返回的问题。  



----
#2017/5/10 10:08:35 
###项目:WebProject4ReactStudy

####遇到问题   
项目从eclipse移植到idea进行react学习。  
移植过程中学习使用maven管理依赖包  
通过在
>http://mvnrepository.com/  

搜索所依赖的包   
得到类似于   

	<dependency>
	    <groupId>org.reactivemongo</groupId>
	    <artifactId>reactivemongo_2.11</artifactId>
	    <version>0.12.3</version>
	</dependency>

的一段xml，把这段文字加入pom.xml然后import，就会自动下载对应包。  


#####包依赖解决后发现404问题
观察log发现文件并没有被编译，目录下没有class文件。  
多方查找发现  
>使用maven管理的项目，需要在pom中加入
>
    <packaging>war</packaging>   
 的配置
  
> 	且<build>下的<finalName>需要与项目的连接名字相同（就是8080后面的名字）

配置完这两个之后如期编译文件了。但是进入视图解析器之后，发现返回的视图资源都找不到。  
检查目录结构，发现web-inf下的资源文件均没有加载。  

多方查找发现主要是Project Structure下的Artifacts设置  
该设置可以配置输出文件的layout。**检查layout发现，资源由一个module"'Web' facet resources 映射加载**   
而Facets的配置就在上面。。。 
发现默认的资源目录是src->main->webapp下。。  


那么方法有两种    
1. 把资源目录调整为web目录下的WEB-INF  
2. 把web目录里面的东西撸到src->main->webapp下面去。

发现改一改pom会自动把路径设置为webapp，那么干脆一劳永逸放弃web文件夹


####最后做大死删除WEB目录，程序依然平稳运行。web目录果然无关紧要  
就很奇怪为什么创建的spring mvc 项目会默认给个web目录，搞事情啊=- =
####结语  
通过观察目录搞定了很多问题，这种不放弃的探索令人受益匪浅。  很感动一直以来一直能在互联网各位大佬与运气的帮助下解决问题。磕磕绊绊走到今天。像我这种头铁的人，改BUG真不是个好活啊。
