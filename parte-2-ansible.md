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

sintaxis básica de un comando ad-hoc

```shellsession
ansible <hosts> -m <módulo> -a "<argumentos>" [-b]
```

#### 1. Conectividad: prueba ssh y ping en el nodo

```shellsession
[root@localhost ansible]# ansible all -m ansible.builtin.ping
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
[root@localhost ansible]# 
```

funcionó

#### 2. Facts: Información que puede obtenera ansible sobre el nodo

```shellsession
[root@localhost ansible]# ansible nodo1 -m ansible.builtin.setup -a "filter=ansible_distribution*"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Rocky",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "9",
        "ansible_distribution_release": "Blue Onyx",
        "ansible_distribution_version": "9.8",
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
[root@localhost ansible]# ansible nodo1 -m ansible.builtin.setup -a "filter=ansible_memtotal_mb"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "ansible_memtotal_mb": 1707,
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
[root@localhost ansible]# ansible nodo1 -m ansible.builtin.command -a "df -h /"
nodo1 | CHANGED | rc=0 >>
S.ficheros          Tamaño Usados  Disp Uso% Montado en
/dev/mapper/rl-root    16G   2.3G   14G  15% /
```

Valla

#### 3. Instalar un paquete

Esta primera ha de fallar

```shellsession
[root@localhost ansible]# ansible web -m ansible.builtin.dnf -a "name=tree state=present"
nodo1 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "This command has to be run under the root user.",
    "results": []
}
[root@localhost ansible]# ansible web -b -m ansible.builtin.dnf -a "name=tree state=present"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
[root@localhost ansible]# 
```

bueno no salio exactamente como se esperaba

##### pasos para que ansible tenga privilegios adminstrativos en el nodo

```shellsession
[root@localhost ansible]# echo 'ansible ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/ansible
[root@localhost ansible]# chmod 0440 /etc/sudoers.d/ansible
[root@localhost ansible]# visudo -cf /etc/sudoers.d/ansible
/etc/sudoers.d/ansible: parsed OK
[root@localhost ansible]# 
```

bien, paso la prueba de visudo

ahora si con ansible

```shellsession
[root@localhost ansible]# ansible web -b -m ansible.builtin.dnf -a "name=tree state=present"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
[root@localhost ansible]# ansible web -b -m ansible.builtin.dnf -a "name=tree state=present"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
[root@localhost ansible]# 
```

Ha funcionado

#### 4. Comprobar como un modulo si es idempotente a diferencia de la shell

```shellsession
[root@localhost ansible]# ansible web -m ansible.builtin.shell -a "echo 'zona=aves' >> ~/notas.txt"
nodo1 | CHANGED | rc=0 >>

[root@localhost ansible]# ansible web -m ansible.builtin.shell -a "echo 'zona=aves' >> ~/notas.txt"
nodo1 | CHANGED | rc=0 >>

[root@localhost ansible]# ansible web -m ansible.builtin.shell -a "echo 'zona=aves' >> ~/notas.txt"
nodo1 | CHANGED | rc=0 >>

[root@localhost ansible]# ansible web -m ansible.builtin.lineinfile -a "path=/home/ansible/notas2.txt line='zona=aves' create=true"
nodo1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "backup": "",
    "changed": true,
    "msg": "line added"
}
[root@localhost ansible]# ansible web -m ansible.builtin.lineinfile -a "path=/home/ansible/notas2.txt line='zona=aves' create=true"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "backup": "",
    "changed": false,
    "msg": ""
}
[root@localhost ansible]# ansible web -m ansible.builtin.lineinfile -a "path=/home/ansible/notas2.txt line='zona=aves' create=true"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "backup": "",
    "changed": false,
    "msg": ""
}
[root@localhost ansible]# ssh ansible@nodo1 'cat ~/notas.txt; echo ---; cat ~/notas2.txt'
zona=aves
zona=aves
zona=aves
---
zona=aves
[root@localhost ansible]# 

```

Efectivamente, un script de shell no es idempotente, a diferencia de un modulo se ansible que si lo es.

#### 5. Archivos, usuarios y servicios

