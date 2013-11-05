A typical production deployment process for leihs could look like the following. This is modeled after the process that is actually in use at the Zürich University of the Arts (ZHdK).

Server landscape:

* rails.zhdk.ch with the following:
  * Apache 2
  * Phusion Passenger
  * RVM
  * Ruby 1.9.3-p448 installed through RVM

* db.zhdk.ch with the following:
  * MariaDB

## Steps to take to deploy a new version of leihs at the ZHdK

With Capistrano installed on your workstation, enter the leihs source code directory and do:

    # cap production deploy

If you have permission as user `leihs@rails.zhdk.ch`, Capistrano will magically deploy a version of leihs to rails.zhdk.ch. The following explains exactly what happens here and in which steps.


