## Evidencia de ejercicios ANSIBLE

## Paso a paso en la VM

#### 1. Crear el usuario ansible y darle nombre al host

```shellsession
[root@localhost ~]# useradd ansible
Creando el fichero del buzón de correo: El fichero ya existe
[root@localhost ~]# passwd ansible
Cambiando la contraseña del usuario ansible.
Nueva contraseña: 
Vuelva a escribir la nueva contraseña: 
passwd: todos los tokens de autenticación se actualizaron exitosamente.
[root@localhost ~]#  echo "127.0.0.1   nodo1" >> /etc/hosts
[root@localhost ~]# getent hosts nodo1
127.0.0.1       nodo1
```

#### 2. Conectarse a nodo1 ( misma máquina ) usando password

```shellsession
[root@localhost ~]# ssh ansible@nodo1
ansible@127.0.0.1's password: 
Last failed login: Fri Oct 23 08:03:00 CST 2026 from 127.0.0.1 on ssh:notty
There were 2 failed login attempts since the last successful login.
[ansible@localhost ~]$ exit
cerrar sesión
Connection to 127.0.0.1 closed.
[root@localhost ~]# cat .ssh/known_hosts
nodo1 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGkauMvuOfkbKsDgkL7brzpEb0qCzHs/547r2IyWIx11
nodo1 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDXpIeOV8BBTdwHuwDAyPBr1ZoXWXI1tqyL2Sp08ZngA3u+eA1GpMoJjzqmGpK05Z8lQgD10hvqYcb9Nu6zwqGY+9LaPiIZ8tPBzEI9djtciwK5aoZ82jZQcJB3X+Cp8H17B957GN5ebInA/4xLsENFZTAHOc6ePbYuTtVcYWZoS/4Yp1jQi49k+Ueex/h3wJ9qiqPUxIPnwFzhJkJtNg2ZYiE+qloXidj1pcwLPiaDN8DmoW8IBDfAq/XXWQY74UqRJ9+sGJ04OmEmwmQHE2cIyEl+goDBeuOPe3w4CoUw+I2ebxp2ANFv5xeIFGW3WOkl+zQsueyYH/0/tV3U23ragi+N8BAY7g2qGwijAKQj+rf5wX1Jfrl3eeEq+QaQkW/qnSMD5irjISTgPAItByrdwhxeqSBe9ENF0kgQHqqtd5cmQ9zaX9PDPsCec6ivNgz3mo8ebI0pcY7pB42pq5bCXY+YT4O0NbQNzbQcuIqYiSFDTAPWHugfZwq3cSC0kwc=
nodo1 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBB1NnJN3pImt6mydrtDvK2LKmzCcYOBFPjhlP+CTYxD3Wrlx2I+K6KDR5jTnYNR9Y7o+N+Dgh6moUQ1KNXeqTPQ=
127.0.0.1 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGkauMvuOfkbKsDgkL7brzpEb0qCzHs/547r2IyWIx11
192.168.122.40 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGkauMvuOfkbKsDgkL7brzpEb0qCzHs/547r2IyWIx11

```

#### 3. Generar claves ssh para el nodo de control ( con usuario root

```shellsession

```
