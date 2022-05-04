#### 1. Приготовьте свой собственный inventory файл prod.yml  
[playbook/inventory/prod.yml](playbook/inventory/prod.yml)  


#### 2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector https://vector.dev  
В [playbook/site.yml](playbook/site.yml) добавлен блок task'ов с `tags=vector`, устанавливающий 
[vector](https://vector.dev) из deb-пакетов:  

<details>
    <summary>ansible-playbook -i inventory/prod.yml site.yml --tags vector</summary>

```shell
vagrant@test-netology:/playbook$ ansible-playbook -i inventory/prod.yml site.yml --tags vector

PLAY [Install Clickhouse & Vector] *****************************************************************************************************************

TASK [Vector. Create work directory] ***************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Get Vector distributive] *************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Unzip archive] ***********************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Install vector binary file] **********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Check Vector installation] ***********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create Vector config vector.toml] ****************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create vector.service daemon] ********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Modify vector.service file] **********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create user vector] ******************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create data_dir] *********************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Remove work directory] ***************************************************************************************************************
changed: [clickhouse-01]

RUNNING HANDLER [Start Vector service] *************************************************************************************************************
changed: [clickhouse-01]

PLAY RECAP *****************************************************************************************************************************************
clickhouse-01              : ok=12   changed=12   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>
  

Также переработан блок task'ов с `tags=clickhouse`, устанавливающий Clickhouse из deb-пакетов на Ubuntu вместо rpm-пакетов:  

<details>
    <summary>ansible-playbook -i inventory/prod.yml site.yml --tags clickhouse</summary>

```shell
vagrant@test-netology:/playbook$ ansible-playbook -i inventory/prod.yml site.yml --tags clickhouse

PLAY [Install Clickhouse & Vector] *****************************************************************************************************************

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static_22.3.3.44_all.deb", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/deb/pool/stable/clickhouse-common-static_22.3.3.44_all.deb"}

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install package clickhouse-common-static] ****************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Install package clickhouse-client] ***********************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Install clickhouse package clickhouse-server] ************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Flush handlers] ******************************************************************************************************************

RUNNING HANDLER [Start clickhouse service] *********************************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Waiting while clickhouse-server is available...] *********************************************************************************
Pausing for 10 seconds (output is hidden)
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Clickhouse. Create database] *****************************************************************************************************************
changed: [clickhouse-01]

PLAY RECAP *****************************************************************************************************************************************
clickhouse-01              : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
```

</details>


Написан [playbook/deinstall.yml](playbook/deinstall.yml) для деинсталяции Clickhouse и Vector. Может использоваться 
как целиком, так и по отдельности с `tags=clickhouse` и `tags=vector`, соответственно:  

<details>
    <summary>ansible-playbook -i inventory/prod.yml deinstall.yml --tags=clickhouse</summary>

```shell
vagrant@test-netology:/playbook$ ansible-playbook -i inventory/prod.yml deinstall.yml --tags=clickhouse

PLAY [Deinstall Clickhouse & Vector] ***************************************************************************************************************

TASK [Clickhouse. Drop database logs] **************************************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Stop clickhouse-server.service daemon] *******************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Deinstall packages] **************************************************************************************************************
changed: [clickhouse-01]

RUNNING HANDLER [Systemd Daemon Reload] ************************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP *****************************************************************************************************************************************
clickhouse-01              : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>



<details>
    <summary>ansible-playbook -i inventory/prod.yml deinstall.yml --tags=vector</summary>

```shell
vagrant@test-netology:/playbook$ ansible-playbook -i inventory/prod.yml deinstall.yml --tags=vector

PLAY [Deinstall Clickhouse & Vector] ***************************************************************************************************************

TASK [Vector. Stop vector.service daemon] **********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Deinstall vector binary file] ********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Remove Vector config] ****************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Remove vector.service daemon] ********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Remove Vector data_dir] **************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Remove user vector] ******************************************************************************************************************
changed: [clickhouse-01]

RUNNING HANDLER [Systemd Daemon Reload] ************************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP *****************************************************************************************************************************************
clickhouse-01              : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>



#### 3. При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.  

Использованы модули:  
1. `ansible.builtin.get_url`  
2. `ansible.builtin.apt`  
3. `ansible.builtin.meta`  
4. `ansible.builtin.pause`  
5. `ansible.builtin.command`  
6. `ansible.builtin.file`  
7. `ansible.builtin.unarchive`  
8. `ansible.builtin.copy`  
9. `ansible.builtin.replace`  
10. `ansible.builtin.user`  
11. `ansible.builtin.service`  
12. `ansible.builtin.systemd`  



#### 4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector.

Требуемые task'и есть в [playbook/site.yml](playbook/site.yml).  


#### 5. Запустите ansible-lint site.yml и исправьте ошибки, если они есть.

```shell
vagrant@test-netology:/playbook$ ansible-lint
WARNING: PATH altered to include /usr/bin

