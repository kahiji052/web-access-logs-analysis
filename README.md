# Proyecto Final | Análisis de logs (acceso al servidor web) mediante Hadoop
En este repositorio, se implementó **Amazon EMR (Elastic MapReduce)** al ser un servicio que facilita el uso Apache Hadoop para procesar grandes volúmenes de datos. En este caso, se utilizó un dataset de logs de acceso de un servidor web proporcionado por Kaggle.

## Dataset
El dataset utilizado es accesible a través del siguiente enlace: [Web Server Access Logs Dataset](https://www.kaggle.com/datasets/eliasdabbas/web-server-access-logs 'web-server-access-logs').

El archivo contiene registros de acceso a un servidor web, donde cada línea de log refleja una solicitud realizada por un usuario, junto con detalles como el método HTTP utilizado, la URL solicitada y el código de respuesta del servidor, entre otros.

## 0. Creación de Clúster
El siguiente comando genera un clúster de Amazon EMR utilizando la versión emr-6.9.0 e instalando Hadoop y Spark. Además, habilita la sustitución automática de nodos no saludables [*para reemplazar automáticamente sin necesidad de intervención manual*] y configura el escalado automático [*terminando las instancias una vez que las tareas se completan*].
```bash
aws emr create-cluster \
--name "MiClusterEMR" \
--release-label "emr-6.9.0" \
--service-role "EMR_DefaultRole" \
--unhealthy-node-replacement \
--ec2-attributes '{"InstanceProfile":"EMR_EC2_DefaultRole","EmrManagedMasterSecurityGroup":"sg-055a650e5c5a86fb0","EmrManagedSlaveSecurityGroup":"sg-0ff63f78ded1d403d","AvailabilityZone":"us-east-1b"}' \
--applications Name=Hadoop Name=Spark \
--instance-groups '[{"InstanceCount":2,"InstanceGroupType":"CORE","Name":"CORE","InstanceType":"m5.xlarge","EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"VolumeType":"gp2","SizeInGB":32},"VolumesPerInstance":2}]}},{"InstanceCount":1,"InstanceGroupType":"MASTER","Name":"MASTER","InstanceType":"m5.xlarge","EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"VolumeType":"gp2","SizeInGB":32},"VolumesPerInstance":2}]}}]' \
--scale-down-behavior "TERMINATE_AT_TASK_COMPLETION" \
--region "us-east-1"
```

## 1. Subida de archivo desde S3
Por privacidad, sólo es modificar las varibles por el nombre del bucket, el archivo de origen y la ruta de destino deseadas.
```bash
aws s3 cp s3://mi-bucket-de-datos/nuevo-archivo.csv /root/destino/nuevo-archivo.csv
```

## 2. Análisis de Logs con Hadoop
### Logs con el método `GET` para imágenes:
```bash
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar grep input/access.log.bz2 salidaget "GET /image/"
hadoop fs -cat salidaget/part-r-00000
```
![image](https://github.com/user-attachments/assets/0ecd8585-36e4-4078-ba9b-7fb5c7844751)

### Logs con el método `POST`:
```bash
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar grep input/access.log.bz2 salidapost "POST"
hadoop fs -cat salidapost/part-r-00000
```
![image](https://github.com/user-attachments/assets/09a4b84c-581e-4fcc-8376-15812ffaa0b5)

### Logs con los códigos de error `404` y `500`:
```bash
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar grep input/access.log.bz2 salidaerror "404|500"
hadoop fs -cat salidaerror/part-r-00000
```
![image](https://github.com/user-attachments/assets/8c1fe44f-f43d-44d1-aa95-cecb4ab7d4a7)
