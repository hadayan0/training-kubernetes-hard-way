03の操作メモ

# Networking
## Virtual Private Cloud Network
```
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
ERROR: (gcloud.compute.networks.create) The required property [project] is not currently set.
It can be set on a per-command basis by re-running your command with the [--project] flag.

You may set it for your current workspace by running:

  $ gcloud config set project VALUE

or it can be set temporarily by the environment variable [CLOUDSDK_CORE_PROJECT]
```
エラーになる。projectが指定されていないとのこと。

GCPのウェブ画面で確認できる自分のproject名は `sandbox-hada` なのでそれを設定。

```
hmba hmbanoMacBook-Air.local ~ 
$ gcloud config set project sandbox-hada                                     
Updated property [core/project].


Updates are available for some Cloud SDK components.  To install them,
please run:
  $ gcloud components update



To take a quick anonymous survey, run:
  $ gcloud survey

hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/sandbox-hada/global/networks/kubernetes-the-hard-way].
NAME                     SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
kubernetes-the-hard-way  CUSTOM       REGIONAL

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network kubernetes-the-hard-way --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network kubernetes-the-hard-way --allow tcp:22,tcp:3389,icmp

hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute networks subnets create kubernetes --network kubernetes-the-hard-way --range 10.240.0.0/24

For the following subnetwork:
 - [kubernetes]
choose a region:
 [1] asia-east1
 [2] asia-east2
 [3] asia-northeast1
 [4] asia-northeast2
 [5] asia-northeast3
 [6] asia-south1
 [7] asia-south2
 [8] asia-southeast1
 [9] asia-southeast2
 [10] australia-southeast1
 [11] australia-southeast2
 [12] europe-central2
 [13] europe-north1
 [14] europe-west1
 [15] europe-west2
 [16] europe-west3
 [17] europe-west4
 [18] europe-west6
 [19] northamerica-northeast1
 [20] northamerica-northeast2
 [21] southamerica-east1
 [22] southamerica-west1
 [23] us-central1
 [24] us-east1
 [25] us-east4
 [26] us-west1
 [27] us-west2
 [28] us-west3
 [29] us-west4
Please enter your numeric choice:  3

Created [https://www.googleapis.com/compute/v1/projects/sandbox-hada/regions/asia-northeast1/subnetworks/kubernetes].
NAME        REGION           NETWORK                  RANGE          STACK_TYPE  IPV6_ACCESS_TYPE  IPV6_CIDR_RANGE  EXTERNAL_IPV6_CIDR_RANGE
kubernetes  asia-northeast1  kubernetes-the-hard-way  10.240.0.0/24  IPV4_ONLY
```
regionの指定を迫られた。多分01の操作をサボったせい。
まあ、hard-way通りに `us-west1` を指定するのも癪なので、居住地に合わせて `[3] asia-northeast1` を指定した。

## Firewall Rules
```
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/sandbox-hada/global/firewalls/kubernetes-the-hard-way-allow-internal].
Creating firewall...done.                                                                          
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW         DENY  DISABLED
kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp        False
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/sandbox-hada/global/firewalls/kubernetes-the-hard-way-allow-external].
Creating firewall...done.                                                                          
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False
```

## Kubernetes Public IP Address
```
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)
hmba hmbanoMacBook-Air.local ~ 
$ gcloud config get-value compute/region

(unset)
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute addresses create kubernetes-the-hard-way \                    
  --region $(gcloud config get-value compute/region)

(unset)
ERROR: (gcloud.compute.addresses.create) argument --region: expected one argument
Usage: gcloud compute addresses create [NAME ...] [optional flags]
  optional flags may be  --addresses | --description | --global | --help |
                         --ip-version | --network | --network-tier |
                         --prefix-length | --purpose | --region | --subnet

For detailed information on this command and its flags, run:
  gcloud compute addresses create --help
hmba hmbanoMacBook-Air.local ~ 
$ gcloud config set compute/region asia-northeast1
Updated property [compute/region].
hmba hmbanoMacBook-Air.local ~ 
$ gcloud config get-value compute/region                   
  
asia-northeast1
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)

Created [https://www.googleapis.com/compute/v1/projects/sandbox-hada/regions/asia-northeast1/addresses/kubernetes-the-hard-way].
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
NAME                     ADDRESS/RANGE   TYPE      PURPOSE  NETWORK  REGION           SUBNET  STATUS
kubernetes-the-hard-way  34.146.242.107  EXTERNAL                    asia-northeast1          RESERVED
```
設定完了が確認できた。

