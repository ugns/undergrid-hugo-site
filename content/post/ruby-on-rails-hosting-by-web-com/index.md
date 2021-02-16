---
title: Ruby on Rails hosting by Web.com
type: post
author: jtbouse
date: "2010-02-07T20:52:42Z"
tags:
- development
- rails
- ruby
- software
- web.com
categories:
- Projects
---
Usually in the past I've done my web hosting on either one of the many servers I own or utilizing VPS hosting providers like [VPSfarm.com](http://vpsfarm.com), [GrokThis.net](http://grokthis.net) or [Linode.com](http://linode.com), but lately with the economy and a price that can't be beat I've been using [Web.com](http://www.web.com) Linux Hosting plan to meet my needs. This has met all my requirements except one, I couldn't run my Ruby on Rails applications that I was working on development for using their services. Well until now that is... Thanks in part to a great Systems Engineer that I've had the pleasure of working with and knowing great strides had been made to improve the feature set to the level that a power user like myself would appreciate adding even more value to the offering.

Recently there had been work being done to add FastCGI access to the Linux Hosting plan which already offers PHP5, Python 2.4 and Perl 5. Ruby is still not available on the system as a whole; however, that doesn't stop you from adding it to your own account which is precisely what I did. Armed with the ability to test out FastCGI I proceeded to work on getting a very simple test RoR app setup and running.

To start with I needed to get Ruby installed. To do this I simply logged into my shell account (another great feature of Web.com's Linux Hosting) and proceeded to download the necessary sources:

{{< highlight bash >}}
  $ wget ftp://ftp.ruby-lang.org/pub/ruby/1.8/ruby-1.8.7-p174.tar.gz
  $ wget http://rubyforge.org/frs/download.php/60718/rubygems-1.3.5.tgz
{{< /highlight >}}

With the source downloaded I then extracted them into the tmp/ directory in my account and built them.

{{< highlight bash >}}
  $ tar -xzf ruby-1.8.7-p174.tar.gz -C tmp/
  $ tar -xzf rubygems-1.3.5.tgz -C tmp/
  $ cd tmp/ruby-1.8.7-p174/
  $ ./configure --prefix=$HOME
  $ make
  $ make install
  $ cd ~/tmp/rubygems-1.3.5/
  $ ruby setup.rb
{{< /highlight >}}

With this completed you should have Ruby 1.8.7 and RubyGem 1.3.5 ready to go. You can test by running the following couple of commands:

{{< highlight bash >}}
  $ ruby -v
    ruby 1.8.7 (2009-06-12 patchlevel 174) [i686-linux]
  $ gem -v
    1.3.5
{{< /highlight >}}

Now we're ready to rock and roll&#8230; We just need a few gems to be thrown in and we'll have Ruby on Rails ready to go and be able to use MySQL for the database. So we'll start off with the basics&#8230;

{{< highlight bash >}}
  $ gem install rails mysql fcgi
{{< /highlight >}}

This should install any other gems that are necessary for dependencies and you can check the final list of gems installed running **_gem list_** later if you're curious to see them all. Now would also be a good time to go ahead and clean-up your tmp/ directory as you won't be needing all that source any more so why use up the disk resources.

Now we just need a Rails application to run, so we'll start out with building one from scratch although if you have one ready to go then you could just upload it to your home directory. For purposes of this I'm just going to setup a Rails webapp aptly called "**test**" as follows:

{{< highlight bash >}}
  $ rails -D -d mysql ~/test
{{< /highlight >}}

Now we just need to go create our webapps public/.htaccess file with the necessary configuration settings. For my example I used the following for my .htaccess file:

{{< highlight htaccess >}}
  Options +FollowSymLinks +ExecCGI
  <IfModule mod_fastcgi.c>
    AddHandler fastcgi-script .fcgi
  </IfModule>
  <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule ^$ index.html [QSA]
    RewriteRule ^([^.]+)$ $1.html [QSA]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ dispatch.fcgi [QSA,L]
  </IfModule>
{{< /highlight >}}

Armed with this we just need to setup the virtual host. I set this up as test.example.com so it ran isolated from my regular non-Rails website. To do this I logged into my support account (outside scope of this entry) and setup **test.example.com** as a website content path as **/test/public**. Then while still logged into the support site created the necessary database tables **test_development** and **test_production**. You could also setup separate database uses at this time for the databases as well if you liked.

Once the databases were created I needed to edit the **test/config/database.yml** file with the necessary database name and credentials. Also set the database host to the appropriate hostname provided for your account. You'll then need to setup and migrate the database.

{{< highlight bash >}}
  cd ~/test
  rake db:migrate
  rake db:setup
{{< /highlight >}}

Everything done you're now ready to test it out and see if it all works. If you open up your browser and go to **test.example.com** and you should see the default Ruby on Rails welcome page. If you click on the "About your application's environment" link it should drop-down and show you the version information for your Rails application. From here you're ready to begin the rest of your webapp development.
