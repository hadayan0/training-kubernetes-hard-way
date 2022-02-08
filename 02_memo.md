02までの下準備。
01の `gcloud config set compute/region` などの設定は行わずに突入した。
多分03で設定を迫られているのはそれのせい。
```
hmba hmbanoMacBook-Air.local ~ 
$ curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
hmba hmbanoMacBook-Air.local ~ 
$ curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 15.8M  100 15.8M    0     0  2632k      0  0:00:06  0:00:06 --:--:-- 3397k
hmba hmbanoMacBook-Air.local ~ 
$ curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  9.7M  100  9.7M    0     0  3268k      0  0:00:03  0:00:03 --:--:-- 3280k
hmba hmbanoMacBook-Air.local ~ 
$ chmod +x cfssl cfssljson 
hmba hmbanoMacBook-Air.local ~ 
$ sudo mv cfssl cfssljson /usr/local/bin/
hmba hmbanoMacBook-Air.local ~ 
$ cfssl version
Version: 1.4.1
Runtime: go1.12.12
hmba hmbanoMacBook-Air.local ~ 
$ cfssljson --version
Version: 1.4.1
Runtime: go1.12.12
hmba hmbanoMacBook-Air.local ~ 
$ k version --client
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.3", GitCommit:"ca643a4d1f7bfe34773c74f79527be4afd95bf39", GitTreeState:"clean", BuildDate:"2021-07-15T21:04:39Z", GoVersion:"go1.16.6", Compiler:"gc", Platform:"darwin/arm64"}
```

