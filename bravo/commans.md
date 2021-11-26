

1. Creating host path on worker node

```sh
	ssh node01

	mkdir drupal-data
	mkdir drupal-mysql-data
```

2. Return to control plane

```sh
	ssh controlplane
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

```

5. Create Deployment and Service for Drupal and MySQL

```sh

```

