---
date: 2013-09-12T12:28:00Z
tags:
- ruby
- vpim
- rails
- outlook
- encoding
title: Creating vcards in rails using vpim - which works in outlook with correct encoding
tumblr_url: http://blog.bigfun.eu/post/61013239846/creating-vcards-in-rails-using-vpim-which-works
url: /2013/09/12/creating-vcards-in-rails-using-vpim-which-works/
---

Short info - how to create vcards in rails, which work correctly in Outlook 20xx.
I’m using awsome vpim gem.
In controller, where you will generate vcard, add this require:

{{< highlight ruby >}}
require ''vpim/vcard''
{{< / highlight >}}

The best working solution is to encode your vcard in utf-8, which is not currently available in vpim, so lets add following piece of code (after the require):

{{< highlight ruby >}}
module Vpim
  class DirectoryInfo
    class Field
      class << self
        alias_method :orig_create, :create

        # we overwrite Field.create for setting the CHARSET
        def create(name, value='', params={})
          # specify the lines you don't want to add a charset to
          lines_to_ignore = %w(BEGIN END VERSION)
          params.merge!({'CHARSET' => 'utf-8'}) unless lines_to_ignore.include? name
          orig_create(name, value, params)
        end
      end

    end
  end
end
{{< / highlight >}}

It''s important to set charset to utf-8, not UTF-8 - Outlook won’t handle it!
Now you have to generate vcard itself - i’m doing it with following piece of code:

{{< highlight ruby >}}
  #controller method
  def vcard
    person = Person.visible.find(params[:id])
    card = Vpim::Vcard::Maker.make2 do |maker|

      #setting up name
      maker.add_name do |name|
        name.prefix = ''
        name.given = person.first_name
        name.family = person.last_name
      end

      # setting up address.
      maker.add_addr do |addr|
        addr.location = 'work'
        addr.street = person.location.street
        addr.postalcode = person.location.zip_code
        addr.region = person.location.city
        addr.country = 'Poland'
      end

      maker.title = person.law_level
      maker.org = 'your company'
      maker.add_field(Vpim::DirectoryInfo::Field.create('office', person.location.name ))
      maker.add_tel(person.phone) { |e| e.location = 'cell'}
      maker.add_tel(person.location.phone) { |e| e.location = 'work'}


      maker.add_email(person.email) { |e| e.location = 'work' }

    end
{{< / highlight >}}

At the end, we have to send the file (still vcard method) :

{{< highlight ruby >}}
 send_data card.to_s, :type => ''text/x-vcard'', :filename => URI::encode(person.name) + ''.vcf''
  end
{{< / highlight >}}


Things to remember:
* Make correct charset
* set content-type (:type option) - you can sent charset also in content-type.
* URI:encode helps when filename includes non-ASCII characters
