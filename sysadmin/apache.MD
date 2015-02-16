MPM-Prefork.conf
----
http://cloudinservice.com/tune-apache-performance-using-mpm-prefork-module/

02 Mar Posted by Olga in Performance, Unix administration | 32 comments
There could be many reasons why your website performance is poor, one of them can possibly be that Apache is not coping with the load. Below you’ll find ready to consume configuration to make Apache performance better using the Apache MPM prefork module.

apache logo

To do this, just include the below lines into your httpd.conf apache configuration file:

<IfModule mpm_prefork_module>
StartServers 2
MinSpareServers 2
MaxSpareServers 5
MaxClients 200 #must be customized
ServerLimit 200 #must be customized
MaxRequestsPerChild 100
</IfModule>

KeepAlive Off

Some explanations are here:

StartServers – this is how many apache instances should start at the very beginning when apache is started. I set it to 2, and it works fine for me.
 

MinSpareServers – minimum number of spare servers that should be running waiting for potential requests. MinSpareServers=2 worked fine for me too.
 

MaxSpareServers – maximum number of spare servers that should be running waiting for potential requests, obviously >= MinSpareServers. In my working example MaxSpareServers=5.
 

MaxClients & ServerLimit. You can use this shell script to determine an average amount of memory consumed by one Apache process. In addition to that it’ll show total amount of memory consumed by all Apache processes. Just unzip and execute as follows:
wget http://cloudinservice.com/wp-content/uploads/2011/03/ap.sh.zip

unzip ap.sh.zip

sh ap.sh

The output will be something like that:

Apache Memory Usage (MB): 1372.6
Average Proccess Size (MB): 54.9041

Try to execute it several times to compare the numbers; good results will be shown when server is under a heavy load. Now when you know average amount of memory consumed by Apache and total amount of memory of your server, it is possible to calculate value to be used for MaxClients setting. For example, if in average one your Apache process consumes 50MB RAM and server RAM is 2GB, and you want to leave 512MB for the rest processes, then:
MaxClients = (2GB – 512MB)/50MB = 30.72 ~ 30.
ServerLimit is, as I understand, the same thing, but while MaxClient setting can be changed on the go without a need to restart Apache, for new ServerLimit value to take effect Apache restart is required. MaxClients should always be <= ServerLimit. To make it easy, I set ServerLimit = MaxClients calculated by above formula.

 

By default MaxRequestsPerChild = 0, which means that httpd process will never expire. However, it can happen that there are some memory leaks in your PHP scripts (especially if you are using lot’s of third-party contributed modules). To protect yourself from accidental memory leaks, you can set how many requests should be processed by Apache process before it dies. In my example I used MaxRequestsPerChild = 100 and it worked fine. You can experiment to find a value that suits you better.
 

KeepAlive directive description is pretty straight forward on Apache site. If KeepAlive is On, it allows multiple requests to be sent over the same TCP connection, which can boost site performance up to 50% for some sites. This mostly works for pages that have lot’s of images. However, I’ve noticed that it can lead to the situation when processes never die and that can load the server a lot. Instead, I’m using a proxy server to serve images and other static content (I’ll make another post to clarify it) and in my configs KeepAlive is set to Off.
