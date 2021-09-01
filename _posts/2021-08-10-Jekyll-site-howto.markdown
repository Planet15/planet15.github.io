---
layout: post
title:  "github에서 Jekyll-Paper-Github를 이용하여 블로그 운영하기"
date:   2021-08-10
last_modified_at: 2021-08-10
categories: [jekyll]
tags: [jekyll]
---

1.github 로그인을 합니다.

2.원하는 아이디를 생성을 하며, 보통 test01 이라는 ID를 만든 경우 test01.github.io를 만들수 있게 됩니다.

3.새로운 Repositories  생성에 자신이 만든 ID 와 github.io를 추가 합니다.
{% highlight sh %}
test01.github.io
{% endhighlight %}

4.생성 후, 작업하고자 하는 위치로 Repository를 clone 합니다.
{% highlight sh %}
# git clone https://github.com/test01/test01.github.io.git
{% endhighlight %}

5.사용하고자 하는 테마를 선택 해야 하며, 저는 한글을 지원하는 [Jekyll-Paper-Github] 선택 하였습니다.
{% highlight sh %}
# git clone https://github.com/ghosind/Jekyll-Paper-Github
{% endhighlight %}

6.Jekyll-Paper-Github의 디렉토리 내용을 전체를 복사 하여 test01.github.io 디렉토리로 옮깁니다.
{% highlight sh %}
# cp -rf Jekyll-Paper-Github/* test01.github.io.git/.
{% endhighlight %}

7.복사된 후 바꿔야 될 내용(_config.yml)을 수정 합니다.
{% highlight yaml %}
title: 페이지의 제일 상위에 표시되는 내용입니다.
email: 저자의 이메일을 기입합니다.
keywords: 관심사 부분이며 대괄호 안에 넣으며 다수인 경우 쉼표로 나눕니다.
baseutl: 만약 다른 디렉토리에서 정보를 읽는 경우라면 디렉토리 표기를 해야 하지만 본 예제에서는 비워두겠습니다.
url: https://test01.github.io 로 명시 하겠습니다.
domain: 현재 저는 github.io로 표기 하였습니다.
language: 언어 지원 부분이며 ko로 설정 합니다.
{% endhighlight %}

{% highlight yaml %}
title: Your title
email: your-email@example.com
name: Your name
description: >- # this means to ignore newlines until "baseurl:"
  Write an awesome description for your new site here.
keywords: [Your keywords]
baseurl: '' # the subpath of your site, e.g. /`blog
url: '' # the base hostname & protocol for your site, e.g. http://example.com
domain: 'example.com' # the domain name for your site, e.g. example.com
language: en
# Supported languages list:
# en: English
# de: Deutsche (German)
# es: Español (Spanish)
# fr: Français (French)
# ja: 日本語 (Japanese)
# pt: Português (Portuguese)
# zh-cn: 简体中文 (Simplified Chinese)
# zh-tw: 繁體中文 (Traditional Chinese)
# ko : 한국어 (Korean)
{% endhighlight %}

8.메뉴 부분에 대한 수정은 _data/menus.yml 파일에서 수정이 가능합니다.
{% highlight yaml %}
- default: menu_index
  url: ''

- default: menu_category
  url: 'categories'

- default: menu_about
  url: 'about'

{% endhighlight %}
9.수정후,코드를 add/commit/push를 합니다.
{% highlight sh %}
# git add -A .
# git commit -m "Init tech blogs"
# git push
{% endhighlight %}

10.이후, https://test01.github.io 통해 확인 합니다.

[Jekyll-Paper-Github]: https://github.com/ghosind/Jekyll-Paper-Github