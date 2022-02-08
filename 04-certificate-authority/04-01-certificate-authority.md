# Provisioning a CA and Generating TLS Certificates
## Certificate Authority

04_01_ca-config.json
```
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
```

8760h = 365day * 24h/day

 encipher 【他動】 〔文書などを〕暗号化する◆【反】decipher

04_01_ca-csr.json
```
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
```

↓cfsslコマンドを実行
```
hmba hmbanoMacBook-Air.local ~/git/training-kubernetes-hard-way (git:refs/heads/04)
$ cfssl gencert -initca 04_01_ca-csr.json | cfssljson -bare ca
2022/02/08 11:36:04 [INFO] generating a new CA key and certificate from CSR
2022/02/08 11:36:04 [INFO] generate received request
2022/02/08 11:36:04 [INFO] received CSR
2022/02/08 11:36:04 [INFO] generating key: rsa-2048
2022/02/08 11:36:04 [INFO] encoded CSR
2022/02/08 11:36:04 [INFO] signed certificate with serial number 299419263342644621883311486216870741322907574111
```

# 使ったコマンド
```
cfssl gencert -initca 04_01_ca-csr.json
cfssljson -bare ca
```
> cfssl gencert -- generate a new key and signed certificate
> Usage of gencert:
>     Generate a new key and cert from CSR:
>         cfssl gencert -initca CSRJSON

> Usage of cfssljson:
>   -bare
>         the response from CFSSL is not wrapped in the API standard response


