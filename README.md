# Ansible role OpenVPN
[![CI Molecule](https://github.com/darexsu/ansible-role-openvpn/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-openvpn/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/58592?color=blue&label=downloads)


  - Role:
      - [platforms](#platforms)
      - [install](#install)
      - [requirements](#requirements)
      - [FAQ](#faq)
      - [merge behaviour](#merge-behaviour)      
  - Playbooks (merge version):
      - [install and configure: OpenVPN v2.4, FirewallD](#install-and-configure-openvpn-firewalld-merge-version)
          - [install: OpenVPN, repo: distribution](#install-openvpn-repo-distribution-merge-version)
          - [install: OpenVPN, repo: third_party](#install-openvpn-repo-third_party-merge-version)
          - [configure: Create Certificate Authority](#configure-create-certificate-authority-merge-version)
          - [configure: OpenVPN-server](#configure-openvpn-server-merge-version)
          - [configure: OpenVPN-client](#configure-openvpn-client-merge-version)
          - [configure: add multiple client.ovpn ](#configure-add-multiple-clientovpn-merge-version)
          - [configure: disable some clients ](#configure-disable-some-clients-merge-version)
  - Playbooks (full version):
      - [install and configure: OpenVPN v2.4, FirewallD](#install-and-configure-openvpn-firewalld-full-version)
          - [install: OpenVPN, repo: distribution](#install-openvpn-repo-distribution-full-version)
          - [install: OpenVPN, repo: third_party](#install-openvpn-repo-third_party-full-version)
          - [configure: Create Certificate Authority](#configure-create-certificate-authority-full-version)
          - [configure: OpenVPN-server](#configure-openvpn-server-full-version)
          - [configure: OpenVPN-client](#configure-openvpn-client-full-version)
          - [configure: add multiple client.ovpn ](#configure-add-multiple-clientovpn-full-version)
          - [configure: disable some clients ](#configure-disable-some-clients-full-version)

### Platforms

|  Testing         | repo: distribution | repo: third_party|
| :--------------: | :----------------: | ---------------- |
| Debian 11        |  OpenVPN 2.5       | OpenVPN 2.5 or latest [openvpn.net]  |
| Ubuntu 20.04     |  OpenVPN 2.4       | OpenVPN 2.4 or latest [openvpn.net]  |
| Ubuntu 18.04     |  OpenVPN 2.4       | OpenVPN 2.4 or latest [openvpn.net]  |
| Oracle Linux 8   |  -                 | OpenVPN 2.4 or latest [EPEL]         |
| Rocky Linux 8    |  -                 | OpenVPN 2.4 or latest [EPEL]         |

### Install

```
ansible-galaxy install darexsu.openvpn --force
```
### Requirements

roles: [FirewallD](https://github.com/darexsu/ansible-role-firewalld) (will automatically be installed)

### FAQ

- Q: Playbooks (merge version)
  - A: Some users prefer that variables that are hashes (aka ‘dictionaries’ in Python terms) are merged. This setting is called ‘merge’. You can use both version - see [merge behaviour](#merge-behaviour). Don't turn on "hash_behaviour" in ansible.cfg

- Q: Easy way to deploy OpenVPN-server. Ready for use
  - A: Use playbook [install and configure: OpenVPN, FirewallD](#install-and-configure-openvpn-firewalld-full-version)

- Q: Enable routing all traffic through vpn?
  - A: by default

- Q: How can I connect to my OpenVPN-server?
  - A: 1) Setup OpenVPN-client for your OS or Android/iOS. 2) Import file "client.ovpn"

- Q: Where are "client.ovpn" files stored?
  - A: /etc/openvpn/client/

- Q: How can I create other ovpn files if OpenVPN and Certificate Authority already installed ?
  - A: Use [configure: add multiple Client.ovpn ](#configure-add-multiple-clientovpn-full-version)

- Q: Using the same ovpn profile from multiple computers
  - A: Edit: server.conf. Add "duplicate-cn". Remove "ifconfig-pool-persist"
### Merge behaviour

Replace or Merge dictionaries (with "hash_behaviour=replace" in ansible.cfg):
```
# Replace             # Merge
---                   ---
  vars:                 vars:
    dict:                 merge:
      a: "value"            dict: 
      b: "value"              a: "value" 
                              b: "value"

# How does merge work?:
Your vars [host_vars]  -->  default vars [current role] --> default vars [include role]
  
  dict:          dict:              dict:
    a: "1" -->     a: "1"    -->      a: "1"
                   b: "2"    -->      b: "2"
                                      c: "3"
    
```

##### Install and configure: Openvpn, FirewallD (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
        version: "2.4"                            # <-- Change
        repo: "third_party"
      # OpenVPN -> install
      openvpn_install:
        enabled: true
      # OpenVPN -> Certificate_Authority
      openvpn_ca:
        enabled: true
        privatekey_passphrase: "change_me"         # <-- Change
        common_name: "{{ ansible_fqdn }}"
        country_name: "US"
        state_or_province_name: "New York"
        locality_name: "New York City"
        organization_name: "Organization_Name"
        organizational_unit_name: "Community"
        email_address: "example@gmail.com"
        basic_constraints: 'CA:TRUE'
      # OpenVPN -> sign with CA
      openvpn_sign_with_ca:
        server:
          enabled: true
        client:
          enabled: true
      # OpenVPN -> config
      openvpn_config:
        server:
          enabled: true
        client:
          enabled: true

      # FirewallD
      firewalld:
        enabled: true
      # FirewallD -> install
      firewalld_install:
        enabled: true
      # FirewallD -> rules
      firewalld_rules:
        public_zone_masquerade:
          enabled: true
        openvpn_service:
          enabled: true
        public_zone_tun:
          enabled: true

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Install: Openvpn, repo: distribution (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
        version: "2.4"
        repo: "distribution"
      # OpenVPN -> install
      openvpn_install:
        enabled: true

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Install: Openvpn, repo: third_party (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
        version: "2.4"
        repo: "third_party"
      # OpenVPN -> install
      openvpn_install:
        enabled: true
 
  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: Create Certificate Authority (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
      # OpenVPN -> Certificate_Authority
      openvpn_ca:
        enabled: true
        path: "/etc/openvpn/ca/"
        diffie_hellman_bit: "2048"
        privatekey_passphrase: "change_me"         # <-- Change
        common_name: "{{ ansible_fqdn }}"
        country_name: "US"
        state_or_province_name: "New York"
        locality_name: "New York City"
        organization_name: "Organization_Name"
        organizational_unit_name: "Community"
        email_address: "example@gmail.com"
        basic_constraints: 'CA:TRUE'   

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: OpenVPN-server (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
        server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
        server_port: "1194"
        server_subnet: "10.8.0.0"
      # OpenVPN -> config
      openvpn_config:
        server:
          enabled: true
          file: "server.conf"
          src: "config_conf.j2"
          path: "/etc/openvpn/server/"
          backup: false
          data: |
            port {{ openvpn.server_port }}
            proto udp
            dev tun
            ca {{ openvpn_ca.path }}{{ ansible_fqdn }}.crt
            cert {{ openvpn_sign_with_ca.server.path }}server.crt
            key {{ openvpn_sign_with_ca.server.path }}server.key
            dh {{ openvpn_ca.path }}dh.pem
            topology subnet
            push "redirect-gateway def1 bypass-dhcp"
            push "dhcp-option DNS 8.8.8.8"
            push "dhcp-option DNS 8.8.4.4"
            server {{ openvpn.server_subnet }} 255.255.255.0
            ifconfig-pool-persist /etc/openvpn/server/ipp.txt
            client-config-dir /etc/openvpn/ccd
            tun-mtu 1500
            keepalive 10 120
            cipher AES-256-GCM
            tls-server
            auth SHA256
            client-to-client
            status /var/log/openvpn/openvpn-status.log
            log /var/log/openvpn/openvpn.log
            verb 2

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: OpenVPN-client (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
        server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
        server_port: "1194"
      # OpenVPN -> config
      openvpn_config:
        client:
          enabled: true
          file: "client.conf"
          src: "config_conf.j2"
          path: "/etc/openvpn/client/"
          backup: false
          data: |
            client
            dev tun
            proto udp
            remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
            remote-cert-tls server
            auth SHA256
            cipher AES-256-GCM
            tun-mtu 1500
            resolv-retry infinite
            nobind
            persist-key
            persist-tun
            log-append /var/log/openvpn/openvpn-client.log
            verb 2

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: add multiple Client.ovpn (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
        server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
        server_port: "1194"
      # OpenVPN -> sign with CA
      openvpn_sign_with_ca:
        client1:
          enabled: true
          path: "/etc/openvpn/client/"
          common_name: "client1"
        client2:
          enabled: true
          path: "/etc/openvpn/client/"
          common_name: "client2"
        client3:
          enabled: true
          path: "/etc/openvpn/client/"
          common_name: "client3"
        client4:
          enabled: true
          path: "/etc/openvpn/client/"
          common_name: "client4"
        client5:
          enabled: true
          path: "/etc/openvpn/client/"
          common_name: "client5"
        # ...
      # OpenVPN -> config (This config import to client.ovpn)
      openvpn_config:
        client1:
          enabled: true
          file: "client1.conf"
          src: "config_conf.j2"
          path: "/etc/openvpn/client/"
          backup: false
          data: |
            client
            dev tun
            proto udp
            remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
            remote-cert-tls server
            auth SHA256
            cipher AES-256-GCM
            tun-mtu 1500
            resolv-retry infinite
            nobind
            persist-key
            persist-tun
            log-append /var/log/openvpn/openvpn-client.log
            verb 2
        client2:
          enabled: true
          file: "client2.conf"
          src: "config_conf.j2"
          path: "/etc/openvpn/client/"
          backup: false
          data: |
            client
            dev tun
            proto udp
            remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
            remote-cert-tls server
            auth SHA256
            cipher AES-256-GCM
            tun-mtu 1500
            resolv-retry infinite
            nobind
            persist-key
            persist-tun
            log-append /var/log/openvpn/openvpn-client.log
            verb 2
        client3:
          enabled: true
          file: "client3.conf"
          src: "config_conf.j2"
          path: "/etc/openvpn/client/"
          backup: false
          data: |
            client
            dev tun
            proto udp
            remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
            remote-cert-tls server
            auth SHA256
            cipher AES-256-GCM
            tun-mtu 1500
            resolv-retry infinite
            nobind
            persist-key
            persist-tun
            log-append /var/log/openvpn/openvpn-client.log
            verb 2
        client4:
          enabled: true
          file: "client4.conf"
          src: "config_conf.j2"
          path: "/etc/openvpn/client/"
          backup: false
          data: |
            client
            dev tun
            proto udp
            remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
            remote-cert-tls server
            auth SHA256
            cipher AES-256-GCM
            tun-mtu 1500
            resolv-retry infinite
            nobind
            persist-key
            persist-tun
            log-append /var/log/openvpn/openvpn-client.log
            verb 2
        client5:
          enabled: true
          file: "client5.conf"
          src: "config_conf.j2"
          path: "/etc/openvpn/client/"
          backup: false
          data: |
            client
            dev tun
            proto udp
            remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
            remote-cert-tls server
            auth SHA256
            cipher AES-256-GCM
            tun-mtu 1500
            resolv-retry infinite
            nobind
            persist-key
            persist-tun
            log-append /var/log/openvpn/openvpn-client.log
            verb 2

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: disable some clients (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # OpenVPN
      openvpn:
        enabled: true
      # OpenVPN -> config
      openvpn_config:
        client:
          enabled: true
          file: "client"
          path: "/etc/openvpn/ccd/"
          data: |
            disable

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Install and configure: Openvpn, FirewallD (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "third_party"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"
    # OpenVPN -> install
    openvpn_install:
      enabled: true
    # OpenVPN -> Certificate_Authority
    openvpn_ca:
      enabled: true
      path: "/etc/openvpn/ca/"
      diffie_hellman_bit: "2048"
      privatekey_passphrase: "change_me"         # <-- Change
      common_name: "{{ ansible_fqdn }}"
      country_name: "US"
      state_or_province_name: "New York"
      locality_name: "New York City"
      organization_name: "Organization_Name"
      organizational_unit_name: "Community"
      email_address: "example@gmail.com"
      basic_constraints: 'CA:TRUE'
    # OpenVPN -> sign with CA
    openvpn_sign_with_ca:
      server:
        enabled: true
        path: "/etc/openvpn/server/"
        common_name: "server"
        key_usage: ["digitalSignature", "keyAgreement"]
        extended_key_usage: "TLS Web Server Authentication"
      client:
        enabled: true
        path: "/etc/openvpn/client/"
        common_name: "client"
        key_usage: ["digitalSignature", "keyAgreement"]
        extended_key_usage: "TLS Web Client Authentication"
    # OpenVPN -> config
    openvpn_config:
      server:
        enabled: true
        file: "server.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/server/"
        backup: false
        data: |
          port {{ openvpn.server_port }}
          proto udp
          dev tun
          ca {{ openvpn_ca.path }}{{ ansible_fqdn }}.crt
          cert {{ openvpn_sign_with_ca.server.path }}server.crt
          key {{ openvpn_sign_with_ca.server.path }}server.key
          dh {{ openvpn_ca.path }}dh.pem
          topology subnet
          push "redirect-gateway def1 bypass-dhcp"
          push "dhcp-option DNS 8.8.8.8"
          push "dhcp-option DNS 8.8.4.4"
          server {{ openvpn.server_subnet }} 255.255.255.0
          ifconfig-pool-persist /etc/openvpn/server/ipp.txt
          client-config-dir /etc/openvpn/ccd
          tun-mtu 1500
          keepalive 10 120
          cipher AES-256-GCM
          tls-server
          auth SHA256
          client-to-client
          status /var/log/openvpn/openvpn-status.log
          log /var/log/openvpn/openvpn.log
          verb 2
      client:
        enabled: true
        file: "client.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/client/"
        backup: false
        data: |
          client
          dev tun
          proto udp
          remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
          remote-cert-tls server
          auth SHA256
          cipher AES-256-GCM
          tun-mtu 1500
          resolv-retry infinite
          nobind
          persist-key
          persist-tun
          log-append /var/log/openvpn/openvpn-client.log
          verb 2

    # FirewallD
    firewalld:
      enabled: true
      service:
        enabled: true
        state: "started"
    # FirewallD -> install
    firewalld_install:
      enabled: true
    # FirewallD -> rules
    firewalld_rules:
      public_zone_masquerade:
        enabled: true
        zone: "public"
        state: enabled
        permanent: true
        masquerade: true
      openvpn_service:
        enabled: true
        zone: "public"
        service: "openvpn"
        state: "enabled"
        permanent: true
      public_zone_tun:
        enabled: true
        zone: "public"
        state: "enabled"
        interface: "tun0"
        permanent: true

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Install: Openvpn, repo: distribution (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "distribution"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"
    # OpenVPN -> install
    openvpn_install:
      enabled: true

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Install: Openvpn, repo: third_party (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "third_party"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"
    # OpenVPN -> install
    openvpn_install:
      enabled: true 

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: Create Certificate Authority (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "third_party"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"
    # OpenVPN -> Certificate_Authority
    openvpn_ca:
      enabled: true
      path: "/etc/openvpn/ca/"
      diffie_hellman_bit: "2048"
      privatekey_passphrase: "change_me"         # <-- Change
      common_name: "{{ ansible_fqdn }}"
      country_name: "US"
      state_or_province_name: "New York"
      locality_name: "New York City"
      organization_name: "Organization_Name"
      organizational_unit_name: "Community"
      email_address: "example@gmail.com"
      basic_constraints: 'CA:TRUE'   

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: OpenVPN-server (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "third_party"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"   
    # OpenVPN -> config
    openvpn_config:
      server:
        enabled: true
        file: "server.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/server/"
        backup: false
        data: |
          port {{ openvpn.server_port }}
          proto udp
          dev tun
          ca {{ openvpn_ca.path }}{{ ansible_fqdn }}.crt
          cert {{ openvpn_sign_with_ca.server.path }}server.crt
          key {{ openvpn_sign_with_ca.server.path }}server.key
          dh {{ openvpn_ca.path }}dh.pem
          topology subnet
          push "redirect-gateway def1 bypass-dhcp"
          push "dhcp-option DNS 8.8.8.8"
          push "dhcp-option DNS 8.8.4.4"
          server {{ openvpn.server_subnet }} 255.255.255.0
          ifconfig-pool-persist /etc/openvpn/server/ipp.txt
          client-config-dir /etc/openvpn/ccd
          tun-mtu 1500
          keepalive 10 120
          cipher AES-256-GCM
          tls-server
          auth SHA256
          client-to-client
          status /var/log/openvpn/openvpn-status.log
          log /var/log/openvpn/openvpn.log
          verb 2

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: OpenVPN-client (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "third_party"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"  
    # OpenVPN -> config
    openvpn_config:      
      client:
        enabled: true
        file: "client.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/client/"
        backup: false
        data: |
          client
          dev tun
          proto udp
          remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
          remote-cert-tls server
          auth SHA256
          cipher AES-256-GCM
          tun-mtu 1500
          resolv-retry infinite
          nobind
          persist-key
          persist-tun
          log-append /var/log/openvpn/openvpn-client.log
          verb 2

  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: add multiple Client.ovpn (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "third_party"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"
    # OpenVPN -> sign with CA
    openvpn_sign_with_ca:
      client1:
        enabled: true
        path: "/etc/openvpn/client/"
        common_name: "client1"
        key_usage: ["digitalSignature", "keyAgreement"]
        extended_key_usage: "TLS Web Client Authentication"
      server2:
        enabled: true
        path: "/etc/openvpn/client/"
        common_name: "client2"
        key_usage: ["digitalSignature", "keyAgreement"]
        extended_key_usage: "TLS Web Client Authentication"
      client3:
        enabled: true
        path: "/etc/openvpn/client/"
        common_name: "client3"
        key_usage: ["digitalSignature", "keyAgreement"]
        extended_key_usage: "TLS Web Client Authentication"
      server4:
        enabled: true
        path: "/etc/openvpn/client/"
        common_name: "client4"
        key_usage: ["digitalSignature", "keyAgreement"]
        extended_key_usage: "TLS Web Client Authentication"
      client5:
        enabled: true
        path: "/etc/openvpn/client/"
        common_name: "client5"
        key_usage: ["digitalSignature", "keyAgreement"]
        extended_key_usage: "TLS Web Client Authentication"
    # OpenVPN -> config (This config import to client.ovpn)
    openvpn_config:
      client1:
        enabled: true
        file: "client1.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/client/"
        backup: false
        data: |
          client
          dev tun
          proto udp
          remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
          remote-cert-tls server
          auth SHA256
          cipher AES-256-GCM
          tun-mtu 1500
          resolv-retry infinite
          nobind
          persist-key
          persist-tun
          log-append /var/log/openvpn/openvpn-client.log
          verb 2
      client2:
        enabled: true
        file: "client2.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/client/"
        backup: false
        data: |
          client
          dev tun
          proto udp
          remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
          remote-cert-tls server
          auth SHA256
          cipher AES-256-GCM
          tun-mtu 1500
          resolv-retry infinite
          nobind
          persist-key
          persist-tun
          log-append /var/log/openvpn/openvpn-client.log
          verb 2
      client3:
        enabled: true
        file: "client3.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/client/"
        backup: false
        data: |
          client
          dev tun
          proto udp
          remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
          remote-cert-tls server
          auth SHA256
          cipher AES-256-GCM
          tun-mtu 1500
          resolv-retry infinite
          nobind
          persist-key
          persist-tun
          log-append /var/log/openvpn/openvpn-client.log
          verb 2
      client4:
        enabled: true
        file: "client4.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/client/"
        backup: false
        data: |
          client
          dev tun
          proto udp
          remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
          remote-cert-tls server
          auth SHA256
          cipher AES-256-GCM
          tun-mtu 1500
          resolv-retry infinite
          nobind
          persist-key
          persist-tun
          log-append /var/log/openvpn/openvpn-client.log
          verb 2
      client5:
        enabled: true
        file: "client5.conf"
        src: "config_conf.j2"
        path: "/etc/openvpn/client/"
        backup: false
        data: |
          client
          dev tun
          proto udp
          remote {{ openvpn.server_ip }} {{ openvpn.server_port }}
          remote-cert-tls server
          auth SHA256
          cipher AES-256-GCM
          tun-mtu 1500
          resolv-retry infinite
          nobind
          persist-key
          persist-tun
          log-append /var/log/openvpn/openvpn-client.log
          verb 2
   
  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
##### Configure: disable some clients (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # OpenVPN
    openvpn:
      enabled: true
      version: "2.4"
      repo: "third_party"
      server_ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"
      server_port: "1194"
      server_subnet: "10.8.0.0"
      net_ipv4_ip_forward: true
      service:
        enabled: true
        state: "started"
    # OpenVPN -> config
    openvpn_config:
      client:
        enabled: true
        file: "client"
        src: "config_conf.j2"
        path: "/etc/openvpn/ccd/"
        backup: false
        data: |
          disable
   
  tasks:
    - name: role darexsu.openvpn
      include_role:
        name: darexsu.openvpn
```
