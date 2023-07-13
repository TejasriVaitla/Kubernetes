# Introduction
This repository provides examples for running Word Count and PageRank algorithms on Apache Spark using Kubernetes and GKE (Google Kubernetes Engine).

# Design
## Word Count
<img width="500" alt="pagerank theory" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/704f4e00-602e-4f1f-98c1-2108ed039866.png">

## PageRank
<img width="200" alt="pagerank theory" src="https://github.com/TejasriVaitla/Cloud-Computing/assets/128747986/887e34ea-6d64-49e5-bcd0-1c1c4bac2378.png">


1. Assume The initial PageRank value for each webpage is 1. <br>
PR(A) = 1  <br>
PR(B) = 1  <br>
PR(C) = 1  <br>
Page B has a link to pages C and A<br>
Page C has a link to page A<br>
Page D has links to all three pages<br>

## Prerequisites

* GKE cluster with one node:
  ```
    gcloud container clusters create spark --num-nodes=1 --machine-type=e2-highmem-2 --region=us-central1
  ```
  <img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/00caad7b-c409-4c6d-9d9c-223ce2e1878e.png">

# Implementation
* NFS Server Provisioner installation:
```
helm repo add stable https://charts.helm.sh/stable
helm install nfs stable/nfs-server-provisioner
--set persistence.enabled=true,persistence.size=5Gi
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/d168595a-a77e-44da-a170-3d0006580601.png">

## Setup
Create a Persistent Volume Claim and a Pod to use NFS:
* Create the file spark-pvc.yaml
* Apply the YAML descriptor
```
kubectl apply -f spark-pvc.yaml
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/362163ef-bc79-4923-b8a6-39a61bee6698.png">

* Prepare your application JAR file
```
docker run -v /tmp:/tmp -it bitnami/spark -- find /opt/bitnami/spark/examples/jars/ -name spark-examples* -exec cp {} /tmp/my.jar \;
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/edb6d96d-4709-47fc-80b9-94d2226f349d.png">

* Add a test file for word count
```
echo "the quick brown fox the fox ate the mouse how now brown cow" > /tmp/test1.txt
```
* Copy the JAR file and test file to the Persistent Volume Claim
```
kubectl cp /tmp/my.jar spark-data-pod:/data/my.jar
kubectl cp /tmp/test1.txt spark-data-pod:/data/test1.txt
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/8aff8c6b-399b-4180-b262-8f2ebfb622e2.png">

## Deploy Apache Spark on Kubernetes
* Deploy Apache Spark using the Bitnami Apache Spark Helm chart
* Create the file spark-chart.yaml
  
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/c3d2f20b-0cfd-42d3-a425-4c2ba5dada77.png">

* Check Pods are running
  
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/dd0d4634-cb6b-4487-b264-fb61debec2be.png">

* Install the chart
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install spark bitnami/spark -f spark-chart.yaml
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/237b5633-97ff-4ab4-922c-a7abfd1c7357.png">

* Get the external IP of the running pod
```
kubectl get svc -l "app.kubernetes.io/instance=spark,app.kubernetes.io/name=spark"
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/7ab4e819-8dac-44e5-af37-0aaced9a253a.png">

* Open the external ip on your browser
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/30a33a14-fa54-48b4-89ae-e338882ef65d.png">

## Word Count Example
1. Submit a word count task
```
kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.4.1-debian-11-r3 \
    -- spark-submit --master spark://104.154.224.162:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.JavaWordCount \
    /data/my.jar /data/test1.txt
```

<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/6bb577a5-10b8-4076-b908-fcd881112ac4.png">

2. View the output of the completed job
* On the browser, you should see the worker node ip address of the finished task
  
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/87f2a3c7-6483-4515-8966-cf38f832e44d.png">

* Get the name of the worker node
```
  kubectl get pods -o wide | grep WORKER-NODE-ADDRESS
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/695cb55c-9570-4869-bb1a-27e685ec373f.png">

* Execute a pod and see the result of the finished tasks
```
kubectl exec -it <worker node name> -- bash
cd /opt/bitnami/spark/work
cat <taskname>/stdout
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/fd3b817a-7237-4ccf-8271-4b418555b744.png">

## PageRank Example
* Execute the Spark master pod
```
kubectl exec -it spark-master-0 -- bash
```
* Go to the directory where pagerank.py is located
```
cd /opt/bitnami/spark/examples/src/main/python
```
* Run the PageRank example using PySpark
```
spark-submit pagerank.py /opt 2
```
<img alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/4a1613bf-3161-4e9b-9be2-3b9dd501e3ac.png">

* Output

<img width="500" alt="image" src="https://github.com/TejasriVaitla/Kubernetes/assets/128747986/f3dd0148-7c80-4cd6-9f5a-1ee2418e30bf.png">

## Detail Design Presentation 
[Word count and PageRank using Kubernetes](https://docs.google.com/presentation/d/1wVUOm_7hzZYzt7eQSv1w6YzLNtseNzErTzEmby0HmE/edit#slide=id.p)
