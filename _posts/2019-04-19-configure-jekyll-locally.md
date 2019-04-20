---
layout: post
title:  "Configure Jekyll locally"
date:   2019-04-19 14:19:01 +0900
categories: jekyll
tag: jekyll
---

* content
{:toc}


Setting up your GitHub Pages site locally with Jekyll


------------------------

# Install rbenv
```
brew install rbenv ruby-build
```
# Add rbenv to bash so that it loads every time you open a terminal
```
echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bash_profile
source ~/.bash_profile
```
# Install Ruby
```
rbenv install 2.6.1
rbenv global 2.6.1
rbenv rehash
ruby -v
```
rbenv-gem-rehash를 설치하면 rehash를 안해줘도 된다.

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

localhost:4000로 접속

# References
[github-help](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll)

[jekyll](ttp://jekyllrb.com)
