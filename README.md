ansible-role-kubernetes-ca
==========================

This playbook is used in [Kubernetes the not so hard way with Ansible (at scaleway) - Part 4 - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-4/). It basically creates two CA's (one for etcd and one for Kubernetes API server) and the certificates needed to secure communication of the Kubernetes components. Besides the Kubernetes API server none of the Kubernetes components should communicate with the etcd cluster directly. For more information see [Kubernetes the not so hard way with Ansible (at scaleway) - Part 4 - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-4/).

Requirements
------------

This playbook needs [CFSSL](https://github.com/cloudflare/cfssl) PKI toolkit binaries installed. You can use [ansible-role-cfssl](https://github.com/githubixx/ansible-role-cfssl) to install CFSSL locally on your machine. If you want to store the generated certificates and CA's locally or on a network share specify the role variables below in `host_vars/localhost`.

Role Variables
--------------

This playbook has quite a few variables. But that's mainly information needed for the certificates.

```
cfssl_conf_directory: /etc/cfssl
cfssl_certificate_owner: root
cfssl_certificate_group: root
```

`cfssl_conf_directory` tells Ansible where to store the CA's and certificate files. To enable Ansible to read the files in later runs you should specify a user and group in `cfssl_certificate_owner` / `cfssl_certificate_group` which has permissions (in most cases this will be the user you use on your workstation).

```
ca_etcd_expiry: 87600h
```
`cfssl_certificate_group` sets the expiry date for etcd root CA.

```
ca_etcd_csr_cn: Etcd
ca_etcd_csr_key_algo: ecdsa
ca_etcd_csr_key_size: 384
ca_etcd_csr_names_c: DE
ca_etcd_csr_names_l: The_Internet
ca_etcd_csr_names_o: Kubernetes
ca_etcd_csr_names_ou: BY
ca_etcd_csr_names_st: Bayern
```
This are used to create the CSR (certificate signing request) of the CA (certificate authority) which we use to sign certifcates for etcd.

```
ca_k8s_apiserver_expiry: 87600h
```
`ca_k8s_apiserver_expiry` sets the expiry date for Kubernetes API server root CA.

```
ca_k8s_apiserver_csr_cn: Kubernetes
ca_k8s_apiserver_csr_key_algo: ecdsa
ca_k8s_apiserver_csr_key_size: 384
ca_k8s_apiserver_csr_names_c: DE
ca_k8s_apiserver_csr_names_l: The_Internet
ca_k8s_apiserver_csr_names_o: Kubernetes
ca_k8s_apiserver_csr_names_ou: BY
ca_k8s_apiserver_csr_names_st: Bayern
```
This are used to create the CSR (certificate signing request) of the CA (certificate authority) which we use to sign certifcates for the Kubernetes API server.

```
etcd_csr_cn: Etcd
etcd_csr_key_algo: ecdsa
etcd_csr_key_size: 384
etcd_csr_names_c: DE
etcd_csr_names_l: The_Internet
etcd_csr_names_o: Kubernetes
etcd_csr_names_ou: BY
etcd_csr_names_st: Bayern
```
This parameters are used to create the CSR for the certificate that is used to secure the etcd communication.

```
k8s_apiserver_csr_cn: Kubernetes
k8s_apiserver_csr_key_algo: ecdsa
k8s_apiserver_csr_key_size: 384
k8s_apiserver_csr_names_c: DE
k8s_apiserver_csr_names_l: The_Internet
k8s_apiserver_csr_names_o: Kubernetes
k8s_apiserver_csr_names_ou: BY
k8s_apiserver_csr_names_st: Bayern
```
This parameters are used to create the CSR for the certificate that is used to secure the Kubernetes API server communication.

```
etcd_cert_hosts:
  - 127.0.0.1
  - etcd0
  - etcd1
  - etcd2
```
Here you add all etcd hosts. I recommend to add the hostname, the fully qualified domain names (FQDN) and the IP addresses of your etcd hosts here. Also always add 127.0.0.1. If you plan to use 5 instead of 3 etcd hosts sometimes in the future add the hosts too even when you only use 3 hosts in the beginning. This will save you work later.

```
k8s_apiserver_cert_hosts:
  - 127.0.0.1
  - 10.32.0.1
  - controller0
  - controller1
  - controller2
  - worker0
  - worker1
  - worker2
  - kubernetes
  - kubernetes.default
  - kubernetes.default.svc
  - kubernetes.default.svc.cluster.local
```
Add all your Kubernetes controller, worker, 10.32.0.1 and 127.0.0.1 here. 10.32.0.1 is the internal virtual IP of the Kubernetes API. If you don't add it in this list KubeDNS later won't work. As with the etcd hosts I recommend to add the hostname, the fully qualified domain names (FQDN) and the IP address for every API server host. If you know that you will add more worker later add them here in advance to save you work later.

Example Playbook
----------------

```
- hosts: k8s_ca

  roles:
    - githubixx.kubernetes-ca
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