```shellsession
[root@localhost ansible]# ansible web -m ansible.builtin.copy -a "src=/root/reporte.txt dest=/home/ansible/reporte-ansible.txt mode=0640"
nodo1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum": "de403ee2581c4d47f6621b4c92e9b9e246003cb2",
    "dest": "/home/ansible/reporte-ansible.txt",
    "gid": 8006,
    "group": "ansible",
    "md5sum": "e91e656a1c829943186ebb6e74772bc0",
    "mode": "0640",
    "owner": "ansible",
    "secontext": "unconfined_u:object_r:user_home_t:s0",
    "size": 47,
    "src": "/home/ansible/.ansible/tmp/ansible-tmp-1793325250.9809897-7334-134445662551160/source",
    "state": "file",
    "uid": 8005
}
[root@localhost ansible]# ansible web -b -m ansible.builtin.file -a "path=/opt/zoologico state=directory mode=0755"
nodo1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "gid": 8002,
    "group": "cuidadores",
    "mode": "0755",
    "owner": "root",
    "path": "/opt/zoologico",
    "secontext": "unconfined_u:object_r:usr_t:s0",
    "size": 6,
    "state": "directory",
    "uid": 0
}
[root@localhost ansible]# ansible web -b -m ansible.builtin.user -a "name=koala comment='Cuidador koalas'"
nodo1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "append": false,
    "changed": true,
    "comment": "Cuidador koalas",
    "group": 8003,
    "home": "/home/koala",
    "move_home": false,
    "name": "koala",
    "shell": "/bin/bash",
    "state": "present",
    "uid": 8002
}
[root@localhost ansible]# ansible web -b -m ansible.builtin.user -a "name=koala state=absent remove=true"
nodo1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "force": false,
    "name": "koala",
    "remove": true,
    "state": "absent"
}
[root@localhost ansible]# ansible web -b -m ansible.builtin.service -a "name=sshd state=started enabled=true"
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "enabled": true,
    "name": "sshd",
    "state": "started",
    "status": {
        "AccessSELinuxContext": "system_u:object_r:sshd_unit_file_t:s0",
        "ActiveEnterTimestamp": "Thu 2026-10-29 17:25:24 CST",
        "ActiveEnterTimestampMonotonic": "7012491",
        "ActiveExitTimestampMonotonic": "0",
        "ActiveState": "active",
        "After": "network.target systemd-journald.socket basic.target system.slice sshd-keygen.target sysinit.target",
        "AllowIsolate": "no",
        "AssertResult": "yes",
        "AssertTimestamp": "Thu 2026-10-29 17:25:24 CST",
        "AssertTimestampMonotonic": "6926429",
        "Before": "shutdown.target multi-user.target",
        "BlockIOAccounting": "no",
        "BlockIOWeight": "[not set]",
        "CPUAccounting": "yes",
        "CPUAffinityFromNUMA": "no",
        "CPUQuotaPerSecUSec": "infinity",
        "CPUQuotaPeriodUSec": "infinity",
        "CPUSchedulingPolicy": "0",
        "CPUSchedulingPriority": "0",
        "CPUSchedulingResetOnFork": "no",
        "CPUShares": "[not set]",
        "CPUUsageNSec": "3827094000",
        "CPUWeight": "[not set]",
        "CacheDirectoryMode": "0755",
        "CanFreeze": "yes",
        "CanIsolate": "no",
        "CanReload": "yes",
        "CanStart": "yes",
        "CanStop": "yes",
        "CapabilityBoundingSet": "cap_chown cap_dac_override cap_dac_read_search cap_fowner cap_fsetid cap_kill cap_setgid cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config cap_mknod cap_lease cap_audit_write cap_audit_control cap_setfcap cap_mac_override cap_mac_admin cap_syslog cap_wake_alarm cap_block_suspend cap_audit_read cap_perfmon cap_bpf cap_checkpoint_restore",
        "CleanResult": "success",
        "CollectMode": "inactive",
        "ConditionResult": "yes",
        "ConditionTimestamp": "Thu 2026-10-29 17:25:24 CST",
        "ConditionTimestampMonotonic": "6926420",
        "ConfigurationDirectoryMode": "0755",
        "Conflicts": "shutdown.target",
        "ConsistsOf": "sshd-keygen.target",
        "ControlGroup": "/system.slice/sshd.service",
        "ControlGroupId": "2977",
        "ControlPID": "0",
        "CoredumpFilter": "0x33",
        "DefaultDependencies": "yes",
        "DefaultMemoryLow": "0",
        "DefaultMemoryMin": "0",
        "Delegate": "no",
        "Description": "OpenSSH server daemon",
        "DevicePolicy": "auto",
        "Documentation": "\"man:sshd(8)\" \"man:sshd_config(5)\"",
        "DynamicUser": "no",
        "EnvironmentFiles": "/etc/sysconfig/sshd (ignore_errors=yes)",
        "ExecMainCode": "0",
        "ExecMainExitTimestampMonotonic": "0",
        "ExecMainPID": "858",
        "ExecMainStartTimestamp": "Thu 2026-10-29 17:25:24 CST",
        "ExecMainStartTimestampMonotonic": "6936615",
        "ExecMainStatus": "0",
        "ExecReload": "{ path=/bin/kill ; argv[]=/bin/kill -HUP $MAINPID ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }",
        "ExecReloadEx": "{ path=/bin/kill ; argv[]=/bin/kill -HUP $MAINPID ; flags= ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }",
        "ExecStart": "{ path=/usr/sbin/sshd ; argv[]=/usr/sbin/sshd -D $OPTIONS ; ignore_errors=no ; start_time=[Thu 2026-10-29 17:25:24 CST] ; stop_time=[n/a] ; pid=858 ; code=(null) ; status=0/0 }",
        "ExecStartEx": "{ path=/usr/sbin/sshd ; argv[]=/usr/sbin/sshd -D $OPTIONS ; flags= ; start_time=[Thu 2026-10-29 17:25:24 CST] ; stop_time=[n/a] ; pid=858 ; code=(null) ; status=0/0 }",
        "ExitType": "main",
        "FailureAction": "none",
        "FileDescriptorStoreMax": "0",
        "FinalKillSignal": "9",
        "FragmentPath": "/usr/lib/systemd/system/sshd.service",
        "FreezerState": "running",
        "GID": "[not set]",
        "GuessMainPID": "yes",
        "IOAccounting": "no",
        "IOReadBytes": "18446744073709551615",
        "IOReadOperations": "18446744073709551615",
        "IOSchedulingClass": "2",
        "IOSchedulingPriority": "4",
        "IOWeight": "[not set]",
        "IOWriteBytes": "18446744073709551615",
        "IOWriteOperations": "18446744073709551615",
        "IPAccounting": "no",
        "IPEgressBytes": "[no data]",
        "IPEgressPackets": "[no data]",
        "IPIngressBytes": "[no data]",
        "IPIngressPackets": "[no data]",
        "Id": "sshd.service",
        "IgnoreOnIsolate": "no",
        "IgnoreSIGPIPE": "yes",
        "InactiveEnterTimestampMonotonic": "0",
        "InactiveExitTimestamp": "Thu 2026-10-29 17:25:24 CST",
        "InactiveExitTimestampMonotonic": "6936908",
        "InvocationID": "4bde42e2c2634d86a64accf58e4ec3bc",
        "JobRunningTimeoutUSec": "infinity",
        "JobTimeoutAction": "none",
        "JobTimeoutUSec": "infinity",
        "KeyringMode": "private",
        "KillMode": "process",
        "KillSignal": "15",
        "LimitAS": "infinity",
        "LimitASSoft": "infinity",
        "LimitCORE": "infinity",
        "LimitCORESoft": "infinity",
        "LimitCPU": "infinity",
        "LimitCPUSoft": "infinity",
        "LimitDATA": "infinity",
        "LimitDATASoft": "infinity",
        "LimitFSIZE": "infinity",
        "LimitFSIZESoft": "infinity",
        "LimitLOCKS": "infinity",
        "LimitLOCKSSoft": "infinity",
        "LimitMEMLOCK": "8388608",
        "LimitMEMLOCKSoft": "8388608",
        "LimitMSGQUEUE": "819200",
        "LimitMSGQUEUESoft": "819200",
        "LimitNICE": "0",
        "LimitNICESoft": "0",
        "LimitNOFILE": "524288",
        "LimitNOFILESoft": "1024",
        "LimitNPROC": "6647",
        "LimitNPROCSoft": "6647",
        "LimitRSS": "infinity",
        "LimitRSSSoft": "infinity",
        "LimitRTPRIO": "0",
        "LimitRTPRIOSoft": "0",
        "LimitRTTIME": "infinity",
        "LimitRTTIMESoft": "infinity",
        "LimitSIGPENDING": "6647",
        "LimitSIGPENDINGSoft": "6647",
        "LimitSTACK": "infinity",
        "LimitSTACKSoft": "8388608",
        "LoadState": "loaded",
        "LockPersonality": "no",
        "LogLevelMax": "-1",
        "LogRateLimitBurst": "0",
        "LogRateLimitIntervalUSec": "0",
        "LogsDirectoryMode": "0755",
        "MainPID": "858",
        "ManagedOOMMemoryPressure": "auto",
        "ManagedOOMMemoryPressureLimit": "0",
        "ManagedOOMPreference": "none",
        "ManagedOOMSwap": "auto",
        "MemoryAccounting": "yes",
        "MemoryAvailable": "infinity",
        "MemoryCurrent": "9076736",
        "MemoryDenyWriteExecute": "no",
        "MemoryHigh": "infinity",
        "MemoryLimit": "infinity",
        "MemoryLow": "0",
        "MemoryMax": "infinity",
        "MemoryMin": "0",
        "MemoryPeak": "12337152",
        "MemorySwapMax": "infinity",
        "MountAPIVFS": "no",
        "NFileDescriptorStore": "0",
        "NRestarts": "0",
        "NUMAPolicy": "n/a",
        "Names": "sshd.service",
        "NeedDaemonReload": "no",
        "Nice": "0",
        "NoNewPrivileges": "no",
        "NonBlocking": "no",
        "NotifyAccess": "main",
        "OOMPolicy": "stop",
        "OOMScoreAdjust": "0",
        "OnFailureJobMode": "replace",
        "OnSuccessJobMode": "fail",
        "Perpetual": "no",
        "PrivateDevices": "no",
        "PrivateIPC": "no",
        "PrivateMounts": "no",
        "PrivateNetwork": "no",
        "PrivateTmp": "no",
        "PrivateUsers": "no",
        "ProcSubset": "all",
        "ProtectClock": "no",
        "ProtectControlGroups": "no",
        "ProtectHome": "no",
        "ProtectHostname": "no",
        "ProtectKernelLogs": "no",
        "ProtectKernelModules": "no",
        "ProtectKernelTunables": "no",
        "ProtectProc": "default",
        "ProtectSystem": "no",
        "RefuseManualStart": "no",
        "RefuseManualStop": "no",
        "ReloadResult": "success",
        "ReloadSignal": "1",
        "RemainAfterExit": "no",
        "RemoveIPC": "no",
        "Requires": "system.slice sysinit.target",
        "Restart": "on-failure",
        "RestartKillSignal": "15",
        "RestartUSec": "42s",
        "RestrictNamespaces": "no",
        "RestrictRealtime": "no",
        "RestrictSUIDSGID": "no",
        "Result": "success",
        "RootDirectoryStartOnly": "no",
        "RuntimeDirectoryMode": "0755",
        "RuntimeDirectoryPreserve": "no",
        "RuntimeMaxUSec": "infinity",
        "RuntimeRandomizedExtraUSec": "0",
        "SameProcessGroup": "no",
        "SecureBits": "0",
        "SendSIGHUP": "no",
        "SendSIGKILL": "yes",
        "Slice": "system.slice",
        "StandardError": "inherit",
        "StandardInput": "null",
        "StandardOutput": "journal",
        "StartLimitAction": "none",
        "StartLimitBurst": "5",
        "StartLimitIntervalUSec": "10s",
        "StartupBlockIOWeight": "[not set]",
        "StartupCPUShares": "[not set]",
        "StartupCPUWeight": "[not set]",
        "StartupIOWeight": "[not set]",
        "StateChangeTimestamp": "Thu 2026-10-29 17:25:24 CST",
        "StateChangeTimestampMonotonic": "7012491",
        "StateDirectoryMode": "0755",
        "StatusErrno": "0",
        "StopWhenUnneeded": "no",
        "SubState": "running",
        "SuccessAction": "none",
        "SyslogFacility": "3",
        "SyslogLevel": "6",
        "SyslogLevelPrefix": "yes",
        "SyslogPriority": "30",
        "SystemCallErrorNumber": "2147483646",
        "TTYReset": "no",
        "TTYVHangup": "no",
        "TTYVTDisallocate": "no",
        "TasksAccounting": "yes",
        "TasksCurrent": "1",
        "TasksMax": "10636",
        "TimeoutAbortUSec": "1min 30s",
        "TimeoutCleanUSec": "infinity",
        "TimeoutStartFailureMode": "terminate",
        "TimeoutStartUSec": "1min 30s",
        "TimeoutStopFailureMode": "terminate",
        "TimeoutStopUSec": "1min 30s",
        "TimerSlackNSec": "50000",
        "Transient": "no",
        "Type": "notify",
        "UID": "[not set]",
        "UMask": "0022",
        "UnitFilePreset": "enabled",
        "UnitFileState": "enabled",
        "UtmpMode": "init",
        "WantedBy": "multi-user.target",
        "Wants": "sshd-keygen.target",
        "WatchdogSignal": "6",
        "WatchdogTimestampMonotonic": "0",
        "WatchdogUSec": "0"
    }
}
[root@localhost ansible]# 

```

