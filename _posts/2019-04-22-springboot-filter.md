---
layout: post
title:  "Spring Boot에서 Filter 적용하기"
date:   2019-04-22 10:55:00 +0900
categories: Backend
tag: SpringBoot
---
* content
{:toc}


## Filter 적용
Spring에서는 다음과 같이 필터를 등록했다

```
  private void addHttpservletRequestFilter(ServletContext servletContext) {
    HttpServletRequestFilter httpServletRequestFilter = new HttpServletRequestFilter();
    FilterRegistration.Dynamic filter = servletContext.addFilter("httpServletRequestFilter", httpServletRequestFilter);
    filter.addMappingForUrlPatterns(null, false, "/*");
  }
```
  
->
Spring Boot에서는 Bean으로 선언해주면 됨  
```
  @Bean
  public HttpServletRequestFilter httpServletRequestFilter() {
    return new HttpServletRequestFilter();
  }
```
## Filter 적용 - 경로 매핑하기  
경로를 매핑해야 하는 경우 FilterRegistrationBean을 사용하면 된다.  
```
  private void addServerAuthorizatioinFilter(ServletContext servletContext) {
    ServerAuthorizationFilter serverAuthorizationFilter = new ServerAuthorizationFilter();
    FilterRegistration.Dynamic filter = servletContext.addFilter("serverAuthorizationFilter", serverAuthorizationFilter);
    filter.addMappingForUrlPatterns(null, false, "/advancedServer", "/advancedServer/*", "/mediaServer", "/mediaServer/*", "/documentServer",
        "/documentServer/*");
  }
```

->
```
  @Bean
  public FilterRegistrationBean serverAuthorizationFilter() {
    ServerAuthorizationFilter serverAuthorizationFilter = new ServerAuthorizationFilter();
    FilterRegistrationBean filterRegistration = new FilterRegistrationBean();
    filterRegistration.setFilter(serverAuthorizationFilter);
    filterRegistration.setMatchAfter(false);
    filterRegistration.addUrlPatterns("/advancedServer", "/advancedServer/*", "/mediaServer", "/mediaServer/*", "/documentServer");
    return filterRegistration;
  }
```