# Compute Instances
## Kubernetes Controllers
```
hmba hmbanoMacBook-Air.local ~ 
$   gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
For the following instance:
 - [controller-]
choose a zone:
 [1] asia-east1-a
 [2] asia-east1-b
 [3] asia-east1-c
 [4] asia-east2-a
 [5] asia-east2-b
 [6] asia-east2-c
 [7] asia-northeast1-a
 [8] asia-northeast1-b
 [9] asia-northeast1-c
 [10] asia-northeast2-a
 [11] asia-northeast2-b
 [12] asia-northeast2-c
 [13] asia-northeast3-a
 [14] asia-northeast3-b
 [15] asia-northeast3-c
 [16] asia-south1-a
 [17] asia-south1-b
 [18] asia-south1-c
 [19] asia-south2-a
 [20] asia-south2-b
 [21] asia-south2-c
 [22] asia-southeast1-a
 [23] asia-southeast1-b
 [24] asia-southeast1-c
 [25] asia-southeast2-a
 [26] asia-southeast2-b
 [27] asia-southeast2-c
 [28] australia-southeast1-a
 [29] australia-southeast1-b
 [30] australia-southeast1-c
 [31] australia-southeast2-a
 [32] australia-southeast2-b
 [33] australia-southeast2-c
 [34] europe-central2-a
 [35] europe-central2-b
 [36] europe-central2-c
 [37] europe-north1-a
 [38] europe-north1-b
 [39] europe-north1-c
 [40] europe-west1-b
 [41] europe-west1-c
 [42] europe-west1-d
 [43] europe-west2-a
 [44] europe-west2-b
 [45] europe-west2-c
 [46] europe-west3-a
 [47] europe-west3-b
 [48] europe-west3-c
 [49] europe-west4-a
 [50] europe-west4-b
Did not print [38] options.
Too many options [88]. Enter "list" at prompt to print choices fully.
Please enter your numeric choice:  ^C

Command killed by keyboard interrupt


hmba hmbanoMacBook-Air.local ~ 
$   gcloud compute instances create controller-0 \   
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --private-network-ip 10.240.0.10 \    
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
For the following instance:
 - [controller-0]
choose a zone:
 [1] asia-east1-a
 [2] asia-east1-b
 [3] asia-east1-c
 [4] asia-east2-a
 [5] asia-east2-b
 [6] asia-east2-c
 [7] asia-northeast1-a
 [8] asia-northeast1-b
 [9] asia-northeast1-c
 [10] asia-northeast2-a
 [11] asia-northeast2-b
 [12] asia-northeast2-c
 [13] asia-northeast3-a
 [14] asia-northeast3-b
 [15] asia-northeast3-c
 [16] asia-south1-a
 [17] asia-south1-b
 [18] asia-south1-c
 [19] asia-south2-a
 [20] asia-south2-b
 [21] asia-south2-c
 [22] asia-southeast1-a
 [23] asia-southeast1-b
 [24] asia-southeast1-c
 [25] asia-southeast2-a
 [26] asia-southeast2-b
 [27] asia-southeast2-c
 [28] australia-southeast1-a
 [29] australia-southeast1-b
 [30] australia-southeast1-c
 [31] australia-southeast2-a
 [32] australia-southeast2-b
 [33] australia-southeast2-c
 [34] europe-central2-a
 [35] europe-central2-b
 [36] europe-central2-c
 [37] europe-north1-a
 [38] europe-north1-b
 [39] europe-north1-c
 [40] europe-west1-b
 [41] europe-west1-c
 [42] europe-west1-d
 [43] europe-west2-a
 [44] europe-west2-b
 [45] europe-west2-c
 [46] europe-west3-a
 [47] europe-west3-b
 [48] europe-west3-c
 [49] europe-west4-a
 [50] europe-west4-b
Did not print [38] options.
Too many options [88]. Enter "list" at prompt to print choices fully.
Please enter your numeric choice:  7

NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [controller-0]: https://www.googleapis.com/compute/v1/projects/sandbox-hada/zones/asia-northeast1-a/operations/operation-1644042491895-5d73f78b99dcc-26991029-9b4bab1e
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
```