vagrant@test-netology:/playbook$ ansible-lint site.yml
WARNING: PATH altered to include /usr/bin
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml

vagrant@test-netology:/playbook$ ansible-lint deinstall.yml
WARNING: PATH altered to include /usr/bin
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: deinstall.yml
```  


#### 6. Попробуйте запустить playbook на этом окружении с флагом --check.  

Имитация выполнения playbook "спотыкается" на зависимости при установке второго из трех пакета Clickhouse:   

<details>
    <summary>ansible-playbook -i inventory/prod.yml site.yml --check</summary>

```shell
vagrant@test-netology:/playbook$ ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Clickhouse & Vector] *****************************************************************************************************************

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static_22.3.3.44_all.deb", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/deb/pool/stable/clickhouse-common-static_22.3.3.44_all.deb"}

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install package clickhouse-common-static] ****************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Install package clickhouse-client] ***********************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"changed": false, "msg": "Dependency is not satisfiable: clickhouse-common-static (= 22.3.3.44)\n"}

RUNNING HANDLER [Start clickhouse service] *********************************************************************************************************

PLAY RECAP *****************************************************************************************************************************************
clickhouse-01              : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0
```

</details>



#### 7. Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.  

<details>
    <summary>ansible-playbook -i inventory/prod.yml site.yml --diff</summary>

```shell
vagrant@test-netology:/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse & Vector] *****************************************************************************************************************

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static_22.3.3.44_all.deb", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/deb/pool/stable/clickhouse-common-static_22.3.3.44_all.deb"}

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install package clickhouse-common-static] ****************************************************************************************
Selecting previously unselected package clickhouse-common-static.
(Reading database ... 40625 files and directories currently installed.)
Preparing to unpack .../clickhouse-common-static_22.3.3.44_amd64.deb ...
Unpacking clickhouse-common-static (22.3.3.44) ...
Setting up clickhouse-common-static (22.3.3.44) ...
changed: [clickhouse-01]

TASK [Clickhouse. Install package clickhouse-client] ***********************************************************************************************
Selecting previously unselected package clickhouse-client.
(Reading database ... 40639 files and directories currently installed.)
Preparing to unpack .../clickhouse-client_22.3.3.44_all.deb ...
Unpacking clickhouse-client (22.3.3.44) ...
Setting up clickhouse-client (22.3.3.44) ...
changed: [clickhouse-01]

TASK [Clickhouse. Install clickhouse package clickhouse-server] ************************************************************************************
Selecting previously unselected package clickhouse-server.
(Reading database ... 40650 files and directories currently installed.)
Preparing to unpack .../clickhouse-server_22.3.3.44_all.deb ...
Unpacking clickhouse-server (22.3.3.44) ...
Setting up clickhouse-server (22.3.3.44) ...
Processing triggers for systemd (245.4-4ubuntu3.13) ...
changed: [clickhouse-01]

TASK [Clickhouse. Flush handlers] ******************************************************************************************************************

RUNNING HANDLER [Start clickhouse service] *********************************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Waiting while clickhouse-server is available...] *********************************************************************************
Pausing for 10 seconds (output is hidden)
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Clickhouse. Create database] *****************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create work directory] ***************************************************************************************************************
--- before
+++ after
@@ -1,5 +1,5 @@
 {
-    "mode": "0775",
+    "mode": "0755",
     "path": "/home/vagrant/vector",
-    "state": "absent"
+    "state": "directory"
 }

changed: [clickhouse-01]

