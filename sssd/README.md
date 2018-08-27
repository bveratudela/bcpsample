# Install SSSD for OS Authentication

SSSD is a native Linux agent that allows you to integrate your Linux hosts with AD/LDAP so that users/groups are visible to the OS. SSSD is not the recommended solution for enterprises but can be used to overcome this gap when necessary. Instead recommend that users look into more enterprise solutions like Centrify. Refer to XXX for sample SSSD configurations when using AD. In this workshop we will use LDAP instead. Start by installing and configuring the SSSD daemon on ALL hosts along with some other auxiliary packages. Note that we are making reference to the use of the rfc2307bis schema, ldap provider, uid and member filters, etc. These values will naturally be different when using AD. Also ensure you do this first on the LDAP server and copy the files to your other hosts to ensure the config file is the same across all hosts:

```
yum -y install sssd sssd-client sssd-tools samba-common oddjob-mkhomedir nscd

echo "[sssd]
config_file_version = 2
services = nss, pam, autofs
domains = default
debug_level = 0

[nss]
filter_users = root,ldap,named,avahi,haldaemon,dbus,radiusd,news,nscd

[pam]

[domain/default]
debug_level = 0
id_provider = ldap
ldap_schema = rfc2307bis
ldap_tls_reqcert = never
auth_provider = ldap
krb5_realm = $(hostname -d | tr [a-z] [A-Z])
ldap_search_base = ${DC}
ldap_user_name = uid
ldap_user_home_directory = homeDirectory
ldap_user_object_class = posixAccount
ldap_group_object_class = posixGroup
ldap_group_member = member
ldap_id_use_start_tls = True
chpass_provider = ldap
ldap_uri = ldap://$(hostname -f):389
ldap_chpass_uri = ldaps://$(hostname -f)
krb5_kdcip = $(hostname -f)
cache_credentials = False
ldap_tls_cacertdir = /etc/openldap/cacerts
entry_cache_timeout = 600
ldap_network_timeout = 3
krb5_server = $(hostname -f)
autofs_provider = ldap

[autofs]
" > /etc/sssd/sssd.conf

chmod 700 /etc/sssd/sssd.conf

chkconfig sssd on
service sssd start

chkconfig nscd on
service nscd start

chkconfig messagebus on
service messagebus start
```

Also, ensure that you enable every host to authenticate using LDAP via SSSD too:

```
authconfig --enableldap --enableldapauth --ldapserver=ldap://${LDAP}:389/ --ldapbasedn="${DC}" --enablecache --disablefingerprint --kickstart --enablemkhomedir --enablerfc2307bis
```
