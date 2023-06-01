# 调用流程分析

在web.xml配置中，请求会被DispatcherServlet拦截。
DispatcherServlet继承于FrameworkServlet，FrameworkServlet继承于HttpServletBean，HttpServletBean继承于HttpServlet。

HttpServlet的doGet和doPost等方法在FrameworkServlet中被重写：
```java
protected final void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    this.processRequest(request, response);
}

protected final void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    this.processRequest(request, response);
}
```

在processRequest方法中调用了doService方法，该方法在FrameworkServlet中为抽象方法，具体实现在DispatcherServlet类中。
DispatcherServlet类的processRequest方法调用了doDispatch方法，主要的请求处理由该方法完成。




# doDispatch方法

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        try {
            ModelAndView mv = null;
            Object dispatchException = null;

            try {
                // 检测当前请求是否为文件上传请求
                processedRequest = this.checkMultipart(request);
                multipartRequestParsed = processedRequest != request;
                // 根据当前的请求地址找到对应的处理类
                mappedHandler = this.getHandler(processedRequest);
                if (mappedHandler == null) {
                    // 找不到处理器抛出异常或返回错误页面
                    this.noHandlerFound(processedRequest, response);
                    return;
                }
                // 拿到能执行这个类的所有方法的适配器（反射工具）
                HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }

                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }

                // 处理器的方法被调用，使用适配器执行目标方法
                // 无论目标方法返回值是什么类型，都会封装成ModelAndView
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }
                // 没有视图名则设置一个默认视图名
                this.applyDefaultViewName(processedRequest, mv);
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            } catch (Exception var20) {
                dispatchException = var20;
            } catch (Throwable var21) {
                dispatchException = new NestedServletException("Handler dispatch failed", var21);
            }
            // 转发到目标页面
            // 根据方法执行完封装的ModelAndView转发到对应页面，并且ModelAndView中的数据可以从请求域中获取
            this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
        } catch (Exception var22) {
            this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
        } catch (Throwable var23) {
            this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
        }

    } finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        } else if (multipartRequestParsed) {
            this.cleanupMultipart(processedRequest);
        }

    }
}
```


流程概述：
* 发送请求时，DispatcherServlet收到请求
* 调用doDispatch()方法进行处理
  * getHandler()方法根据当前请求在HandlerMapping中找到这个请求的映射信息，获取目标处理器类
  * getHandlerAdapter()方法根据当前处理器类获取到能执行这个处理器方法的适配器
  * 使用刚才获取到的适配器(AnnotationMethodHandlerAdatper，Spring5之后为RequestMappingHandlerAdapter)执行目标方法
  * 目标方法执行后会返回一个ModelAndView对象
  * 根据ModelAndView的信息转发到具体的页面，并可以在请求域中取出ModelAndView中的模型数据




# getHandler()方法

getHandler()会返回目标处理器的执行链：
```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        Iterator var2 = this.handlerMappings.iterator();

        while(var2.hasNext()) {
            HandlerMapping mapping = (HandlerMapping)var2.next();
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }

    return null;
}
```

handlerMapping：处理器映射，里面保存了每一个处理器能处理哪些请求的映射信息。
IOC容器启动创建Controller对象时会扫描每个处理器能处理哪些请求，将信息存入handlerMapping的handerMap属性中。




# getHandleAdapter()方法

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    if (this.handlerAdapters != null) {
        Iterator var2 = this.handlerAdapters.iterator();

        while(var2.hasNext()) {
            HandlerAdapter adapter = (HandlerAdapter)var2.next();
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
    }

    throw new ServletException("No adapter for handler [" + handler + "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

RequestMappingHandlerAdapter：能解析注解方法的适配器，处理器类中只要有标了注解的方法就能使用。




# SpringMVC的九大组件

在DispatcherServlet中有几个引用类型的属性：
```java
// 文件上传解析器
private MultipartResolver multipartResolver;
// 区域信息解析器：和国际化有关
private LocaleResolver localeResolver;
// 主题解析器：强大的主题效果更换
private ThemeResolver themeResolver;
// Handler映射信息
private List<HandlerMapping> handlerMappings;
// Handler的适配器
private List<HandlerAdapter> handlerAdapters;
// SpringMVC的异常解析功能：异常解析器
private List<HandlerExceptionResolver> handlerExceptionResolvers;

