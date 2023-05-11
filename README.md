# REMEDIACIÓN AÑADIR ALARMAS A VOLÚMENES EBS ADJUNTOS A INSTANCIA

*Para este caso ya se tiene creado un tema SNS configurado previamente con el objetivo llamado “AlertaRecursosInutilizados”.*

La creación de la alarma se espera con los siguientes parámetros:

- Agregar o editar alarma = Crear una nueva alarma
- Nombre de alarma = Default
- Tema SNS para la notificacion de la alarma = AlertaRecursosInutilizados
- Umbrales de alarma:
  - Agrupar por = Maximo
  - Tipo de dato para muestra = Tiempo de inactividad
  - Alarma cuando = >=
  - Segundos = 600
  - Periodos consecutivos = 4
  - Periodo = 6 horas

**1.** Conectarse mediante la CLI (awscli)


**2.** Se busca primero listar todos los volúmenes que estén adjuntos a una instancia.

Metodo CLI:
```markdown
aws ec2 describe-volumes --filters Name=status,Values=in-use --query "Volumes[?Attachments[0].InstanceId].VolumeId" --output text
```

Agregar la secuencia `> file.csv` para exportar a **CSV**


**2.** Identificar el comando específico según la necesidad.

*(reemplazar por los valores correctos en <>)*
```bash
aws cloudwatch put-metric-alarm --alarm-name <alarm-namee> --alarm-description "Alarma de inactividad para el volumen EBS" --actions-enabled --alarm-actions <arn:aws:sns:us-east-1:xxxxxxxxxx:AlertaRecursosInutilizados> --metric-name VolumeIdleTime --namespace AWS/EBS --statistic Maximum --period 21600 --evaluation-periods 4 --threshold 600 --comparison-operator GreaterThanOrEqualToThreshold --dimensions Name=VolumeId,Value=<vol-id> --unit Seconds
```

**3.** Crear el script.

Metodo Bash `(mi_script.sh)``

#!/bin/bash


#Definir el comando como una variable
COM="aws cloudwatch put-metric-alarm --alarm-name awsebs-{}-GreaterThanOrEqualToThreshold-VolumeIdleTime --alarm-description 'Alarma de inactividad para el volumen EBS' --actions-enabled --alarm-actions arn:aws:sns:us-east-1:xxxxxxxxxx:AlertaRecursosInutilizados --metric-name VolumeIdleTime --namespace AWS/EBS --statistic Maximum --period 21600 --evaluation-periods 4 --threshold 600 --comparison-operator GreaterThanOrEqualToThreshold --dimensions Name=VolumeId,Value={} --unit Seconds"


```bash
#Definir la lista de identificadores de volumen
VOL_IDS=(
    "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" "vol-00101001010101010" ... )


#iterar sobre la lista de identificadores de volumen y ejecutar el comandp
for id in "${VOL_IDS[@]}"
do
  eval "${COM//\{\}/$id}"
done
```

**4.** Ejecución.

- Ubicarse en el mismo directorio del script: `cd ~/scripts`

- Visualizar el archivo del script creado: `ls -l`

- Dar los permisos para la ejecución: `chmod +x mi_script.sh`

- Ejecutar el script: `./mi_script.sh`
