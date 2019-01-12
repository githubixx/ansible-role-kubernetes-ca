CHANGELOG
---------

**7.0.0+1.12.3**

- use correct semantic versioning as described in https://semver.org. Needed for Ansible Galaxy importer as it now insists on using semantic versioning.
- moved changelog entries to separate file
- make Ansible linter happy
- no major changes but decided to start a new major release as versioning scheme changed quite heavily

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