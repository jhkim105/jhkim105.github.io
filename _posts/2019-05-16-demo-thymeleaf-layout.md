---
layout: post
title:  "Thymeleaf Fragments and Layout dialect"
date:   2019-05-16 18:15:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}



# Overview
Thymeleaf fragments와 Layout dialect를 사용해서 페이지를 구성하는 방법에 대해 살펴보자.


# Fragments in Thymeleaf
Page를 다음과 같이 영역을 Fragment로 나눈다.
![Page Fragments]({{site.url}}/assets/images/2019-05/thymeleaf-layout-01.png)  

작업전  
![Page Fragments 작업전]({{site.url}}/assets/images/2019-05/thymeleaf-layout-02.png)  

## Framents

### common.html
Page에서 나누고 싶은 부분은 th:fragment로 지정하여 별도 html로 분리
```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <ul th:fragment="sidebar" class="navbar-nav bg-gradient-primary sidebar sidebar-dark accordion" id="accordionSidebar">
      <!-- Sidebar - Brand -->
      <a class="sidebar-brand d-flex align-items-center justify-content-center" href="index.html">
        <div class="sidebar-brand-icon rotate-n-15">
          <i class="fas fa-laugh-wink"></i>
        </div>
        <div class="sidebar-brand-text mx-3">SB Admin <sup>2</sup></div>
      </a>
      <!-- Divider -->
      <hr class="sidebar-divider my-0">


      <!-- Nav Item - Settings -->
      <li class="nav-item">
        <a class="nav-link" href="@{/index}">
          <i class="fas fa-fw fa-table"></i>
          <span>Settings</span></a>
      </li>
      <!-- Divider -->
      <hr class="sidebar-divider d-none d-md-block">
      <!-- Sidebar Toggler (Sidebar) -->
      <div class="text-center d-none d-md-inline">
        <button class="rounded-circle border-0" id="sidebarToggle"></button>
      </div>
    </ul>
</html>
```

## content.html
th:insert로 삽입하거나, th:replace로 교체
```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
...
    <!-- Sidebar -->
    <ul th:replace="fragments/common.html :: sidebar"></ul>
    <!-- End of Sidebar -->
```

## Fragments 적용 후
![Fragments 적용 후]({{site.url}}/assets/images/2019-05/thymeleaf-layout-03.png)  

# Layout(decorator)
Thymeleaf Layout Dialect를 사용하면 Sitemesh처럼 Layout(decorator) 사용이 가능하다  
[https://ultraq.github.io/thymeleaf-layout-dialect/](https://ultraq.github.io/thymeleaf-layout-dialect/)  

## maven dependency
```
    <dependency>
      <groupId>nz.net.ultraq.thymeleaf</groupId>
      <artifactId>thymeleaf-layout-dialect</artifactId>
    </dependency>
```

## Layout
layout 파일에서 content 자리에 아래 태크를 삽입  
```
<div layout:fragment="content"></div>
```
    
content page에서  
* layout:decorate="~{layout/default.html}" 로 decorator를 지정  
* content자리에 layout과 동일하게 tag 지정  
    ```
    <div layout:fragment="content">
    ```


### layouts/default.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <meta name="description" content="">
  <meta name="author" content="">
  <title>SB Admin 2 - Blank</title>
  <!-- Custom fonts for this template-->
  <link href="/webjars/font-awesome/5.8.2/css/all.min.css" rel="stylesheet" type="text/css">
  <link href="https://fonts.googleapis.com/css?family=Nunito:200,200i,300,300i,400,400i,600,600i,700,700i,800,800i,900,900i" rel="stylesheet">
  <!-- Custom styles for this template-->
  <link href="/webjars/startbootstrap-sb-admin-2/4.0.2/css/sb-admin-2.min.css" rel="stylesheet">
</head>
<body id="page-top">
  <!-- Page Wrapper -->
  <div id="wrapper">
    <!-- Sidebar -->
    <ul th:replace="fragments/common.html :: sidebar"></ul>
    <!-- End of Sidebar -->
    <!-- Content Wrapper -->
    <div id="content-wrapper" class="d-flex flex-column">
      <!-- Main Content -->
      <div layout:fragment="content"></div>
      <!-- End of Main Content -->
      <!-- Footer -->
      <footer th:replace="fragments/common.html :: footer"></footer>
      <!-- End of Footer -->
    </div>
    <!-- End of Content Wrapper -->
  </div>
  <!-- End of Page Wrapper -->
  <!-- Scroll to Top Button-->
  <a class="scroll-to-top rounded" href="#page-top">
    <i class="fas fa-angle-up"></i>
  </a>
  <!-- Logout Modal-->
  <div class="modal fade" id="logoutModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="exampleModalLabel">Ready to Leave?</h5>
          <button class="close" type="button" data-dismiss="modal" aria-label="Close">
            <span aria-hidden="true">×</span>
          </button>
        </div>
        <div class="modal-body">Select "Logout" below if you are ready to end your current session.</div>
        <div class="modal-footer">
          <button class="btn btn-secondary" type="button" data-dismiss="modal">Cancel</button>
          <a class="btn btn-primary" href="login.html">Logout</a>
        </div>
      </div>
    </div>
  </div>
  <!-- Bootstrap core JavaScript-->
  <script src="/webjars/jquery/3.0.0/jquery.min.js"></script>
  <script src="/webjars/bootstrap/4.3.1/js/bootstrap.bundle.min.js"></script>
  <!-- Core plugin JavaScript-->
  <script src="/webjars/jquery-easing/1.4.1/jquery.easing.min.js"></script>
  <!-- Custom scripts for all pages-->
  <script src="/webjars/startbootstrap-sb-admin-2/4.0.2/js/sb-admin-2.min.js"></script>
</body>
</html>
```

### content.html
```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout/default.html}">
<head>
  <title>Content page 1</title>
</head>
<body>
  <!-- Main Content -->
  <div layout:fragment="content" id="content">
    <!-- Topbar -->
    <nav th:replace="fragments/common.html :: topbar"></nav>
    <!-- End of Topbar -->
    <!-- Begin Page Content -->
    <div class="container-fluid">
      <!-- Page Heading -->
      <h1 class="h3 mb-4 text-gray-800">Blank Page</h1>
    </div>
    <!-- /.container-fluid -->
  </div>
  <!-- End of Main Content -->
</body>
</html>
```

content page에서 script tag 지정한 것이 제일 마지막에 로드되게 하려면
layout 파일에 아래 태그 삽입하고 content에서 다음처럼 같은 이름으로 사용하면 된다.  
layout/defalut.html  
    ```
        <script layout:fragment="script"></script> 
    ```

content.html  
    ```
        <script layout:fragment="script" src="./js/index.js"></script>
    ```
