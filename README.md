# kube-sidecar-injector

This repo is used for [a tutorial at Medium](https://medium.com/ibm-cloud/diving-into-kubernetes-mutatingadmissionwebhook-6ef3c5695f74) to create a Kubernetes [MutatingAdmissionWebhook](https://kubernetes.io/docs/admin/admission-controllers/#mutatingadmissionwebhook-beta-in-19) that injects a nginx sidecar container into pod prior to persistence of the object.

## Prerequisites

- [git](https://git-scm.com/downloads)
- [go](https://golang.org/dl/) version v1.17+
- [docker](https://docs.docker.com/install/) version 19.03+
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version v1.19+
- Access to a Kubernetes v1.19+ cluster with the `admissionregistration.k8s.io/v1` API enabled. Verify that by the following command:

```
kubectl api-versions | grep admissionregistration.k8s.io
```
The result should be:
```
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
```

> Note: In addition, the `MutatingAdmissionWebhook` and `ValidatingAdmissionWebhook` admission controllers should be added and listed in the correct order in the admission-control flag of kube-apiserver.

## Build and Deploy

1. Build and push docker image:

```bash
make docker-build docker-push IMAGE=quay.io/<your_quayio_username>/sidecar-injector:latest
```

2. Deploy the kube-sidecar-injector to kubernetes cluster:

```bash
make deploy IMAGE=quay.io/<your_quayio_username>/sidecar-injector:latest
```

3. Verify the kube-sidecar-injector is up and running:

```bash
# kubectl -n sidecar-injector get pod
# kubectl -n sidecar-injector get pod
NAME                                READY   STATUS    RESTARTS   AGE
sidecar-injector-7c8bc5f4c9-28c84   1/1     Running   0          30s
```

## How to use

1. Create a new namespace `test-ns` and label it with `sidecar-injector=enabled`:

```
# kubectl create ns test-ns
# kubectl label namespace test-ns sidecar-injection=enabled
# kubectl get namespace -L sidecar-injection
NAME                 STATUS   AGE   SIDECAR-INJECTION
default              Active   26m
test-ns              Active   13s   enabled
kube-public          Active   26m
kube-system          Active   26m
sidecar-injector     Active   17m
```

2. Deploy an app in Kubernetes cluster, take `alpine` app as an example
# 为了更简化前台的配置，将不启用filebeat.yml配置，使用命令行来启动filebeat
# 注入的filebeat修改过filebeat.yml的，因为原生的filebeat使用命令行会有以下问题:
# 修改filebeat.yml，删掉自带的output
bash-4.2$ filebeat -e -E output.kafka.hosts=["kafka:9092"] -E output.kafka.topic="my-topic" -E filebeat.inputs.0.paths=["/path/to/logs/*"]
Exiting: error unpacking config data: more than one namespace configured accessing 'output' (source:'filebeat.yml')


```bash
kubectl apply -f test.yaml
```

3. Verify sidecar container is injected:

```
#[root@VM-13-7-centos ~/ycw/webhook]# kubectl get po -n test-ns
#NAME                    READY   STATUS    RESTARTS   AGE
#sleep-fc87db5db-pxplz   2/2     Running   0          2m38s
```

## Troubleshooting

Sometimes you may find that pod is injected with sidecar container as expected, check the following items:

1. The sidecar-injector pod is in running state and no error logs.
2. The namespace in which application pod is deployed has the correct labels(`sidecar-injector=enabled`) as configured in `mutatingwebhookconfiguration`.
3. Check if the application pod has annotation `sidecar-injector-webhook.morven.me/inject:"yes"`.
