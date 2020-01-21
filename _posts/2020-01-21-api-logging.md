---
layout: post
title:  "Api Logging"
date:   2020-01-21 20:00:00 +0900
categories: Backend
tag: Spring
---

* content
{:toc}


Request Response Loggging Using AbstractLoggingFilter

## ApiLoggingFilter.java
```
import com.rsupport.commons.core.util.StringUtils;
import lombok.extern.slf4j.Slf4j;
import lombok.val;
import org.springframework.http.MediaType;
import org.springframework.web.filter.AbstractRequestLoggingFilter;
import org.springframework.web.util.ContentCachingRequestWrapper;
import org.springframework.web.util.ContentCachingResponseWrapper;
import org.springframework.web.util.WebUtils;


import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Arrays;
import java.util.List;


@Slf4j
public class ApiLoggingFilter extends AbstractRequestLoggingFilter {


  private static final List<MediaType> LOGGING_TYPE = Arrays.asList(
      MediaType.APPLICATION_FORM_URLENCODED,
      MediaType.APPLICATION_JSON,
      MediaType.APPLICATION_JSON_UTF8,
      MediaType.APPLICATION_XML,
      MediaType.valueOf("application/*+json"),
      MediaType.valueOf("application/*+xml")
  );


  @Override
  protected void beforeRequest(HttpServletRequest httpServletRequest, String s) {
    log.info("{}", s);
  }


  @Override
  protected void afterRequest(HttpServletRequest httpServletRequest, String s) {
    log.info("{}", s);
  }


  private void writeResponse(HttpServletResponse responseToCache) {
    String responseData = getResponseData(responseToCache);
    log.debug("response:{}", responseData);
  }


  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException,
      IOException {
    boolean isFirstRequest = !this.isAsyncDispatch(request);
    Object requestToUse = request;
    if (this.isIncludePayload() && isFirstRequest && !(request instanceof ContentCachingRequestWrapper)) {
      requestToUse = new ContentCachingRequestWrapper(request, this.getMaxPayloadLength());
    }


    boolean shouldLog = this.shouldLog((HttpServletRequest)requestToUse);
    if (shouldLog && isFirstRequest) {
      this.beforeRequest((HttpServletRequest)requestToUse, this.getBeforeMessage((HttpServletRequest)requestToUse));
    }


    ContentCachingResponseWrapper responseToCache = new ContentCachingResponseWrapper(response);


    try {
      filterChain.doFilter((ServletRequest)requestToUse, responseToCache);
    } finally {
      if (shouldLog && !this.isAsyncStarted((HttpServletRequest)requestToUse)) {
        this.afterRequest((HttpServletRequest)requestToUse, this.getAfterMessage((HttpServletRequest)requestToUse));
        this.writeResponse(responseToCache);
      }
      responseToCache.copyBodyToResponse();
    }
  }


  @Override
  protected boolean shouldLog(HttpServletRequest request) {
    String contentType = request.getContentType();
    if (StringUtils.isBlank(contentType))
      return false;
    val mediaType = MediaType.valueOf(request.getContentType());
    val enabledLoggingType = LOGGING_TYPE.stream().anyMatch(visibleType -> visibleType.includes(mediaType));
    return enabledLoggingType;
  }


  private static String getResponseData(final HttpServletResponse response) {
    String payload = null;
    ContentCachingResponseWrapper wrapper = WebUtils.getNativeResponse(response, ContentCachingResponseWrapper.class);
    if (wrapper != null) {
      byte[] buf = wrapper.getContentAsByteArray();
      if (buf.length > 0) {
        try {
          payload = new String(buf, 0, buf.length, wrapper.getCharacterEncoding());
        } catch (IOException e) {
          // ignored
        }
      }
    }
    return payload;
  }


  private String getBeforeMessage(HttpServletRequest request) {
    String prefix = String.format("before request[%s] [", request.getMethod());
    String suffix = "]";
    return this.createMessage(request, prefix, suffix);
  }


  private String getAfterMessage(HttpServletRequest request) {
    String prefix = String.format("request[%s] [", request.getMethod());
    String suffix = "]";
    return this.createMessage(request, prefix, suffix);
  }


}
```

## WebAppInitiallizer.java  
```
@Slf4j
public class WebAppInitializer extends AbstractWebAppInitializer {


  @Override
  public void onStartup(ServletContext servletContext) throws ServletException {
    super.onStartup(servletContext);
    addListener(servletContext);
    addCharacterEncodingFilter(servletContext);
    addApiLoggingFilter(servletContext);


...
  private void addApiLoggingFilter(ServletContext servletContext) {
    ApiLoggingFilter apiLoggingFilter = new ApiLoggingFilter();
    apiLoggingFilter.setIncludePayload(true);
    apiLoggingFilter.setIncludeQueryString(true);
    apiLoggingFilter.setIncludeHeaders(true);
    apiLoggingFilter.setMaxPayloadLength(10000);
    FilterRegistration.Dynamic filter = servletContext.addFilter("apiLoggingFilter", apiLoggingFilter);
    filter.addMappingForUrlPatterns(null, false, "/*");
  }
```