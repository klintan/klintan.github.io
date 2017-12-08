---
layout: post
title:  "Using and installing NodeJS Couchbase for AWS Lambda"
date:   2017-12-07 17:49:50 -0800
categories: lambda aws couchbase
---

### Create folder:
https://aws.amazon.com/blogs/compute/nodejs-packages-in-lambda/
mkdir lambdaTestFunction
cd lambdaTestFunction

Install libuv-devel:



### Manually:
from https://github.com/libuv/libuv/blob/master/README.md

wget https://dist.libuv.org/dist/v1.18.0/libuv-v1.18.0.tar.gz 
tar -xvzf libuv-v1.18.0.tar.gz
sudo yum install libtool -y
cd libuv-v1.18.0
sh autogen.sh
./configure
make
make check
make install

Do NOT feel tempted to google how to "yum install" libuv, because manually installing it is a bit intimidating. How do I know ? Because I did the same, and the answer is: "yum install libuv-devel libuv --enablerepo=epel". At the time of writing the version installed is 0.10.34 which unfortunately is to old to use when building libcouchbase.


### Install nodejs: 

https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-a-centos-7-server

cd ~
wget http://nodejs.org/dist/v0.10.30/node-v0.10.30-linux-x64.tar.gz

sudo tar --strip-components 1 -xzvf node-v* -C /usr/local

Install an example node library (aws-sdk in this case) and install it in the lambdaTestFunction (--prefix <folder> <npm-package>)
npm install --prefix=~/lambdaTestFunction aws-sdk


### Upgrade node and tools: 

Step 1: Update your Amazon Linux machine and install required libraries and tools

sudo yum update
sudo yum install gcc44 gcc-c++ libgcc44 cmake –y
wget http://nodejs.org/dist/v0.10.33/node-v0.10.33.tar.gz
tar -zxvf node-v0.10.33.tar.gz
cd node-v0.10.33 && ./configure && make
sudo make install

### Install libev, libevent and libev-devel and libevent-devel:
sudo yum install libevent libev -y
sudo yum install libev-devel -y

### Install openssl:
yum install -y openssl-devel

Build libcouchbase package:
$ cmake -D CMAKE_BUILD_TYPE=RELEASE -D BUILD_SHARED_LIBS=NO -D CMAKE_INSTALL_PREFIX=~/libcouchbase-dist ~/libcouchbase/


### Install couchnode packacked with the prebuild libcouchbase

PKG_CONFIG_PATH=~/libcouchbase/lib/pkgconfig/  npm install –-prefix=~/lambdaTestFunction couchbase

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
