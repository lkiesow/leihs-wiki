You can hook up leihs to an LDAP server. So far, we've only tried ActiveDirectory, but at four different universities. We're reasonably sure the adapter should work for any LDAP server. If it doesn't work for yours, please submit a pull request with the changes you need.

#### Modifying config/LDAP.yml

Copy config/LDAP.yml.example to config/LDAP.yml and adapt the configuration to your own LDAP server:

        production:
          host: ldapserver.example.org
          port: 636
          encryption: simple_tls
          #log_file: log/ldap_server.log
          #log_level: warn
          master_bind_dn: CN=LeihsEnumUser,OU=NonHuman,OU=users,DC=example,DC=org
          master_bind_pw: 12345
          base_dn: OU=users,DC=example,DC=org
          admin_dn: CN=grpAllLeihsAdmins,OU=Users,OU=Groups,DC=example,DC=org
          unique_id_field: userPrincipalName
          search_field: sAMAccountName

`unique_id_field`, `master_bind_dn` and `master_bind_pw` are all required.

WARNING: be sure to include a space after the `:` characters:
* wrong: `master_bind_pw:12345`
* correct: `master_bind_pw: 12345`

#### Distinguished Names
'Distinguished Name' is LDAP speak for "complete, unambiguous path to an object inside LDAP". You will need to specify DN strings in LDAP.yml. It can be hard /error prone to build a complete DN string by hand. To make your life easier copy the whole distinguishedName of an object out of the 'Active Directory Users and Computers' console:
* Open the 'Active Directory Users and Computers' console
* In the view menu enable "Advanced Features"
* Open the properties of an OU, User or Group and look for the "Attribute-Editor" tab
* Copy the contents of the attribute `distinguishedName` to the value of `master_bind_dn` in the config file (click on "Edit" first).
* Paste the whole string to your config file where needed.

If you are unable to access the Users and Computers console, you may build the DN strings by hand. If using ActiveDirectory and a Windows client joined to your domain, you can find out the Distiguished Name of the currently logged on user via this command on a Windows session:
`whoami /FQDN`

Examples
* A user: `CN=LeihsEnumUser,OU=NonHuman,OU=users,DC=example,DC=org`
* An Object Unit (folder): `OU=users,DC=example,DC=org`
* A user group object: `CN=grpAllLeihsAdmins,OU=Users,OU=Groups,DC=example,DC=org`

#### host
The hostname of your LDAP server.
Active Directory: enter your AD domain name here, not just a hostname. The domain name resolves to all available Domain Controllers - good for redundancy. In the example above the value should read
'example.org'
A regular FQDN of any single Domaincontroller will work too, but without redundancy of course.

#### port, encryption
Talk to LDAP server encrypted over SSL (LDAPS) or unencrypted in plaintext LDAP.
Using encryption for LDAP traffic between leihs and LDAP server is highly recommended, as passwords will be sent in plaintext over the network using `encryption: none`. Be aware that LDAP over SSL requires a certificate on your LDAP server. It may be self-signed, though as no validation of the cert is performed.

If you want to use encryption for LDAP traffic:
```
port: 636
encryption: simple_tls
```

If you do not want to use encryption:
```
port: 389
encryption: none
```
Open your firewall to allow either 636 TCP or 389 (UDP, TCP) on your LDAP server.

