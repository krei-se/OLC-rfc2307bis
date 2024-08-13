# OLC-rfc2307bis

Simple replacing of nis in favor of rfc2307bis:

Debian has cn=config by default.

### SCHEMA FOR RFC2307bis:

```
cd /etc/ldap/schema
wget https://github.com/jtyr/rfc2307bis/raw/master/rfc2307bis.schema
schema2ldif rfc2307bis.schema > rfc2307bis.ldif
```

- make sure slapd is started and dump the old nis schema:

`ldapsearch -Y EXTERNAL -H ldapi:/// -b cn={2}nis,cn=schema,cn=config > oldnis.ldif`

- create a deletion ldif for all attributes and objects of nis:

`touch delete_nis_attributes_and_objects.ldif`

i did it for you, [here it is](https://github.com/krei-se/OLC-rfc2307bis/raw/main/delete_nis_attributes_and_objects.ldif)

- apply the deletion ldif

`ldapmodify -Y EXTERNAL -H ldapi:/// -f delete_nis_attributes_and_objects.ldif`

- add the rfc2307bis.ldif:

`ldapadd -Y EXTERNAL -H ldapi:/// -f rfc2307bis.ldif`

if you don't want the empty cn={2}nis.ldif and rfc2307bis on {4} but {2}rfc2307bis.ldif:

- remove the old nis-schema now by STOPPING THE SERVER and deleting the file in slapd.d:

```
systemctl stop slapd
rm /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}nis.ldif
mv /etc/ldap/slapd.d/cn=config/cn=schema/cn={4}rfc2307bis.ldif  /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}rfc2307bis.ldif
systemctl start slapd
```

That's it, enjoy your clean schemas!