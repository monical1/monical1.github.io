问题发生背景

项目中有一些transformer类中使用 `beanCopier`来实现bean转换，该类来自cglib，20190830线上接口返回出错，涉及到三处api

```java
private static BeanCopier BEAN_COPIER = BeanCopier.create(TPSAreaDTO.class, ShowHeatMapDTO.class, false);
```

发生了如下错误

`java.lang.IncompatibleClassChangeError: net/sf/cglib/core/DebuggingClassWriter`

具体的stack error

```
2019-08-30 19:49:01.177 - - [ERROR] Pigeon-Server-Request-Processor-40-thread-1 ShowHeat2DTOTransformer2 #XMDT#{__traceId__=1318699557884336472}#XMDT#
java.lang.IncompatibleClassChangeError: net/sf/cglib/core/DebuggingClassWriter
	at net.sf.cglib.core.DefaultGeneratorStrategy.getClassVisitor(DefaultGeneratorStrategy.java:30) ~[cglib-3.2.0.jar:?]
	at net.sf.cglib.core.DefaultGeneratorStrategy.generate(DefaultGeneratorStrategy.java:24) ~[cglib-3.2.0.jar:?]
	at net.sf.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:231) ~[cglib-3.2.0.jar:?]
	at net.sf.cglib.core.KeyFactory$Generator.create(KeyFactory.java:149) ~[cglib-3.2.0.jar:?]
	at net.sf.cglib.core.KeyFactory.create(KeyFactory.java:117) ~[cglib-3.2.0.jar:?]
	at net.sf.cglib.core.KeyFactory.create(KeyFactory.java:109) ~[cglib-3.2.0.jar:?]
	at net.sf.cglib.core.KeyFactory.create(KeyFactory.java:105) ~[cglib-3.2.0.jar:?]
	at net.sf.cglib.beans.BeanCopier.<clinit>(BeanCopier.java:32) ~[cglib-3.2.0.jar:?]
	at com.maoyan.show.deal.biz.transformer.seat.ShowHeat2DTOTransformer2.<clinit>(ShowHeat2DTOTransformer2.java:35) [maoyan-show-deal-biz-1.0.0-SNAPSHOT.jar:?]
	at com.maoyan.show.deal.biz.remoteservice.stat.ShowDealStatRemoteServiceImpl.calculateShowHeatMap(ShowDealStatRemoteServiceImpl.java:175) [maoyan-show-deal-biz-1.0.0-SNAPSHOT.jar:?]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[?:1.8.0_151]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[?:1.8.0_151]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[?:1.8.0_151]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[?:1.8.0_151]
	at com.dianping.pigeon.remoting.provider.service.method.ServiceMethod.invoke(ServiceMethod.java:170) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.BusinessProcessFilter.invoke(BusinessProcessFilter.java:79) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.BusinessProcessFilter.invoke(BusinessProcessFilter.java:34) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.ProviderProcessHandlerFactory$1.handle(ProviderProcessHandlerFactory.java:99) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.GatewayProcessFilter.invoke(GatewayProcessFilter.java:242) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.GatewayProcessFilter.invoke(GatewayProcessFilter.java:51) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.ProviderProcessHandlerFactory$1.handle(ProviderProcessHandlerFactory.java:99) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.SecurityFilter.invoke(SecurityFilter.java:31) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.SecurityFilter.invoke(SecurityFilter.java:15) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.ProviderProcessHandlerFactory$1.handle(ProviderProcessHandlerFactory.java:99) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.GenericProcessFilter.invoke(GenericProcessFilter.java:62) [dpsf-net-2.10.8.jar:?]
	at com.dianping.pigeon.remoting.provider.process.filter.GenericProcessFilter.invoke(GenericProcessFilter.java:24) [dpsf-net-2.10.8.jar:?]
```



