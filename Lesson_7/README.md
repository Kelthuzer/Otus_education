# Размещаем свой RPM в своем репозитории
## Создать свой RPM пакет
- щагрузим требуемы для выполнения задания пакеты
  
  ``` bash
  yum install -y wget rpmdevtools rpm-build createrepo \
   yum-utils cmake gcc git nano
  ```
  
- создадим директирию для исходников и загрузим их
  
  ``` bash
  mkdir rpm && cd rpm
  yumdownloader --source nginx
  ```
  
- результат выполенния команд <br>
  <img width="1899" height="361" alt="image" src="https://github.com/user-attachments/assets/81a9e44e-8b09-44fa-9bab-4cdfffa6a335" /> <br>

- поставим все зависимости для сборки пакета nginx
  
  ``` bash
  rpm -Uvh nginx*.src.rpm
  yum-builddep nginx
  ```
- получаем <br>
  <img width="1902" height="280" alt="image" src="https://github.com/user-attachments/assets/d7337a17-c2b9-4fab-8153-04c335321625" /> <br>
- качаем исходный кот модуля `ngx_brotli`
  
  ``` bash
  cd /root
  git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
  cd ngx_brotli/deps/brotli
  mkdir out && cd out
  ```
  на выходе
  <img width="956" height="325" alt="image" src="https://github.com/user-attachments/assets/68cee7ad-3ac3-4a43-9e47-f809e78f1d2f" /> <br>
- Собираем модуль
  
  ``` bash
  cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
  cmake --build . --config Release -j 2 --target brotlienc
  cd ../../../..
  ```
  
  <details>
    <summary>Результат выполенения  </summary>
    <img width="881" height="910" alt="Снимок экрана 2026-05-26 140712" src="https://github.com/user-attachments/assets/f555fc78-ce43-4158-b62b-dba7f8f98fe1" />
  </details>  
  
- правим `spec` для nginx
  
  <img width="658" height="318" alt="image" src="https://github.com/user-attachments/assets/7e422346-ee46-484c-9cce-0639aaccf4b4" />

- собираем пакет (я уже был в нужной дериктории т.к правил спек)
  `rpmbuild -ba nginx.spec -D 'debug_package %{nil}'`
  Результат
  ``` bash
  Выполняется(%clean): /bin/sh -e /var/tmp/rpm-tmp.e6zcQf
  + umask 022
  + cd /root/rpmbuild/BUILD
  + cd nginx-1.20.1
  + /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.20.1-24.el9.3.alma.1.x86_64
  + RPM_EC=0
  ++ jobs -p
  + exit 0
  ```

 - проверим результат `ll /rpmbuild/RPMS/x86_64/`
   <img width="865" height="168" alt="image" src="https://github.com/user-attachments/assets/2f3aa911-181b-4498-9472-4905a923de46" />
- Копируем пакеты в общий каталог и убедим что nginx работает
  ``` bash
  cp ~/rpmbuild/RPMS/noarch/* ~/rpmbuild/RPMS/x86_64
  cd ~/rpmbuild/RPMS/x86_64
  yum localinstall *.rpm
  systemctl start nginx
  systemctl status nginx
  ```
```Результат
  Выполнено!
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-05-26 14:27:28 EET; 7ms ago
    Process: 6284 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 6301 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 6375 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 6420 (nginx)
      Tasks: 5 (limit: 35135)
     Memory: 10.8M (peak: 11.6M)
        CPU: 32ms
     CGroup: /system.slice/nginx.service
             ├─6420 "nginx: master process /usr/sbin/nginx"
             ├─6421 "nginx: worker process"
             ├─6422 "nginx: worker process"
             ├─6423 "nginx: worker process"
             └─6424 "nginx: worker process"

мая 26 14:27:28 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
мая 26 14:27:28 localhost.localdomain nginx[6301]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
мая 26 14:27:28 localhost.localdomain nginx[6301]: nginx: configuration file /etc/nginx/nginx.conf test is successful
мая 26 14:27:28 localhost.localdomain systemd[1]: Started The nginx HTTP and reverse proxy server.
```
## Создать свой репозиторий и разместить там ранее собранный RPM  
- Создаем директририю для репозитория, скопируем ранее созданные пакеты, и запустим репозиторий
```bash
mkdir /usr/share/nginx/html/repo
cp ~/rpmbuild/RPMS/x86_64/*.rpm /usr/share/nginx/html/repo/
createrepo /usr/share/nginx/html/repo/
```
- получим
  <img width="531" height="142" alt="image" src="https://github.com/user-attachments/assets/b9da0465-c4de-4420-825e-6bfb137ea640" />
- поправим конфиг для nginx для автоматическго индексирвоания `nano /etc/nginx/nginx.conf` ищем сервер и добавлем после `root` (чисто истетически)
  
  ```
  index index.html index.htm;
	autoindex on;
  ```
- поверяем что конфиг не поиблся `nginx -t` если
  ```
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
  ```  
  то перезагружам nginx `systemctl restart nginx.service`
- првоерим что получилось `lynx http://localhost/repo/` попросит установить пакет соглашаемся
  и нас встерчает наш репозиторий
  <img width="1037" height="314" alt="image" src="https://github.com/user-attachments/assets/611ef538-227f-49cc-a41b-197492636777" />
- добавим наш репозиторий
 ```bash
cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
```  
<img width="535" height="164" alt="image" src="https://github.com/user-attachments/assets/223c2ced-5dd4-4480-b6f2-f88b8477b611" />

- добавим пакет `percona` в наш репозиторий  
  ```bash
  cd /usr/share/nginx/html/repo/
  wget https://repo.percona.com/yum/percona-release-latest.noarch.rpm
  createrepo /usr/share/nginx/html/repo/
  yum makecache
  yum list | grep otus
  ```
  <img width="1046" height="497" alt="image" src="https://github.com/user-attachments/assets/cd13fc19-c2c4-44c2-a4d2-c6aea17a8b37" />
- установим пакет из нашего репозитория  
`yum install -y percona-release.noarch`  

<img width="1900" height="796" alt="image" src="https://github.com/user-attachments/assets/44d6ec72-a26a-465c-aad3-71ad3959e2be" />

