# NFS
С помощью vagrant  поднимаем две виртуальные машины (nfss - сервер nfs, nfsc - клиент):
```
vagrant up
```
# Настраиваем сервер NFS 
Заходим на сервер 
```
vagrant ssh nfss
```
NFS сервер уже установлен в CentOS 7, нам необходимо до установить утилиты для отладки:
```
yum install nfs-utils
``
Далле включаем firewall и проверяем, его статус 
```
systemctl enable firewalld --now 
systemctl status firewalld 
```
Разрешаем в firewall доступ к сервисам NFS:
```
firewall-cmd --add-service="nfs3" -add-service="rpc-bind" --add-service="mountd" --permanent
firewall-cmd --reload
```
Включаем сервер NFS
```
systemctl enable nfs --now 
```
Проверяем наличие слушайемых портов (2049/udp,2049/tcp,20048/udp,20048/tcp,111/udp,111/tcp)
```
ss -tnplu
```
Создаем и настраиваем директорию
```
mkdir -p /srv/share/upload
chown -R nfsnobody: /srv/share
chmod 0777 /srv/share/upload

В файле  `/etc/exports/` , добавляем структуру, которая позвполит экспортировать созданную директорию.
```
cat << EOF > /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
EOF
```
Экспортируем созданную директорию 
```
exportfs -r
```
# Настраиваем клиент NFS
Заходим на клиент 
```
vagrant ssh nfsc
```
Устанавливаем утилиты 
``
yum install nfs-utils
``
Далле включаем firewall и проверяем, его статус 
```
systemctl enable firewalld --now 
systemctl status firewalld 
```

Добавляем в файл `/etc/fstab` строку:
```
echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0" >> /etc/fstab
systemctl daemon-reload
systemctl restart remote-fs.target
```
Далее переходим в директорию `/mnt` и проверяем монтирования:
```
mount | grep mnt
```


