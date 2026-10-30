## Evidencia de ejercicios ANSIBLE

## 01 - Paso a paso en la VM

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
[root@localhost ~]# # ssh-keygen -t ed25519 -C "root@control"
[root@localhost ~]# ls -l ~/.ssh/
total 28
-rw-r--r--. 1 root root  109 oct 23 09:32 config
-rw-------. 1 root root  432 oct 23 09:17 github_mquinteror1000
-rw-------. 1 root root  113 oct 23 09:17 github_mquinteror1000.pub
-rw-------. 1 root root  419 oct 23 09:36 id_ed25519
-rw-r--r--. 1 root root  108 oct 23 09:36 id_ed25519.pub
-rw-------. 1 root root 1000 oct 23 09:17 known_hosts
-rw-r--r--. 1 root root   87 oct 23 08:16 known_hosts.old
[root@localhost ~]# cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFVWx2Thz5aILUhyG2xb0OjQ9X8QXk5E1JdObixSIqcw root@localhost.localdomain
[root@localhost ~]# 
```

Se puede ver que si se generó el par de claves

#### 4. usar ssh-coy-id para configurar acceso mediante clave ssh

```shellsession
[root@localhost ~]# ssh-copy-id ansible@nodo1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
ansible@127.0.0.1's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'ansible@nodo1'"
and check to make sure that only the key(s) you wanted were added.

[root@localhost ~]# ssh ansible@nodo1
Last login: Thu Oct 29 18:31:35 2026
```

Funciona!!

#### 5. ejecutando comandos remotos

```shellsession
[root@localhost ~]# ssh ansible@nodo1 'hostname; whoami; uptime'
localhost.localdomain
ansible
 18:36:59 up  1:11,  1 user,  load average: 0.01, 0.03, 0.00
[root@localhost ~]# for h in nodo1 nodo1 nodo1; do ssh ansible@$h 'date'; done
jue 29 oct 2026 18:37:07 CST
jue 29 oct 2026 18:37:07 CST
jue 29 oct 2026 18:37:07 CST
[root@localhost ~]# 

```

Han funcionado, como entra por ssh directo autenticando con la clave.

#### Provocar una falla de autenticacion a propósito, moficando los permisos

```shellsession
[root@localhost ~]# chmod 777 /home/ansible/.ssh
[root@localhost ~]# ssh ansible@nodo1
ansible@nodo1's password: 
```

Ahora pide password, porque el directorio /home/ansible/.ssh no cumple con las politicas de permisos para ser usado

Diagnosticar con ssh modo verboso ssh -v y filtrando Autentications

```shellsession
[root@localhost ~]# ssh -v ansible@nodo1 2>&1 | grep -i -E "offering|authentications"
debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
debug1: Offering public key: /root/.ssh/id_ed25519 ED25519 SHA256:9vWA7QZ5hgKorSlO01bHKcXcNzBb1JTvidXtRqWnpD4
ansible@nodo1's password: debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
```

sabemos no ha querido autenticar con claves ssh, por eso pide password.

Arreglemoslo, dejando los permisos como se remcomienda 700 para /home/ansible/.ssh y 600 para /home/ansible/.ssh/authorized_keys

```shellsession
[root@localhost ~]# chmod 700 /home/ansible/.ssh
[root@localhost ~]# chmod 600 /home/ansible/.ssh/authorized_keys
[root@localhost ~]# ssh ansible@nodo1 hostname
localhost.localdomain
[root@localhost ~]# ssh ansible@nodo1
Last login: Thu Oct 29 18:34:47 2026 from 127.0.0.1
```

ha funcionado, como lo hcemos todo como root, podemos usmear los home de otros usuarios, pero lo correcto sería que ansible sea quien corrija sus permisos

## 02 - copiar archivos de un punto a otro

#### 1. scp: origen -> destino

local a remoto

```shellsession
[root@localhost ~]# echo "Reporte de aves - $(date)" > /root/reporte.txt
[root@localhost ~]# scp /root/reporte.txt ansible@nodo1:/home/ansible/
reporte.txt    
```

remoto a local

```shellsession
[root@localhost ~]# scp ansible@nodo1:/etc/os-release /tmp/os-release-nodo1
os-release        
```

carpeta completa

```shellsession
[root@localhost ~]# scp ansible@nodo1:/etc/os-release /tmp/os-release-nodo1
os-release                                                                                                                                                                        100%  579     1.0MB/s   00:00    
[root@localhost ~]# mkdir -p /root/sitio && for i in 1 2 3; do echo "pagina $i" > /root/sitio/p$i.html; done
[root@localhost ~]# scp -r /root/sitio ansible@nodo1:~
p2.html                                                                                                                                                                           100%    9    23.2KB/s   00:00    
p3.html                                                                                                                                                                           100%    9    39.2KB/s   00:00    
p4.html                                                                                                                                                                           100%    6    33.0KB/s   00:00    
p1.html                                                                                                                                                                           100%    9    45.4KB/s   00:00    
[root@localhost ~]# ssh ansible@nodo1 'ls -l ~/sitio ~/reporte.txt'
-rw-r--r--. 1 ansible ansible 47 oct 29 19:07 /home/ansible/reporte.txt

