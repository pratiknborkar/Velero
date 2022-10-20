### MinIO Object Storage 
It is a high performance object storage solution that provides an Amazon Web Services 
S3-compatible API and supports all core S3 features.
MinIO is built to deploy anywhere - public or private cloud, baremetal infrastructure, 
orchestrated environments, and edge infrastructure.

### Installation of MinIO
``
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio

sudo mv minio /usr/local/bin/

or 

sudo mv minio /usr/bin/
``

### Start MinIO Server 
```
sudo minio server /minio
```

### Access MinIO Externally using port 9000
```
[root@centosvm01 ~]# sudo minio server /minio
WARNING: Detected Linux kernel version older than 4.0.0 release, there are some known potential performance problems with this kernel version. MinIO recommends a minimum of 4.x.x linux kernel version for best performance
WARNING: Detected default credentials 'minioadmin:minioadmin', we recommend that you change these values with 'MINIO_ROOT_USER' and 'MINIO_ROOT_PASSWORD' environment variables
MinIO Object Storage Server
Copyright: 2015-2022 MinIO, Inc.
License: GNU AGPLv3 <https://www.gnu.org/licenses/agpl-3.0.html>
Version: RELEASE.2022-10-20T00-55-09Z (go1.19.2 linux/amd64)

Status:         1 Online, 0 Offline. 
API: http://10.0.0.201:9000  http://192.168.61.148:9000  http://127.0.0.1:9000       
RootUser: minioadmin 
RootPass: minioadmin 
Console: http://10.0.0.201:44889 http://192.168.61.148:44889 http://127.0.0.1:44889    
RootUser: minioadmin 
RootPass: minioadmin 

Command-line: https://min.io/docs/minio/linux/reference/minio-mc.html#quickstart
   $ mc alias set myminio http://10.0.0.201:9000 minioadmin minioadmin

Documentation: https://min.io/docs/minio/linux/index.html
```