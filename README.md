## Velero Backup and Restore

Velero is an open source tool developed by VMware and used for backing and restoring resources in a 
Kubernetes cluster, performing disaster recovery, and can be used for migrating resources and 
persistent volumes to another Kubernetes cluster.

Velero features such as scheduled backups, retention schedules, and pre- or post-backup hooks for custom actions. 
Velero can help protect data stored in persistent volumes and makes your entire Kubernetes cluster more resilient.


### Download Velero 1.9.2 Release

```
wget https://github.com/heptio/velero/releases/download/v1.9.2/velero-v1.9.2-linux-amd64.tar.gz
tar zxf velero-v1.9.2-linux-amd64.tar.gz
sudo mv velero-v1.9.2-linux-amd64/velero /usr/local/bin/
rm -rf velero*

```
### Create credentials file i have tested in minio (Needed for velero initialization)

```
cat <<EOF>> minio.credentials
[default]
aws_access_key_id=minio
aws_secret_access_key=minio123
EOF
```

### Install Velero in the Kubernetes Cluster

velero install \
   --provider velero.io/aws \
   --plugins velero/velero-plugin-for-aws:v1.9.2,digitalocean/velero-plugin:v1.9.2 \
   --bucket velero \
   --secret-file ./minio.credentials \
   --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://192.168.29.71:9000
  
``` 
root@kmaster:~#  velero backup create backup

root@kmaster:~# velero backup get
NAME     STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
backup   Completed   0        0          2022-10-20 21:35:57 +0000 UTC   29d       default            <none>
root@kmaster:~# 
````