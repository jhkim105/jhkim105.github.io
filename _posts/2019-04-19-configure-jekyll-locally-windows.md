---
layout: post
title:  "Configure Jekyll locally on Windows"
date:   2019-04-19 14:19:01 +0900
categories: Etc
tag: Jekyll
---

* content
{:toc}


Setting up your GitHub Pages site locally with Jekyll - Windows


------------------------

# Install ruby
https://rubyinstaller.org/downloads/  
  
![]({{site.url}}/assets/images/2019-05/jekyll-local-windows-01.png)



# Install Bundler
```
gem install bundler     
```

# Theme download
[jekyllthemes.org](http://jekyllthemes.org)

# Install Jekyll using bundle
Add Gemfile to root director(jhkim105.github.io)
```
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins
```
Install bundle
```
bundle install
```

# Run your jekyll site locally
Context path를 /로 하려면 _config.yml 수정
```
baseurl: ""
```

Run Server
```
bundle exec jekyll serve
```

character 관련 에러  
ruby설치할때 utf-8 관련 체크박스 있던데, 기본으로 uncheck된 상태로 설치했더니, 그런가 보다
  
![]({{site.url}}/assets/images/2019-05/jekyll-local-windows-02.png)
  
charseter set 변경 후 실행하면 됨
```
chcp 65001
bundle exec jekyll serve
```

localhost:4000로 접속

# References
[jekyll](ttp://jekyllrb.com)
