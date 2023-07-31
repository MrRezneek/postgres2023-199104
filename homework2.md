- создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом  
<span style="color:green">создал ВМ в яндекс клауд</span>  
- поставить на нем Docker Engine  
<span style="color:green">curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER</span>
- сделать каталог /var/lib/postgres  
<span style="color:green">sudo mkdir /var/lib/postgres</span>
- развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql  
<span style="color:green">sudo docker network create pg-net</span>  
<span style="color:green">sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15</span>
- развернуть контейнер с клиентом postgres 
- подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк    
<span style="color:green">sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres</span>  
<span style="color:green">create table test(id int);</span>  
<span style="color:green">insert into test(id) values (1);</span>     
- подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера  
<span style="color:green">psql -p 5432 -U postgres -h 158.160.38.137 -d postgres -W;</span>  
- удалить контейнер с сервером    
<span style="color:green">sudo docker stop 77dc03c55dfb</span> 
<span style="color:green">sudo docker rm 77dc03c55dfb</span> 
- создать его заново  
<span style="color:green">sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15</span>
- подключится снова из контейнера с клиентом к контейнеру с сервером  
<span style="color:green">sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres</span>  
- проверить, что данные остались на месте    
<span style="color:green">select * from test;</span>  