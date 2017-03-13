---
date: 2013-06-23T13:50:00Z
tags:
- rails
- ruby
- rails3
- i18n
- quicknote
title: I18n::InvalidPluralizationData in rails 3.2 with polish locale
tumblr_url: http://blog.bigfun.eu/post/53668303696/i18ninvalidpluralizationdata-in-rails-32-with
url: /2013/06/23/i18ninvalidpluralizationdata-in-rails-32-with/
---
<!--more-->
In case u receive this error with such explanation:

> translation data can not be used with :count => 2

Remember that you need to define a “few” and “many” key for translation for particular entity, e.g.:

{{< highlight yaml >}}
pl:
  activerecord:
    models:
      article:
        one: “Artykuł”
        few: “Artykuły”
        many: “Artykułów”
        other: “Artykuły”
{{< / highlight >}}
