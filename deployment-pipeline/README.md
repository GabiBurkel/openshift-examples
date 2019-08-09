For this example to work you need to create a persistentVolume and persistentVolumeClaim. This 
can either be done via Webconsole or as described in this blogpost which was  taken from 
https://developers.redhat.com/blog/2017/04/05/adding-persistent-storage-to-minishift-cdk-3-in-minutes/

The steps to be executed are: 
logon to minishift console via "minishift ssh" --> become root "sudo -i" --> run: 
   #> mkdir -p /mnt/sda1/var/lib/minishift/openshift.local.volumes/pv/simplewebserver
   #> chmod 777 -R /mnt/sda1/var/lib/minishift/openshift.local.volumes/pv

logout of minishift and create the object via oc: 

# create a persistent volume

cat << PV | oc create -f -
apiVersion: v1
kind: PersistentVolume
metadata:
 name: simplewebserver-pv3
spec:
 capacity:
  storage: 2Gi
 accessModes:
  - ReadWriteOnce
 storageClassName: slow
 hostPath:
  path: /mnt/sda1/var/lib/minishift/openshift.local.volumes/pv/simplewebserver
PV


# create a persistent volume claim

cat << PVC | oc create -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
 name: simplewebserver-pvc3
spec:
 accessModes:
  - ReadWriteOnce
 resources:
  requests:
   storage: 2Gi
 storageClassName: slow
 selector:
  name: simplewebserver-pv3
PVC

# patch the deploymentconfig to add the storageClassName
oc set volumes dc/simplewebserver --add --name=extra-storage -t pvc --claim-name=simplewebserver-pvc --overwrite

