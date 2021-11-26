

1. Creating host path folders on worker node

```sh
	ssh node01

	mkdir /drupal-data && \
	mkdir /drupal-mysql-data
```

2. Return to control plane

```sh 
  exit 
```

3. Create secret

```sh	
	kubectl create secret generic drupal-mysql-secret \
	--from-literal=MYSQL_USER=root \
	--from-literal=MYSQL_ROOT_PASSWORD=root-password \
	--from-literal=MYSQL_DATABASE=drupal-database	
```

4. Create PV and PVC for Drupal and MySQL

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-pv
  labels:
    name: drupal-pv
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /drupal-data
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-mysql-pv
  labels:
    name: drupal-mysql-pv
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /drupal-mysql-data
EOF
```

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-pvc
spec:
  selector: 
    matchLabels: 
      name: drupal-pv
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---      
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-mysql-pvc
spec:
  selector: 
    matchLabels: 
      name: drupal-mysql-pv
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF
```

5. Create Deployment and Service for MySQL

```sh
cat <<EOF | kubectl apply -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal-mysql
  labels:
    app: drupal-mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drupal-mysql
  template:
    metadata:
      labels:
        app: drupal-mysql        
    spec:
      containers:
      - name: drupal-mysql
        image: mysql:5.7
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3306
        env:
        # - name: MYSQL_USER
        #   valueFrom:
        #     secretKeyRef:
        #       name: drupal-mysql-secret
        #       key: MYSQL_USER
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: drupal-mysql-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: drupal-mysql-secret
              key: MYSQL_DATABASE
        volumeMounts:
          - mountPath: /var/lib/mysql
            name: vol-drupal
            subPath: dbdata      
      volumes:
        - name: vol-drupal
          persistentVolumeClaim:
            claimName: drupal-mysql-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: drupal-mysql-service
spec:
  type: ClusterIP
  selector:
    app: drupal-mysql
  ports:
    - name: drupal-mysql
      protocol: TCP
      port: 3306
      targetPort: 3306       
EOF
```

6. Create Deployment and Service for Drupal

```sh
cat <<EOF | kubectl apply -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal
  labels:
    app: drupal
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drupal
  replicas: 1
  template:
    metadata:
      labels:
        app: drupal        
    spec:
      initContainers:
      - name: init-sites-volume
        image: drupal:8.6
        command:
        - /bin/bash
        - -c
        args: ['cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R']
        volumeMounts:
        - mountPath: /data
          name: vol-drupal
      containers:
      - name: drupal
        image: drupal:8.6
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        volumeMounts:
          - mountPath: /var/www/html/modules
            name: vol-drupal
            subPath: modules
          - mountPath: /var/www/html/profiles
            name: vol-drupal
            subPath: profiles        
          - mountPath: /var/www/html/sites
            name: vol-drupal
            subPath: sites  
          - mountPath: /var/www/html/themes
            name: vol-drupal
            subPath: themes  
      volumes:
        - name: vol-drupal
          persistentVolumeClaim:
            claimName: drupal-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: drupal-service
spec:
  selector:
    app: drupal
  type: NodePort
  ports:
    - name: drupal
      protocol: TCP
      port: 80
      nodePort: 30095
EOF
```

7. Now you can access Drupal at **NODE_IP**:30095



