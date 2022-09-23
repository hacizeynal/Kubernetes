
## verify pods and services with single command

cloud_user@ip-10-0-1-101:~$ kubectl get pods,services
```
NAME                                  READY   STATUS    RESTARTS   AGE
pod/busybox                           1/1     Running   0          16m
pod/store-products-576bb96d6d-7qqdl   1/1     Running   0          8m47s
pod/store-products-576bb96d6d-n644b   1/1     Running   0          8m47s
pod/store-products-576bb96d6d-ss4f5   1/1     Running   0          8m47s
pod/store-products-576bb96d6d-w64g2   1/1     Running   0          8m47s

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP   17m
service/store-products   ClusterIP   10.100.125.134   <none>        80/TCP    2m38s
```

## verify pods and services with single command with wide command

cloud_user@ip-10-0-1-101:~$ kubectl get pods,services -o wide
```
NAME                                  READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE
pod/busybox                           1/1     Running   0          17m   10.244.1.4   ip-10-0-1-102   <none>
pod/store-products-576bb96d6d-7qqdl   1/1     Running   0          10m   10.244.2.2   ip-10-0-1-103   <none>
pod/store-products-576bb96d6d-n644b   1/1     Running   0          10m   10.244.1.5   ip-10-0-1-102   <none>
pod/store-products-576bb96d6d-ss4f5   1/1     Running   0          10m   10.244.2.4   ip-10-0-1-103   <none>
pod/store-products-576bb96d6d-w64g2   1/1     Running   0          10m   10.244.2.3   ip-10-0-1-103   <none>

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE     SELECTOR
service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP   18m     <none>
service/store-products   ClusterIP   10.100.125.134   <none>        80/TCP    3m50s   app=store-products
```


## send request to port 80 via curl command (from inside) ,cluster IP is responding for deployment.

cloud_user@ip-10-0-1-101:~$ curl 10.100.125.134:80
```
{
	"Products":[
		{
			"Name":"Apple",
			"Price":1000.00,
		},
		{
			"Name":"Banana",
			"Price":5.00,
		},
		{
			"Name":"Orange",
			"Price":1.00,
		},
		{
			"Name":"Pear",
			"Price":0.50,
		}
	]
}
```

