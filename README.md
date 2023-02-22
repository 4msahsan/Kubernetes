<pre>
<h1> *** Install MicroK8s *** </h1>
root@microk8s:~# snap install microk8s --classic
microk8s (1.26/stable) v1.26.1 from Canonical✓ installed
root@microk8s:~# microk8s start
root@microk8s:~#

<h1> Install Metallb for LoadBalancer on vm </h1>

root@microk8s:~# kk  apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
namespace/metallb-system created
customresourcedefinition.apiextensions.k8s.io/addresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io created
customresourcedefinition.apiextensions.k8s.io/communities.metallb.io created
customresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io created
serviceaccount/controller created
serviceaccount/speaker created
role.rbac.authorization.k8s.io/controller created
role.rbac.authorization.k8s.io/pod-lister created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/controller created
rolebinding.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
secret/webhook-server-cert created
service/webhook-service created
deployment.apps/controller created
daemonset.apps/speaker created
validatingwebhookconfiguration.admissionregistration.k8s.io/metallb-webhook-configuration created
root@microk8s:~#


root@microk8s:~# kk get all -n metallb-system
NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-68bf958bf9-gmg82   1/1     Running   0          32s
pod/speaker-s4nrs                 1/1     Running   0          32s

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/webhook-service   ClusterIP   10.152.183.84   <none>        443/TCP   32s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   32s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           32s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-68bf958bf9   1         1         1       32s
root@microk8s:~#


***
root@microk8s:~# cat configuresLayer2.yml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.0.240-192.168.0.250


root@microk8s:~# kk create -f configuresLayer2.yml
ipaddresspool.metallb.io/first-pool created
root@microk8s:~# kk get all -n metallb-system
NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-68bf958bf9-gmg82   1/1     Running   0          3m33s
pod/speaker-s4nrs                 1/1     Running   0          3m33s

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/webhook-service   ClusterIP   10.152.183.84   <none>        443/TCP   3m33s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   3m33s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           3m33s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-68bf958bf9   1         1         1       3m33s
root@microk8s:~#


root@microk8s:~# microk8s kubectl create deployment test-nginx --image=nginx
deployment.apps/test-nginx created
root@microk8s:~# microk8s kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
test-nginx-6b4d69f6f8-7phms   0/1     ContainerCreating   0          10s


root@microk8s:~#  microk8s kubectl expose pod test-nginx-6b4d69f6f8-7phms --type="LoadBalancer" --port 80
service/test-nginx-6b4d69f6f8-7phms exposed
root@microk8s:~# kk get svc
NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
kubernetes                    ClusterIP      10.152.183.1     <none>          443/TCP        10m
test-nginx-6b4d69f6f8-7phms   LoadBalancer   10.152.183.181   192.168.0.240   80:32043/TCP   12s
root@microk8s:~#


<h1> *** Helm *** </h1>
curl -fsSL -o get_helm.sh \
https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
root@microk8s:~#



root@microk8s:~# chmod 700 get_helm.sh
root@microk8s:~#  ./get_helm.sh
Downloading https://get.helm.sh/helm-v3.11.1-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
root@microk8s:~#

root@microk8s:~# helm version
version.BuildInfo{Version:"v3.11.1", GitCommit:"293b50c65d4d56187cd4e2f390f0ada46b4c4737", GitTreeState:"clean", GoVersion:"go1.18.10"}
root@microk8s:~#

root@microk8s:~# helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
root@microk8s:~#



root@microk8s:~# microk8s enable helm3
Infer repository core for addon helm3
Addon core/helm3 is already enabled

root@microk8s:~# microk8s helm3 version
version.BuildInfo{Version:"v3.9.1+unreleased", GitCommit:"0b977ed36f2db4f947a7a107fc3f5298401c4a96", GitTreeState:"clean", GoVersion:"go1.19.5"}
root@microk8s:~#


