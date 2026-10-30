## Parte 2 Ansible

#### 1. Instalar paquetes

Debe ser **ansible-core**  el paquete a instalar, no ansible u otro parecido, solo el base

```shellsession
[root@localhost ~]# dnf install -y ansible-core
Última comprobación de caducidad de metadatos hecha hace 0:53:39, el jue 29 oct 2026 18:33:51.
El paquete ansible-core-1:2.14.18-3.el9_8.1.x86_64 ya está instalado.
Dependencias resueltas.
Nada por hacer.
¡Listo!
[root@localhost ~]# ansible --version
ansible [core 2.14.18]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.25 (main, Aug  6 2026, 00:00:00) [GCC 11.5.0 20240719 (Red Hat 11.5.0-14)] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
[root@localhost ~]# mkdir -p /root/devops2026/ansible && cd /root/devops2026/ansible
[root@localhost ansible]# pwd
/root/devops2026/ansible
```

Hasta aqui esta instalado el paquete y creado el directorio /root/devops2026/ansible

#### 2. Archivos de configuracion en /root/devops2026/ansible

En el directorio de trabajo se crean los archivos **inventario.ini** y **ansible.cfg** cada uno con su respectivo contendido segun el manual del profesor

```shellsession
[root@localhost ansible]# touch inventario.ini
[root@localhost ansible]# touch ansible.cfg
[root@localhost ansible]# vim inventario.ini 
[root@localhost ansible]# vim ansible.cfg 
[root@localhost ansible]# cat inventario.ini 
[web]
nodo1  ansible_host=127.0.0.1

[zoologico:children]
web
[root@localhost ansible]# car ansible.cfg 
-bash: car: orden no encontrada
[root@localhost ansible]# cat ansible.cfg 
[defaults]
inventory   = ./inventario.ini
remote_user = ansible
forks       = 5

[root@localhost ansible]# 
```

verificar que  esté correcta la configuración

```shellsession
[root@localhost ansible]# ansible --version | grep "config file"
  config file = /root/devops2026/ansible/ansible.cfg
[root@localhost ansible]# ansible-inventory --graph
@all:
  |--@ungrouped:
  |--@zoologico:
  |  |--@web:
  |  |  |--nodo1
[root@localhost ansible]# ansible-inventory --list
{
    "_meta": {
        "hostvars": {
            "nodo1": {
                "ansible_host": "127.0.0.1"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "zoologico"
        ]
    },
    "web": {
        "hosts": [
            "nodo1"
        ]
    },
    "zoologico": {
        "children": [
            "web"
        ]
    }
}
[root@localhost ansible]# ansible all --list-hosts
  hosts (1):
    nodo1
[root@localhost ansible]# 
```

Parace que está bien

### 2. Actividades Básicas ( comandos ad-hoc )