#### 6. La documentación vive en la terminal

```shellsession
[root@localhost ansible]# ansible-doc -l | wc -l
72
[root@localhost ansible]# ansible-doc ansible.builtin.user
> ANSIBLE.BUILTIN.USER    (/usr/lib/python3.9/site-packages/ansible/modules/user.py)

        Manage user accounts and user attributes. For Windows targets, use the [ansible.windows.win_user] module instead.

ADDED IN: version 0.2 of ansible-core

OPTIONS (= is mandatory):

- append
        If `true', add the user to the groups specified in `groups'.
        If `false', user will only be added to the groups specified in `groups', removing them from all other groups.
        default: false
        type: bool

- authorization
        Sets the authorization of the user.
        Does nothing when used with other platforms.
        Can set multiple authorizations using comma separation.
        To delete all authorizations, use `authorization='''.
        Currently supported on Illumos/Solaris.
        default: null
        type: str
        added in: version 2.8 of ansible-core


- comment
        Optionally sets the description (aka `GECOS') of user account.
        default: null
        type: str

- create_home
        Unless set to `false', a home directory will be made for the user when the account is created or if the home directory does not exist.
        Changed from `createhome' to `create_home' in Ansible 2.5.
        aliases: [createhome]
        default: true
        type: bool

- expires
        An expiry time for the user in epoch, it will be ignored on platforms that do not support this.
        Currently supported on GNU/Linux, FreeBSD, and DragonFlyBSD.
        Since Ansible 2.6 you can remove the expiry time by specifying a negative value. Currently supported on GNU/Linux and FreeBSD.
        default: null
        type: float
        added in: version 1.9 of ansible-core


