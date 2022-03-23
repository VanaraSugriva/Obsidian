id: 202203021553

# # # How to run Netbox IPAM in Docker containers

**_Source:_** 

https://computingforgeeks.com/how-to-run-netbox-ipam-tool-in-docker-containers/

## Description

-   Управление Vlan
-   Управление VRF
-   IPAM – управление IP-адресами
-   DCIM – управление инфраструктурой центра обработки данных
-   Управление провайдерами каналов связи
-   Многосайтовость
-   Единая конвергентная база данных
-   Оповещения
-   Управление подключениями – интерфейсы/консоли/питание
-   Персонализация заголовка для логотипа и т.д.

Содержание

1.  [Начало работы.](https://itsecforu.ru/2021/11/25/%F0%9F%90%B3-%D0%BA%D0%B0%D0%BA-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D1%82%D0%B8%D1%82%D1%8C-%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82-netbox-ipam-%D0%B2-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5/?ysclid=l09jdkx5up#nachalo-raboty)
2.  [1. Установите Docker и Docker-Compose на Linux](https://itsecforu.ru/2021/11/25/%F0%9F%90%B3-%D0%BA%D0%B0%D0%BA-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D1%82%D0%B8%D1%82%D1%8C-%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82-netbox-ipam-%D0%B2-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5/?ysclid=l09jdkx5up#1-ustanovite-docker-i-docker-compose-na-linux)
3.  [2. Настройка IPAM-сервера Netbox](https://itsecforu.ru/2021/11/25/%F0%9F%90%B3-%D0%BA%D0%B0%D0%BA-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D1%82%D0%B8%D1%82%D1%8C-%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82-netbox-ipam-%D0%B2-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5/?ysclid=l09jdkx5up#2-nastroyka-ipam-servera-netbox)
4.  [3. Доступ к веб-интерфейсу инструмента Netbox IPAM](https://itsecforu.ru/2021/11/25/%F0%9F%90%B3-%D0%BA%D0%B0%D0%BA-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D1%82%D0%B8%D1%82%D1%8C-%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82-netbox-ipam-%D0%B2-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5/?ysclid=l09jdkx5up#3-dostup-k-veb-interfeysu-instrumenta-netbox)
5.  [Заключение](https://itsecforu.ru/2021/11/25/%F0%9F%90%B3-%D0%BA%D0%B0%D0%BA-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D1%82%D0%B8%D1%82%D1%8C-%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82-netbox-ipam-%D0%B2-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5/?ysclid=l09jdkx5up#zaklyuchenie)


__

```
## На Debian/Ubuntu
sudo apt update && sudo apt upgrade
sudo apt install curl vim git
```
```
docker -v
```
```
sudo usermod -aG docker $USER
newgrp docker
```
```
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url  | grep docker-compose-linux-x86_64 | cut -d '"' -f 4 | wget -qi -
```
```
chmod +x docker-compose-linux-x86_64
```
```
sudo mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
```
```
docker-compose version
```
```
sudo systemctl start docker && sudo systemctl enable docker
```
```
git clone -b release https://github.com/netbox-community/netbox-docker.git
```
```
cd netbox-docker
```
```
tee docker-compose.override.yml <<EOF
version: '3.4'
services:
  netbox:
    ports:
      - 8000:8080
EOF
```
```
docker-compose pull
```
```
docker-compose up
# docker-compose up -d # to detach and return to terminal/ for view logs docker-compose logs -f
# docker-compose down, 
```


__

Keys: [[docker]] [[docker-compose]] [[Netbox]] [[IPAM]] [[DCIM]]

**_Status:_**  #done


**_Type:_**  #pro


**_Tags:_**  #pro #it
