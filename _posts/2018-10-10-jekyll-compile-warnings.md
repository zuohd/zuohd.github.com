---
layout: post
title:  "解决Jekyll编译报错"
date:   2018-10-10 10:51:48 +0800
categories: Jekyll
tags: [blog tricks]
author: Soderberg
---
在执行jekyll build 命令时遇到了如下警告：
``` warnnings
C:/Ruby25-x64/lib/ruby/2.5.0/fileutils.rb:90: warning: already initialized constant FileUtils::VERSION
C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/fileutils-1.1.0/lib/fileutils.rb:92: warning: previous definition of VERSION was here
C:/Ruby25-x64/lib/ruby/2.5.0/fileutils.rb:1188: warning: already initialized constant FileUtils::Entry_::S_IF_DOOR
C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/fileutils-1.1.0/lib/fileutils.rb:1267: warning: previous definition of S_IF_DOOR was here
C:/Ruby25-x64/lib/ruby/2.5.0/fileutils.rb:1444: warning: already initialized constant FileUtils::Entry_::DIRECTORY_TERM
C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/fileutils-1.1.0/lib/fileutils.rb:1539: warning: previous definition of DIRECTORY_TERM was here
C:/Ruby25-x64/lib/ruby/2.5.0/fileutils.rb:1448: warning: already initialized constant FileUtils::Entry_::SYSCASE
C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/fileutils-1.1.0/lib/fileutils.rb:1543: warning: previous definition of SYSCASE was here
C:/Ruby25-x64/lib/ruby/2.5.0/fileutils.rb:1501: warning: already initialized constant FileUtils::OPT_TABLE
C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/fileutils-1.1.0/lib/fileutils.rb:1596: warning: previous definition of OPT_TABLE was here
C:/Ruby25-x64/lib/ruby/2.5.0/fileutils.rb:1555: warning: already initialized constant FileUtils::LOW_METHODS
C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/fileutils-1.1.0/lib/fileutils.rb:1650: warning: previous definition of LOW_METHODS was here
C:/Ruby25-x64/lib/ruby/2.5.0/fileutils.rb:1562: warning: already initialized constant FileUtils::METHODS
C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/fileutils-1.1.0/lib/fileutils.rb:1657: warning: previous definition of METHODS was here
```
应该是gem升级安装问题，遂google之，查询到以下解决方法：
``` commandlines
gem uninstall fileutils
gem update --default

```

> 参考来源
:https://stackoverflow.com/questions/51334732/rails-5-2-0-with-ruby-2-5-1-console-warning-already-initialized-constant