1. 问题排查

   项目deal里出现了`cglib 3.2`

   ```
   <dependency>
     <groupId>cglib</groupId>
     <artifactId>cglib</artifactId>
     <version>3.2.0</version>
   </dependency>
   ```

   

   而在2019-07-23 因为调用公司内部服务，引入了对应dependence

   ```
   <dependency>
     <groupId>com.meituan.movie</groupId>
     <artifactId>movie-common</artifactId>
     <version>2.0.25</version>
   </dependency>
   ```

   对应的依赖关系

   ```
   [INFO]
   [INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ maoyan-show-deal-biz ---
   
   [INFO] +- cglib:cglib:jar:3.2.0:compile
   [INFO] |  +- org.ow2.asm:asm:jar:5.0.3:compile
   [INFO] |  \- org.apache.ant:ant:jar:1.9.4:compile
   [INFO] |     \- org.apache.ant:ant-launcher:jar:1.9.4:compile
   
   
   [INFO] +- com.meituan.movie:movie-common:jar:2.0.25:compile
   [INFO] |  +- dom4j:dom4j:jar:1.6.1:compile
   [INFO] |  |  \- xml-apis:xml-apis:jar:1.0.b2:compile
   [INFO] |  +- org.apache.cxf:cxf-rt-frontend-jaxws:jar:2.5.2:compile
   [INFO] |  |  +- xml-resolver:xml-resolver:jar:1.2:compile
   [INFO] |  |  +- asm:asm:jar:3.1:compile
   [INFO] |  |  +- org.apache.cxf:cxf-api:jar:2.5.2:compile
   [INFO] |  |  |  +- org.apache.cxf:cxf-common-utilities:jar:2.5.2:compile
   [INFO] |  |  |  |  \- org.codehaus.woodstox:woodstox-core-asl:jar:4.1.1:runtime
   [INFO] |  |  |  |     \- org.codehaus.woodstox:stax2-api:jar:3.1.1:runtime
   [INFO] |  |  |  +- org.apache.ws.xmlschema:xmlschema-core:jar:2.0.1:compile
   [INFO] |  |  |  +- org.apache.neethi:neethi:jar:3.0.1:compile
   [INFO] |  |  |  \- wsdl4j:wsdl4j:jar:1.6.2:compile
   [INFO] |  |  +- org.apache.cxf:cxf-rt-core:jar:2.5.2:compile
   [INFO] |  |  |  +- com.sun.xml.bind:jaxb-impl:jar:2.1.13:compile
   [INFO] |  |  |  \- org.apache.geronimo.specs:geronimo-javamail_1.4_spec:jar:1.7.1:compile
   [INFO] |  |  +- org.apache.cxf:cxf-rt-bindings-soap:jar:2.5.2:compile
   [INFO] |  |  |  +- org.apache.cxf:cxf-tools-common:jar:2.5.2:compile
   [INFO] |  |  |  \- org.apache.cxf:cxf-rt-databinding-jaxb:jar:2.5.2:compile
   [INFO] |  |  +- org.apache.cxf:cxf-rt-bindings-xml:jar:2.5.2:compile
   [INFO] |  |  +- org.apache.cxf:cxf-rt-frontend-simple:jar:2.5.2:compile
   [INFO] |  |  \- org.apache.cxf:cxf-rt-ws-addr:jar:2.5.2:compile
   [INFO] |  +- org.apache.httpcomponents:httpclient-cache:jar:4.3.5:compile
   [INFO] |  +- org.apache.httpcomponents:httpmime:jar:4.3.5:compile
   [INFO] |  +- com.sankuai.meituan:mt-common:jar:1.1.1:compile
   [INFO] |  |  \- asm:asm-all:jar:3.3.1:compile
   [INFO] |  +- org.apache.commons:commons-email:jar:1.2:compile
   [INFO] |  |  \- javax.mail:mail:jar:1.4:compile
   [INFO] |  +- common-money:money:jar:1.0.2-SNAPSHOT:compile
   [INFO] |  +- org.javaswift:joss:jar:0.9.9-SNAPSHOT:compile
   [INFO] |  \- com.meituan.service.mobile:service-common:jar:1.0.8:compile
   
   [INFO] ------------------------------------------------------------------------
   [INFO] BUILD SUCCESS
   [INFO] ------------------------------------------------------------------------
   [INFO] Total time: 17.215 s
   [INFO] Finished at: 2019-08-30T20:23:27+08:00
   [INFO] Final Memory: 34M/320M
   [INFO] ------------------------------------------------------------------------
   ```

   ```
   !?master ~/resources/workspaces4idea/maoyan/maoyan-show-deal/maoyan-show-deal-biz> mvn dependency:tree | grep 'asm'
   [INFO] |  +- org.ow2.asm:asm:jar:5.0.3:compile
   [INFO] |  |  +- asm:asm:jar:3.1:compile
   [INFO] |  |  \- asm:asm-all:jar:3.3.1:compiled
   ```

   定位到问题

   依赖包引用了`asm:asm:jar:3.1:compile`

   

   具体的原因

   net.sf.cglib.core.GeneratorStrategy#generate

   ```
   DefaultGeneratorStrategy
   
   public byte[] generate(ClassGenerator cg) throws Exception {
     DebuggingClassWriter cw = getClassVisitor();
     transform(cg).generateClass(cw);
     return transform(cw.toByteArray());
   }
   
   // 类public class DebuggingClassWriter extends ClassVisitor {}
   
   // cglib3.2.0 依赖底层asm5.0.3 对应的ClassVisitor 是抽象类
   // 而外部依赖引入的asm3.1 对应ClassVisitor 是接口
   
   
   ```

   ![https://upload.cc/i1/2019/08/30/2dVmw8.jpg](https://upload.cc/i1/2019/08/30/2dVmw8.jpg)

3. 解决方案

   （1）删除cglib-3.2.0，使用spring-core-4.3.17-release 的beanCopier

   > /org/springframework/spring-core/4.3.17.RELEASE/spring-core-4.3.17.RELEASE.jar!/org/springframework/cglib/core/AbstractClassGenerator.class:193

   >  org.springframework.cglib.core.GeneratorStrategy#generate

   其中GeneratorStrategy 为`this.strategy = DefaultGeneratorStrategy.INSTANCE;`

   ```java
   public class DefaultGeneratorStrategy implements GeneratorStrategy {
       public static final DefaultGeneratorStrategy INSTANCE = new DefaultGeneratorStrategy();
   
       public DefaultGeneratorStrategy() {
       }
   
       public byte[] generate(ClassGenerator cg) throws Exception {
           DebuggingClassWriter cw = this.getClassVisitor();
           this.transform(cg).generateClass(cw);
           return this.transform(cw.toByteArray());
       }
   }
   ```

   

   （2）对movie-common删除asm 对应的依赖，保证`classpath`只有一份asm

   （3）使用cglib 2.2 代替3.2.0