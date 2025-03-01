# COnfiguring HTPasswd Identity Provider

- Create a file with users and their password

```bash
htpasswd -c -B -b /tmp/htpasswd ocpadmin ocpadmin
```

- Creata a secret in openshift-config namespace using below command

```bash
oc create secret generic htpass-secret --from-file=htpasswd=/tmp/htpasswd -n openshift-config 
```

- Then edit the OAuth cluster config

```bash
oc edit oauth cluster
```

```yaml
spec:
  identityProviders:
  - name: htpass-identity 
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret 
```

- Provide cluster admin permission to the user by apply cluster role binding

```bash
oc adm policy add-cluster-role-to-user cluster-admin ocpadmin
```
