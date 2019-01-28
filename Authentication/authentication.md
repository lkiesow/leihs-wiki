Authentication in _leihs_
=========================

The authentication within _leihs_ has been rewritten in version 5. 
The new design provides two internal authentication methods:

1. password authentication (rewritten and available as of version 5.0) ,
2. e-mail authentication (planned for version 5.1), and 

any number of _external authentication systems_ which can be bound to leihs via
an API. See also [External Authentication](./external_authentication).

Before version 5.0 binding custom authentication systems to _leihs_ required
custom code to be integrated within leihs. Such an approach is neither
necessary nor intended from _leihs_ 5.0 on.


