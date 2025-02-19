# Configuring Internal image registry using Object storage in Openshift

## Requirements

1. ODF operator need to be installed and should be configured and ready to create object bucket

## Procedure

1. First create object bucket claim using the below yaml file.

```yaml
cat <<EOF >> image-registry-bucket.yaml
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: image-registry-bucket
  namespace: openshift-image-registry
spec:
  storageClassName: ocs-storagecluster-ceph-rgw
  generateBucketName: image-registry-bucket
EOF
```

- for creating the object bucket claim run the below command

```bash
$ oc create -f image-registry-bucket.yaml
```

2. Gather details like bucket name, bucket endpoint, Access key and Secret access key.

```bash
BUCKET_HOST=$(oc get -n openshift-image-registry configmap image-registry-bucket -o jsonpath='{.data.BUCKET_HOST}')

BUCKET_NAME=$(oc get -n openshift-image-registry configmap image-registry-bucket -o jsonpath='{.data.BUCKET_NAME}')

ACCESS_KEY_ID=$(oc get -n openshift-image-registry secret image-registry-bucket -o jsonpath='{.data.AWS_ACCESS_KEY_ID}' | base64 -d)

SECRET_ACCESS_KEY=$(oc get -n openshift-image-registry secret image-registry-bucket -o jsonpath='{.data.AWS_SECRET_ACCESS_KEY}' | base64 -d)

```

3. Using access key and secret access key create secret.In openshift-image-registry namespace.

```bash

oc create -n openshift-image-registry secret generic image-registry-private-configuration-user --from-literal=REGISTRY_STORAGE_S3_ACCESSKEY="${ACCESS_KEY_ID}" --from-literal=REGISTRY_STORAGE_S3_SECRETKEY="${SECRET_ACCESS_KEY}"

```

4. Update to cluster config using the bucket details like S3 bucketname, bucket endpoint and  is present at trustedCA and management state using the below command

- If you don't have ca-bundle.crt run the below command 
```bash

$ oc patch config.image/cluster -p '{"spec":{"managementState":"Managed","replicas":1,"storage":{"managementState":"Unmanaged","s3":{"bucket":'\"$BUCKET_NAME\"',"regionEndpoint":'\"http://$BUCKET_HOST\"',"virtualHostedStyle":false,"encrypt":false}}}}' --type=merge

```

- If you have ca-bundle.crt, create configMap and then run the below command 
```bash

$ oc create cm image-registry-s3-bundle --from-file=ca-bundle.crt=<CA_BUNDLE.CRT>  -n openshift-config

$ oc patch config.image/cluster -p '{"spec":{"managementState":"Managed","replicas":1,"storage":{"managementState":"Unmanaged","s3":{"bucket":'\"$BUCKET_NAME\"',"regionEndpoint":'\"https://$BUCKET_HOST\"',"virtualHostedStyle":false,"encrypt":false, "trustedCA":{"name":"image-registry-s3-bundle"}}}}}' --type=merge

```
