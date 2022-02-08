# Provisioning a CA and Generating TLS Certificates
## Client and Server Certificates

04-02-admin-csr.json
```
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
```
diff:
- CNが `Kubernetes` じゃなくて `admin`
- names[0].Oが `Kubernetes` じゃなくて `system:masters`
- names[0].OUが `CA` じゃなくて `Kubernetes The Hard Way`

https://www.cybertrust.co.jp/sureserver/support/csr.html
- CNはコモンネーム（URLのFQDN）
- Oは組織名（Organization?）
- OUは組織単位名（部署名 Organization Unit?）

↓コマンド
```
hmba hmbanoMacBook-Air.local ~/git/training-kubernetes-hard-way/04-certificate-authority (git:refs/heads/04)
$ cfssl gencert \                             
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=04-01-ca-config.json \
  -profile=kubernetes \
  04-02-admin-csr.json | cfssljson -bare admin
2022/02/08 12:23:45 [INFO] generate received request
2022/02/08 12:23:45 [INFO] received CSR
2022/02/08 12:23:45 [INFO] generating key: rsa-2048
2022/02/08 12:23:45 [INFO] encoded CSR
2022/02/08 12:23:45 [INFO] signed certificate with serial number 336067184612571158889045048154923494369187183172
2022/02/08 12:23:45 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

# 使ったコマンド

```
$ cfssl gencert \                             
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=04-01-ca-config.json \
  -profile=kubernetes \
  04-02-admin-csr.json
```

> cfssl gencert -- generate a new key and signed certificate
> Usage of gencert:
>     Generate a new key and cert from CSR:
>         cfssl gencert -ca cert -ca-key key [-config config] [-profile profile] [-hostname hostname] CSRJSON
> 

```
cfssljson -bare admin
```
割愛