TASK [Vector. Get Vector distributive] *************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Unzip archive] ***********************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Install vector binary file] **********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Check Vector installation] ***********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create Vector config vector.toml] ****************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create vector.service daemon] ********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Modify vector.service file] **********************************************************************************************************
--- before: /lib/systemd/system/vector.service
+++ after: /lib/systemd/system/vector.service
@@ -8,7 +8,7 @@
 User=vector
 Group=vector
 ExecStartPre=/usr/bin/vector validate
-ExecStart=/usr/bin/vector
+ExecStart=/usr/bin/vector --config /etc/vector/vector.toml
 ExecReload=/usr/bin/vector validate
 ExecReload=/bin/kill -HUP $MAINPID
 Restart=no

changed: [clickhouse-01]

TASK [Vector. Create user vector] ******************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create data_dir] *********************************************************************************************************************
--- before
+++ after
@@ -1,6 +1,6 @@
 {
-    "group": 0,
-    "owner": 0,
+    "group": 1001,
+    "owner": 1001,
     "path": "/var/lib/vector",
-    "state": "absent"
+    "state": "directory"
 }

changed: [clickhouse-01]

TASK [Vector. Remove work directory] ***************************************************************************************************************
--- before
+++ after
@@ -1,42 +1,4 @@
 {
     "path": "/home/vagrant/vector",
-    "path_content": {
-        "directories": [
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/bin",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sources",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sinks",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/transforms"
-        ],
-        "files": [
-            "/home/vagrant/vector/vector-0.21.1-x86_64-unknown-linux-gnu.tar.gz",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/README.md",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/LICENSE",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/bin/vector",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd/hardened-vector.service",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd/vector.default",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd/vector.service",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/vector.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/wrapped_json.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/es_s3_hybrid.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/file_to_prometheus.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/prometheus_to_console.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/docs_example.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/environment_variables.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/file_to_cloudwatch_metrics.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/stdio.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/vector.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sources/apache_logs.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sinks/s3_archives.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sinks/es_cluster.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/transforms/apache_sample.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/transforms/apache_parser.toml"
-        ]
-    },
-    "state": "directory"
+    "state": "absent"
 }

changed: [clickhouse-01]

RUNNING HANDLER [Start Vector service] *************************************************************************************************************
changed: [clickhouse-01]

PLAY RECAP *****************************************************************************************************************************************
clickhouse-01              : ok=19   changed=17   unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
```

</details>  



<details>
    <summary>Проверяем изменения на хосте clickhouse-01 (192.168.10.10):</summary>

```shell
vagrant@clickhouse-01:~$ ip -br a
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::a00:27ff:feb1:285d/64
eth1             DOWN
eth2             UP             192.168.10.10/24 fe80::a00:27ff:fee7:7655/64

vagrant@clickhouse-01:~$ id vector
uid=1001(vector) gid=1001(vector) groups=1001(vector)

vagrant@clickhouse-01:~$ vector --version
vector 0.21.1 (x86_64-unknown-linux-gnu 18787c0 2022-04-22)

vagrant@clickhouse-01:~$ systemctl status clickhouse-server
● clickhouse-server.service - ClickHouse Server (analytic DBMS for big data)
     Loaded: loaded (/lib/systemd/system/clickhouse-server.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-05-04 13:36:13 UTC; 10min ago
   Main PID: 34202 (clckhouse-watch)
      Tasks: 206 (limit: 467)
     Memory: 309.1M
     CGroup: /system.slice/clickhouse-server.service
             ├─34202 clickhouse-watchdog        --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid
             └─34207 /usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid

May 04 13:36:13 clickhouse-01 systemd[1]: Started ClickHouse Server (analytic DBMS for big data).
May 04 13:36:13 clickhouse-01 clickhouse-server[34202]: Processing configuration file '/etc/clickhouse-server/config.xml'.
May 04 13:36:13 clickhouse-01 clickhouse-server[34202]: Logging trace to /var/log/clickhouse-server/clickhouse-server.log
May 04 13:36:13 clickhouse-01 clickhouse-server[34202]: Logging errors to /var/log/clickhouse-server/clickhouse-server.err.log
May 04 13:36:15 clickhouse-01 clickhouse-server[34207]: Processing configuration file '/etc/clickhouse-server/config.xml'.
May 04 13:36:15 clickhouse-01 clickhouse-server[34207]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/config.xml'.
May 04 13:36:15 clickhouse-01 clickhouse-server[34207]: Processing configuration file '/etc/clickhouse-server/users.xml'.
May 04 13:36:15 clickhouse-01 clickhouse-server[34207]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/users.xml'.

vagrant@clickhouse-01:~$ clickhouse-client
ClickHouse client version 22.3.3.44 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 22.3.3 revision 54455.

clickhouse-01 :) SHOW DATABASES;

