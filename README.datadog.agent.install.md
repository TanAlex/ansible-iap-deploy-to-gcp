# README

### Reference
https://www.youtube.com/watch?v=EYoqwiXFrlQ


## Procedures

### Add hosts and projects to inventory.gcp.yaml file

### Install Ansible if it is not installed (ubuntu)
```
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get -y update
sudo apt-get install ansible -y
```

### Go to your Datadog website to get instructions
https://app.datadoghq.com/account/settings#agent/ansible

### Install datadog role
```
ansible-galaxy install datadog.datadog
# If it is already installed, ensure you get the latest version:
ansible-galaxy install --force datadog.datadog
```

### Create or add to your datadog-playbook.yaml
Remember to update your datadog_api_key to the one you use

```
- hosts: ungrouped
  roles:
    - { role: datadog.datadog, become: yes }
  vars:
    datadog_api_key: "998xxxxee77"
    datadog_site: "datadoghq.com"
    datadog_checks:
      process:
        init_config:
        instances:
          - name: ssh
            search_string: ['ssh', 'sshd']
          - name: syslog
            search_string: ['rsyslog']
            cpu_check_interval: 0.2
            exact_match: true
            ignore_denied_access: true
```

### Follow the README.md to install ansible and setup inventory.gcp.yaml file
```
ansible-inventory all -i inventory.gcp.yaml --list

ansible-inventory --graph

ansible-playbook datadog-playbook.yaml -i inventory.gcp.yaml
```

### The playbooks details 
```
/home/ttan/.ansible/roles/datadog.datadog/
```