- force
        This only affects `state=absent', it forces removal of the user and associated directories on supported platforms.
        The behavior is the same as `userdel --force', check the man page for `userdel' on your system for details and support.
[root@localhost ansible]# ansible-doc -s ansible.builtin.lineinfile
- name: Manage lines in text files
  lineinfile:
      attributes:            # The attributes the resulting filesystem object should have. To get supported flags look at the man page for `chattr' on the target system. This string should contain the attributes
                             # in the same order as the one displayed by `lsattr'. The `=' operator is assumed as default, otherwise `+' or `-' operators need to be included in the
                             # string.
      backrefs:              # Used with `state=present'. If set, `line' can contain backreferences (both positional and named) that will get populated if the `regexp' matches. This parameter changes the operati>
                             # of the module slightly; `insertbefore' and `insertafter' will be ignored, and if the `regexp' does not match anywhere in the file, the file will be
                             # left unchanged. If the `regexp' does match, the last matching line will be replaced by the expanded line parameter. Mutually exclusive with
                             # `search_string'.
      backup:                # Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.
      create:                # Used with `state=present'. If specified, the file will be created if it does not already exist. By default it will fail if the file is missing.
      firstmatch:            # Used with `insertafter' or `insertbefore'. If set, `insertafter' and `insertbefore' will work with the first line that matches the given regular expression.
      group:                 # Name of the group that should own the filesystem object, as would be fed to `chown'. When left unspecified, it uses the current group of the current user unless you are root, in
                             # which case it can preserve the previous ownership.
      insertafter:           # Used with `state=present'. If specified, the line will be inserted after the last match of specified regular expression. If the first match is required, use(firstmatch=yes). A
                             # special value is available; `EOF' for inserting the line at the end of the file. If specified regular expression has no matches, EOF will be used
                             # instead. If `insertbefore' is set, default value `EOF' will be ignored. If regular expressions are passed to both `regexp' and `insertafter',
                             # `insertafter' is only honored if no match for `regexp' is found. May not be used with `backrefs' or `insertbefore'.
      insertbefore:          # Used with `state=present'. If specified, the line will be inserted before the last match of specified regular expression. If the first match is required, use `firstmatch=yes'. A
                             # value is available; `BOF' for inserting the line at the beginning of the file. If specified regular expression has no matches, the line will be
                             # inserted at the end of the file. If regular expressions are passed to both `regexp' and `insertbefore', `insertbefore' is only honored if no match for
                             # `regexp' is found. May not be used with `backrefs' or `insertafter'.
      line:                  # The line to insert/replace into the file. Required for `state=present'. If `backrefs' is set, may contain backreferences that will get expanded with the `regexp' capture groups if
                             # the regexp matches.
      mode:                  # The permissions the resulting filesystem object should have. For those used to `/usr/bin/chmod' remember that modes are actually octal numbers. You must either add a leading zero so
                             # that Ansible's YAML parser knows it is an octal number (like `0644' or `01777') or quote it (like `'644'' or `'1777'') so Ansible receives a string and
                             # can do its own conversion from string into number. Giving Ansible a number without following one of these rules will end up with a decimal number which
                             # will have unexpected results. As of Ansible 1.8, the mode may be specified as a symbolic mode (for example, `u+rwx' or `u=rw,g=r,o=r'). If `mode' is
                             # not specified and the destination filesystem object *does not* exist, the default `umask' on the system will be used when setting the mode for the
                             # newly created filesystem object. If `mode' is not specified and the destination filesystem object *does* exist, the mode of the existing filesystem
                             # object will be used. Specifying `mode' is the best way to ensure filesystem objects are created with the correct permissions. See CVE-2020-1736 for
                             # further details.
      others:                # All arguments accepted by the [ansible.builtin.file] module also work here.
      owner:                 # Name of the user that should own the filesystem object, as would be fed to `chown'. When left unspecified, it uses the current user unless you are root, in which case it can preser>
                             # the previous ownership. Specifying a numeric username will be assumed to be a user ID and not a username. Avoid numeric usernames to avoid this
                             # confusion.
      path:                  # (required) The file to modify. Before Ansible 2.3 this option was only usable as `dest', `destfile' and `name'.
      regexp:                # The regular expression to look for in every line of the file. For `state=present', the pattern to replace if found. Only the last line found will be replaced. For `state=absent', t>
                             # pattern of the line(s) to remove. If the regular expression is not matched, the line will be added to the file in keeping with `insertbefore' or
                             # `insertafter' settings. When modifying a line the regexp should typically match both the initial state of the line as well as its state after
                             # replacement by `line' to ensure idempotence. Uses Python regular expressions. See https://docs.python.org/3/library/re.html.
      search_string:         # The literal string to look for in every line of the file. This does not have to match the entire line. For `state=present', the line to replace if the string is found in the file.
                             # Only the last line found will be replaced. For `state=absent', the line(s) to remove if the string is in the line. If the literal expression is not
                             # matched, the line will be added to the file in keeping with `insertbefore' or `insertafter' settings. Mutually exclusive with `backrefs' and `regexp'.
      selevel:               # The level part of the SELinux filesystem object context. This is the MLS/MCS attribute, sometimes known as the `range'. When set to `_default', it will use the `level' portion of t>
                             # policy if available.
[root@localhost ansible]#
```

Bastante información


