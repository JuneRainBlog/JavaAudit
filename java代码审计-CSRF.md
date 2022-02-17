### 原理

跨站请求伪造,通过伪装来自授信用用户的请求来利用受信任的网站。

### 要点

快速方法：

```
查看是否存在CSRF防范机制，如Spring Security、
是否检验Referer
是否给cookie设置SameSite属性和敏感操作是否会生成CSRF token
安卓端⼀般⽆需考虑CSRF问题
```

### 防御

总结：除了携带 Cookie 中的信息之外，还需要携带一个随机数

- 可以使用interceptor或者filter或者servlet校验Referer
- 开启框架的安全机制：比如Spring Security自带的安全机制：http.csrf
- 使用JWT进行身份校验，因为自带token所以可以防御csrf
- 一些高危操作，比如支付，修改密码等，需要使用短信验证码验证



防御的代码

使用interceptor来校验Referer，以下代码为例：

```
@Component
public class RefererInterceptor extends HandlerInterceptorAdapter {
	private AntPathMatcher matcher = new AntPathMatcher();
	@Autowired
	private RefererProperties properties;
	@Override
	public boolean preHandle(HttpServletRequest req, HttpServletResponse resp, Object handler) throws Exception {
		String referer = req.getHeader("referer");
		String host = req.getServerName();
		if ("POST".equals(req.getMethod())) {  // 只验证POST请求
			if (referer == null) { //若referer为空
				resp.setStatus(HttpServletResponse.SC_FORBIDDEN); //返回403
				return false;
			} 
			java.net.URL url = null;
			try {
				url = new java.net.URL(referer);
			} catch (MalformedURLException e) {
				resp.setStatus(HttpServletResponse.SC_FORBIDDEN); // URL解析异常，也置为403
				return false;
			}
			if (!host.equals(url.getHost())) {  // 首先判断请求域名和referer域名是否相同
				if (properties.getRefererDomain() != null) {  // 如果不等，判断是否在白名单中
					for (String s : properties.getRefererDomain()) {
						if (s.equals(url.getHost())) {
							return true;
						}
					}
				}
				return false;
			}
		}
		return true;
	}
}
```

可以使用Servlet来校验Referer，以下代码为例：

```
public class RefererServlet extends HttpServlet {
 
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        String header = request.getHeader("Referer");
        //String domainName = null;
        String[] domain = { "localhost", "test.localhost", "admin.localhost" };
        boolean key=false;
        for (int i = 0; i < domain.length; i++) {
            if (header != null && header.startsWith("http://" + domain[i]) && header.endsWith(domain[i])) {
                key=true;
            }
        }
        if(key==true) {
            response.getWriter().write("成功读到打到数据");
        }else{
            response.getWriter().write("非法请求");
        }
    }
 
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

给cookie设置SameSite属性

```
@Component
public class CookieServiceInterceptor extends HandlerInterceptorAdapter {

@Override
public boolean preHandle(
	HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
	return true;
}

@Override
public void postHandle(
	HttpServletRequest request, HttpServletResponse response, Object handler, 
	ModelAndView modelAndView) throws Exception {
		//检查返回中是否存在"set-cookie"，若存在，则给其加上"SameSite"属性
		Collection<String> headers = response.getHeaders(HttpHeaders.SET_COOKIE);
		boolean firstHeader = true;
		for (String header : headers) { // 可能存在多个"Set-Cookie"属性，故采用for循环
			if (firstHeader) {
				response.setHeader(HttpHeaders.SET_COOKIE, String.format("%s; %s",  header, "SameSite=strict")); //可根据业务设置strict或者lax
				firstHeader = false;
				continue;
			}
		response.addHeader(HttpHeaders.SET_COOKIE, String.format("%s; %s",  header, "SameSite=strict"));
	}
}
@Override
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
	Object handler, Exception exception) throws Exception {}
}
```

设置CSRF token

```
CSRF token生成可以借助Spring Security框架，默认提供CSRF防护。Spring Security的CSRF防护是基于Filter来实现的，Spring Security提供多种保存token的策略，既可以保存在cookie中，也可以保存在session中，保存方法可以手动指定。

protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
	request.setAttribute(HttpServletResponse.class.getName(), response);
	// 通过tokenRepository从request中获取csrf token
	CsrfToken csrfToken = this.tokenRepository.loadToken(request);
	final boolean missingToken = csrfToken == null;
	// 如果未获取到token则新生成token并保存
	if (missingToken) {
		csrfToken = this.tokenRepository.generateToken(request);
		this.tokenRepository.saveToken(csrfToken, request, response);
	}
	request.setAttribute(CsrfToken.class.getName(), csrfToken);
	request.setAttribute(csrfToken.getParameterName(), csrfToken);
	// 判断是否需要进行csrf token校验
	if (!this.requireCsrfProtectionMatcher.matches(request)) {
		filterChain.doFilter(request, response);
		return;
	}
	// 获取前端传过来的实际token
	String actualToken = request.getHeader(csrfToken.getHeaderName());
	if (actualToken == null) {
		actualToken = request.getParameter(csrfToken.getParameterName());
	}
	// 校验两个token是否相等
	if (!csrfToken.getToken().equals(actualToken)) {
		if (this.logger.isDebugEnabled()) {
			this.logger.debug("Invalid CSRF token found for "
				+ UrlUtils.buildFullRequestUrl(request));
		}
		// 如果是token缺失导致，则抛出MissingCsrfTokenException异常
		if (missingToken) {
			this.accessDeniedHandler.handle(request, response,
				new MissingCsrfTokenException(actualToken));
		}
		// 如果不是同一个token则抛出InvalidCsrfTokenException异常
		else {
			this.accessDeniedHandler.handle(request, response,
			new InvalidCsrfTokenException(csrfToken, actualToken));
		}
		return;
	}
	// 执行下一个过滤器
	filterChain.doFilter(request, response);
}
```



