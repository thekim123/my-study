### 1. date parse error
```
java.text.ParseException: Unparseable date: "20250316"

at com.portal.guard.controller.GuardLogController$$EnhancerBySpringCGLIB$$991aa6dd.doAction(<generated>) [classes!/:0.0.1-SNAPSHOT]
```

### 2. export 결재시 NPE
```
java.lang.NullPointerException: null
        at com.portal.approval.service.impl.ApprovalServiceImpl.updateDocument(ApprovalServiceImpl.java:917) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.approval.service.impl.ApprovalServiceImpl.doApprovalAction(ApprovalServiceImpl.java:679) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.approval.service.impl.ApprovalServiceImpl$$FastClassBySpringCGLIB$$47b03c73.invoke(<generated>) ~[classes!/:0.0.1-SNAPSHOT]
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:793) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:123) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:388) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:123) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:388) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:708) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at com.portal.approval.service.impl.ApprovalServiceImpl$$EnhancerBySpringCGLIB$$f58d30da.doApprovalAction(<generated>) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.export.service.impl.ExportReqServiceImpl.saveExportReqApproval(ExportReqServiceImpl.java:236) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.export.service.impl.ExportReqServiceImpl$$FastClassBySpringCGLIB$$ab9521b.invoke(<generated>) ~[classes!/:0.0.1-SNAPSHOT]
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:793) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:123) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:388) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119) ~[spring-tx-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:708) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at com.portal.export.service.impl.ExportReqServiceImpl$$EnhancerBySpringCGLIB$$5d7f2f32.saveExportReqApproval(<generated>) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.export.controller.ExportReqController.procSaveExportReqApproval(ExportReqController.java:209) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.export.controller.ExportReqController.doAction(ExportReqController.java:181) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.export.controller.ExportReqController$$FastClassBySpringCGLIB$$c37bb2d9.invoke(<generated>) ~[classes!/:0.0.1-SNAPSHOT]
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:793) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.security.access.intercept.aopalliance.MethodSecurityInterceptor.invoke(MethodSecurityInterceptor.java:61) ~[spring-security-core-5.6.7.jar!/:5.6.7]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:708) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at com.portal.export.controller.ExportReqController$$EnhancerBySpringCGLIB$$cc9416fb.doAction(<generated>) ~[classes!/:0.0.1-SNAPSHOT]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_362]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_362]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_362]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_362]
        at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205) ~[spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:150) ~[spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:117) ~[spring-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895) ~[spring-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808) ~[spring-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1071) ~[spring-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:964) ~[spring-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006) [spring-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909) [spring-webmvc-5.3.23.jar!/:5.3.23]
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:681) [tomcat-embed-core-9.0.65.jar!/:4.0.1]
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883) [spring-webmvc-5.3.23.jar!/:5.3.23]
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:764) [tomcat-embed-core-9.0.65.jar!/:4.0.1]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) [tomcat-embed-websocket-9.0.65.jar!/:na]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.65.jar!/:na]
        at com.portal.core.security.filter.CorsFilter.doFilter(CorsFilter.java:51) [classes!/:0.0.1-SNAPSHOT]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:337) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.invoke(FilterSecurityInterceptor.java:115) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.doFilter(FilterSecurityInterceptor.java:81) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:122) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:116) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:126) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:81) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter(AnonymousAuthenticationFilter.java:109) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter(SecurityContextHolderAwareRequestFilter.java:149) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter(RequestCacheAwareFilter.java:63) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at com.portal.core.security.filter.HictSecurityFilter.doFilter(HictSecurityFilter.java:187) [classes!/:0.0.1-SNAPSHOT]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:103) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:89) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.web.filter.CorsFilter.doFilterInternal(CorsFilter.java:91) [spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.header.HeaderWriterFilter.doHeadersAfter(HeaderWriterFilter.java:90) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.header.HeaderWriterFilter.doFilterInternal(HeaderWriterFilter.java:75) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:110) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:80) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal(WebAsyncManagerIntegrationFilter.java:55) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:221) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:186) [spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:354) [spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:267) [spring-web-5.3.23.jar!/:5.3.23]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201) [spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5.3.23.jar!/:5.3.23]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:197) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:135) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:360) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.coyote.ajp.AjpProcessor.service(AjpProcessor.java:433) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:890) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1789) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1191) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.65.jar!/:na]
        at java.lang.Thread.run(Thread.java:750) [na:1.8.0_362]

```



