root@localhost:~# cat nfs-pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 250Gi               # Size of the PV, matching your SFS size
  accessModes:
    - ReadWriteMany              # Set the access mode for NFS
  nfs:
    path: /data     # default path of sfs in e2e
    server: sfs ip         # Your NFS server's IP
    readOnly: false              # Set to true if you want it to be read-only
  persistentVolumeReclaimPolicy: Retain  # Retain, Recycle, or Delete after use

root@localhost:~# cat nfs-pvc.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
  namespace: devops-proj        # Your specified namespace
spec:
  accessModes:
    - ReadWriteMany              # Match the access mode of the PV
  resources:
    requests:
      storage: 250Gi            # Requesting the same size as the PV

root@localhost:~# cat nginx-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nfs-pod
  namespace: devops-proj        # Your specified namespace
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: "/app"   # Directory in the pod where NFS will be mounted
      name: nfs-storage
  volumes:
  - name: nfs-storage
    persistentVolumeClaim:
      claimName: nfs-pvc                 # Link to the PVC created earlier

      kindly grant permission in console of sfs