/home/ansible/sitio:
total 16
-rw-r--r--. 1 ansible ansible 9 oct 29 19:09 p1.html
-rw-r--r--. 1 ansible ansible 9 oct 29 19:09 p2.html
-rw-r--r--. 1 ansible ansible 9 oct 29 19:09 p3.html
-rw-r--r--. 1 ansible ansible 6 oct 29 19:09 p4.html
```

#### 2. rsync solo lo que cambió

```shellsession
[root@localhost ~]# dnf install -y rsync
Última comprobación de caducidad de metadatos hecha hace 0:37:55, el jue 29 oct 2026 18:33:51.
El paquete rsync-3.2.7-1.el9_8.x86_64 ya está instalado.
Dependencias resueltas.
Nada por hacer.
¡Listo!
[root@localhost ~]#  rsync -avz /root/sitio/ ansible@nodo1:/home/ansible/respaldo/
sending incremental file list
created directory /home/ansible/respaldo
./
p1.html
p2.html
p3.html
p4.html

sent 349 bytes  received 140 bytes  326.00 bytes/sec
total size is 33  speedup is 0.07
[root@localhost ~]# rsync -avz /root/sitio/ ansible@nodo1:/home/ansible/respaldo/ 
sending incremental file list

sent 139 bytes  received 12 bytes  100.67 bytes/sec
total size is 33  speedup is 0.22
[root@localhost ~]# echo "nueva" > /root/sitio/p4.html
[root@localhost ~]# rsync -avz /root/sitio/ ansible@nodo1:/home/ansible/respaldo/
sending incremental file list
p4.html

sent 180 bytes  received 41 bytes  147.33 bytes/sec
total size is 33  speedup is 0.15
[root@localhost ~]# rm /root/sitio/p1.html
rm: ¿borrar el fichero regular '/root/sitio/p1.html'? (s/n) s
[root@localhost ~]# rsync -avz --delete --dry-run /root/sitio/ ansible@nodo1:/home/ansible/respaldo/
sending incremental file list
deleting p1.html
./

sent 124 bytes  received 33 bytes  314.00 bytes/sec
total size is 24  speedup is 0.15 (DRY RUN)

```

#### 3. sftp modo interactivo

```shellsession
[root@localhost ~]# sftp ansible@nodo1
Connected to nodo1.
sftp> wd ; ls
Invalid command.
sftp> pwd
Remote working directory: /home/ansible
sftp> ld
Invalid command.
sftp> ls
reporte.txt  respaldo     sitio        
sftp> get reporte.txt /tmp/reporte-bajado.txt
Fetching /home/ansible/reporte.txt to /tmp/reporte-bajado.txt
open local "/tmp/reporte-bajado.txt": Permission denied
sftp> put /etc/hostname
Uploading /etc/hostname to /home/ansible/hostname
hostname                                                                                                                                                                          100%    1     0.9KB/s   00:00    
sftp> bye 

```

#### 4. Motivación para ansible

¿Por qué ansible es mejor que un script interactivo?

```shellsession
[root@localhost ~]# vim configura.sh
```

con este contendido

```bash
#!/bin/bash
for h in nodo1; do
  ssh ansible@$h 'echo "zona=aves" >> ~/config.txt'
done
```

se ejecuta 3 veces

```shellsession
[root@localhost ~]# sh configura.sh 
[root@localhost ~]# sh configura.sh 
[root@localhost ~]# sh configura.sh 
[root@localhost ~]# ls
anaconda-ks.cfg  configura.sh  devops2026  evidencia-examen.sh  evidencia-martin-quintero.txt  reporte.txt  script-evidencia.sh  sitio
[root@localhost ~]# cat /home/ansible/config.txt 
zona=aves
zona=aves
zona=aves
[root@localhost ~]# 
```

efectivamente, tres veces hizo lo mismo