```
hmba hmbanoMacBook-Air.local ~ 
$ for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done
For the following instance:
 - [worker-0]
choose a zone:
 [1] asia-east1-a
 [2] asia-east1-b
 [3] asia-east1-c
 [4] asia-east2-a
 [5] asia-east2-b
 [6] asia-east2-c
 [7] asia-northeast1-a
 [8] asia-northeast1-b
 [9] asia-northeast1-c
 [10] asia-northeast2-a
 [11] asia-northeast2-b
 [12] asia-northeast2-c
 [13] asia-northeast3-a
 [14] asia-northeast3-b
 [15] asia-northeast3-c
 [16] asia-south1-a
 [17] asia-south1-b
 [18] asia-south1-c
 [19] asia-south2-a
 [20] asia-south2-b
 [21] asia-south2-c
 [22] asia-southeast1-a
 [23] asia-southeast1-b
 [24] asia-southeast1-c
 [25] asia-southeast2-a
 [26] asia-southeast2-b
 [27] asia-southeast2-c
 [28] australia-southeast1-a
 [29] australia-southeast1-b
 [30] australia-southeast1-c
 [31] australia-southeast2-a
 [32] australia-southeast2-b
 [33] australia-southeast2-c
 [34] europe-central2-a
 [35] europe-central2-b
 [36] europe-central2-c
 [37] europe-north1-a
 [38] europe-north1-b
 [39] europe-north1-c
 [40] europe-west1-b
 [41] europe-west1-c
 [42] europe-west1-d
 [43] europe-west2-a
 [44] europe-west2-b
 [45] europe-west2-c
 [46] europe-west3-a
 [47] europe-west3-b
 [48] europe-west3-c
 [49] europe-west4-a
 [50] europe-west4-b
Did not print [38] options.
Too many options [88]. Enter "list" at prompt to print choices fully.
Please enter your numeric choice:  7

NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [worker-0]: https://www.googleapis.com/compute/v1/projects/sandbox-hada/zones/asia-northeast1-a/operations/operation-1644282078461-5d77741327e03-63dd04b8-d7510377
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
For the following instance:
 - [worker-1]
choose a zone:
 [1] asia-east1-a
 [2] asia-east1-b
 [3] asia-east1-c
 [4] asia-east2-a
 [5] asia-east2-b
 [6] asia-east2-c
 [7] asia-northeast1-a
 [8] asia-northeast1-b
 [9] asia-northeast1-c
 [10] asia-northeast2-a
 [11] asia-northeast2-b
 [12] asia-northeast2-c
 [13] asia-northeast3-a
 [14] asia-northeast3-b
 [15] asia-northeast3-c
 [16] asia-south1-a
 [17] asia-south1-b
 [18] asia-south1-c
 [19] asia-south2-a
 [20] asia-south2-b
 [21] asia-south2-c
 [22] asia-southeast1-a
 [23] asia-southeast1-b
 [24] asia-southeast1-c
 [25] asia-southeast2-a
 [26] asia-southeast2-b
 [27] asia-southeast2-c
 [28] australia-southeast1-a
 [29] australia-southeast1-b
 [30] australia-southeast1-c
 [31] australia-southeast2-a
 [32] australia-southeast2-b
 [33] australia-southeast2-c
 [34] europe-central2-a
 [35] europe-central2-b
 [36] europe-central2-c
 [37] europe-north1-a
 [38] europe-north1-b
 [39] europe-north1-c
 [40] europe-west1-b
 [41] europe-west1-c
 [42] europe-west1-d
 [43] europe-west2-a
 [44] europe-west2-b
 [45] europe-west2-c
 [46] europe-west3-a
 [47] europe-west3-b
 [48] europe-west3-c
 [49] europe-west4-a
 [50] europe-west4-b
Did not print [38] options.
Too many options [88]. Enter "list" at prompt to print choices fully.
Please enter your numeric choice:  7

NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [worker-1]: https://www.googleapis.com/compute/v1/projects/sandbox-hada/zones/asia-northeast1-a/operations/operation-1644282090151-5d77741e4e03c-7157d85f-fd5fa5cf
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
For the following instance:
 - [worker-2]
choose a zone:
 [1] asia-east1-a
 [2] asia-east1-b
 [3] asia-east1-c
 [4] asia-east2-a
 [5] asia-east2-b
 [6] asia-east2-c
 [7] asia-northeast1-a
 [8] asia-northeast1-b
 [9] asia-northeast1-c
 [10] asia-northeast2-a
 [11] asia-northeast2-b
 [12] asia-northeast2-c
 [13] asia-northeast3-a
 [14] asia-northeast3-b
 [15] asia-northeast3-c
 [16] asia-south1-a
 [17] asia-south1-b
 [18] asia-south1-c
 [19] asia-south2-a
 [20] asia-south2-b
 [21] asia-south2-c
 [22] asia-southeast1-a
 [23] asia-southeast1-b
 [24] asia-southeast1-c
 [25] asia-southeast2-a
 [26] asia-southeast2-b
 [27] asia-southeast2-c
 [28] australia-southeast1-a
 [29] australia-southeast1-b
 [30] australia-southeast1-c
 [31] australia-southeast2-a
 [32] australia-southeast2-b
 [33] australia-southeast2-c
 [34] europe-central2-a
 [35] europe-central2-b
 [36] europe-central2-c
 [37] europe-north1-a
 [38] europe-north1-b
 [39] europe-north1-c
 [40] europe-west1-b
 [41] europe-west1-c
 [42] europe-west1-d
 [43] europe-west2-a
 [44] europe-west2-b
 [45] europe-west2-c
 [46] europe-west3-a
 [47] europe-west3-b
 [48] europe-west3-c
 [49] europe-west4-a
 [50] europe-west4-b
Did not print [38] options.
Too many options [88]. Enter "list" at prompt to print choices fully.
Please enter your numeric choice:  7

NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [worker-2]: https://www.googleapis.com/compute/v1/projects/sandbox-hada/zones/asia-northeast1-a/operations/operation-1644282094542-5d7774227df58-fd5e54f1-c5e8f823
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute instances list --filter="tags.items=kubernetes-the-hard-way"
NAME          ZONE               MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
controller-0  asia-northeast1-a  e2-standard-2               10.240.0.10  34.85.61.245   RUNNING
worker-0      asia-northeast1-a  e2-standard-2               10.240.0.20  34.85.43.220   RUNNING
worker-1      asia-northeast1-a  e2-standard-2               10.240.0.21  35.200.94.101  RUNNING
worker-2      asia-northeast1-a  e2-standard-2               10.240.0.22  34.146.44.245  RUNNING
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute instances --help
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute instances create --help
hmba hmbanoMacBook-Air.local ~ 
$ gcloud compute ssh controller-0
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/hmba/.ssh/google_compute_engine
Your public key has been saved in /Users/hmba/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:295cVrEOv522zvF7+wMNRwT5u66Cq9dJyUMPVvfUAXo hmba@hmbanoAir
The key's randomart image is:
+---[RSA 3072]----+
|             o=oo|
|            .o oo|
|           ..E+o.|
|           +.. o+|
|        S + +.+.o|
|         o = o++ |
|        . = o ++.|
|         + * o.+B|
|       .o.o +.=X@|
+----[SHA256]-----+
No zone specified. Using zone [asia-northeast1-a] for instance: [controller-0].
Updating project ssh metadata...⠶Updated [https://www.googleapis.com/compute/v1/projects/sandbox-hada].
Updating project ssh metadata...done.                                                              
Waiting for SSH key to propagate.
Warning: Permanently added 'compute.7563844660264936979' (ED25519) to the list of known hosts.
Enter passphrase for key '/Users/hmba/.ssh/google_compute_engine': 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-1028-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Feb  8 01:05:26 UTC 2022

  System load:  0.16               Processes:             111
  Usage of /:   1.3% of 193.66GB   Users logged in:       0
  Memory usage: 4%                 IPv4 address for ens4: 10.240.0.10
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

5 updates can be applied immediately.
To see these additional updates run: apt list --upgradable


*** System restart required ***
Last login: Tue Feb  8 01:05:26 2022 from 153.229.201.97
hmba@controller-0:~$ exit
logout
Connection to 34.85.61.245 closed.

```

# 出てきたコマンドまとめ
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
gcloud config set project sandbox-hada                                     
gcloud compute networks subnets create kubernetes --network kubernetes-the-hard-way --range 10.240.0.0/24
```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```
internal communicationはすべてのプロコトルで許可

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
```
external communicationは、SSH(22), ICMP, and HTTPS(6443) を許可。
（普通HTTPSは443だが、ここでは6443を使用している？）

```
gcloud config get-value compute/region
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)
```
事前に `gcloud config set compute/region asia-northeast1` を実行していないと、
`$(gcloud config get-value compute/region)` の結果が帰ってこないため、エラーになるやつ

```
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```

```
gcloud compute instances create controller-0 \
  --async \
  --boot-disk-size 200GB \
  --can-ip-forward \
  --image-family ubuntu-2004-lts \
  --image-project ubuntu-os-cloud \
  --machine-type e2-standard-2 \
  --private-network-ip 10.240.0.1${i} \
  --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
  --subnet kubernetes \
  --tags kubernetes-the-hard-way,controller
```

```
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done
```

```
gcloud compute instances list --filter="tags.items=kubernetes-the-hard-way"
```

```
gcloud compute ssh controller-0
```
