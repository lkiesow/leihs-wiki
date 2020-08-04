Running through a Reverse Proxy / HTTP Gateway
==============================================

The default deployment of leihs will setup a reverse proxy within the
deployment target/host.  It is possible to run this host in a private network
and route traffic through an additional reverse proxy.

The following depicts a simple example of such a setup.

![ZHdK Reverse Proxy Setup](./Reverse-Proxy/ZHdK-Reverse-Proxy.svg)


Setting up a reverse proxy for leihs is mostly straight forward. Note that the
`external_base_url` setting must reflect the proper name. For the example of
the ZHdK the proper value is `https://leihs.zhdk.ch`.


Switch-AAI Behind a HTTP Reverse Proxy
======================================

We do recommend to run a potential Switch-AAI/Shibboleth authentication
adapter, see also [External Authentication Systems in _leihs_](external_authentication),
behind a reverse proxy and not on the reverse proxy itself and neither on the
machine where leihs is deployed to.  This is not a hard requirement but we
found such a setup more flexible and less troublesome.

Some additional configuration on the primary reverse proxy is needed in order
not to derail the switch-aai apache module (it seems that it does ignores some
of the additional headers added by the reverse proxys).  The configuration
values for the Apache HTTP Server read like the following:

        SSLProxyVerify none
        SSLProxyCheckPeerCN off
        SSLProxyCheckPeerName off
        SSLProxyCheckPeerExpire off

We **recommend to run the authentication adapter** (see also [external authentication](external_authentication))
on the **same host where the shibboleth module is running**. If you diverge
from this setup additional measures must be taken to secure authentication.

