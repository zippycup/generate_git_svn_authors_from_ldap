# Generate authors file from ldap for use with git-svn migration

## Description
This script will read information from ldap to create the necessary file for input to git-svn migration ( e.g. rpm git-svn-1.8.3.1 )

## Requirement
admin user with access to read ldap

## Instructions
* cut and past this script to your host. filename: get_authors.sh
```
#!/bin/bash

ou='People'  #you can include addition ou(s) separated by a space.
domain='mydomain'
domain_extension='net'
ldap_server='ldap-1.mydomain.net'
ldap_user='cn=myadminuser,dc=mydomain,dc=net'
ldap_pass='mypassword'
unsorted='unsorted.txt'
output='authors.txt'

rm -f ${unsorted}

for i in ${ou}
do
  ldapsearch -D "${ldap_user}" -w "${ldap_pass}" -p 389 -h ${ldap_server} -b "ou=${i},dc=${domain},dc=${domain_extension}" |
  awk '/^cn:/ { firstname = $2 } /^sn:/ { lastname = $2 } /^mail:/ { email = $2 } /^uid:/ { uid = $2} /^$/ { if ( email != "") {print uid " = " firstname " " lastname " <" email ">"} email = ""}' >> ${unsorted}
done

cat ${unsorted} | sort > ${output}
rm -f ${unsorted}
```

* run script
```
sh -x get_authors.sh
```
* This will generate a file named authors.txt as formatted
```
# uid = cn sn <email>
jdoe = John Doe <john.doe@mydomain.net>
```
