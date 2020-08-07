ansible-role-kubernetes-ca
==========================

This role is used in [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/). It basically creates two CA's: One for etcd and one for Kubernetes components (needed to secure communication of the Kubernetes components). Besides the Kubernetes API server none of the Kubernetes components should have a need to communicate with the etcd cluster directly. For infrastructure components like [Cilium](https://cilium.io/) for K8s networking or [Traefik](https://traefik.io) for ingress it may make sense to reuse the already existing etcd cluster. For more information see [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `9.0.0+1.18.4` means this is release 9.0.0 of this role and it's meant to be used with Kubernetes version `1.18.4` (while normally it should work with basically any Kubernetes version >= 1.18.0 but I tested it with the version tagged). If the role itself changes `X.Y.Z` before `+` will increase. If the Kubernetes version changes `X.Y.Z` after `+` will increase and also the role patch version will increase (e.g. from `9.0.0` to `9.0.1`). This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release.

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

# Owner of the certificate files (you should probably change this)
k8s_ca_certificate_owner: "root"

# Group to which the certificate files belongs to (you should probably change this)
k8s_ca_certificate_group: "root"

# Specifies Ansible's hosts group which contains all K8s controller
# nodes (as specified in Ansible's "hosts" file).
k8s_ca_controller_nodes_group: "k8s_controller"

# As above but for the K8s etcd nodes.
k8s_ca_etcd_nodes_group: "k8s_etcd"

# As above but for the K8s worker nodes.
k8s_ca_worker_nodes_group: "k8s_worker"

# This role will include the IP address of the interface you specify here in
# the etcd, kube-apiserver and kubelet certificate SAN (subject alternative name).
# This is the interface where all the Kubernetes cluster services communicates
# and should be an encrypted network. Valid interface names are normally:
# "wg0" (WireGuard), "peervpn0" (PeerVPN) or "tap0".
k8s_interface: "tap0"

# Expiry for etcd root certificate
ca_etcd_expiry: "87600h"

# Certificate authority (CA) parameters for etcd certificates. This CA is used
# to sign certificates used by etcd (like peer and server certificates) and
# etcd clients (like "Kube API Server", "Traefik" and "Cilium" e.g.).
ca_etcd_csr_cn: "Etcd"
ca_etcd_csr_key_algo: "rsa"
ca_etcd_csr_key_size: "2048"
ca_etcd_csr_names_c: "DE"
ca_etcd_csr_names_l: "The_Internet"
ca_etcd_csr_names_o: "Kubernetes"
ca_etcd_csr_names_ou: "BY"
ca_etcd_csr_names_st: "Bayern"

# Expiry for Kubernetes API server root certificate
ca_k8s_apiserver_expiry: "87600h"

# Certificate authority (CA) parameters for Kubernetes API server. The CA is
# used to sign certifcates for various Kubernetes services like Kubernetes API
# server e.g.
ca_k8s_apiserver_csr_cn: "Kubernetes"
ca_k8s_apiserver_csr_key_algo: "rsa"
ca_k8s_apiserver_csr_key_size: "2048"
ca_k8s_apiserver_csr_names_c: "DE"
ca_k8s_apiserver_csr_names_l: "The_Internet"
ca_k8s_apiserver_csr_names_o: "Kubernetes"
ca_k8s_apiserver_csr_names_ou: "BY"
ca_k8s_apiserver_csr_names_st: "Bayern"

# CSR parameter for etcd server certificate. The server certificate is used by
# etcd server and verified by client for server identity (for example
# "Kubernetes API server").
# etcd parameter: --cert-file and --key-file
etcd_server_csr_cn: "Etcd"
etcd_server_csr_key_algo: "rsa"
etcd_server_csr_key_size: "2048"
etcd_server_csr_names_c: "DE"
etcd_server_csr_names_l: "The_Internet"
etcd_server_csr_names_o: "Kubernetes"
etcd_server_csr_names_ou: "BY"
etcd_server_csr_names_st: "Bayern"

