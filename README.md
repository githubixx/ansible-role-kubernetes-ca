ansible-role-kubernetes-ca
==========================

This role is used in [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/). It basically creates two CA's (one for etcd and one for Kubernetes API server) and the certificates needed to secure communication of the Kubernetes components. Besides the Kubernetes API server none of the Kubernetes components should communicate with the etcd cluster directly. For more information see [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org) (well kind of). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `r1.0.0_v1.8.0` means this is release 1.0.0 of this role and it's meant to be used with Kubernetes version 1.8.0. If the role itself changes `rX.Y.Z` will increase. If the Kubernetes version changes `vX.Y.Z` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release.

**r6.0.0_v1.10.4**

- support Ubuntu 18.04

- Remove PeerVPN dependency when generating certificates (use `k8s_interface` variable instead of `peervpn_conf_interface`). When generating certificates previously the values of `peervpn_conf_interface` variables were used and added to the certificate. Now instead `k8s_interface` variable is used. For almost all people that change shouldn't have any effect because the values of both variables should have been the same. If not check if the generated certificates are ok for you. The reason for this change is to allow the usage of a different VPN solution like [WireGuard](https://github.com/githubixx/ansible-role-wireguard) e.g.

- always gather facts as first task

- update README

**r5.0.0_v1.10.4**

- Implemented changes needed for Kubernetes v1.10.x.
- Renamed certificate files `cert-kube-proxy*` -> `cert-k8s-proxy*` to be in pair with the other certificate file names
- Added `k8s_controller_manager_csr_*` variables for kube-controller-manager client certificate
- Added `k8s_scheduler_csr_*` variables for kube-scheduler client certificate

**r4.0.1_v1.9.8**

- No changes. Just added Git tag for Kubernetes v1.9.8 to be in pair with controller and worker roles.

**r4.0.1_v1.9.3**

- No changes. Just added Git tag for Kubernetes v1.9.3 to be in pair with controller and worker roles.

**r4.0.1_v1.8.4**

- Changed default of `k8s_ca_conf_directory` to `{{ '~/k8s/certs' | expanduser }}`. By default this will expand to user's LOCAL $HOME (the user that run's `ansible-playbook ...` plus `/k8s/certs`. That means if the user's `$HOME` directory is e.g. `/home/da_user` then `k8s_ca_conf_directory` will have a value of `/home/da_user/k8s/certs`. As the user normally has write access to his `$HOME` directory we don't rely on the parent directory permission if we deploy the role without root permissions. If you defined this variable with a different value before this change then you don't need to bother about this change.

**r3.0.0_v1.8.4**

- include worker node names/ips into CSR (certificate signing request) for kube-apiserver certificate

**r1.0.1_v1.8.0**

- renamed `cfssl_*` variables to `k8s_ca_*`
- change defaults for key algos and sizes to match settings in https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md
- added admin client certificate
- added kubelet client certificates used in Kuberetes 1.7/1.8
- added kube-proxy client certificate used in Kubernetes 1.7/1.8
- Hostname,FQDN,internal IP and PeerVPN IP of all controller hosts are added automatically to Kubernetes API server certificate now
- Hostname,FQDN,internal IP and PeerVPN IP for every worker host certificate is added automatically to the worker certificate

**r1.0.0_v1.6.0**

- Initial Ansible role.

Requirements
------------

This playbook needs [CFSSL](https://github.com/cloudflare/cfssl) PKI toolkit binaries installed. You can use [ansible-role-cfssl](https://github.com/githubixx/ansible-role-cfssl) to install CFSSL locally on your machine. If you want to store the generated certificates and CA's locally or on a network share specify the role variables below in `host_vars/localhost` or in `group_vars/all`.

Role Variables
--------------

This playbook has quite a few variables. But that's mainly information needed for the certificates.

```
k8s_ca_conf_directory: "{{ '~/k8s/certs' | expanduser }}"
k8s_ca_certificate_owner: "root"
k8s_ca_certificate_group: "root"
```

`k8s_ca_conf_directory` tells Ansible where to store the CA's and certificate files. To enable Ansible to read the files in later runs you should specify a user and group in `k8s_ca_certificate_owner` / `k8s_ca_certificate_group` which has permissions (in most cases this will be the user you use on your workstation).

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
CSR parameter for kube-controller-manager service account key pair. The kube-controller-manager leverages a key pair to generate and sign service account tokens as described in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

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
  - kubernetes.default.svc.cluster.local
```
Here the same is basically true as with `etcd_cert_hosts` but we also include the Kubernetes service IP `10.32.0.1` (which you will get BTW if you execute `nslookup kubernetes` later in one of the pods). We also include 127.0.0.1 (localhost) and we include some default Kubernetes hostname's that are available by default if KubeDNS is deployed.

In general I need to do more testing to be sure if the values in `etcd_cert_hosts` and `k8s_apiserver_cert_hosts` are really needed. But while developing this Kubernetes roles it was annoying if you get error's because the certificates are not correct. And it takes a while to replace them. So I'll keep it here for now until I know better ;-)

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
