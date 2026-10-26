# Zabbix-Monitoring-LAB
Projeto para monitoramento de redes em um ambiente com dispositivos Linux, Windows, Mikrotik e Cisco

## Para esse projeto será necessario ter instalado uma máquina virtual, com o EVE-NG instalado e Winbox 4 baixado

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

## 4- DNS Servers ( IP > DNS )

<img width="1198" height="672" alt="image" src="https://github.com/user-attachments/assets/75a7db4a-961f-49df-83a4-fb7f4a7d0762" />

## 5- Criar novo Super Usuário ( System > Users > New )

<img width="1194" height="697" alt="image" src="https://github.com/user-attachments/assets/b60db8b9-ec39-410d-a345-b8857a7422c2" />

## 6- Após relogar no usuário criado, apagar o usuário admin padrão ( System > Users > Selecionar admin > Remove )

<img width="745" height="454" alt="image" src="https://github.com/user-attachments/assets/1c4515f4-71ab-44ee-b8fe-37e3680174c3" />

## 7- Ativar RoMON ( Tools > RoMON )

<img width="1198" height="668" alt="image" src="https://github.com/user-attachments/assets/f81df374-a4f1-4e49-9d6d-8f96c23d547a" />

## 8- Logar com conta MikroTik para aumento de banda ( System > License > Renew License )

<img width="438" height="226" alt="image" src="https://github.com/user-attachments/assets/2eeea699-96d6-45a6-8bfc-8b244ff3bb94" />


# Configurações R1

## Adicionar IP PTP a ether2 e ether3 ( IP > Addresses > New )

<img width="408" height="458" alt="image" src="https://github.com/user-attachments/assets/d84dcf21-30a4-44a9-89d5-5ac649fede5f" />

## Criar NAT masquerade com | chain srcnat / out interface ether1 / Action masquerade

<img width="694" height="642" alt="image" src="https://github.com/user-attachments/assets/87a9ff4b-0365-4b5c-b649-84322e9a6c66" />

<img width="693" height="248" alt="image" src="https://github.com/user-attachments/assets/b95ee6ce-fc16-4dc2-bb0a-7932158383e8" />

## Criar rotas para Rede LAN e SERVERS ( IP > Routes > New )

<img width="1194" height="699" alt="image" src="https://github.com/user-attachments/assets/42c17b2c-fddc-4722-85ef-04481078709b" />

<img width="811" height="455" alt="image" src="https://github.com/user-attachments/assets/177812bd-5d28-4c1f-aa1c-4c5b4bcc1872" />


# Configurações MK-LAN

## Criar Bridge LAN e adicionar portas a ela ( Bridge > New | Ports > New )

<img width="1047" height="643" alt="image" src="https://github.com/user-attachments/assets/08a71b5c-b662-4d9c-af16-fa955e2fe201" />
<img width="1047" height="223" alt="image" src="https://github.com/user-attachments/assets/9092305d-db22-41f9-8420-6ef8cf59a0ed" />

## Criar PTP para R1 ( IP > Addresses > New )

<img width="408" height="457" alt="image" src="https://github.com/user-attachments/assets/04d6cc8a-89c9-4bec-8db1-1eed7b3bfc19" />

## Criar rede LAN ( IP > Addresses > New )
 
<img width="405" height="454" alt="image" src="https://github.com/user-attachments/assets/e124ae43-ab45-4ae7-9344-2c1781d219c0" />

## Criar DHCP Server para a Bridge LAN ( IP > DHCP Server > DHCP Setup )

<img width="786" height="455" alt="image" src="https://github.com/user-attachments/assets/ed2800ab-7ad1-49ff-87be-0309a86fb09d" />

## Rota DEFAULT ( IP > Routes > New )

<img width="625" height="612" alt="image" src="https://github.com/user-attachments/assets/a03428ad-83c9-49e8-831d-bdd31bce7491" />


# Configurações Switch CISCO ( SW1 )

## Criar IP PTP e Rota para R1

enable
configure terminal
hostname SW1
int e0/0
no switchport
ip address 10.0.0.6 255.255.255.252
no shut
ip routing
ip route 0.0.0.0 0.0.0.0 10.0.0.5
end
write memory

## Criar Vlan e SVI para os Servers

enable
configure terminal
vlan 10
name SERVERS
exit
int range e0/1-2
switchport mode access
switchport access vlan 10
int vlan 10
ip address 172.16.0.1 255.255.255.0
no shut
exit
ip default-gateway 172.16.0.1
end
write memory

