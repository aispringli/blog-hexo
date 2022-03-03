---
title: kubernetes使用cert-manager自动签发letsencrypt证书
comments: true
mp3: /music/blog.mp3
tags:
  - k8s
categories:
  - k8s
keywords:
  - k8s
  - cert manager
  - Let's Encrypt
date: 2021-09-16 13:54:05
updated: 2021-09-16 13:54:05
---


## 参考内容
+ [Let's Encrypt](https://letsencrypt.org/)
+ [cert-manager](https://cert-manager.io/docs/)
+ [jetstack/cert-manager](https://github.com/jetstack/cert-manager)

## 部署 cert-manager

1. 部署 crds
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.crds.yaml
```

2. 创建 Namespace
```
kubectl create namespace cert-manager
```

3. 部署 cert-manager
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```
4. 创建私钥

生成签名密钥对
```
# Generate a CA private key
$ openssl genrsa -out ca.key 2048

# Create a self signed Certificate, valid for 10yrs with the 'signing' option set
$ openssl req -x509 -new -nodes -key ca.key -subj "/CN=${COMMON_NAME}" -days 3650 -reqexts v3_req -extensions v3_ca -out ca.crt
```

将签名密钥对保存为Secret
```
kubectl create secret tls ca-key-pair \
   --cert=ca.crt \
   --key=ca.key \
   --namespace=default
```




4. 创建ClusterIssuer  
我们需要先创建一个签发机构，cert-manager 给我们提供了 Issuer 和ClusterIssuer 这两种用于创建签发机构的自定义资源对象，Issuer 只能用来签发自己所在 namespace 下的证书，ClusterIssuer 可以签发任意 namespace 下的证书，这里以 ClusterIssuer 为例
```
kubectl apply -f cluster-issuer.yaml
```
```
cluster-issuer.yaml
piVersion: v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod #这里是issuer的名称，后面要使用
spec:
  acme:
    # 邮箱，证书过期前会发邮件到这个邮箱
    email: admin@aispring.cloud
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: issuer-key
    solvers:
    - http01:
        ingress:
          class: nginx
```
## 签发证书
Let’s Encrypt常见生成证书的方式是cerbot,可以使用http 01和dns 01两种方式认证方式，http 01不支持生成通配符证书，这里需要生成通配符域名，所以才用dns 01认证方式。

若dns服务商是阿里云，官方暂不支持，有开源的AliDNS-Webhook供我们使用。

1. 部署 AliDNS-Webhook
```
kubectl apply -f https://raw.githubusercontent.com/pragkent/alidns-webhook/master/deploy/bundle.yaml

```

2. 创建阿里云访问凭据
```
apiVersion: v1
kind: Secret
metadata:
  name: alidns-secret
  namespace: cert-manager
data:
  access-key: YOUR_ACCESS_KEY
  secret-key: YOUR_SECRET_KEY
```

3. 创建 ClusterIssuer 发行证书
```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # Change to your letsencrypt email
    email: certmaster@example.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-staging-account-key
    solvers:
    - dns01:
        webhook:
          groupName: acme.yourcompany.com
          solverName: alidns
          config:
            region: ""
            accessKeySecretRef:
              name: alidns-secret
              key: access-key
            secretKeySecretRef:
              name: alidns-secret
              key: secret-key
```

4. 签发证书
```
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: example-tls
spec:
  secretName: example-com-tls
  commonName: example.com
  dnsNames:
  - example.com
  - "*.example.com"
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

5. 测试证书
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod #需要使用这个标记，letsencrypt-prod是上面issuer的名称
  name: nginx
  namespace: default
spec:
  rules:
  - host: dev.arfront.cn
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        pathType: ImplementationSpecific
  tls:
  - hosts:
    - dev.arfront.cn 
    secretName: dev.arfront.cn #证书的域名

```