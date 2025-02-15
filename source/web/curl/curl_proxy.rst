.. _curl_proxy:

=================
curl代理
=================

命令行
=======

http/https代理
-------------------

- 命令行 ``curl`` 可以直接使用代理::

   curl https://reqbin.com/echo -x myproxy.com:8080 -U login:password

举例，我在 :ref:`nodejs_dev_env` 需要安装 ``nvm`` ，但是 raw.githubusercontent.com 被墙，所以需要通过以下命令通过代理访问::

   curl -x 192.168.10.9:3128 -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

socks5代理
------------

- 命令行 ``curl`` 可以使用 ``socks5h`` 方式::

   curl -x socks5h://localhost:1080 -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

持久化配置
==========

http/https代理
---------------------

- 环境变量设置::

   export http_proxy=http://192.168.10.9:3128
   export https_proxy=$http_proxy

- 或者采用 ``~/.curlrc`` ::

   proxy=192.168.10.9:3128

socks5代理
------------

比较特别，对于socks代理，变量是使用全部大写字母::

   export ALL_PROXY=socks5h://localhost:1080

参考
=======

- `How do I use Curl with a proxy? <https://reqbin.com/req/c-ddxflki5/curl-proxy-server>`_
- `performing HTTP requests with cURL (using PROXY) <https://stackoverflow.com/questions/9445489/performing-http-requests-with-curl-using-proxy>`_
- `How to use curl command with proxy username/password on Linux/ Unix <https://www.cyberciti.biz/faq/linux-unix-curl-command-with-proxy-username-password-http-options/>`_
- `How to Use Socks5 Proxy in Curl <https://blog.emacsos.com/use-socks5-proxy-in-curl.html>`_
