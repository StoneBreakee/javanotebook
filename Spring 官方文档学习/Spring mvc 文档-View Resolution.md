# 1.1.8 View Resolution #
----------
Spring中定义了**ViewResolver**和**View**这两个接口，通过这两个接口，我们不必实现视图技术，就可以将数据模型渲染到呈现给客户端的视图中。ViewResovler 的功能是将视图逻辑名映射到物理视图，此时的物理视图还没有model数据填充，model数据的填充在View.render(...)中处理。View 的功能是 在移交给特定视图技术之前的数据准备问题。

下面以ThymeleafView为例，分析View，ViewResolver和DispatcherServlet之间的运作关系。

### View.java

用于Web交互的MVC View视图,实现View接口的类负责将内容和model数据渲染给客户端,一个视图中可以包含多个model数据
每一个视图都应该对应与一个bean，这些view 是通过ViewResolver实例化为bean对象。

> View在MVC中的过程：
> 
> - 1. 未装配Model数据的View，由ViewResolver.resovleViewName(...)生成
> - 2. 装配过model数据的View，由View.render(...)完成
> - 3. 由DispatcherServlet的render(...)方法，依次调用1，2两步，并将装配后的View成品放入response输出流中

<pre><code>
public interface View {
    // 
    // 根据给定的model数据渲染视图
    // 第一步是 准备(装配)request：在jsp视图中，意味着将model数据设置为request对象中的一个键值属性。
    // 第二步是 生成真正的供客户端可浏览的数据，不仅仅是网页，诸如excel,pdf,json等都可以
    // @param response 我们需要将可浏览的视图放入到response输出流中返回给客户端
    // 
    void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
</code></pre>
### ViewResolver.java
用于将物理逻辑视图名映射为视图View实例，此时model数据还未装配到可视视图中。在spring web项目中，可能同时存在多个视图解析器，组成 **视图解析器链(ViewResolver Chaining)**。针对一个逻辑视图名，视图解析器链中的每个视图解析器都会去解析该逻辑视图名称，直到有一个解析成功为止。因此在配置视图解析器时，可以设置该视图解析器的优先级，数值越小优先级越高。
<pre><code>
public interface ViewResolver {
    /**
     * Resolve the given view by name.
     * @return the View object, or {@code null} if not found (optional, to allow for ViewResolver chaining)
     * @throws Exception if the view cannot be resolved (typically in case of problems creating an actual View object)
     */
    View resolveViewName(String viewName, Locale locale) throws Exception;
}
<code></pre>
### DispatcherServlet.java

DispatcherServlet前端控制器在将请求拦截后，将请求映射到相应的处理器上，由处理器处理之后返回一个逻辑视图名称，该请求在spring web中的最后一步，是由dispatcherServlet.render(..)处理。
<pre>
/**
 * Render the given ModelAndView.
 * This is the last stage in handling a request. It may involve resolving the view by name.
 * @param request current HTTP servlet request，model数据由request保存，在渲染的过程中将model从request中提取出来
 * @param response current HTTP servlet response,将渲染之后的View放入到response的输出流中
 */
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    View view;
    String viewName = mv.getViewName();
    // We need to resolve the view name.
    // 在这里调用ViewResolver.resolveViewName(...)
    view = resolveViewName(viewName, mv.getModelInternal(), locale, request); 
    if (view == null) {
    	throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
    			"' in servlet with name '" + getServletName() + "'");
    // 在这里调用View.render(..)
    view.render(mv.getModelInternal(), request, response);
}

/**
 * 将逻辑视图名解析为View对象，该View对象尚未被渲染.
 * 默认的实现是遍历DispatcherServlet中所有的ViewResolver.
 * @param model the model to be passed to the view
 * @param request current HTTP servlet request
 */
protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
        Locale locale, HttpServletRequest request) throws Exception {
    // this.viewResolvers : List of ViewResolvers used by this servlet 
    // ViewResolver Chaining
    if (this.viewResolvers != null) {
    	for (ViewResolver viewResolver : this.viewResolvers) {
    		View view = viewResolver.resolveViewName(viewName, locale);
    		if (view != null) {
    			return view;
    		}
    	}
    }
    return null;
}
</pre>
### ThymeleafView.java
ThymeleafView实现了View接口，其render(..)方法如下
<pre>
public void render(final Map<String, ?> model, final HttpServletRequest request, final HttpServletResponse response)
        throws Exception {
    renderFragment(this.markupSelectors, model, request, response);
}

protected void renderFragment(final Set<String> markupSelectorsToRender, final Map<String, ?> model, final HttpServletRequest request,
        final HttpServletResponse response)
        throws Exception {
    ...
	Thymeleaf用到了模板技术,在将view渲染并放进response outputstream进行以下步骤：
    1.获取模板名称
    2.提取request中的model数据和参数model进行合并
    3.标记文本选择器，用于将标记文本和model数据进行替换
    ...
    // If we have chosen to not output anything until processing finishes, we will use a buffer
    // 返回一个PrintWriter对象用于将渲染之后的视图成品放入到字符输出流中返回给客户端
    final Writer templateWriter =
            (producePartialOutputWhileProcessing? response.getWriter() : new FastStringWriter(1024));
	// 进行渲染并将结果视图放进response outputstream中，返回给客户端
    viewTemplateEngine.process(templateName, processMarkupSelectors, context, templateWriter);
}
</pre>