## Verificar as configurações

ping 8.8.8.8
ping 192.168.0.1
show ip int brief
sh vlan brief
sh ip route

<img width="1117" height="567" alt="image" src="https://github.com/user-attachments/assets/e6d0985d-77d4-4af3-a8ed-79129d6408a5" />
<img width="1120" height="296" alt="image" src="https://github.com/user-attachments/assets/65b54276-eeca-4f4b-b869-9624516f5556" />


# Definindo IP Estático para Servidor Linux

## Descobrir o nome da interface de rede

ip a

<img width="1034" height="225" alt="image" src="https://github.com/user-attachments/assets/1772e7ef-8dd1-4b12-b198-c7207a09cbd5" />

## Configurar o arquivo de internet

sudo nano /etc/netplan/50-cloud-init.yaml

network:
  version: 2
  ethernets:
    ens3:
      dhcp4: false
      addresses:
        - 172.16.0.101/24
      routes:
        - to: default
          via: 172.16.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
          
Control O > Enter > Control X para salvar e sair

## Testar e Confirmar a configuração 

sudo netplan generate
sudo netplan try
ip a

<img width="847" height="226" alt="image" src="https://github.com/user-attachments/assets/2819886e-4ad4-499a-89b5-b93eb2f5a1b5" />

# Definir IP Estático no Zabbix

## Descobrir nome da interface

ip a
<img width="1117" height="402" alt="image" src="https://github.com/user-attachments/assets/38a9e951-6b37-4180-99d7-3bc14af29674" />

## Alterar arquivo de configuração de rede

vi /etc/sysconfig/network-scripts/ifcfg-eth0

Pressione i para editar e coloque a seguinte configuração

TYPE=Ethernet
DEVICE=eth0
NAME=eth0
BOOTPROTO=none
ONBOOT=yes
IPADDR=172.16.0.100
PREFIX=24
GATEWAY=172.16.0.1
DNS1=8.8.8.8

esc > :wq = para salvar e sair

## Aplicar as configurações

systemctl restart network

## Testar

ip a
<img width="1118" height="458" alt="image" src="https://github.com/user-attachments/assets/0f1d9827-5a7d-4aa2-b001-a6b3f9ec7081" />

# Adicionar rota do computador para o LAB Zabbix

## Para acessar o zabbix do seu proprio computador é necessario criar uma rota no VMWare, para isso será necessario usar o ip recebido na interface ether 1 do dispositivo MikroTik R1, no caso desse lab será o IP 192.168.14.135

ip route add 172.16.0.0/24 via 192.168.14.135
ip route

<img width="974" height="137" alt="image" src="https://github.com/user-attachments/assets/c690bb24-8867-4679-a444-85efca9e695e" />

## Teste de conectividade

ping -c 4 172.16.0.100
<img width="879" height="268" alt="image" src="https://github.com/user-attachments/assets/77e95074-78e2-460f-8562-d0c88284d308" />

# Acessando Zabbix

## No navegador do seu computador digite o endereço 172.16.0.100, aparecerá a tela de login do Zabbix
Username: Admin
Password: zabbix

# Adicionando Hosts ao Zabbix

## Windows

Para adicionar um dispositivo Windows ao Zabbix iremos utilizar o Zabbix Agent, baixando o pelo link: https://www.zabbix.com/download_agents

<img width="1014" height="717" alt="image" src="https://github.com/user-attachments/assets/07cdef2f-7ac7-4c8c-984e-6c293696c010" />

No Zabbix vamos em Data collection > Hosts

<img width="1918" height="944" alt="image" src="https://github.com/user-attachments/assets/ff209121-2a24-4a11-86bb-be4122827f43" />

Clique em Create Host no canto superior direito e irá colocar as seguintes informações sendo o IP do Agent de acordo com o ip do windows
<img width="1917" height="943" alt="image" src="https://github.com/user-attachments/assets/1b17ecf9-f464-485f-979d-61229da84eb5" />

## Linux 