### 3. EAI 에러
```
org.springframework.web.client.ResourceAccessException: I/O error on POST request for "https://10.5.225.101:5100/restv2/UNI_ARV_01.REST_API:REST_ARV_API/IFARV001": Unsupported or unrecognized SSL message; nested exception is javax.net.ssl.SSLException: Unsupported or unrecognized SSL message
        at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:785) ~[spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:711) ~[spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.client.RestTemplate.exchange(RestTemplate.java:602) ~[spring-web-5.3.23.jar!/:5.3.23]
        at com.portal.eai.api.EaiHttpRequest.sendRequest(EaiHttpRequest.java:74) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.eai.api.EaiHttpRequest.registerRequest(EaiHttpRequest.java:42) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.eai.command.EaiRequestAction.lambda$sendApprovalInformation$1(EaiRequestAction.java:42) ~[classes!/:0.0.1-SNAPSHOT]
        at java.util.stream.ForEachOps$ForEachOp$OfRef.accept(ForEachOps.java:183) ~[na:1.8.0_362]
        at java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:175) ~[na:1.8.0_362]
        at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1384) ~[na:1.8.0_362]
        at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:482) ~[na:1.8.0_362]
        at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:472) ~[na:1.8.0_362]
        at java.util.stream.ForEachOps$ForEachOp.evaluateSequential(ForEachOps.java:150) ~[na:1.8.0_362]
        at java.util.stream.ForEachOps$ForEachOp$OfRef.evaluateSequential(ForEachOps.java:173) ~[na:1.8.0_362]
        at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[na:1.8.0_362]
        at java.util.stream.ReferencePipeline.forEach(ReferencePipeline.java:485) ~[na:1.8.0_362]
        at com.portal.eai.command.EaiRequestAction.sendApprovalInformation(EaiRequestAction.java:40) ~[classes!/:0.0.1-SNAPSHOT]
        at com.portal.eai.service.EaiServiceImpl.sendApprovalResult(EaiServiceImpl.java:28) ~[classes!/:0.0.1-SNAPSHOT]
        at sun.reflect.GeneratedMethodAccessor2470.invoke(Unknown Source) ~[na:na]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_362]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_362]
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:198) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at org.springframework.aop.interceptor.AsyncExecutionInterceptor.lambda$invoke$0(AsyncExecutionInterceptor.java:115) ~[spring-aop-5.3.23.jar!/:5.3.23]
        at scouter.agent.wrapper.async.WrTaskCallable.call(WrTaskCallable.java:38) ~[na:na]
        at java.util.concurrent.FutureTask.run(FutureTask.java:266) ~[na:1.8.0_362]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) ~[na:1.8.0_362]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) ~[na:1.8.0_362]
        at java.lang.Thread.run(Thread.java:750) ~[na:1.8.0_362]
Caused by: javax.net.ssl.SSLException: Unsupported or unrecognized SSL message
        at sun.security.ssl.SSLSocketInputRecord.handleUnknownRecord(SSLSocketInputRecord.java:455) ~[na:1.8.0_362]
        at sun.security.ssl.SSLSocketInputRecord.decode(SSLSocketInputRecord.java:184) ~[na:1.8.0_362]
        at sun.security.ssl.SSLTransport.decode(SSLTransport.java:109) ~[na:1.8.0_362]
        at sun.security.ssl.SSLSocketImpl.decode(SSLSocketImpl.java:1397) ~[na:1.8.0_362]
        at sun.security.ssl.SSLSocketImpl.readHandshakeRecord(SSLSocketImpl.java:1305) ~[na:1.8.0_362]
        at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:440) ~[na:1.8.0_362]
        at sun.net.www.protocol.https.HttpsClient.afterConnect(HttpsClient.java:559) ~[na:1.8.0_362]
        at sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:197) ~[na:1.8.0_362]
        at sun.net.www.protocol.https.HttpsURLConnectionImpl.connect(HttpsURLConnectionImpl.java:167) ~[na:1.8.0_362]
        at org.springframework.http.client.SimpleBufferingClientHttpRequest.executeInternal(SimpleBufferingClientHttpRequest.java:76) ~[spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.http.client.AbstractBufferingClientHttpRequest.executeInternal(AbstractBufferingClientHttpRequest.java:48) ~[spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.http.client.AbstractClientHttpRequest.execute(AbstractClientHttpRequest.java:66) ~[spring-web-5.3.23.jar!/:5.3.23]
        at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:776) ~[spring-web-5.3.23.jar!/:5.3.23]
        ... 28 common frames omitted

```