# CSR parameter for etcd peer certificate. The peer certificate is used by etcd
# cluster members as they communicate with each other in both ways.
# etcd parameter: --peer-cert-file and --peer-key-file
etcd_peer_csr_cn: "Etcd"
etcd_peer_csr_key_algo: "rsa"
etcd_peer_csr_key_size: "2048"
etcd_peer_csr_names_c: "DE"
etcd_peer_csr_names_l: "The_Internet"
etcd_peer_csr_names_o: "Kubernetes"
etcd_peer_csr_names_ou: "BY"
etcd_peer_csr_names_st: "Bayern"

# CSR parameter for etcd clients. One such client is "kube-apiserver" e.g.
# and is defined in "etcd_additional_clients" variable (see below). All
# certificates issued for etcd clients will use this parameters.
etcd_client_csr_cn_prefix: "etcd"
etcd_client_csr_key_algo: "rsa"
etcd_client_csr_key_size: "2048"
etcd_client_csr_names_c: "DE"
etcd_client_csr_names_l: "The_Internet"
etcd_client_csr_names_o: "Kubernetes"
etcd_client_csr_names_ou: "BY"
etcd_client_csr_names_st: "Bayern"

# CSR parameter for Kubernetes API server certificate. Used to secure the
# Kubernetes API server communication.
k8s_apiserver_csr_cn: "Kubernetes"
k8s_apiserver_csr_key_algo: "rsa"
k8s_apiserver_csr_key_size: "2048"
k8s_apiserver_csr_names_c: "DE"
k8s_apiserver_csr_names_l: "The_Internet"
k8s_apiserver_csr_names_o: "Kubernetes"
k8s_apiserver_csr_names_ou: "BY"
k8s_apiserver_csr_names_st: "Bayern"

# CSR parameter for the admin user (used for the admin user and which "kubectl"
# will use).
k8s_admin_csr_cn: "admin"
k8s_admin_csr_key_algo: "rsa"
k8s_admin_csr_key_size: "2048"
k8s_admin_csr_names_c: "DE"
k8s_admin_csr_names_l: "The_Internet"
k8s_admin_csr_names_o: "system:masters" # DO NOT CHANGE!
k8s_admin_csr_names_ou: "BY"
k8s_admin_csr_names_st: "Bayern"

# CSR parameter for kubelet client certificates. The `kubelet` process
# (a.k.a. Kubernetes worker) also needs to authenticate itself against the
# API server. These variables are used to create the CSR file which in turn
# is used to create the `kubelet` certificate. Kubernetes uses a special-purpose
# authorization mode (https://kubernetes.io/docs/admin/authorization/node/)
# called "Node Authorizer", that specifically authorizes API requests made by
# kubelets (https://kubernetes.io/docs/concepts/overview/components/#kubelet).
# In order to be authorized by the "Node Authorizer", Kubelets must use a
# credential that identifies them as being in the `system:nodes` group,
# with a username of `system:node:<nodeName>`
k8s_worker_csr_key_algo: "rsa"
k8s_worker_csr_key_size: "2048"
k8s_worker_csr_names_c: "DE"
k8s_worker_csr_names_l: "The_Internet"
k8s_worker_csr_names_o: "system:nodes" # DO NOT CHANGE!
k8s_worker_csr_names_ou: "BY"
k8s_worker_csr_names_st: "Bayern"

# CSR parameter for the kube-proxy client certificate
k8s_kube_proxy_csr_cn: "system:kube-proxy" # DO NOT CHANGE!
k8s_kube_proxy_csr_key_algo: "rsa"
k8s_kube_proxy_csr_key_size: "2048"
k8s_kube_proxy_csr_names_c: "DE"
k8s_kube_proxy_csr_names_l: "The_Internet"
k8s_kube_proxy_csr_names_o: "system:node-proxier" # DO NOT CHANGE!
k8s_kube_proxy_csr_names_ou: "BY"
k8s_kube_proxy_csr_names_st: "Bayern"

# CSR parameter for the kube-scheduler client certificate
k8s_scheduler_csr_cn: "system:kube-scheduler" # DO NOT CHANGE!
k8s_scheduler_csr_key_algo: "rsa"
k8s_scheduler_csr_key_size: "2048"
k8s_scheduler_csr_names_c: "DE"
k8s_scheduler_csr_names_l: "The_Internet"
k8s_scheduler_csr_names_o: "system:kube-scheduler" # DO NOT CHANGE!
k8s_scheduler_csr_names_ou: "BY"
k8s_scheduler_csr_names_st: "Bayern"

