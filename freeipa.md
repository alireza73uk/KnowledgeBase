# Freeipa

## docker compose
```
services:
  freeipa:
    image: freeipa/freeipa-server:{{ freeipa_server_version }}
    container_name: freeipa
    restart: always
    hostname: {{ freeipa_service_address }}
    environment:
      TZ: "Asia/Tehran"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - {{ freeipa_data_path }}:/data
    command:
      - --domain={{ domain_name }}
      - --realm={{ domain_name }}
      - --http-pin={{ freeipa_admin_password }}
      - --dirsrv-pin={{ freeipa_admin_password }}
      - --ds-password={{ freeipa_admin_password }}
      - --admin-password={{ freeipa_admin_password }}
      - --unattended
    ports:
      - "{{ freeipa_kerberos_port }}:88"
      - "{{ freeipa_kerberos_port }}:88/udp"
      - "{{ freeipa_ldap_port }}:389"
      - "{{ freeipa_https_ports }}:443"
      - "{{ freeipa_kpasswd_port }}:464"
      - "{{ freeipa_kpasswd_port }}:464/udp"
      - "{{ freeipa_ldaps_port }}:636"

```
## wildcard certificate from freeipa

In freeipa container or machine do:

`kinit admin`

`ipa certprofile-show caIPAserviceCert --out wildcard.cfg`

Modify `wildcard.cfg` with these values:

`profileId=wildcard`

remove all index 12 and add:

```
policyset.serverCertSet.12.constraint.class_id=noConstraintImpl
policyset.serverCertSet.12.constraint.name=No Constraint
policyset.serverCertSet.12.default.class_id=subjectAltNameExtDefaultImpl
policyset.serverCertSet.12.default.name=Subject Alternative Name Extension Default
policyset.serverCertSet.12.default.params.subjAltNameNumGNs=2
policyset.serverCertSet.12.default.params.subjAltExtGNEnable_0=true
policyset.serverCertSet.12.default.params.subjAltExtType_0=DNSName
policyset.serverCertSet.12.default.params.subjAltExtPattern_0=*.$request.req_subject_name.cn$
policyset.serverCertSet.12.default.params.subjAltExtGNEnable_1=true
policyset.serverCertSet.12.default.params.subjAltExtType_1=DNSName
policyset.serverCertSet.12.default.params.subjAltExtPattern_1=$request.req_subject_name.cn$
```

Remove index 11 and make sure to modify this line `policyset.serverCertSet.list=1,2,3,4,5,6,7,8,9,10,12`  notice missing index 11.

Now import the modified configuration as a new profile called wildcard:


`ipa certprofile-import wildcard --file wildcard.cfg --desc 'Wildcard certificates' --store 1`

Next, set up a CA ACL to allow the wildcard profile to be used with the example.com host:


`ipa caacl-add wildcard-hosts`
```
-----------------------------
Added CA ACL "wildcard-hosts"
-----------------------------
  ACL name: wildcard-hosts
  Enabled: TRUE
```

`ipa caacl-add-profile wildcard-hosts --certprofiles wildcard`

```
  ACL name: wildcard-hosts
  Enabled: TRUE
  CAs: ipa
  Profiles: wildcard
-------------------------
Number of members added 1
-------------------------
```

`ipa host-add example.com --force`

`ipa caacl-add-host wildcard-hosts --hosts example.com`

```
  ACL name: wildcard-hosts
  Enabled: TRUE
  CAs: ipa
  Profiles: wildcard
  Hosts: example.com
-------------------------
Number of members added 1
-------------------------
```



`ipa caacl-add-ca wildcard-hosts --cas ipa`

```
  ACL name: wildcard-hosts
  Enabled: TRUE
  CAs: ipa
-------------------------
Number of members added 1
-------------------------
```


Then create a CSR with subject CN=example.com and without any SANs, and issue the certificate.

The issued certificate will have *.example.com as its

```
ftweedal% ipa cert-request my.csr \
    --principal host/example.com \
    --profile wildcard
```

### Thanks to

1- https://frasertweedale.github.io/blog-redhat/posts/2017-02-20-freeipa-wildcard-certs.html

2- https://frasertweedale.github.io/blog-redhat/posts/2017-06-26-freeipa-wildcard-san.html
