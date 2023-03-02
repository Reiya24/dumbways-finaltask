# setup cloudflare
## domain
buka halaman cloudflare > website > pilih domain aktif > dns
add record, masukan domain dan ip public dari gateway
![](.11setup_domain_images/e1a82b8a.png)

## API token
buka halaman cloudflare > my profile > API token > global API
key, klik view, copy API token

# setup ansible
buat ansible playbook 
```shell
---
- hosts: gateway
  become: true
  gather_facts: true

  tasks:
    - name: "install nginx menggunakan apt"
      apt:
        name: nginx
        state: latest
        update_cache: true

    - name: "jalankan service nginx"
      service:
        name: nginx
        state: started

    - name: "copy file konfigurasi nginx"
      copy:
        src: sites-enabled/
        dest: /etc/nginx/sites-enabled

    - name: "restart service nginx"
      service:
        name: nginx
        state: reloaded
    - name: update snapd
      shell: "sudo snap install core; sudo snap refresh core"
    - name: install certbot
      shell: "sudo snap install --classic certbot"
    - name: certbot env
      shell: "sudo ln -s /snap/bin/certbot /usr/bin/certbot"
      ignore_errors: yes
    - name: allow plugin
      shell: "sudo snap set certbot trust-plugin-with-root=ok"
    - name: install plugin
      shell: "sudo snap install certbot-dns-cloudflare"
    - name: buat file credentials
      copy:
        dest: /home/{{ansible_user}}/cloudflare.ini
        content: |
          dns_cloudflare_email = {{email}}
          dns_cloudflare_api_key = {{api_token}}
#    - name: jalankan certbot
#      shell: "sudo certbot -i nginx --non-interactive --agree-tos --email reiya2307@gmail.com --dns-cloudflare --dns-cloudflare-credentials //home/reiya24/cloudflare.ini -d reiya.my.id  -d *.reiya.my.id"
echo "0 12 1 * * /usr/bin/certbot renew --quiet" >> /var/spool/cron/crontabs/reiya24

  vars:
    - email: "reiya2307@gmail.com"
    - api_token: "api_token"

```
![](.11setup_domain_images/1a43b058.png)

jalankan certbot
```shell
ansible-playbook nama_file.yaml
```
![](.11setup_domain_images/8a407fa5.png)

setup reverse proxy berhasil
![](.11setup_domain_images/aef02d5e.png)