SHOW DATABASES

Query id: 9407ebb7-af5e-494e-b20f-ae617dd8766d

┌─name───────────────┐
│ INFORMATION_SCHEMA │
│ default            │
│ information_schema │
│ logs               │
│ system             │
└────────────────────┘

5 rows in set. Elapsed: 0.063 sec.

clickhouse-01 :) q
Bye.

vagrant@clickhouse-01:~$
```

</details>




#### 8. Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.  

<details>
    <summary>ansible-playbook -i inventory/prod.yml site.yml --diff</summary>

```shell
vagrant@test-netology:/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse & Vector] *****************************************************************************************************************

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static_22.3.3.44_all.deb", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/deb/pool/stable/clickhouse-common-static_22.3.3.44_all.deb"}

TASK [Clickhouse. Get clickhouse distrib] **********************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install package clickhouse-common-static] ****************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install package clickhouse-client] ***********************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse package clickhouse-server] ************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Flush handlers] ******************************************************************************************************************

TASK [Clickhouse. Waiting while clickhouse-server is available...] *********************************************************************************
Pausing for 10 seconds (output is hidden)
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Clickhouse. Create database] *****************************************************************************************************************
ok: [clickhouse-01]

TASK [Vector. Create work directory] ***************************************************************************************************************
--- before
+++ after
@@ -1,5 +1,5 @@
 {
-    "mode": "0775",
+    "mode": "0755",
     "path": "/home/vagrant/vector",
-    "state": "absent"
+    "state": "directory"
 }

changed: [clickhouse-01]

TASK [Vector. Get Vector distributive] *************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Unzip archive] ***********************************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Install vector binary file] **********************************************************************************************************
ok: [clickhouse-01]

TASK [Vector. Check Vector installation] ***********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Create Vector config vector.toml] ****************************************************************************************************
ok: [clickhouse-01]

TASK [Vector. Create vector.service daemon] ********************************************************************************************************
changed: [clickhouse-01]

TASK [Vector. Modify vector.service file] **********************************************************************************************************
--- before: /lib/systemd/system/vector.service
+++ after: /lib/systemd/system/vector.service
@@ -8,7 +8,7 @@
 User=vector
 Group=vector
 ExecStartPre=/usr/bin/vector validate
-ExecStart=/usr/bin/vector
+ExecStart=/usr/bin/vector --config /etc/vector/vector.toml
 ExecReload=/usr/bin/vector validate
 ExecReload=/bin/kill -HUP $MAINPID
 Restart=no

changed: [clickhouse-01]

TASK [Vector. Create user vector] ******************************************************************************************************************
ok: [clickhouse-01]

TASK [Vector. Create data_dir] *********************************************************************************************************************
ok: [clickhouse-01]

TASK [Vector. Remove work directory] ***************************************************************************************************************
--- before
+++ after
@@ -1,42 +1,4 @@
 {
     "path": "/home/vagrant/vector",
-    "path_content": {
-        "directories": [
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/bin",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sources",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sinks",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/transforms"
-        ],
-        "files": [
-            "/home/vagrant/vector/vector-0.21.1-x86_64-unknown-linux-gnu.tar.gz",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/README.md",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/LICENSE",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/bin/vector",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd/hardened-vector.service",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd/vector.default",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/etc/systemd/vector.service",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/vector.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/wrapped_json.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/es_s3_hybrid.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/file_to_prometheus.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/prometheus_to_console.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/docs_example.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/environment_variables.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/file_to_cloudwatch_metrics.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/stdio.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/vector.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sources/apache_logs.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sinks/s3_archives.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/sinks/es_cluster.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/transforms/apache_sample.toml",
-            "/home/vagrant/vector/vector-x86_64-unknown-linux-gnu/config/examples/namespacing/transforms/apache_parser.toml"
-        ]
-    },
-    "state": "directory"
+    "state": "absent"
 }

changed: [clickhouse-01]