# CSR parameter for the kube-controller-manager client certificate
k8s_controller_manager_csr_cn: "system:kube-controller-manager" # DO NOT CHANGE!
k8s_controller_manager_csr_key_algo: "rsa"
k8s_controller_manager_csr_key_size: "2048"
k8s_controller_manager_csr_names_c: "DE"
k8s_controller_manager_csr_names_l: "The_Internet"
k8s_controller_manager_csr_names_o: "system:kube-controller-manager" # DO NOT CHANGE!
k8s_controller_manager_csr_names_ou: "BY"
k8s_controller_manager_csr_names_st: "Bayern"

# CSR parameter for kube-controller-manager service account key pair.
# The "kube-controller-manager" leverages a key pair to generate and sign
# service account tokens as described in the managing service accounts
# documentation: https://kubernetes.io/docs/admin/service-accounts-admin/
# Hint: Think twice if you want to change this key pair for a K8s cluster
# that has already pods deployed. The private key will be used to sign
# generated service account tokens. The public key will be used to verify
# the tokens during authentication. So if you have pods running e.g. with
# the `default` service account and you roll out a new key pair the "Token
# Controller" (https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#token-controller)
# which is part of the controller manager won't be able to verify the
# already existing service accounts anymore. So this might cause trouble
# for your running pods.
k8s_controller_manager_sa_csr_cn: "service-accounts"
k8s_controller_manager_sa_csr_key_algo: "rsa"
k8s_controller_manager_sa_csr_key_size: "2048"
k8s_controller_manager_sa_csr_names_c: "DE"
k8s_controller_manager_sa_csr_names_l: "The_Internet"
k8s_controller_manager_sa_csr_names_o: "Kubernetes"
k8s_controller_manager_sa_csr_names_ou: "BY"
k8s_controller_manager_sa_csr_names_st: "Bayern"

# Add additional etcd hosts that should be included in the certificates SAN.
# The task "Generate list of IP addresses and hostnames needed for etcd certificate"
# in this role will automatically add the hostname, the fully qualified domain name
# (FQDN), the internal IP address and the VPN IP address (e.g. the WireGuard IP)
# of your etcd hosts to a list which is needed to create the certificate.
# But "127.0.0.1" and "localhost" should be included too.
#
# If you plan to expand your etcd cluster from 3 to 5 hosts later e.g. and know the
# hostname, the fully qualified domain name (FQDN), the internal IP address and
# esp. the VPN IP address (e.g. the WireGuard IP) of that hosts upfront then add
# them here too. This will save you a lot of work later as you don't need to
# change the certificate files of the already running etcd daemons.
etcd_cert_hosts:
  - 127.0.0.1
  - localhost

# For "k8s_apiserver_cert_hosts" the same is basically true as with
# `etcd_cert_hosts` but we also include the Kubernetes service IP `10.32.0.1`
# (which you will get btw if you execute `nslookup kubernetes` later in one
# of the pods). We also include "127.0.0.1" and "localhost" and we include
# some Kubernetes hostname's that are available by default if "CoreDNS"
# is deployed.
k8s_apiserver_cert_hosts:
  - localhost
  - 127.0.0.1
  - 10.32.0.1
  - kubernetes
  - kubernetes.default
  - kubernetes.default.svc
  - kubernetes.default.svc.cluster
  - kubernetes.default.svc.cluster.local

# This list should contain all etcd clients that wants to connect to the etcd
# cluster. The most important client is "kube-apiserver" of course. So you
# definitely want to keep "k8s-apiserver-etcd" in this list.
# If other clients like "Traefik" or "Cilium" should also be able to connect
# to the Kubernetes etcd cluster and store their state there further client
# certificates can be issued. So if "Traefik" and "Cilium" should be able to
# connect to the etcd cluster via TLS the list would look like this:
#
# etcd_additional_clients:
#   - k8s-apiserver-etcd
#   - traefik
#   - cilium
#
# This will generate additional files in the directory specified in 
# "k8s_ca_conf_directory" variable e.g. "cert-traefik*" and "cert-cilium*".
# So instead of running a separate etcd cluster for "Traefik" and/or
# "Cilium" the already running etcd cluster for Kubernetes can be used in
# this case.
#
etcd_additional_clients:
  - k8s-apiserver-etcd
```

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
