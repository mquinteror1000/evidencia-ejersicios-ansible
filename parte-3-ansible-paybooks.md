## Parte 3 Playbooks de ansible

### Playbook 1 - Usuarios con loop

Creamos el archivo **usuarios.yml** en la carpeta de trabajo /root/devops2026/ansible

```shellsession
[root@localhost ansible]# pwd
/root/devops2026/ansible
[root@localhost ansible]# touch usuarios.yml
[root@localhost ansible]# vim usuarios.yml
```

con el siguiente contenido

```yml
---
- name: Cuidadores del zoológico
  hosts: web
  become: true
  vars:
    cuidadores:
      - koala
      - panda
      - lince
  tasks:
    - name: Crear grupo cuidadores
      ansible.builtin.group:
        name: cuidadores
        state: present

    - name: Crear usuarios
      ansible.builtin.user:
        name: "{{ item }}"
        groups: cuidadores
        append: true
        shell: /bin/bash
      loop: "{{ cuidadores }}"

    - name: Directorio compartido
      ansible.builtin.file:
        path: /opt/zoologico
        state: directory
        group: cuidadores
        mode: "2770"
```