RUNNING HANDLER [Start Vector service] *************************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP *****************************************************************************************************************************************
clickhouse-01              : ok=18   changed=7    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
```

</details>  





#### 9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.  

[playbook/site.yml](playbook/site.yml) содержит 2 блока задач:  

1. Первый блок объединяет последовательность задач по инсталяции Clickhouse. Блоку соответствует тэг `clickhouse`. 
В блоке используются параметры:  
`clickhouse_version: "22.3.3.44"` - версия Clickhouse     
`clickhouse_packages: ["clickhouse-client", "clickhouse-server", "clickhouse-common-static"]` - список пакетов для установки    
Task'и:  
`TASK [Clickhouse. Get clickhouse distrib]` - скачивает deb-пакеты с дистрибутивами с помощью модуля `ansible.builtin.get_url`  
`TASK [Clickhouse. Install package clickhouse-common-static]` - устанавливает deb-пакет с помощью модуля `ansible.builtin.apt`   
`TASK [Clickhouse. Install package clickhouse-client]` - устанавливает deb-пакет с помощью модуля `ansible.builtin.apt`  
`TASK [Clickhouse. Install clickhouse package clickhouse-server]` - устанавливает deb-пакеты с помощью модуля `ansible.builtin.apt`   
`TASK [Clickhouse. Flush handlers]` - инициирует внеочередной запуск хандлера `Start clickhouse service`   
`RUNNING HANDLER [Start clickhouse service]` - для старта сервера `clickhouse` в хандлере используется модуль `ansible.builtin.service`  
`TASK [Clickhouse. Waiting while clickhouse-server is available...]` - устанавливает паузу в 10 секунд с помощью модуля `ansible.builtin.pause`, чтобы сервер Clickhouse успел запуститься. Иначе следующая задача по созданию БД может завершиться ошибкой, т.к. сервер еще не успел подняться    
`TASK [Clickhouse. Create database]` - создает инстанс базы данных Clickhouse  
  

2. Второй блок объединяет последовательность задач по инсталяции Vector. Блоку соответствует тэг `vector`. 
В блоке используются параметры:  
`vector_version: "0.21.1"` - версия Vector  
`vector_os_arh: "x86_64"` - архитектура ОС  
`vector_workdir: "/home/vagrant/vector"` - рабочий каталог, в котором будут сохранены скачанные deb-пакеты  
`vector_os_user: "vector"` - имя пользователя-владельца Vector в ОС  
`vector_os_group: "vector"` - имя группы пользователя-владельца Vector в ОС  
Task'и:  
`TASK [Vector. Create work directory]` - создает рабочий каталог, в котором будут сохранены скачанные deb-пакеты, с помощью модуля `ansible.builtin.file`  
`TASK [Vector. Get Vector distributive]` - скачивает архив с дистрибутивом с помощью модуля `ansible.builtin.get_url`  
`TASK [Vector. Unzip archive]` - распаковывает скачанный архив с помощью модуля `ansible.builtin.unarchive`  
`TASK [Vector. Install vector binary file]` - копирует исполняемый файл Vector в `/usr/bin` с помощью модуля `ansible.builtin.copy`  
`TASK [Vector. Check Vector installation]` - проверяет, что бинарный файл Vector работает корректно, с помощью модуля `ansible.builtin.command`  
`TASK [Vector. Create Vector config vector.toml]` - создает файл `/etc/vector/vector.toml` с конфигом Vector с помощью модуля `ansible.builtin.copy`  
`TASK [Vector. Create vector.service daemon]` - создает файл юнита systemd `/lib/systemd/system/vector.service` с помощью модуля `ansible.builtin.copy`  
`TASK [Vector. Modify vector.service file]` - редактирует файл юнита systemd `/lib/systemd/system/vector.service` с помощью модуля `ansible.builtin.replace`  
`TASK [Vector. Create user vector]` - создает пользователя ОС с помощью модуля `ansible.builtin.user`    
`TASK [Vector. Create data_dir]` - создает каталог дял данных Vector с помощью модуля `ansible.builtin.file`  
`TASK [Vector. Remove work directory]` - удаляет рабочий каталог с помощью модуля `ansible.builtin.file`  
`RUNNING HANDLER [Start Vector service]` - инициируется запуск хандлера `Start Vector service`, обновляющего конфигурацию systemd и стартующего сервис `vector.service` с помощью модуля`ansible.builtin.systemd`   