root@microk8s:~# microk8s helm3 search hub wordpress
URL                                                     CHART VERSION   APP VERSION             DESCRIPTION
https://artifacthub.io/packages/helm/kube-wordp...      0.1.0           1.1                     this is my wordpress package
https://artifacthub.io/packages/helm/truecharts...      1.1.15          6.1.1                   The WordPress rich content management system ca...
https://artifacthub.io/packages/helm/bitnami-ak...      15.2.13         6.1.0                   WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/shubham-wo...      0.1.0           1.16.0                  A Helm chart for Kubernetes
https://artifacthub.io/packages/helm/bitnami/wo...      15.2.45         6.1.1                   WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/camptocamp...      0.6.10          4.8.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/devops/wor...      0.12.0          1.16.0                  Wordpress helm chart
https://artifacthub.io/packages/helm/risserlabs...      0.0.1           latest                  open source software you can use to create a be...
https://artifacthub.io/packages/helm/groundhog2...      0.7.5           6.1.1-apache            A Helm chart for Wordpress on Kubernetes
https://artifacthub.io/packages/helm/riftbit/wo...      12.1.16         5.8.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/sikalabs/w...      0.2.0                                   Simple Wordpress
https://artifacthub.io/packages/helm/mcouliba/w...      0.1.0           1.16.0                  A Helm chart for Kubernetes
https://artifacthub.io/packages/helm/homeenterp...      0.5.0           5.9.3-php8.1-apache     Blog server
https://artifacthub.io/packages/helm/wordpress-...      1.0.0           1.1                     This is a package for configuring wordpress and...
https://artifacthub.io/packages/helm/securecode...      3.16.0-alpha3   4.0                     Insecure & Outdated Wordpress Instance: Never e...
https://artifacthub.io/packages/helm/wordpressm...      1.0.0                                   This is the Helm Chart that creates the Wordpre...
https://artifacthub.io/packages/helm/bitpoke/wo...      0.12.1          v0.12.1                 Bitpoke WordPress Operator Helm Chart
https://artifacthub.io/packages/helm/veilis-hel...      0.12.1999       v0.12.1999              Veilis WordPress Operator Helm Chart
https://artifacthub.io/packages/helm/veilis-hel...      0.12.3999       v0.12.3999              Helm chart for deploying a WordPress site on Ve...
https://artifacthub.io/packages/helm/bitpoke/wo...      0.12.3          v0.12.3                 Helm chart for deploying a WordPress site on Bi...
https://artifacthub.io/packages/helm/kube-wordp...      0.1.0           0.0.1-alpha             Helm Chart for Wordpress installation on MySQL ...
https://artifacthub.io/packages/helm/riotkit-or...      2.1.0-alpha4    v2.1-alpha4             Lightweight Wordpress installation with additio...
https://artifacthub.io/packages/helm/phntom/bin...      0.0.4           0.0.3                   www.binaryvision.co.il static wordpress
https://artifacthub.io/packages/helm/gh-shessel...      2.1.8           6.1.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/sikalabs/w...      0.1.2
https://artifacthub.io/packages/helm/gh-shessel...      4.0.1           6.1.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/wordpressh...      0.1.0           1.16.0                  A Helm chart for Kubernetes
https://artifacthub.io/packages/helm/wordpress-...      0.1.0           1.1                     this is chart create the wordpress with suitabl...
https://artifacthub.io/packages/helm/wordpress-...      0.0.1                                   Helm Chart for wordpress-gatsby
https://artifacthub.io/packages/helm/bitpoke/bi...      1.8.10          1.8.10                  The Bitpoke App for WordPress provides a versat...
https://artifacthub.io/packages/helm/sonu-wordp...      1.0.0           2                       This is my custom chart to deploy wo

root@microk8s:~# microk8s helm3 repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" already exists with the same configuration, skipping
root@microk8s:~#  microk8s helm3 repo list
NAME    URL
bitnami https://charts.bitnami.com/bitnami
root@microk8s:~#


root@microk8s:~#  microk8s helm3 search repo bitnami
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/airflow                                 14.0.12         2.5.1           Apache Airflow is a tool to express and execute...
bitnami/apache                                  9.2.16          2.4.55          Apache HTTP Server is an open-source HTTP serve...
bitnami/appsmith                                0.1.12          1.9.7           Appsmith is an open source platform for buildin...
bitnami/argo-cd                                 4.4.10          2.6.2           Argo CD is a continuous delivery tool for Kuber...
bitnami/argo-workflows                          5.1.9           3.4.5           Argo Workflows is meant to orchestrate Kubernet...
bitnami/aspnet-core                             4.0.6           7.0.3           ASP.NET Core is an open-source framework for we...
bitnami/cassandra                               10.0.3          4.1.0           Apache Cassandra is an open source distributed ...
bitnami/cert-manager                            0.9.1           1.11.0          cert-manager is a Kubernetes add-on to automate...
bitnami/clickhouse                              3.0.2           23.1.3          ClickHouse is an open-source column-oriented OL...
bitnami/common                                  2.2.3           2.2.3           A Library Helm Chart for grouping common logic ...
bitnami/concourse                               2.0.4           7.9.1           Concourse is

root@microk8s:~# microk8s helm3 show chart bitnami/wordpress
annotations:
  category: CMS
  licenses: Apache-2.0
apiVersion: v2
appVersion: 6.1.1
dependencies:
- condition: memcached.enabled
  name: memcached
  repository: https://charts.bitnami.com/bitnami
  version: 6.x.x
- condition: mariadb.enabled
  name: mariadb
  repository: https://charts.bitnami.com/bitnami
  version: 11.x.x
- name: common
  repository: https://charts.bitnami.com/bitnami
  tags:
  - bitnami-common
  version: 2.x.x
description: WordPress is the world's most popular blogging and content management
  platform. Powerful yet simple, everyone from students to global corporations use
  it to build beautiful, functional websites.
home: https://github.com/bitnami/charts/tree/main/bitnami/wordpress
icon: https://bitnami.com/assets/stacks/wordpress/img/wordpress-stack-220x234.png
keywords:
- application
- blog
- cms
- http
- php
- web
- wordpress
maintainers:
- name: Bitnami
  url: https://github.com/bitnami/charts
name: wordpress
sources:
- https://github.com/bitnami/containers/tree/main/bitnami/wordpress
- https://wordpress.org/
version: 15.2.45

root@microk8s:~#

root@microk8s:~# microk8s helm3 install wordpress bitnami/wordpress
NAME: wordpress
LAST DEPLOYED: Wed Feb 22 18:38:13 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: wordpress
CHART VERSION: 15.2.45
APP VERSION: 6.1.1

** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
root@microk8s:~#

root@microk8s:~# kk  get svc --namespace default -w wordpress
NAME        TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
wordpress   LoadBalancer   10.152.183.156   192.168.0.241   80:30933/TCP,443:32766/TCP   27m
</pre>