### 4. Gate inout
```
### Error updating database.  Cause: java.sql.SQLIntegrityConstraintViolationException: (conn=274061) Duplicate entrRIMARY'
### The error may exist in URL [jar:file:/home/uniadmmin/webapp/security-portal-0.0.1-SNAPSHOT.jar!/BOOT-INF/classes
### The error may involve com.portal.iogate.repository.IogateVisitsRepository.insertVisitPersonInOut-Inline
### The error occurred while setting parameters
### SQL: /* mapper-iogate-visits.xml - insertVisitPersonInOut */         INSERT INTO VISIT_PSN_INOUT(               * 문서ID */             ,VST_SEQ /* 방문자 순번 */             ,VISIT_DT /* 방문일자 */             ,SEQ /* 순번 */              ,VISIT_PASS_NO /* 방문출입증번호 */             ,IN_PASS_DTM /* 입문일자시간 */             ,IN_PLACE_CR_ID /* 입문확인자ID */             ,PSN_RMK /* 입문인원비고 */             ,LOST_YN /* 출입증 분실여부 */           */             ,MOBILE_SEAL_RMK /* 모바일봉인스티커비고 */             ,CREATE_DTM /* 생성일시 */             ,CREA/* 수정일시 */             ,MODIFY_ID /* 수정자 */         ) VALUES (              ? /* 시스템id */             ,? //             ,? /* 방문일자 */             ,1 /* 순번 */             ,'IN' /* 출입/출문상태 */             ,? /* 방시간 */             ,? /* 입문장소코드 */             ,? /* 입문확인자ID */             ,? /* 입문인원비고 */         ,? /* 모바일봉인스티커건수 */             ,? /* 모바일봉인스티커비고 */             ,? /* 생성일시 */             */             ,? /* 수정자 */         )
### Cause: java.sql.SQLIntegrityConstraintViolationException: (conn=274061) Duplicate entry '0000-VI00060843-1-20250
; (conn=274061) Duplicate entry '0000-VI00060843-1-20250317-1' for key 'PRIMARY'; nested exception is java.sql.SQLIn274061) Duplicate entry '0000-VI00060843-1-20250317-1' for key 'PRIMARY'
        at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.doTranslate(SQLErrorCodeSQLExceptionTr!/:5.3.23]
        at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExce23.jar!/:5.3.23]
        at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.jav
        at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:441) ~[mybatis
        at com.sun.proxy.$Proxy79.insert(Unknown Source) ~[na:na]
        at org.mybatis.spring.SqlSessionTemplate.insert(SqlSessionTemplate.java:272) ~[mybatis-spring-2.0.6.jar!/:2.
        at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:62) ~[mybatis-3.5.7.jar!/:3.5.7]
        at org.apache.ibatis.binding.MapperProxy$PlainMethodInvoker.invoke(MapperProxy.java:145) ~[mybatis-3.5.7.jar
        at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:86) ~[mybatis-3.5.7.jar!/:3.5.7]
        at com.sun.proxy.$Proxy183.insertVisitPersonInOut(Unknown Source) ~[na:na]
        at com.portal.iogate.service.impl.IogateVisitsServiceImpl.savePersonInOut(IogateVisitsServiceImpl.java:199)
        at com.portal.iogate.service.impl.IogateVisitsServiceImpl.saveInout(IogateVisitsServiceImpl.java:339) ~[clas
        at com.portal.iogate.service.impl.IogateVisitsServiceImpl$$FastClassBySpringCGLIB$$b3a5b46f.invoke(<generate
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.3.23.jar!/:5.3.2
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[s
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInt:5.3.23]
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAar!/:5.3.23]
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[s
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:9
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[s
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:70
        at com.portal.iogate.service.impl.IogateVisitsServiceImpl$$EnhancerBySpringCGLIB$$ae219b57.saveInout(<genera
        at com.portal.iogate.controller.IogateVisitsController.doAction(IogateVisitsController.java:154) ~[classes!/
        at com.portal.iogate.controller.IogateVisitsController$$FastClassBySpringCGLIB$$4d0c131f.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.3.23.jar!/:5.3.2
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[s
        at org.springframework.security.access.intercept.aopalliance.MethodSecurityInterceptor.invoke(MethodSecurity-5.6.7.jar!/:5.6.7]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[s
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:9
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[s
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:70
        at com.portal.iogate.controller.IogateVisitsController$$EnhancerBySpringCGLIB$$e05feed3.doAction(<generated>
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_362]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_362]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_362]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_362]
        at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205) ~
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.jav
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(Servl-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(Reng-webmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(Requestbmvc-5.3.23.jar!/:5.3.23]
        at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapt3.23]
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1071) ~[spring-webmvc
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:964) ~[spring-webmvc-5
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006) [spring-webmv
        at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909) [spring-webmvc-5.3.23.
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:681) [tomcat-embed-core-9.0.65.jar!/:4.0.1]
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883) [spring-webmvc-5.3.23
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:764) [tomcat-embed-core-9.0.65.jar!/:4.0.1]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227) [tomcat
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-c
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) [tomcat-embed-websocket-9.0.65.jar
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-c
        at com.portal.core.security.filter.CorsFilter.doFilter(CorsFilter.java:51) [classes!/:0.0.1-SNAPSHOT]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-c
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:337)
        at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.invoke(FilterSecurityIntercepr!/:5.6.7]
        at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.doFilter(FilterSecurityIntercar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.ja6.7]
        at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.ja6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:12
        at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:81
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter(AnonymousAuthentic5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter(SecurityContng-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter(RequestCacheAwareFilter.ja.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at com.portal.core.security.filter.HictSecurityFilter.doFilter(HictSecurityFilter.java:187) [classes!/:0.0.1
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:103) [spri
        at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:89) [sprin
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.web.filter.CorsFilter.doFilterInternal(CorsFilter.java:91) [spring-web-5.3.23.jar!/:5
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.header.HeaderWriterFilter.doHeadersAfter(HeaderWriterFilter.java:90) [sp
        at org.springframework.security.web.header.HeaderWriterFilter.doFilterInternal(HeaderWriterFilter.java:75) [
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersist.6.7.jar!/:5.6.7]
        at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersist6.7.jar!/:5.6.7]
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal(spring-security-web-5.6.7.jar!/:5.6.7]
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5
        at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)
        at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:221) [spring-sec
        at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:186) [spring-security-we
        at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:354) [spri
        at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:267) [spring-web
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-c
        at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-c
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:197) [tomcat-embed-core-9.
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97) [tomcat-embed-core-9.0
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541) [tomcat-embed-core
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:135) [tomcat-embed-core-9.0.65.j
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.65.ja
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78) [tomcat-embed-core-9.0.6
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:360) [tomcat-embed-core-9.0.65.jar
        at org.apache.coyote.ajp.AjpProcessor.service(AjpProcessor.java:433) [tomcat-embed-core-9.0.65.jar!/:na]
        at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65) [tomcat-embed-core-9.0.6
        at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:890) [tomcat-embed-cor
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1789) [tomcat-embed-core-9.
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.65
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1191) [tomcat-embed-c
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659) [tomcat-embed-c
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0
        at java.lang.Thread.run(Thread.java:750) [na:1.8.0_362]
Caused by: java.sql.SQLIntegrityConstraintViolationException: (conn=274061) Duplicate entry '0000-VI00060843-1-20250
        at org.mariadb.jdbc.internal.util.exceptions.ExceptionFactory.createException(ExceptionFactory.java:70) ~[ma
        at org.mariadb.jdbc.internal.util.exceptions.ExceptionFactory.create(ExceptionFactory.java:158) ~[mariadb-ja
        at org.mariadb.jdbc.MariaDbStatement.executeExceptionEpilogue(MariaDbStatement.java:262) ~[mariadb-java-clie
        at org.mariadb.jdbc.ClientSidePreparedStatement.executeInternal(ClientSidePreparedStatement.java:229) ~[mari
        at org.mariadb.jdbc.ClientSidePreparedStatement.execute(ClientSidePreparedStatement.java:149) ~[mariadb-java
        at net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy.execute(PreparedStatementSpy.java:443) ~[log4jdbc-log4j2
        at com.zaxxer.hikari.pool.ProxyPreparedStatement.execute(ProxyPreparedStatement.java:44) ~[HikariCP-4.0.3.ja
        at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.execute(HikariProxyPreparedStatement.java) ~[HikariCP
        at org.apache.ibatis.executor.statement.PreparedStatementHandler.update(PreparedStatementHandler.java:47) ~[
        at org.apache.ibatis.executor.statement.RoutingStatementHandler.update(RoutingStatementHandler.java:74) ~[my
        at org.apache.ibatis.executor.SimpleExecutor.doUpdate(SimpleExecutor.java:50) ~[mybatis-3.5.7.jar!/:3.5.7]
        at org.apache.ibatis.executor.BaseExecutor.update(BaseExecutor.java:117) ~[mybatis-3.5.7.jar!/:3.5.7]
        at org.apache.ibatis.executor.CachingExecutor.update(CachingExecutor.java:76) ~[mybatis-3.5.7.jar!/:3.5.7]
        at sun.reflect.GeneratedMethodAccessor359.invoke(Unknown Source) ~[na:na]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_362]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_362]
        at org.apache.ibatis.plugin.Plugin.invoke(Plugin.java:64) ~[mybatis-3.5.7.jar!/:3.5.7]
        at com.sun.proxy.$Proxy113.update(Unknown Source) ~[na:na]
        at org.apache.ibatis.session.defaults.DefaultSqlSession.update(DefaultSqlSession.java:194) ~[mybatis-3.5.7.j
        at org.apache.ibatis.session.defaults.DefaultSqlSession.insert(DefaultSqlSession.java:181) ~[mybatis-3.5.7.j
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_362]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_362]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_362]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_362]
        at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:427) ~[mybatis
        ... 122 common frames omitted
Caused by: org.mariadb.jdbc.internal.util.exceptions.MariaDbSqlException: Duplicate entry '0000-VI00060843-1-2025031
        at org.mariadb.jdbc.internal.util.exceptions.MariaDbSqlException.of(MariaDbSqlException.java:34) ~[mariadb-j
        at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.exceptionWithQuery(AbstractQueryProtocol.java:19
        at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.exceptionWithQuery(AbstractQueryProtocol.java:17
        at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.executeQuery(AbstractQueryProtocol.java:321) ~[m
        at org.mariadb.jdbc.ClientSidePreparedStatement.executeInternal(ClientSidePreparedStatement.java:220) ~[mari
        ... 143 common frames omitted
Caused by: java.sql.SQLException: Duplicate entry '0000-VI00060843-1-20250317-1' for key 'PRIMARY'
        at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.readErrorPacket(AbstractQueryProtocol.java:1695)
        at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.readPacket(AbstractQueryProtocol.java:1557) ~[ma
        at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.getResult(AbstractQueryProtocol.java:1520) ~[mar
        at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.executeQuery(AbstractQueryProtocol.java:318) ~[m
        ... 144 common frames omitted
```