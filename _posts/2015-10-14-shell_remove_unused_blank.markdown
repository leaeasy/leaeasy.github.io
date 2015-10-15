---
layout: post
title:  "Shell去掉空白行、多余空格"
date:   2015-10-14 10:41:01
categories: shell
tags: shell
image: /assets/article_images/2015-10-14-shell/bash.jpg
---
# 去掉空白行

{% highlight bash %}
tr -s ["\012"] < test.txt > newtest.txt

sed -n '/./p' temp_file5 > newtest.txt

grep . temp_file5 > newtest.txt

grep -v '^$' temp_file5 > newtest.txt
{% endhighlight %}

# 去掉多余空格

{% highlight bash %}
cat test.txt | tr -s [:space:]

sed 's/__*/_/g' test.txt     //这里，_表示一个空格

{% endhighlight %}

