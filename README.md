## Velero Backup and Restore

Velero is an open source tool developed by VMware and used for backing and restoring resources in a 
Kubernetes cluster, performing disaster recovery, and can be used for migrating resources and 
persistent volumes to another Kubernetes cluster.

Velero features such as scheduled backups, retention schedules, and pre- or post-backup hooks for custom actions. 
Velero can help protect data stored in persistent volumes and makes your entire Kubernetes cluster more resilient.

### Download Velero 1.17.2 Release

```
wget https://github.com/heptio/velero/releases/download/v1.17.2/velero-v1.17.2-linux-amd64.tar.gz
tar zxf velero-v1.17.2-linux-amd64.tar.gz
sudo mv velero-v1.17.2-linux-amd64/velero /bin/
rm -rf velero*
```
### Create credentials file i have tested in minio (Needed for velero initialization)
```
cat <<EOF>> minio.credentials
[default]
aws_access_key_id=minioadmin
aws_secret_access_key=minioadmin
EOF
```
### Create credentials For OCI
```
[default]
aws_access_key_id=<OCI_CUSTOMER_SECRET_KEY_ID>
aws_secret_access_key=<OCI_CUSTOMER_SECRET_KEY>
```

### Install Velero in the Kubernetes Cluster
```
velero install \
   --provider velero.io/aws \
   --plugins velero/velero-plugin-for-aws:v1.9.2,digitalocean/velero-plugin:v1.9.2 \
   #--plugins velero/velero-plugin-for-gcp:v1.2.0 \
   --bucket velero \
   --secret-file ./minio.credentials \
   --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://192.168.29.71:9000
```

### Install Velero in OKE Cluster
```
velero install \
  --provider aws \
  --image docker.io/velero/velero:v1.16.2 \
  --plugins docker.io/velero/velero-plugin-for-aws:v1.12.2 \
  --bucket Velero-Backup \
  --prefix <Namespace> \
  --use-volume-snapshots=false \
  --secret-file /root/velero/credentials-velero \
  --backup-location-config \
region=us-east-1,s3ForcePathStyle=true,s3Url=https://<Namespace>.compat.objectstorage.ap-sydney-1.oraclecloud.com,checksumAlgorithm="" \
  --use-node-agent \
  --wait

```

### Lets backup all the resources from default namespace 
``` 
root@kmaster:~#  velero backup create backup
or
root@kmaster:~#  velero backup create backup --include-namespaces default
root@kmaster:~# velero backup get
NAME     STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
backup   Completed   0        0          2022-10-20 21:35:57 +0000 UTC   29d       default            <none>
root@kmaster:~# 
````
### List all the backups using below command.
```
root@kmaster:~# velero get backup
NAME     STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
backup   Completed   0        0          2022-10-20 21:35:57 +0000 UTC   29d       default            <none>
root@kmaster:~# 
```
### Restore default namespace
```
root@kmaster:~# velero restore get
NAME       BACKUP   STATUS      STARTED                         COMPLETED                       ERRORS   WARNINGS   CREATED                         SELECTOR
restore1   backup   Completed   2022-10-20 21:38:46 +0000 UTC   2022-10-20 21:38:51 +0000 UTC   0        1          2022-10-20 21:38:46 +0000 UTC   <none>
root@kmaster:~# velero restore create restore2 --from-backup backup
Restore request "restore2" submitted successfully.
Run `velero restore describe restore2` or `velero restore logs restore2` for more details.
root@kmaster:~# 
root@kmaster:~# velero restore get
NAME       BACKUP   STATUS      STARTED                         COMPLETED                       ERRORS   WARNINGS   CREATED                         SELECTOR
restore1   backup   Completed   2022-10-20 21:38:46 +0000 UTC   2022-10-20 21:38:51 +0000 UTC   0        1          2022-10-20 21:38:46 +0000 UTC   <none>
restore2   backup   Completed   2022-10-20 21:57:19 +0000 UTC   2022-10-20 21:57:24 +0000 UTC   0        1          2022-10-20 21:57:19 +0000 UTC   <none>
root@kmaster:~# 
```

### Restore default namespace in OKE
```
[root@jumphost ~]# velero backup get
NAME             STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
argocd-app       Completed   0        0          2026-02-04 04:31:32 +0800 +08   29d       default            <none>
monitoring-app   Completed   0        0          2026-02-04 04:32:05 +0800 +08   29d       default            <none>

[root@jumphost ~]# velero restore  get
[root@jumphost ~]# velero restore  create argocd-app-restore --from-backup argocd-app
Restore request "argocd-app-restore" submitted successfully.
Run `velero restore describe argocd-app-restore` or `velero restore logs argocd-app-restore` for more details.

[root@jumphost ~]# velero restore describe argocd-app-restore | grep Phase:
Phase:                       Completed

[root@jumphost ~]# velero restore  create monitoring-app-restore --from-backup monitoring-app
Restore request "monitoring-app-restore" submitted successfully.
Run `velero restore describe monitoring-app-restore` or `velero restore logs monitoring-app-restore` for more details.

[root@jumphost ~]# velero restore describe monitoring-app-restore | grep Phase:
Phase:                                 Completed

[root@jumphost ~]# velero restore get
NAME                     BACKUP           STATUS      STARTED                         COMPLETED                       ERRORS   WARNINGS   CREATED                         SELECTOR
argocd-app-restore       argocd-app       Completed   2026-02-04 04:42:27 +0800 +08   2026-02-04 04:42:33 +0800 +08   0        2          2026-02-04 04:42:27 +0800 +08   <none>
monitoring-app-restore   monitoring-app   Completed   2026-02-04 04:43:28 +0800 +08   2026-02-04 04:43:45 +0800 +08   0        7          2026-02-04 04:43:28 +0800 +08   <none>

[root@jumphost ~]# velero restore describe monitoring-app-restore | grep Phase:
Phase:                       Completed

[root@jumphost ~]# velero restore get
NAME                     BACKUP           STATUS      STARTED                         COMPLETED                       ERRORS   WARNINGS   CREATED                         SELECTOR
argocd-app-restore       argocd-app       Completed   2026-02-04 04:42:27 +0800 +08   2026-02-04 04:42:33 +0800 +08   0        2          2026-02-04 04:42:27 +0800 +08   <none>
monitoring-app-restore   monitoring-app   Completed   2026-02-04 04:43:28 +0800 +08   2026-02-04 04:43:45 +0800 +08   0        7          2026-02-04 04:43:28 +0800 +08   <none>
[root@jumphost ~]#

```
### Scheduled backup 
```
Create a backup every 6 hours
velero schedule create backupschedule --schedule="0 */6 * * *"

Create a backup every 6 hours with the @every notation
velero schedule create backupschedule --schedule="@every 6h"

Create a daily backup of the demo namespace
velero schedule create backupschedule --schedule="@every 24h" --include-namespaces demo

Create a weekly backup, each living for 90 days (2160 hours)
velero schedule create backupschedule --schedule="@every 168h" --ttl 2160h0m0s
```
### Remove Velero 
```
velero uninstall --force
```
