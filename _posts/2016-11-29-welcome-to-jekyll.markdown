---
layout: post
title:  "Welcome to Jekyll!"
date:   2016-11-29 16:11:47 +0800
categories: tools
tags: jekyll
---

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

以下示例在`Fedora 28`上进行

## 安装  

### 首先你要有`Ruby`环境

```shell
sudo dnf install ruby-devel
ruby --version
#=> ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux]
gem --version
#=> 2.7.6
gcc --version
#=> gcc (GCC) 8.1.1 20180712 (Red Hat 8.1.1-5)
g++ --version
#=> g++ (GCC) 8.1.1 20180712 (Red Hat 8.1.1-5)
make --version
#=> GNU Make 4.2.1
```

### 安装`jekyll`与`bundler`

```shell
gem install jekyll bundler
```

其中`bundler`安装成功，`jekyll`安装失败，失败原因：

> ERROR: Failed to build gem native extension.
> gcc: error: /usr/lib/rpm/redhat/redhat-hardened-cc1: No such file or directory

解决：

```shell
sudo dnf install redhat-rpm-config
```

重新安装`gem install jekyll`成功

### 本地预览`github page`

```shell
git clone github-page-URL
cd ./github-page

# 安装所需控件
bundle install --path ~/.gem/bundle 

# 运行
bundle exec jekyll serve
# 运行(include draft)
bundle exec jekyll serve --draft

# 浏览器打开 http://127.0.0.1:4000 查看结果
```

`bundle install`过程中安装`nokogiri`失败，原因  

> fatal error: zlib.h: No such file or directory

解决： 

```shell
sudo dnf install zlib-devel
# ubuntu 上执行sudo apt-get install libz-dev
```

`bundle install`之后运行`bundle exec jekyll serve`时报错：

> Ignoring commonmarker-0.17.13 because its extensions are not built. Try: gem pristine commonmarker --version 0.17.13  
> Could not find commonmarker-0.17.13 in any of the sources

根据提示执行`gem pristine commonmarker --version 0.17.13`解决问题

### 语法

#### 屏蔽Tag转义

{%raw%} raw - endraw {%endraw%}

### 参考文献

1. Github Pages  
  [Using Jekyll as a static site generator with GitHub Pages][github-pages-jekyll]
2. Date Format  
  [Jekyll Date Formatting Examples][Jekyll Date Formatting]

[github-pages-jekyll]: https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/
[Jekyll Date Formatting]: http://alanwsmith.com/jekyll-liquid-date-formatting-examples
