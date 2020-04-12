[![pipeline status](https://gitlab.com/miyuk/ansible_cert_update/badges/master/pipeline.svg)](https://gitlab.com/miyuk/ansible_cert_update/commits/master)

# ansible_cert_update

Ansibleサーバの特定ディレクトリにある情報からNGINXとBIG-IPの証明書を更新するPlaybookです

## Requirements
1. yum package
  ```bash
    yum install -y openssl
  ```

2. pip library
  ```bash
    pip install --upgrade -r requirements.txt
  ```


## Variables

下記パラメータを投入して`ansible-playbook`実行  
詳細は、各Roleの`defaults`参照

```yaml
---
### update_cert.ymlで利用 ###
certificate_directory: ./certs
certificate_files_owner: root
certificate_files_group: root
certificate_files_mode: 0755
certificate_csr_filename: "{{ certificate_common_name }}.csr"
certificate_crt_filename: "{{ certificate_common_name }}.crt"
certificate_key_filename: "{{ certificate_common_name }}.key"
certificate_fullchain_filename: "{{ certificate_common_name }}.fullchain.crt"
certificate_force_create_key: no
certificate_force_create_csr: no
certificate_force_create_crt: no
certificate_remaining_days: 30
certificate_key_size: 2048

# Originサーバ情報
origin_certificate_directory: /var/lib/awx/cert_update/certs

# 証明書情報設定
certificate_common_name: example.net
certificate_country_name: JP
certificate_state_or_province_name: Tokyo
certificate_locality_name: Tokyo
certificate_organization_name: example.net
certificate_email_address: root@example.net
certificate_subject_alt_name:
  - 'DNS:*.example.net'
  - 'DNS:example.net'

# Let's Encryptアカウント設定
letsencrypt_key_directory: ./letsencrypt
letsencrypt_key_filename: account.key
letsencrypt_key_size: 2048
letsencrypt_email: root@example.net
letsencrypt_force_create_key: no

# CloudFlareアカウント設定
cloudflare_domain: example.net
```

```yaml
---
### deploy_cert_bigip.ymlで利用 ###
# ローカルホスト側の証明書パス
bigip_src_crt_path: ./certs/example.net.fullchain.crt
bigip_src_key_path: ./certs/example.net.key

# BIG-IP側証明書配置パス
bigip_partition: Common
bigip_dest_crt_path: "/config/ssl/ssl.crt/{{ bigip_dest_crt_name }}"
bigip_dest_key_path: "/config/ssl/ssl.key/{{ bigip_dest_key_name }}"
bigip_dest_crt_name: example-wildcard_cert
bigip_dest_key_name: example-wildcard_cert
bigip_dest_ssl_profile_name: clientssl-wildcard
```

```yaml
---
### deploy_cert_nginx.ymlで利用 ###
# ローカルホスト側の証明書パス
nginx_src_crt_path: ./certs/example.net.fullchain.crt
nginx_src_key_path: ./certs/example.net.key

# 配置するサーバ側の証明書パス
nginx_dest_crt_force_create: yes
nginx_dest_key_force_create: yes
nginx_dest_crt_path: /opt/nginx/data/nginx/certs/server.crt
nginx_dest_key_path: /opt/nginx/data/nginx/certs/server.key
nginx_dest_crt_mode: 0644
nginx_dest_key_mode: 0600
nginx_dest_files_owner: '600'
nginx_dest_files_group: '600'

# 更新後のNGINX再起動設定
nginx_restart: yes
nginx_docker_dir: /opt/nginx
nginx_daemon_type: docker
nginx_docker_name: nginx
```

```yaml
---
### deploy_cert_haproxy.ymlで利用 ###
# ローカルホスト側の証明書パス
haproxy_src_crt_path: "{{ certificate_directory }}/{{ certificate_crt_filename }}"
haproxy_src_key_path: "{{ certificate_directory }}/{{ certificate_crt_filename }}"

# 配置するサーバ側の証明書パス
haproxy_dest_crt_and_key_force_create: yes
haproxy_dest_crt_and_key_path: /etc/haproxy/ssl/server.crt
haproxy_dest_crt_and_key_mode: 0600
haproxy_dest_files_owner: root
haproxy_dest_files_group: root

# 更新後のhaproxy再起動設定
haproxy_restart: yes
haproxy_docker_dir: /opt/haproxy
haproxy_daemon_type: docker
haproxy_docker_name: haproxy
```