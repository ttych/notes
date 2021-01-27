---
title: Manage CA certificate for gem command
date: 2021-01-27 12:41:00 +0100
tags: ruby
---

## The initial error

Here are the kind of error, you can have with gem command, when gem
source is not trusted.

```
ERROR:  While executing gem ... (OpenSSL::SSL::SSLError)
    SSL_connect returned=6 errno=0 state=SSLv3 read finished A
```

```
ERROR:  You must add /.../... to your local trusted store
ERROR:  SSL verification error at depth 0: unable to get local issuer certificate (20)
```

## Context

This error raises when using `gem` command with non-official gem
repositories.

For example in a company context, having internal mirrors and/or
internal gem repositories, the gem command will generate such errors,
since the configured gem sources are in https, and the couple (fqdn,
certificate) is not _supported_ by gem.

## Use env variable

One solution is to force the CA certificate with environment variable
at each `gem` command execution.

{% highlight sh %}
SSL_CERT_FILE= gem install my
{% endhighlight %}

## Add CA certificate in Ruby distribution

Another solution is to inject the CA certificate in the Ruby
distribution, so that at each execution the certificates will be
loaded and used.

`gem` comes with pre-configured certificates

{% highlight sh %}
% gem which rubygems
.../ruby-2.7.2/lib/ruby/2.7.0/rubygems.rb
% ls .../ruby-2.7.2/lib/ruby/2.7.0/rubygems/ssl_certs
index.rubygems.org  rubygems.global.ssl.fastly.net  rubygems.org
{% endhighlight %}

Let suppose you are in the company **company.com** and you have an
internal repository with the dns suffix **company.com**, you have to
create a directory with this suffix in **ssl_certs** directory and put
you CA certificate in this repository.

{% highlight sh %}
% mkdir .../ruby-2.7.2/lib/ruby/2.7.0/rubygems/ssl_certs/company.com
% cp CA_cert.pem .../ruby-2.7.2/lib/ruby/2.7.0/rubygems/ssl_certs/company.com/
{% endhighlight %}

Then the gem command should work without any issue.
