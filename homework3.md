- создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере  
<span style="color:green">создал ВМ в Yandex Cloud</span>  
- поставьте на нее PostgreSQL 15 через sudo apt  
<span style="color:green">sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15</span>  

- проверьте что кластер запущен через sudo -u postgres pg_lsclusters  
<span style="color:green">Ver Cluster Port Status Owner    Data directory              Log file</span>   
<span style="color:green">15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log</span>  

- зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым  
postgres=# create table test(c1 text);  
postgres=# insert into test values('1');  
\q  
<span style="color:green">sudo -u postgres psql</span>   
<span style="color:green">create table test(c1 text);</span>   
<span style="color:green">CREATE TABLE</span>   
<span style="color:green">insert into test values('1');</span>   
<span style="color:green">INSERT 0 1</span>   
<span style="color:green">postgres=# \q</span>   

- остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop  
<span style="color:green">sudo -u postgres pg_ctlcluster 15 main stop</span>   

- создайте новый диск к ВМ размером 10GB  
<span style="color:green">создал диск по инструкции вложенной в урок</span>  

- добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk  
<span style="color:green">добавил по инструкции для Yandex Cloud</span>  

- проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux  
<span style="color:green">выполнил по инструкции, в моём случае был диск /dev/vdb</span> 

- перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)  
<span style="color:green">перезагрузил ВМ, диск остался, проверял командой: </span>   
<span style="color:green">df -h -x tmpfs</span>  

- сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/  
<span style="color:green">выполнил команду</span>  

- перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data  
<span style="color:green">выполнил команду</span>  

- попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start  
<span style="color:green">выполнил команду, возникла ошибка</span>  

- напишите получилось или нет и почему  
<span style="color:green">не получилось, ошибка:</span>  
<span style="color:green">Error: /var/lib/postgresql/15/main is not accessible or does not exist</span>   
<span style="color:green">Мы переместили все данные кластера в другую директорию, в настройках новую директорию не указали</span>  

- задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
<span style="color:green">в postgresql.conf надо изменить data_directory</span>   

- напишите что и почему поменяли  
<span style="color:green">в postgresql.conf в data_directory указал директорию /mnt/data/15/main , т.к. в эту директорию мы всё перенесли</span>   

- попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start  
<span style="color:green">выполнил команду</span>  

- напишите получилось или нет и почему  
<span style="color:green">кластер запустился, т.к. теперь в настройках в data_directory указана верная директория</span>  

- зайдите через через psql и проверьте содержимое ранее созданной таблицы  
<span style="color:green">sudo -u postgres psql</span>   
<span style="color:green">select * from test;</span>    

- задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите 

PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.   
 Создаю ВМ в Yandex Cloud:  
 yc compute instance create \  
  --name pg-instance2 \  
  --hostname pg-instance2 \  
  --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts \  
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \  
  --zone ru-central1-a \  
  --metadata-from-file ssh-keys=/home/lozhkin/.ssh/lozhkin.txt  

устанавливаем PG  
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15  

останавливаем PG  
sudo -u postgres pg_ctlcluster 15 main stop  

Удаляем файлы с данными  
sudo rm -r /var/lib/postgresql/15  

диск уже проинициализирован, надо его смонтировать
sudo mkdir -p /mnt/data  
sudo mount -o defaults /dev/vdb1 /mnt/data  

в  /etc/fstab добавляем строчку чтобы монтировался атоматом при перезапуске   
LABEL=datapartition /mnt/data ext4 defaults 0 2 

в /mnt/data/15/main/postgresql.conf в data_directory указал директорию /mnt/data/15/main  

запускаем кластер
sudo -u postgres pg_ctlcluster 15 main start   

заходим через через psql и проверяем содержимое ранее созданной таблицы  
sudo -u postgres psql  
select * from test;