Para adicionar um dispositivo Linux ao Zabbix iremos utilizar o Zabbix Agent, seguindo a documentação do zabbix: [https://www.zabbix.com/download_agents](https://www.zabbix.com/download?zabbix=7.4&os_distribution=ubuntu&os_version=26.04&components=agent&db=&ws=)

sudo -s
wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu26.04_all.deb
dpkg -i zabbix-release_latest_7.4+ubuntu26.04_all.deb
apt update
apt install zabbix-agent
systemctl restart zabbix-agent
systemctl enable zabbix-agent


Configurar conexão com o Servidor

sudo nano /etc/zabbix/zabbix_agentd.conf
systemctl restart zabbix-agent
systemctl enable zabbix-agent

Logo após adicionar no zabbix com as informações configuradas acima
<img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/5e9d1a3f-5556-4859-9d6b-175fa4ac3099" />


## MikroTik

Para adicionar um dispositivo MikroTik no Zabbix iremos utilizar o protocolo SNMP

No MikroTik vamos ativar o protocolo em IP > SNMP

<img width="1198" height="670" alt="image" src="https://github.com/user-attachments/assets/061111bc-b3f9-4294-94fb-f7f750a83186" />

Logo após vamos adiciona-lo a comunidade clicando em communities > New, e colocar as seguintes informações

<img width="521" height="411" alt="image" src="https://github.com/user-attachments/assets/2e72cd41-90bf-4c32-a66c-19b410fe4f89" />

No Zabbix iremos utilizar as seguintes informações

<img width="1917" height="942" alt="image" src="https://github.com/user-attachments/assets/1e98b82e-70a7-4d2a-bfaf-24fa93ffed38" />

## Cisco

Para adicionar um dispositivo Cisco no Zabbix iremos utilizar o protocolo SNMP

No Cisco vamos usar o seguinte comando para ativar e configurar o SNMP

enable
configure terminal
snmp-server community Cisco RO
snmp-server host 172.16.0.100 version 2c Cisco
end
write memory

No Zabbix iremos utilizar as seguintes informações

<img width="1916" height="943" alt="image" src="https://github.com/user-attachments/assets/33c2df81-34d7-4316-836e-1b169a9d6830" />

# Resultado Esperado

<img width="1918" height="942" alt="image" src="https://github.com/user-attachments/assets/86d46663-d313-4d10-bf2f-fd0499204aef" />


# Instalando e configurando Grafana no servidor Zabbix

## Abertura de Portas

Para utilizar o grafana precisamos abrir a porta 3000 e para isso iremos utilizar o pacote firewalld

## Instalação e inicialização do FirewallD

sudo dnf install firewalld -y
sudo systemctl start firewalld
sudo systemctl status firewalld

## Abrindo as portas 3000, 80, 10050 e 10051 e serviços http e https

sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --permanent --add-port=10050/tcp
sudo firewall-cmd --permanent --add-port=10051/tcp
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

## Verificar Portas e serviços liberados

sudo firewall-cmd --list-ports
sudo firewall-cmd --list-services


## Instalando Pacote Grafana

Para baixar e instalar o pacote vamos seguir a documentação do grafana [https://www.grafana.com/docs](https://grafana.com/docs/grafana/latest/setup-grafana/installation/redhat-rhel-fedora/) 

Importar GPG key

sudo dnf install wget -y
wget -q -O gpg.key https://rpm.grafana.com/gpg.key
sudo rpm --import gpg.key

Criar e editar documento de configuração ( utilizar a tecla i para iniciar edição e esc > :wq para salvar e sair )

vi /etc/yum.repos.d/grafana.repo

[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1

Instalar e iniciar o grafana

sudo dnf install grafana
sudo systemctl start grafana-server
sudo systemctl status grafana-server

## Acessar grafana pelo seu navegador no ip 172.16.0.100:3000

Username: admin
Password: admin

# Configurando o Grafana

## 1- Adicionar nova conexão ( Administration > Plugins and data > Plugins > zabbix > Install > Enable)

<img width="1903" height="944" alt="image" src="https://github.com/user-attachments/assets/8837c2fa-14e1-42e7-81a0-11b784d270fe" />

## 2- Adicionar Fonte de Dados ( Connections > Data sources > Add data source > Zabbix) 

<img width="1919" height="942" alt="image" src="https://github.com/user-attachments/assets/63adffc7-5b71-437d-9b11-35f6d44f05b3" />

## 3- Configurar Fonte de Dados

URL: http://172.16.0.100/api_jsonrpc.php
Authentication method: Forward OAuth Identity
Zabbix Connection:
Username: Admin
Password: zabbix

Save & Test

<img width="1902" height="940" alt="image" src="https://github.com/user-attachments/assets/0a376cf4-81a9-41a8-8533-fa062624a0b0" />

# Adicionando Paineis ( Dashboard > New Dashboard > Edit Panel )

<img width="1917" height="943" alt="image" src="https://github.com/user-attachments/assets/1bfef324-ca9e-4db7-b31c-a4ef3cc3e395" />