Warning: Net::LDAP library used by leihs does not validate certificate authenticity using `simple_tls`. **It will accept any certificate the LDAP server presents without warning.** Other, stricter methods of SSL are currently not implemented in leihs. This is not a huge problem, as long as you trust your DNS server.
[http://www.rubydoc.info/github/ruby-ldap/ruby-net-ldap/Net/LDAP:encryption](http://www.rubydoc.info/github/ruby-ldap/ruby-net-ldap/Net/LDAP:encryption)


#### master_bind
Distinguished Name of an LDAP user object. `master_bind_dn` and `master_bind_pw` are the credentials for an LDAP user that has permission to query enough of the LDAP tree so that it can find all the users you want to grant entry to leihs to. Needs read-only access to your LDAP.
As usual, it is best practice to create a new user in your Active Directory, dedicated only to this purpose. Name it `LeihsEnumUser` for example.
Mind your password policy: if the password changes in AD automatically, you will not be able to log in to Leihs anymore. If this happens, just update the password of your master_bind user in your config file.

To prevent this, make sure to check the "Password never expires" box for this user in 'Users and Computers' console.

#### base_dn
Distinguished Name of an Active Directory Object Unit (OU): the Ldap-tree that is searched for usernames. Active Directory: Copy the `distinguishedName` of the top Organization Unit your users are stored in. The search will look in all subdirectories of this OU for a user with an attribute you specify in `search_field`.

#### admin_dn
Distinguished Name of the LDAP group you intend to give admin rights to in Leihs. This MUST be a group, a single user is not a tested/supported scenario.

#### unique_id_field
`unique_id_field` is any attribute-field of an LDAP user, that contains a completely unique ID for the user in question. This can be the same as `search_field` if you are sure that it's unique.
DBR: I have read that `sAMAccountName` normally is unique on the Active Directory domain level (not forest level). For practical purposes it can be considered 'unique' I guess. Source:
[http://blogs.msdn.com/b/openspecification/archive/2009/07/10/understanding-unique-attributes-in-active-directory.aspx](http://blogs.msdn.com/b/openspecification/archive/2009/07/10/understanding-unique-attributes-in-active-directory.aspx)

Alternative: `userPrincipalName` is guaranteed to be unique, even on the forest level. If you are unsure, I would suggest you use `userPrincipalName` instead of `sAMAccountName` for the unique ID.

Warning: do not use the AD `objectGUID`attribute for this. Though tempting, the value cannot be written to database and will cause an error on first login. You will need a string-based attribute with a maximum of 255 characters for `unique_id_field` (field in database is varchar(255) ).

#### search_field
The `search_field` dictates what users will have to write in the "Login" field on login.
You may have to adapt the `search_field` option to point to the LDAP attribute that contains your usernames. In Active Directory this should normally be the `sAMAccountName` attribute, which is the "User-Logon-Name (pre Windows 2000)" in the GUI.

#### Example Admin
For the sake of simplicity we will use a user called `LeihsAdmin`, which is a member of the group `grpAllLeihsAdmins` we configured above as `admin_dn`. Therefore, this user will automatically have complete administrative control in the Leihs dashboard. When I speak of 'LeihsAdmin', your LDAP user is meant.

#### Requirements for LDAP users in Leihs
The LDAP users you intend to use as Leihs users need to have at least the following attributes set (not empty). **This applies to the admin users as well**.
* `email`
* `givenName` (the surname, e.g. Alexander)
* `sn` (the family name, e. g. Smith)

If a needed user attribute is empty, you will get an ugly looking error message on first login.

Make *sure* you do not use any email address for the LeihsAdmin user that is already taken by any user account in the local Leihs database. Even if local authentication is disabled, Leihs will still keep the local users and will not create your LeihsAdmin in the database if the address is already taken (Error: >Could not create new user for 'leihsadmin' from LDAP source. Contact your leihs system administrator>). Also make sure you do not have a local database user with the same login as one of your LDAP users.


#### Enabling LDAP authentication in the system
Finally, you need to tell Leihs that you want to use LDAP and not the user credentials stored in your local database. We will keep local database authentication enabled at first. This is to show you that local and LDAP authentication may be enabled at the same time. This might be useful as a fall-back method in case there is a problem with the initial LDAP configuration.

Start a Rails console inside your Leihs Legacy directory:

    $ su leihs-legacy
    $ cd /leihs/legacy
    $ export PATH=$HOME/.rubies/ruby-<leihs_legacy_ruby_version>/bin:$PATH
    $ RAILS_ENV=production bundle exec rails console

You can find the value for `leihs_legacy_ruby_version` in `/leihs/deploy/all.yml`.

Then enable LDAP authentication as default but leave database authentication enabled for now:

    ldap = AuthenticationSystem.find_or_initialize_by(class_name: 'LdapAuthentication')
    ldap.name ||= 'LDAP Authentication'
    ldap.is_default = true
    ldap.is_active = true
    ldap.save

    db = AuthenticationSystem.find_or_initialize_by(class_name:'DatabaseAuthentication')
    db.name ||= 'Database Authentication'
    db.is_default = false
    db.is_active = true
    db.save
    exit

#### First login with LeihsAdmin over LDAP
Now restart your Rails application. Next time you try to access it, you should be forwarded to `/authenticator/ldap/login` instead of `/authenticator/db/login`, where you can log in via LDAP. If login was successful, the information stored in LDAP (email, surname, name, phone, etc.) gets copied to your Leihs database as a new user is created in table `users`. It can be (locally) edited later in the admin dashboard of Leihs. You may still login with the local database users by simply changing the URL in your browser to ...`/authenticator/db/login`

If anything goes wrong give the Leihs webserver user write permission to `./log/` (the log directory inside the Leihs directory). Check `production.log` after a failed login attempt and search for 'leihsAdmin'. The error messages were very straightforward to understand in my "testing" (or better: "flailing about, trying to get the damn thing to work")
DerBachmannRocker, Windows Server 2012 AD + Leihs 3.16

You might need to delete a half-created user from the database manually in case the first login goes wrong (e.g. because of some error). Simply delete the the offending row from the database table `users` and try again with a new config. I got "eMail already taken" in the log, for example, as the user was already created but not 100% correctly (DBR).

After you eliminated all errors (no errors visible in browser and the logfile) you should be able to login successfully for the first time. A successful first login results in the following being created:
* table `users`: a new row for LeihsAdmin containing information out of LDAP (email, etc.). Note the user's `ID`
* table `access_rights`: a new row with `user_id` equal to the ID of LeihsAdmin. `role` should be 'admin'.

You might get thrown back to the login page as your user is created in the database and not immediately see
the Leihs interface. In this case simply enter your credentials for LeihsAdmin one more time. This time you should be able to see the Leihs dashboard and have admin permissions (verify this in the `users` page).

Delete your browser cache if repeated login attempts fail unexpectedly.


#### User-Information is updated on logon
I tested changing user information in Active Directory afterwards (email, telephone, address). On next login of the user, his/her information was updated from Active Directory to the Leihs user table. Updating works only in the direction Active Directory -> Leihs.
Therefore, do not manually update user information in Leihs, or it will get overwritten.
Allow for some time to pass for the information to be pulled from AD again (AD replication, caching, etc.). In my testing, the information was only updated to the current values after I waited a bit. DBR, Leihs 3.34.0


#### Turning off database authentication for good
After you made your first login with LeihsAdmin and everything runs as it should, you might disable database authentication for your production server:

    $ su leihs-legacy
    $ cd /leihs/legacy
    $ export PATH=$HOME/.rubies/ruby-<leihs_legacy_ruby_version>/bin:$PATH
    $ RAILS_ENV=production bundle exec rails console

You can find the value for `leihs_legacy_ruby_version` in `/leihs/deploy/all.yml`.

Enable only LDAP authentication and switch off database authentication:

    ldap = AuthenticationSystem.find_or_initialize_by(class_name: 'LdapAuthentication')
    ldap.name ||= 'LDAP Authentication'
    ldap.is_default = true
    ldap.is_active = true
    ldap.save

    db = AuthenticationSystem.find_or_initialize_by(class_name:'DatabaseAuthentication')
    db.name ||= 'Database Authentication'
    db.is_default = false
    db.is_active = false
    db.save
    exit

Restart your webserver afterwards.

## Encryption (LDAPS) with Active Directory
I successfully tested binding to Active Directory with Leihs 3.34.0 and a Server 2012 R2 RODC Domain Controller over SSL (using 'simple_tls' and port '636'. A wireshark trace did not show any cleartext transmission of user credentials between the leihs web server and Domain Controller.
Disclaimer: The LDAP ruby code was modified by me, before testing. I *think* the vanilla version of the code should work just fine, as I did not touch those parts. I hope these modifications will be integrated into the main branch of leihs in the future (pull request pending).
DBR, June 2016

####Cert requirements
Your Active Directory Server needs a certificate for SSL to work.
Please read up on Knowledgebase article 321051. You cert will need to match certain criteria to be accepted for LDAP usage by your domain controller. 
[https://support.microsoft.com/en-us/kb/321051](https://support.microsoft.com/en-us/kb/321051)

All you need to do is to import a suitable certificate (using MMC console with snapin "Certificates") into the local machine personal store of your Domain Controller. Upon reboot, LDAP will look for certificates and start serving connections to port 636, if a matching cert is found.

Your Domain Controller will automatically use the first certificate it deems fitting out of its
`local machine\personal` store. You can however, if you want to enforce a specific certificate to be used, import the Cert into the Active Directory integrated Cert store for the machine `NTDS\personal`. This will override the local certificates.
See:
[http://social.technet.microsoft.com/wiki/contents/articles/2980.ldap-over-ssl-ldaps-certificate.aspx](http://social.technet.microsoft.com/wiki/contents/articles/2980.ldap-over-ssl-ldaps-certificate.aspx)

####Testing LDAPS without leihs
Use `ldp.exe` test tool (included in Windows Server) to connect to your Domain Controller. Choose `Connection` -> `connect...`
Enter your DC's hostname, port 636 and tick the 'SSL' box. Click 'connect'. On successful connection over SSL, I got the following results:

```
ld = ldap_sslinit("ldapserver.example.org", 636, 1);
Error 0 = ldap_set_option(hLdap, LDAP_OPT_PROTOCOL_VERSION, 3);
Error 0 = ldap_connect(hLdap, NULL);
Error 0 = ldap_get_option(hLdap,LDAP_OPT_SSL,(void*)&lv);
Host supports SSL, SSL cipher strength = 256 bits
Established connection to ldapserver.
Retrieving base DSA information...
```
### LDAP Library
Leihs uses the gem net-ldap for connectivity to LDAP (as of leihs 3.16.0).
[https://github.com/ruby-ldap/ruby-net-ldap](https://github.com/ruby-ldap/ruby-net-ldap)
It is a port of the perl library Net::LDAP. More detailed information about the parameters can be found in the source code/documentation. Normally the information given above should be enough to get Leihs running, so this is just for reference.
[http://search.cpan.org/~marschap/perl-ldap/lib/Net/LDAP.pod](http://search.cpan.org/~marschap/perl-ldap/lib/Net/LDAP.pod)


#### Contribute!
Feel free to improve our LDAP connector and send us a pull request on GitHub. Alternatively, you could also pay some Rails developers (even us!) to develop this feature for you.

#### A fair warning
Warning: Please make sure that your Rails application server has SSL enabled before you put this configuration into production. You don't want to send passwords unencrypted over the web using plain HTTP.
