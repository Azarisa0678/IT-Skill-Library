# Ansible — Referenzmodul

Playbooks, Roles, Inventories, Vault, Best Practices, AWX/AAP.

---

## Projektstruktur (Best Practice)

```
ansible/
├── ansible.cfg
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   │       ├── all.yml
│   │       ├── webservers.yml
│   │       └── databases.yml
│   └── staging/
│       └── hosts.yml
├── playbooks/
│   ├── site.yml            # Master-Playbook
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── postgresql/
├── group_vars/
├── host_vars/
└── requirements.yml        # Ansible Galaxy Abhängigkeiten
```

```ini
# ansible.cfg
[defaults]
inventory          = inventories/production
roles_path         = roles
remote_user        = ansible
private_key_file   = ~/.ssh/ansible_ed25519
host_key_checking  = True
retry_files_enabled = False
stdout_callback    = yaml
callback_whitelist = timer, profile_tasks
gathering          = smart
fact_caching       = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600
forks              = 20

[privilege_escalation]
become       = True
become_method = sudo
become_user  = root

[ssh_connection]
pipelining    = True
ssh_args      = -o ControlMaster=auto -o ControlPersist=60s
```

---

## Inventory (YAML-Format)

```yaml
# inventories/production/hosts.yml
all:
  vars:
    ansible_python_interpreter: /usr/bin/python3

  children:
    webservers:
      hosts:
        web01:
          ansible_host: 10.0.1.11
          nginx_worker_processes: 4
        web02:
          ansible_host: 10.0.1.12
          nginx_worker_processes: 4
      vars:
        http_port: 80
        https_port: 443

    databases:
      hosts:
        db01:
          ansible_host: 10.0.2.11
          postgresql_max_connections: 200
        db02:
          ansible_host: 10.0.2.12
          postgresql_max_connections: 200
      vars:
        postgresql_version: "16"

    loadbalancers:
      hosts:
        lb01:
          ansible_host: 10.0.0.10
```

---

## Produktionsreifes Playbook

```yaml
# playbooks/webservers.yml
---
- name: Webserver einrichten und konfigurieren
  hosts: webservers
  gather_facts: true
  become: true
  any_errors_fatal: false
  serial: "30%"      # Rolling Update: 30% der Hosts gleichzeitig

  pre_tasks:
    - name: Wartungsmodus aktivieren (Load Balancer)
      uri:
        url: "https://lb01/api/backends/{{ inventory_hostname }}/drain"
        method: POST
        headers:
          Authorization: "Bearer {{ lb_api_token }}"
      delegate_to: localhost
      when: "'production' in group_names"

    - name: Pakete aktualisieren
      apt:
        update_cache: true
        cache_valid_time: 3600

  roles:
    - role: common
      tags: [common]
    - role: nginx
      tags: [nginx, webserver]
    - role: myapp
      tags: [myapp]

  post_tasks:
    - name: Health-Check
      uri:
        url: "https://{{ ansible_host }}/healthz"
        status_code: 200
        validate_certs: true
      retries: 5
      delay: 10
      register: health_check
      until: health_check.status == 200

    - name: Wartungsmodus deaktivieren
      uri:
        url: "https://lb01/api/backends/{{ inventory_hostname }}/enable"
        method: POST
        headers:
          Authorization: "Bearer {{ lb_api_token }}"
      delegate_to: localhost
      when: "'production' in group_names"

  handlers:
    - name: Nginx neustarten
      service:
        name: nginx
        state: restarted
    - name: Nginx neu laden
      service:
        name: nginx
        state: reloaded
```

---

## Role-Struktur (nginx)

```yaml
# roles/nginx/tasks/main.yml
---
- name: Nginx installieren
  package:
    name: nginx
    state: present
  notify: Nginx neustarten

- name: Nginx-Konfiguration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: "0644"
    validate: "/usr/sbin/nginx -t -c %s"
  notify: Nginx neu laden

- name: VHost-Konfigurationen
  template:
    src: vhost.conf.j2
    dest: "/etc/nginx/sites-available/{{ item.name }}.conf"
    mode: "0644"
  loop: "{{ nginx_vhosts }}"
  notify: Nginx neu laden

- name: VHosts aktivieren
  file:
    src: "/etc/nginx/sites-available/{{ item.name }}.conf"
    dest: "/etc/nginx/sites-enabled/{{ item.name }}.conf"
    state: link
  loop: "{{ nginx_vhosts }}"
  notify: Nginx neu laden

- name: Nginx starten und aktivieren
  service:
    name: nginx
    state: started
    enabled: true
```

```yaml
# roles/nginx/defaults/main.yml
nginx_worker_processes: auto
nginx_worker_connections: 1024
nginx_keepalive_timeout: 65
nginx_server_tokens: "off"
nginx_vhosts: []
```

---

## Ansible Vault

```bash
# Vault-verschlüsselte Datei erstellen
ansible-vault create group_vars/all/vault.yml

# Bestehende Datei verschlüsseln
ansible-vault encrypt group_vars/production/secrets.yml

# Bearbeiten
ansible-vault edit group_vars/production/secrets.yml

# Passwort-Datei (für CI/CD)
echo "SuperGeheimesPasswort" > ~/.vault_password
chmod 600 ~/.vault_password

# Playbook mit Vault ausführen
ansible-playbook playbooks/site.yml \
    --vault-password-file ~/.vault_password

# Einzelne Variable verschlüsseln (in Variablen-Datei einbetten)
ansible-vault encrypt_string 'geheimes_passwort' \
    --name 'db_password' \
    --vault-password-file ~/.vault_password
```

```yaml
# group_vars/databases/vault.yml (verschlüsselt gespeichert)
vault_db_password: "GeheimesPasswort123!"
vault_db_replication_password: "ReplicationGeheim!"

# group_vars/databases/vars.yml (Klartext, referenziert Vault)
db_password: "{{ vault_db_password }}"
db_replication_password: "{{ vault_db_replication_password }}"
```

---

## Nützliche Ad-hoc-Befehle

```bash
# Ping / Erreichbarkeit prüfen
ansible all -m ping
ansible webservers -m ping

# Fakten sammeln
ansible web01 -m setup
ansible webservers -m setup -a "filter=ansible_distribution*"

# Befehl ausführen
ansible webservers -m command -a "uptime"
ansible webservers -m shell -a "df -h | grep -v tmpfs"

# Datei kopieren
ansible webservers -m copy -a "src=/tmp/config dest=/etc/myapp/ mode=0644"

# Service verwalten
ansible webservers -m service -a "name=nginx state=reloaded"

# Paket installieren
ansible webservers -m apt -a "name=htop state=present" --become

# Playbook-Syntax prüfen
ansible-playbook --syntax-check playbooks/site.yml

# Dry-Run
ansible-playbook --check --diff playbooks/webservers.yml

# Nur bestimmte Tags ausführen
ansible-playbook playbooks/site.yml --tags "nginx,config"

# Bestimmten Host überspringen
ansible-playbook playbooks/site.yml --limit "webservers:!web02"

# Verbose für Debugging
ansible-playbook playbooks/site.yml -vvv
```

---

## Ansible AWX/AAP (Tower) — Kurzreferenz

```bash
# AWX CLI (awxkit)
pip install awxkit
awx login --conf.host https://awx.firma.de \
           --conf.username admin --conf.password geheim

# Job starten
awx job_templates launch \
    --name "Deploy WebServers" \
    --extra_vars '{"version": "1.5.0"}' \
    --monitor

# Inventory aktualisieren
awx inventory_sources update --name "Production Inventory" --monitor
```