private RequestToViewNameTranslator viewNameTranslator;
// SpringMVC中运行重定向携带数据的功能
private FlashMapManager flashMapManager;
// 视图解析器
private List<ViewResolver> viewResolvers;
```

这些属性称为SpringMVC的九大组件，它们都是接口。SpringMVC在工作的时候，关键位置都是由这些组件完成的。在初始化时会为这些组件赋值。

九大组件初始化方法：
```java
protected void initStrategies(ApplicationContext context) {
    this.initMultipartResolver(context);
    this.initLocaleResolver(context);
    this.initThemeResolver(context);
    this.initHandlerMappings(context);
    this.initHandlerAdapters(context);
    this.initHandlerExceptionResolvers(context);
    this.initRequestToViewNameTranslator(context);
    this.initViewResolvers(context);
    this.initFlashMapManager(context);
}
```

初始化HandlerMapping：
```java
private void initHandlerMappings(ApplicationContext context) {
    this.handlerMappings = null;
    if (this.detectAllHandlerMappings) {
        Map<String, HandlerMapping> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList(matchingBeans.values());
            AnnotationAwareOrderComparator.sort(this.handlerMappings);
        }
    } else {
        try {
            HandlerMapping hm = (HandlerMapping)context.getBean("handlerMapping", HandlerMapping.class);
            this.handlerMappings = Collections.singletonList(hm);
        } catch (NoSuchBeanDefinitionException var3) {
        }
    }

    if (this.handlerMappings == null) {
        this.handlerMappings = this.getDefaultStrategies(context, HandlerMapping.class);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("No HandlerMappings declared for servlet '" + this.getServletName() + "': using default strategies from DispatcherServlet.properties");
        }
    }

}
```

大致流程：去容器中寻找对应组件，找不到则使用默认配置。




# 目标方法的执行

mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
这个语句核心是调用invokeHandlerMethod()方法执行目标方法。

```java
protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
    ServletWebRequest webRequest = new ServletWebRequest(request, response);

    Object result;
    try {
        WebDataBinderFactory binderFactory = this.getDataBinderFactory(handlerMethod);
        ModelFactory modelFactory = this.getModelFactory(handlerMethod, binderFactory);
        ServletInvocableHandlerMethod invocableMethod = this.createInvocableHandlerMethod(handlerMethod);
        if (this.argumentResolvers != null) {
            invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
        }

        if (this.returnValueHandlers != null) {
            invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
        }

        invocableMethod.setDataBinderFactory(binderFactory);
        invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);
        ModelAndViewContainer mavContainer = new ModelAndViewContainer();
        mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
        modelFactory.initModel(webRequest, mavContainer, invocableMethod);
        mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);
        AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
        asyncWebRequest.setTimeout(this.asyncRequestTimeout);
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        asyncManager.setTaskExecutor(this.taskExecutor);
        asyncManager.setAsyncWebRequest(asyncWebRequest);
        asyncManager.registerCallableInterceptors(this.callableInterceptors);
        asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);
        if (asyncManager.hasConcurrentResult()) {
            result = asyncManager.getConcurrentResult();
            mavContainer = (ModelAndViewContainer)asyncManager.getConcurrentResultContext()[0];
            asyncManager.clearConcurrentResult();
            LogFormatUtils.traceDebug(this.logger, (traceOn) -> {
                String formatted = LogFormatUtils.formatValue(result, !traceOn);
                return "Resume with async result [" + formatted + "]";
            });
            invocableMethod = invocableMethod.wrapConcurrentResult(result);
        }
        // 真正执行目标方法
        invocableMethod.invokeAndHandle(webRequest, mavContainer, new Object[0]);
        if (!asyncManager.isConcurrentHandlingStarted()) {
            ModelAndView var15 = this.getModelAndView(mavContainer, modelFactory, webRequest);
            return var15;
        }

        result = null;
    } finally {
        webRequest.requestCompleted();
    }

    return (ModelAndView)result;
}
```


invokeAndHandle方法：
```java
public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
    Object returnValue = this.invokeForRequest(webRequest, mavContainer, providedArgs);
    this.setResponseStatus(webRequest);
    if (returnValue == null) {
        if (this.isRequestNotModified(webRequest) || this.getResponseStatus() != null || mavContainer.isRequestHandled()) {
            this.disableContentCachingIfNecessary(webRequest);
            mavContainer.setRequestHandled(true);
            return;
        }
    } else if (StringUtils.hasText(this.getResponseStatusReason())) {
        mavContainer.setRequestHandled(true);
        return;
    }

    mavContainer.setRequestHandled(false);
    Assert.state(this.returnValueHandlers != null, "No return value handlers");

    try {
        this.returnValueHandlers.handleReturnValue(returnValue, this.getReturnValueType(returnValue), mavContainer, webRequest);
    } catch (Exception var6) {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace(this.formatErrorForReturnValue(returnValue), var6);
        }

        throw var6;
    }
}
```




# 视图解析器

任何方法的返回值，最终都会被包装成ModelAndView对象。
来到页面的方法：
```java
this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
```

视图渲染流程：将域中的数据在页面展示，页面是用来渲染模型数据的。
实际调用的渲染页面方法：
```java
this.render(mv, request, response);
```

render方法实现：
```java
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    Locale locale = this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale();
    response.setLocale(locale);
    String viewName = mv.getViewName();
    View view;
    if (viewName != null) {
        view = this.resolveViewName(viewName, mv.getModelInternal(), locale, request);
        if (view == null) {
            throw new ServletException("Could not resolve view with name '" + mv.getViewName() + "' in servlet with name '" + this.getServletName() + "'");
        }
    } else {
        view = mv.getView();
        if (view == null) {
            throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a View object in servlet with name '" + this.getServletName() + "'");
        }
    }

    if (this.logger.isTraceEnabled()) {
        this.logger.trace("Rendering view [" + view + "] ");
    }

    try {
        if (mv.getStatus() != null) {
            response.setStatus(mv.getStatus().value());
        }

        view.render(mv.getModelInternal(), request, response);
    } catch (Exception var8) {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Error rendering view [" + view + "]", var8);
        }

        throw var8;
    }
}
```

View与ViewResolver：ViewResolver的作用是根据视图名（方法的返回值）得到View对象。
根据方法名获取视图：
```java
protected View resolveViewName(String viewName, @Nullable Map<String, Object> model, Locale locale, HttpServletRequest request) throws Exception {
    if (this.viewResolvers != null) {
        Iterator var5 = this.viewResolvers.iterator();

        while(var5.hasNext()) {
            ViewResolver viewResolver = (ViewResolver)var5.next();
            // ViewResolver视图解析器根据方法的返回值，得到一个View对象
            View view = viewResolver.resolveViewName(viewName, locale);
            if (view != null) {
                return view;
            }
        }
    }

    return null;
}
```


resolveViewName实现：
```java
public View resolveViewName(String viewName, Locale locale) throws Exception {
    if (!this.isCache()) {
        return this.createView(viewName, locale);
    } else {
        Object cacheKey = this.getCacheKey(viewName, locale);
        View view = (View)this.viewAccessCache.get(cacheKey);
        if (view == null) {
            synchronized(this.viewCreationCache) {
                view = (View)this.viewCreationCache.get(cacheKey);
                if (view == null) {
                    view = this.createView(viewName, locale);
                    if (view == null && this.cacheUnresolved) {
                        view = UNRESOLVED_VIEW;
                    }

                    if (view != null && this.cacheFilter.filter(view, viewName, locale)) {
                        this.viewAccessCache.put(cacheKey, view);
                        this.viewCreationCache.put(cacheKey, view);
                    }
                }
            }
        } else if (this.logger.isTraceEnabled()) {
            this.logger.trace(formatKey(cacheKey) + "served from cache");
        }

        return view != UNRESOLVED_VIEW ? view : null;
    }
}
```

createView方法：
```java
protected View createView(String viewName, Locale locale) throws Exception {
    if (!this.canHandle(viewName, locale)) {
        return null;
    } else {
        String forwardUrl;
        if (viewName.startsWith("redirect:")) {
            forwardUrl = viewName.substring("redirect:".length());
            RedirectView view = new RedirectView(forwardUrl, this.isRedirectContextRelative(), this.isRedirectHttp10Compatible());
            String[] hosts = this.getRedirectHosts();
            if (hosts != null) {
                view.setHosts(hosts);
            }

            return this.applyLifecycleMethods("redirect:", view);
        } else if (viewName.startsWith("forward:")) {
            forwardUrl = viewName.substring("forward:".length());
            InternalResourceView view = new InternalResourceView(forwardUrl);
            return this.applyLifecycleMethods("forward:", view);
        } else {
            return super.createView(viewName, locale);
        }
    }
}
```

视图解析器得到View对象的流程就是所有配置的视图解析器都来尝试根据视图名（返回值）得到View对象，如果能得到就返回，得不到就换下一个视图解析器。
调用View对象的render方法：
```java
public void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("View " + this.formatViewName() + ", model " + (model != null ? model : Collections.emptyMap()) + (this.staticAttributes.isEmpty() ? "" : ", static attributes " + this.staticAttributes));
    }

    Map<String, Object> mergedModel = this.createMergedOutputModel(model, request, response);
    this.prepareResponse(request, response);
    // 渲染要给页面输出的所有数据
    this.renderMergedOutputModel(mergedModel, this.getRequestToExpose(request), response);
}
```

InternalResourceView的渲染方法：
```java
protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 将模型中的所有数据取出来放在request域中
    this.exposeModelAsRequestAttributes(model, request);
    this.exposeHelpers(request);
    String dispatcherPath = this.prepareForRendering(request, response);
    RequestDispatcher rd = this.getRequestDispatcher(request, dispatcherPath);
    if (rd == null) {
        throw new ServletException("Could not get RequestDispatcher for [" + this.getUrl() + "]: Check that the corresponding file exists within your web application archive!");
    } else {
        if (this.useInclude(request, response)) {
            response.setContentType(this.getContentType());
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Including [" + this.getUrl() + "]");
            }

            rd.include(request, response);
        } else {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Forwarding to [" + this.getUrl() + "]");
            }

            rd.forward(request, response);
        }

    }
}
```

```java
protected void exposeModelAsRequestAttributes(Map<String, Object> model, HttpServletRequest request) throws Exception {
    model.forEach((name, value) -> {
        if (value != null) {
            request.setAttribute(name, value);
        } else {
            request.removeAttribute(name);
        }

    });
}
```

总结：视图解析器只是为了得到视图对象，视图对象才能真正地转发（将模型数据全部放在请求域中）或者重定向到页面。



# 运行流程

1. 前端控制器(DispatcherServlet)收到请求，调用doDispatch进行处理
2. 根据HandlerMapping中保存的请求映射信息，找到处理当前请求的处理器执行链（包含拦截器）
3. 根据当前处理器找到HandlerAdapter（适配器）
4. 拦截器的preHandle先执行
5. 适配器执行目标方法，并返回ModelAndView
   * ModelAttribute注解标注的方法提前运行
   * 确认目标方法参数，如果是自定义类型，从隐含模型中获取
   * 若获取不到，则判断是否为SessionAttributes标准的属性
   * 都获取不到就通过反射创建对象
6. 拦截器的postHandle执行
7. 处理结果（页面渲染流程）
   * 如果有异常，使用异常解析器处理
   * 调用render方法进行页面渲染，视图解析器得到视图对象，调用render方法
8. 执行拦截器的afterCompletion方法
