# OLC-rfc2307bis

Simple replacing of nis in favor of rfc2307bis:

![ADS1](https://raw.githubusercontent.com/krei-se/OLC-rfc2307bis/main/ads1.png)
![ADS2](https://raw.githubusercontent.com/krei-se/OLC-rfc2307bis/main/ads2.png)


Debian has cn=config by default.

### SCHEMA FOR RFC2307bis:

```
cd /etc/ldap/schema
wget https://github.com/jtyr/rfc2307bis/raw/master/rfc2307bis.schema
schema2ldif rfc2307bis.schema > rfc2307bis.ldif
```

- make sure slapd is started and dump the old nis schema:

```
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn={2}nis,cn=schema,cn=config > oldnis.ldif
```

- create a deletion ldif for all attributes and objects of nis:

```
touch delete_nis_attributes_and_objects.ldif
```

now add before objects and attributes the delete directives. Make sure not to include blank lines or the import will stop halfway.

```
dn: cn={2}nis,cn=schema,cn=config
changetype: modify
delete: olcObjectClasses
olcObjectClasses: ...
-
delete: olcObjectAttributes
olcObjectAttributes: ...

```

Tedious? I did it for you, [here it is](https://github.com/krei-se/OLC-rfc2307bis/raw/main/delete_nis_attributes_and_objects.ldif)

- apply the deletion ldif

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f delete_nis_attributes_and_objects.ldif
```

- add the rfc2307bis.ldif:

```
ldapadd -Y EXTERNAL -H ldapi:/// -f rfc2307bis.ldif
```

This will work on a running server without breaking connections / clients.

Now if you don't want the empty cn={2}nis.ldif and rfc2307bis on {4} but {2}rfc2307bis.ldif:

- remove the old nis-schema now by STOPPING THE SERVER and deleting the file in slapd.d:

```
systemctl stop slapd
rm /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}nis.ldif
mv /etc/ldap/slapd.d/cn=config/cn=schema/cn={4}rfc2307bis.ldif  /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}rfc2307bis.ldif
systemctl start slapd
```

That's it, enjoy your clean schemas!

I know there are lots of warnings and backups online, but if you replace all attributes and objects nothing will break - the schema is flattened anyway and the index-numbering is relaxed, openldap will not complain about a skipped index and just append the next number.

You can do a backup and i also did, but i couldn't restore from it in a testrun, so i'm not sure if this even matters that much ;)