# Zabbix-Monitoring-LAB
Projeto para monitoramento de redes em um ambiente com dispositivos Linux, Windows, Mikrotik e Cisco

## Para esse projeto será necessario ter instalado uma máquina virtual, com o EVE-NG instalado

# Overview
<img width="987" height="775" alt="image" src="https://github.com/user-attachments/assets/de6338d7-ec7c-4acf-a17c-a6495fce131a" />



## 1 - Imagens utilizadas
Nesse projeto para o dispositivo Cisco utilizei a imagem i86bi_linux_l2, presente para download no link abaixo:
https://www.eve-ng.net/index.php/documentation/supported-images/?utm_source=chatgpt.com

Para dispositivos Mikrotik utilizei a versão 7 presente no seguinte link:
https://mikrotik.com/download/chr?utm_source=chatgpt.com

## 2 Range de IP's

Como simulador de link de internet utilizei o NAT da própria VM usando a Management Cloud 0
Para a rede LAN, onde ficará os computadores dos funcionários, utilizei o range 192.168.0.X/24, como ponto a ponto, utilizei o range 10.0.0.0/30 e 10.0.0.4/30 e para o data center, onde ficará os servidores Zabbix e o servidor Linux utilizei o range 172.16.0.X/24, sendo que no data center utilizarei IP estático sendo o servidor Linux o IP 172.16.0.101/24 e o servidor Zabbix 172.16.0.100/24

# Configuração Inicial - Todos Roteadores e Switches Mikrotik

## 1- Alterar o nome do dispositivo ( System > Identity )

<img width="1195" height="669" alt="image" src="https://github.com/user-attachments/assets/97e765f3-e811-4429-9eca-fde54b1a271d" />

## 2- Configuração de Time Zone ( System > Clock )

<img width="1196" height="701" alt="image" src="https://github.com/user-attachments/assets/efc1a46f-f42f-4b85-89dd-8bc31c490a91" />

## 3- Configuração de NTP ( System > NTP Client )

<img width="1197" height="669" alt="image" src="https://github.com/user-attachments/assets/fa8eb6f4-0f7c-4595-983c-05500148dd8f" />

## 4- Criar novo Super Usuário ( System > Users > New )

<img width="1194" height="697" alt="image" src="https://github.com/user-attachments/assets/b60db8b9-ec39-410d-a345-b8857a7422c2" />

## 5- Após relogar no usuário criado, apagar o usuário admin padrão ( System > Users > Selecionar admin > Remove )

<img width="745" height="454" alt="image" src="https://github.com/user-attachments/assets/1c4515f4-71ab-44ee-b8fe-37e3680174c3" />

## 6- Ativar RoMON ( Tools > RoMON )

<img width="1198" height="668" alt="image" src="https://github.com/user-attachments/assets/f81df374-a4f1-4e49-9d6d-8f96c23d547a" />


















