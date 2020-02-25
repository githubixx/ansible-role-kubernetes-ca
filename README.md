ansible-role-kubernetes-ca
==========================

This role is used in [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/). It basically creates two CA's (one for etcd and one for Kubernetes API server) and the certificates needed to secure communication of the Kubernetes components. Besides the Kubernetes API server none of the Kubernetes components should communicate with the etcd cluster directly. For more information see [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `7.0.0+1.12.3` means this is release 7.0.0 of this role and it's meant to be used with Kubernetes version `1.12.3` (while normally it should work with basically any version >= 1.8.0 but I tested it with the version tagged). If the role itself changes `X.Y.Z` before `+` will increase. If the Kubernetes version changes `X.Y.Z` after `+` will increase and also the role patch version will increase (e.g. from `7.0.0` to `7.0.1`). This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release.

Changelog
---------

see [CHANGELOG.md](https://github.com/githubixx/ansible-role-kubernetes-ca/blob/master/CHANGELOG.md)

Requirements
------------

This playbook needs [CFSSL](https://github.com/cloudflare/cfssl) PKI toolkit binaries installed. You can use [ansible-role-cfssl](https://github.com/githubixx/ansible-role-cfssl) to install CFSSL locally on your machine. If you want to store the generated certificates and CA's locally or on a network share specify the role variables below in `host_vars/localhost` or in `group_vars/all` e.g.

Role Variables
--------------

This playbook has quite a few variables. But that's mainly information needed for the certificates.

```
# The directory where to store the certificates. By default this
# will expand to user's LOCAL $HOME (the user that run's "ansible-playbook ..."
# plus "/k8s/certs". That means if the user's $HOME directory is e.g.
# "/home/da_user" then "k8s_ca_conf_directory" will have a value of
# "/home/da_user/k8s/certs".
k8s_ca_conf_directory: "{{ '~/k8s/certs' | expanduser }}"
# Directory permissions for directory specified in "k8s_ca_conf_directory"
k8s_ca_conf_directory_perm: "0770"
# File permissions for certificates, csr, and so on
k8s_ca_file_perm: "0660"
# Owner of the certificate files
k8s_ca_certificate_owner: "root"
# Group to which the certificate files belongs to
k8s_ca_certificate_group: "root"

# Specifies Ansible's hosts group which contains all K8s controller
# nodes (as specified in Ansible's "hosts" file).
k8s_ca_controller_nodes_group: "k8s_controller"
# As above but for the K8s etcd nodes.
k8s_ca_etcd_nodes_group: "k8s_etcd"
# As above but for the K8s worker nodes.
k8s_ca_worker_nodes_group: "k8s_worker"
```

`k8s_ca_conf_directory` tells Ansible where to store the CA's and certificate files. To enable Ansible to read the files in later runs you should specify a user and a group in `k8s_ca_certificate_owner` / `k8s_ca_certificate_group` which has permissions (in most cases this will be the user you use on your workstation).

```
ca_etcd_expiry: "87600h"
```
`ca_etcd_expiry` sets the expiry date for etcd root CA.

```
ca_etcd_csr_cn: "Etcd"
ca_etcd_csr_key_algo: "rsa"
ca_etcd_csr_key_size: "2048"
ca_etcd_csr_names_c: "DE"
ca_etcd_csr_names_l: "The_Internet"
ca_etcd_csr_names_o: "Kubernetes"
ca_etcd_csr_names_ou: "BY"
ca_etcd_csr_names_st: "Bayern"
```
This are used to create the CSR (certificate signing request) of the CA (certificate authority) which we use to sign certifcates for etcd.

```
ca_k8s_apiserver_expiry: "87600h"
```
`ca_k8s_apiserver_expiry` sets the expiry date for Kubernetes API server root CA.

```
ca_k8s_apiserver_csr_cn: "Kubernetes"
ca_k8s_apiserver_csr_key_algo: "rsa"
ca_k8s_apiserver_csr_key_size: "2048"
ca_k8s_apiserver_csr_names_c: "DE"
ca_k8s_apiserver_csr_names_l: "The_Internet"
ca_k8s_apiserver_csr_names_o: "Kubernetes"
ca_k8s_apiserver_csr_names_ou: "BY"
ca_k8s_apiserver_csr_names_st: "Bayern"
```
This are used to create the CSR (certificate signing request) of the CA (certificate authority) which we use to sign certifcates for the Kubernetes API server.

```
etcd_csr_cn: "Etcd"
etcd_csr_key_algo: "rsa"
etcd_csr_key_size: "2048"
etcd_csr_names_c: "DE"
etcd_csr_names_l: "The_Internet"
etcd_csr_names_o: "Kubernetes"
etcd_csr_names_ou: "BY"
etcd_csr_names_st: "Bayern"
```
This parameters are used to create the CSR for the certificate that is used to secure the etcd communication.

```
k8s_apiserver_csr_cn: "Kubernetes"
k8s_apiserver_csr_key_algo: "rsa"
k8s_apiserver_csr_key_size: "2048"
k8s_apiserver_csr_names_c: "DE"
k8s_apiserver_csr_names_l: "The_Internet"
k8s_apiserver_csr_names_o: "Kubernetes"
k8s_apiserver_csr_names_ou: "BY"
k8s_apiserver_csr_names_st: "Bayern"
```
This parameter are used to create the CSR for the certificate that is used to secure the Kubernetes API server communication.

```
k8s_admin_csr_cn: "admin"
k8s_admin_csr_key_algo: "rsa"
k8s_admin_csr_key_size: "2048"
k8s_admin_csr_names_c: "DE"
k8s_admin_csr_names_l: "The_Internet"
k8s_admin_csr_names_o: "system:masters" # DO NOT CHANGE!
k8s_admin_csr_names_ou: "BY"
k8s_admin_csr_names_st: "Bayern"
```
This variables are used to create the CSR for the certificate that is used for the admin user (and which `kubectl` will use).

```
k8s_worker_csr_key_algo: "rsa"
k8s_worker_csr_key_size: "2048"
k8s_worker_csr_names_c: "DE"
k8s_worker_csr_names_l: "The_Internet"
k8s_worker_csr_names_o: "system:nodes" # DO NOT CHANGE!
k8s_worker_csr_names_ou: "BY"
k8s_worker_csr_names_st: "Bayern"
```
CSR parameter for kubelet client certificates.

```
k8s_controller_manager_csr_key_algo: "rsa"
k8s_controller_manager_csr_key_size: "2048"
k8s_controller_manager_csr_names_c: "DE"
k8s_controller_manager_csr_names_l: "The_Internet"
k8s_controller_manager_csr_names_o: "system:kube-controller-manager" # DO NOT CHANGE!
k8s_controller_manager_csr_names_ou: "BY"
k8s_controller_manager_csr_names_st: "Bayern"
```
This variables are needed to generate the CSR for the `kube-controller-manager` client certificate.

```
k8s_scheduler_csr_key_algo: "rsa"
k8s_scheduler_csr_key_size: "2048"
k8s_scheduler_csr_names_c: "DE"
k8s_scheduler_csr_names_l: "The_Internet"
k8s_scheduler_csr_names_o: "system:kube-scheduler" # DO NOT CHANGE!
k8s_scheduler_csr_names_ou: "BY"
k8s_scheduler_csr_names_st: "Bayern"
```
This variables are needed to generate the CSR for the `kube-scheduler` client certificate.

```
k8s_controller_manager_sa_csr_cn: "service-accounts"
k8s_controller_manager_sa_csr_key_algo: "rsa"
k8s_controller_manager_sa_csr_key_size: "2048"
k8s_controller_manager_sa_csr_names_c: "DE"
k8s_controller_manager_sa_csr_names_l: "The_Internet"
k8s_controller_manager_sa_csr_names_o: "Kubernetes"
k8s_controller_manager_sa_csr_names_ou: "BY"
k8s_controller_manager_sa_csr_names_st: "Bayern"
```
CSR parameter for kube-controller-manager service account key pair. The kube-controller-manager leverages a key pair to generate and sign service account tokens as described in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation. Hint: Think twice if you want to change this key pair for a K8s cluster that has already pods deployed. The private key will be used to sign generated service account tokens. The public key will be used to verify the tokens during authentication. So if you have pods running e.g. with the `default` service account and you roll out a new key pair the [Token Controller](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#token-controller) which is part of the controller manager won't be able to verify the already existing service accounts anymore. So this might cause trouble for your running pods.

```
k8s_kube_proxy_csr_cn: "system:kube-proxy" # DO NOT CHANGE!
k8s_kube_proxy_csr_key_algo: "rsa"
k8s_kube_proxy_csr_key_size: "2048"
k8s_kube_proxy_csr_names_c: "DE"
k8s_kube_proxy_csr_names_l: "The_Internet"
k8s_kube_proxy_csr_names_o: "system:node-proxier" # DO NOT CHANGE!
k8s_kube_proxy_csr_names_ou: "BY"
k8s_kube_proxy_csr_names_st: "Bayern"
```
CSR parameter for the kube-proxy client certificate.

```
etcd_cert_hosts:
  - 127.0.0.1
  - etcd0
  - etcd1
  - etcd2
```
Here you can add additional etcd hosts that should be included in the certificate. In general the role will include all IP's, hostnames and FQDN that are needed.

```
k8s_apiserver_cert_hosts:
  - 127.0.0.1
  - 10.32.0.1
  - kubernetes
  - kubernetes.default
  - kubernetes.default.svc
  - kubernetes.default.svc.cluster
  - kubernetes.default.svc.cluster.local
```
Here the same is basically true as with `etcd_cert_hosts` but we also include the Kubernetes service IP `10.32.0.1` (which you will get btw if you execute `nslookup kubernetes` later in one of the pods). We also include 127.0.0.1 (localhost) and we include some default Kubernetes hostname's that are available by default if KubeDNS/CoreDNS is deployed.

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
