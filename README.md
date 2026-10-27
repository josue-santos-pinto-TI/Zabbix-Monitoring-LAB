# Zabbix Monitoring Lab

Projeto de monitoramento de redes em um ambiente virtualizado, utilizando dispositivos Linux, Windows, MikroTik e Cisco, com integração entre **Zabbix** e **Grafana** para monitoramento e visualização de métricas.

## Pré-requisitos

Para reproduzir este projeto, será necessário ter:

- [VMware Workstation](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion) instalado.
- [EVE-NG](https://www.eve-ng.net/) instalado e configurado.
- [WinBox 4](https://mikrotik.com/download) baixado.
- Imagens dos dispositivos de rede utilizadas no laboratório.
- Máquinas virtuais para os servidores Linux e Zabbix.

---

## Sumário

- [Visão geral](#visão-geral)
- [1. Imagens utilizadas](#1-imagens-utilizadas)
- [2. Planejamento de endereçamento IP](#2-planejamento-de-endereçamento-ip)
- [3. Configuração inicial dos dispositivos MikroTik](#3-configuração-inicial-dos-dispositivos-mikrotik)
- [4. Configuração do R1](#4-configuração-do-r1)
- [5. Configuração do MK-LAN](#5-configuração-do-mk-lan)
- [6. Configuração do switch Cisco SW1](#6-configuração-do-switch-cisco-sw1)
- [7. Configuração do servidor Linux](#7-configuração-do-servidor-linux)
- [8. Configuração do IP estático do Zabbix](#8-configuração-do-ip-estático-do-zabbix)
- [9. Configuração de rota para acesso ao laboratório](#9-configuração-de-rota-para-acesso-ao-laboratório)
- [10. Acesso à interface web do Zabbix](#10-acesso-à-interface-web-do-zabbix)
- [11. Adicionando hosts ao Zabbix](#11-adicionando-hosts-ao-zabbix)
- [12. Resultado esperado](#12-resultado-esperado)
- [13. Instalação e configuração do Grafana](#13-instalação-e-configuração-do-grafana)
- [14. Configuração do Grafana](#14-configuração-do-grafana)
- [15. Criação de dashboards](#15-criação-de-dashboards)

---

# Visão geral

A topologia do projeto contempla uma rede corporativa simulada, com roteamento entre a LAN, os servidores e a rede externa.

<img width="987" height="775" alt="image" src="https://github.com/user-attachments/assets/de6338d7-ec7c-4acf-a17c-a6495fce131a" />

---

# 1. Imagens utilizadas

Para montar o laboratório no EVE-NG, foram utilizadas as seguintes imagens:

### Cisco

Imagem utilizada: `i86bi_linux_l2`.

Documentação e imagens suportadas pelo EVE-NG:

https://www.eve-ng.net/index.php/documentation/supported-images/?utm_source=chatgpt.com

### MikroTik

Versão utilizada: RouterOS 7, por meio da imagem CHR (Cloud Hosted Router).

Download oficial:

https://mikrotik.com/download/chr?utm_source=chatgpt.com

---

# 2. Planejamento de endereçamento IP

Para simular o acesso à internet, será utilizado o NAT do VMware, conectado à Management Cloud 0 do EVE-NG.

## 2.1. Redes utilizadas

| Segmento | Endereçamento | Finalidade |
|---|---|---|
| LAN | `192.168.0.0/24` | Computadores dos funcionários |
| Ponto a ponto 1 | `10.0.0.0/30` | Conexão entre roteadores |
| Ponto a ponto 2 | `10.0.0.4/30` | Conexão entre roteador e switch Cisco |
| Data Center | `172.16.0.0/24` | Servidores Linux e Zabbix |
| Gerenciamento VMware | `192.168.14.0/24` | Comunicação com o ambiente virtualizado |

## 2.2. Endereços dos servidores

| Dispositivo | Endereço IP | Máscara |
|---|---|---|
| Servidor Zabbix | `172.16.0.100` | `/24` |
| Servidor Linux | `172.16.0.101` | `/24` |
| Gateway do Data Center | `172.16.0.1` | `/24` |

Os servidores do Data Center utilizarão endereços IP estáticos.

---

# 3. Configuração inicial dos dispositivos MikroTik

As configurações desta seção devem ser aplicadas a todos os roteadores e switches MikroTik do laboratório.

## 3.1. Alterar o nome do dispositivo

No WinBox, acesse:

`System > Identity`

Defina um nome para identificar o dispositivo na topologia.

<img width="1195" height="669" alt="image" src="https://github.com/user-attachments/assets/97e765f3-e811-4429-9eca-fde54b1a271d" />

## 3.2. Configurar o fuso horário

Acesse:

`System > Clock`

Configure o fuso horário correspondente à localização do laboratório.

<img width="1196" height="701" alt="image" src="https://github.com/user-attachments/assets/efc1a46f-f42f-4b85-89dd-8bc31c490a91" />

## 3.3. Configurar o cliente NTP

Acesse:

`System > NTP Client`

Configure o cliente NTP para sincronizar o relógio do dispositivo.

<img width="1197" height="669" alt="image" src="https://github.com/user-attachments/assets/fa8eb6f4-0f7c-4595-983c-05500148dd8f" />

## 3.4. Configurar os servidores DNS

Acesse:

`IP > DNS`

Configure os servidores DNS que serão utilizados pelo dispositivo.

<img width="1198" height="672" alt="image" src="https://github.com/user-attachments/assets/75a7db4a-961f-49df-83a4-fb7f4a7d0762" />

## 3.5. Criar um novo usuário administrador

Acesse:

`System > Users > New`

Crie um novo usuário com permissões administrativas.

<img width="1194" height="697" alt="image" src="https://github.com/user-attachments/assets/b60db8b9-ec39-410d-a345-b8857a7422c2" />

## 3.6. Remover o usuário padrão

Após confirmar que o novo usuário consegue acessar o dispositivo, faça login com a nova conta.

Em seguida, acesse:

`System > Users`

Selecione o usuário `admin` padrão e clique em `Remove`.

<img width="745" height="454" alt="image" src="https://github.com/user-attachments/assets/1c4515f4-71ab-44ee-b8fe-37e3680174c3" />

## 3.7. Ativar o RoMON

Acesse:

`Tools > RoMON`

Ative o RoMON conforme a necessidade do laboratório.

<img width="1198" height="668" alt="image" src="https://github.com/user-attachments/assets/f81df374-a4f1-4e49-9d6d-8f96c23d547a" />

## 3.8. Renovar a licença

Acesse:

`System > License > Renew License`

Faça login com a conta MikroTik para verificar as opções de licença disponíveis.

<img width="438" height="226" alt="image" src="https://github.com/user-attachments/assets/2eeea699-96d6-45a6-8bfc-8b244ff3bb94" />

---

# 4. Configuração do R1

O R1 será responsável pelo roteamento entre as redes internas e pela saída para a rede externa simulada.

## 4.1. Configurar os endereços IP das interfaces

Acesse:

`IP > Addresses > New`

Configure os endereços IP das interfaces `ether2` e `ether3`, conforme o planejamento de endereçamento da topologia.

<img width="408" height="458" alt="image" src="https://github.com/user-attachments/assets/d84dcf21-30a4-44a9-89d5-5ac649fede5f" />

## 4.2. Configurar o NAT

Crie uma regra NAT com os seguintes parâmetros:

- **Chain:** `srcnat`
- **Out. Interface:** `ether1`
- **Action:** `masquerade`

O NAT permitirá que os dispositivos da rede interna utilizem o endereço da interface externa do R1 para acessar a rede externa.

<img width="694" height="642" alt="image" src="https://github.com/user-attachments/assets/87a9ff4b-0365-4b5c-b649-84322e9a6c66" />

<img width="693" height="248" alt="image" src="https://github.com/user-attachments/assets/b95ee6ce-fc16-4dc2-bb0a-7932158383e8" />

## 4.3. Criar rotas para as redes LAN e SERVERS

Acesse:

`IP > Routes > New`

Configure as rotas necessárias para que o R1 alcance as redes LAN e SERVERS por meio dos próximos saltos correspondentes à topologia.

<img width="1194" height="699" alt="image" src="https://github.com/user-attachments/assets/42c17b2c-fddc-4722-85ef-04481078709b" />

<img width="811" height="455" alt="image" src="https://github.com/user-attachments/assets/177812bd-5d28-4c1f-aa1c-4c5b4bcc1872" />

---

# 5. Configuração do MK-LAN

O MK-LAN será responsável pela conectividade dos dispositivos da rede local.

## 5.1. Criar uma bridge para a LAN

Acesse:

- `Bridge > New` para criar a bridge.
- `Bridge > Ports > New` para adicionar as interfaces à bridge.

Adicione as portas que serão utilizadas pelos dispositivos da LAN.

<img width="1047" height="643" alt="image" src="https://github.com/user-attachments/assets/08a71b5c-b662-4d9c-af16-fa955e2fe201" />

<img width="1047" height="223" alt="image" src="https://github.com/user-attachments/assets/9092305d-db22-41f9-8420-6ef8cf59a0ed" />

## 5.2. Configurar o enlace ponto a ponto com o R1

Acesse:

`IP > Addresses > New`

Configure o endereço IP da interface responsável pelo enlace ponto a ponto com o R1.

<img width="408" height="457" alt="image" src="https://github.com/user-attachments/assets/04d6cc8a-89c9-4bec-8db1-1eed7b3bfc19" />

## 5.3. Configurar o endereço IP da LAN

Acesse:

`IP > Addresses > New`

Adicione o endereço IP correspondente à rede LAN, utilizando a bridge criada anteriormente quando aplicável.

<img width="405" height="454" alt="image" src="https://github.com/user-attachments/assets/e124ae43-ab45-4ae7-9344-2c1781d219c0" />

## 5.4. Configurar o servidor DHCP

Acesse:

`IP > DHCP Server > DHCP Setup`

Selecione a bridge LAN e siga o assistente para configurar o servidor DHCP.

<img width="786" height="455" alt="image" src="https://github.com/user-attachments/assets/ed2800ab-7ad1-49ff-87be-0309a86fb09d" />

## 5.5. Criar a rota padrão

Acesse:

`IP > Routes > New`

Configure a rota padrão para que o MK-LAN encaminhe o tráfego destinado a outras redes para o R1.

<img width="625" height="612" alt="image" src="https://github.com/user-attachments/assets/a03428ad-83c9-49e8-831d-bdd31bce7491" />

---

# 6. Configuração do switch Cisco SW1

O switch Cisco será configurado com um enlace ponto a ponto para o R1 e uma VLAN para a rede de servidores.

## 6.1. Configurar o enlace ponto a ponto e a rota padrão

Entre no modo de configuração e aplique:

```text
enable
configure terminal
hostname SW1

interface e0/0
 no switchport
 ip address 10.0.0.6 255.255.255.252
 no shutdown
exit

ip routing
ip route 0.0.0.0 0.0.0.0 10.0.0.5

end
write memory
```

A interface `e0/0` será utilizada como interface de camada 3 para o enlace com o R1.

## 6.2. Criar a VLAN e a SVI para os servidores

Crie a VLAN 10, associe as interfaces destinadas aos servidores e configure a interface virtual da VLAN (SVI).

```text
enable
configure terminal

vlan 10
 name SERVERS
exit

interface range e0/1-2
 switchport mode access
 switchport access vlan 10
exit

interface vlan 10
 ip address 172.16.0.1 255.255.255.0
 no shutdown
exit

ip default-gateway 172.16.0.1

end
write memory
```

**Observação:** como o switch está configurado com `ip routing`, a SVI pode atuar como interface de roteamento. Nesse caso, o gateway padrão de um switch de camada 3 deve ser definido por meio de uma rota padrão, quando necessário, em vez de depender de `ip default-gateway`.

Além disso, o endereço `172.16.0.1` precisa ser único na rede e corresponder ao gateway efetivamente adotado para o Data Center.

## 6.3. Verificar as configurações

Execute os comandos abaixo para validar a conectividade e a configuração do switch:

```text
ping 8.8.8.8
ping 192.168.0.1
show ip interface brief
show vlan brief
show ip route
```

<img width="1117" height="567" alt="image" src="https://github.com/user-attachments/assets/e6d0985d-77d4-4af3-a8ed-79129d6408a5" />

<img width="1120" height="296" alt="image" src="https://github.com/user-attachments/assets/65b54276-eeca-4f4b-b869-9624516f5556" />

---

# 7. Configuração do servidor Linux

Nesta etapa, será configurado um endereço IP estático no servidor Linux, que utilizará o endereço `172.16.0.101/24`.

## 7.1. Identificar a interface de rede

Execute:

```bash
ip a
```

Identifique o nome da interface de rede. Neste exemplo, será utilizada a interface `ens3`.

<img width="1034" height="225" alt="image" src="https://github.com/user-attachments/assets/1772e7ef-8dd1-4b12-b198-c7207a09cbd5" />

## 7.2. Editar o arquivo de configuração de rede

Abra o arquivo do Netplan:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Adicione ou ajuste a configuração abaixo:

```yaml
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
```

Para salvar e sair do Nano, pressione `Ctrl + O`, `Enter` e `Ctrl + X`.

## 7.3. Aplicar e testar a configuração

Execute:

```bash
sudo netplan generate
sudo netplan try
ip a
```

Confirme que a interface apresenta o endereço `172.16.0.101/24`.

<img width="847" height="226" alt="image" src="https://github.com/user-attachments/assets/2819886e-4ad4-499a-89b5-b93eb2f5a1b5" />

---

# 8. Configuração do IP estático do Zabbix

O servidor Zabbix utilizará o endereço IP estático `172.16.0.100/24`.

## 8.1. Identificar a interface de rede

Execute:

```bash
ip a
```

Identifique a interface que será configurada. Neste exemplo, será utilizada a interface `eth0`.

<img width="1117" height="402" alt="image" src="https://github.com/user-attachments/assets/38a9e951-6b37-4180-99d7-3bc14af29674" />

## 8.2. Editar o arquivo de configuração

Abra o arquivo:

```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

Pressione `i` para iniciar a edição e insira:

```ini
TYPE=Ethernet
DEVICE=eth0
NAME=eth0
BOOTPROTO=none
ONBOOT=yes
IPADDR=172.16.0.100
PREFIX=24
GATEWAY=172.16.0.1
DNS1=8.8.8.8
```

Para salvar e sair, pressione `Esc` e digite:

```text
:wq
```

## 8.3. Aplicar as configurações

Reinicie o serviço de rede:

```bash
systemctl restart network
```

## 8.4. Validar o endereço IP

Execute:

```bash
ip a
```

Confirme que a interface está configurada com o endereço `172.16.0.100/24`.

<img width="1118" height="458" alt="image" src="https://github.com/user-attachments/assets/0f1d9827-5a7d-4aa2-b001-a6b3f9ec7081" />

---

# 9. Configuração de rota para acesso ao laboratório

Para acessar o Zabbix diretamente do computador Windows, será necessário criar uma rota para a rede `172.16.0.0/24` por meio do roteador MikroTik R1.

Neste laboratório, o endereço IP recebido pelo R1 na interface `ether1` é `192.168.14.135`, pertencente à rede de gerenciamento do VMware.

## 9.1. Adicionar a rota no EVE-NG

Execute no EVE-NG:

```bash
ip route add 172.16.0.0/24 via 192.168.14.135
```

Para consultar as rotas configuradas:

```bash
ip route
```

<img width="974" height="137" alt="image" src="https://github.com/user-attachments/assets/c690bb24-8867-4679-a444-85efca9e695e" />

## 9.2. Testar a conectividade

Execute:

```bash
ping -c 4 172.16.0.100
```

O objetivo é confirmar que o EVE-NG consegue alcançar o servidor Zabbix.

<img width="879" height="268" alt="image" src="https://github.com/user-attachments/assets/77e95074-78e2-460f-8562-d0c88284d308" />

---

# 10. Acesso à interface web do Zabbix

Após configurar o endereçamento e validar a conectividade, abra o navegador do computador e acesse:

**URL:** http://172.16.0.100/zabbix

Utilize as credenciais iniciais abaixo, caso ainda não tenham sido alteradas:

- **Usuário:** `Admin`
- **Senha:** `zabbix`

---

# 11. Adicionando hosts ao Zabbix

O Zabbix permite monitorar diferentes sistemas operacionais e dispositivos de rede por meio de agentes e protocolos como SNMP.

## 11.1. Monitoramento de dispositivos Windows

Para monitorar um dispositivo Windows, será utilizado o Zabbix Agent.

### Download do agente

Acesse a página oficial:

https://www.zabbix.com/download_agents

Baixe o agente compatível com a versão do Zabbix e com o sistema operacional do dispositivo.

<img width="1014" height="717" alt="image" src="https://github.com/user-attachments/assets/07cdef2f-7ac7-4c8c-984e-6c293696c010" />

### Adicionar o host no Zabbix

No painel web, acesse:

`Data collection > Hosts`

<img width="1918" height="944" alt="image" src="https://github.com/user-attachments/assets/ff209121-2a24-4a11-86bb-be4122827f43" />

Clique em **Create host** e preencha os campos de acordo com a configuração do agente instalado no Windows.

Informe o endereço IP do dispositivo Windows e associe o template adequado ao sistema operacional.

<img width="1917" height="943" alt="image" src="https://github.com/user-attachments/assets/1b17ecf9-f464-485f-979d-61229da84eb5" />

## 11.2. Monitoramento de dispositivos Linux

Para monitorar um servidor Linux, será utilizado o Zabbix Agent.

### Consultar a documentação oficial

Acesse:

https://www.zabbix.com/download?zabbix=7.4&os_distribution=ubuntu&os_version=26.04&components=agent&db=&ws=

### Instalar o agente

Os comandos abaixo são um exemplo para Ubuntu 26.04 e Zabbix 7.4. Confirme a compatibilidade entre a versão do Ubuntu e os pacotes disponíveis antes de executar.

```bash
sudo -s

wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu26.04_all.deb

dpkg -i zabbix-release_latest_7.4+ubuntu26.04_all.deb

apt update
apt install zabbix-agent
```

### Configurar a comunicação com o servidor Zabbix

Edite o arquivo de configuração:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Configure os parâmetros de acordo com o endereço do servidor e o endereço IP do host monitorado. Por exemplo:

```ini
Server=172.16.0.100
ServerActive=172.16.0.100
Hostname=Linux-Server
```

O valor de `Hostname` deve corresponder ao nome configurado para o host na interface do Zabbix.

Reinicie e habilite o agente:

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
sudo systemctl status zabbix-agent
```

Após a configuração, adicione o dispositivo Linux na interface web do Zabbix e associe o template Linux apropriado.

<img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/5e9d1a3f-5556-4859-9d6b-175fa4ac3099" />

## 11.3. Monitoramento de dispositivos MikroTik

Para monitorar os dispositivos MikroTik, será utilizado o protocolo SNMP.

### Ativar o SNMP no MikroTik

No WinBox, acesse:

`IP > SNMP`

Habilite o serviço SNMP.

<img width="1198" height="670" alt="image" src="https://github.com/user-attachments/assets/061111bc-b3f9-4294-94fb-f7f750a83186" />

### Criar uma comunidade SNMP

Acesse a seção `Communities` e clique em `New`.

Configure a comunidade de acordo com os parâmetros que serão utilizados no Zabbix.

<img width="521" height="411" alt="image" src="https://github.com/user-attachments/assets/2e72cd41-90bf-4c32-a66c-19b410fe4f89" />

### Adicionar o dispositivo no Zabbix

Na interface web, crie um novo host e configure a interface SNMP com o endereço IP de gerenciamento do MikroTik.

Associe um template compatível com o modelo e a versão do RouterOS.

<img width="1917" height="942" alt="image" src="https://github.com/user-attachments/assets/1e98b82e-70a7-4d2a-bfaf-24fa93ffed38" />

## 11.4. Monitoramento de dispositivos Cisco

Para monitorar o switch Cisco, será utilizado o protocolo SNMP versão 2c.

### Configurar o SNMP no Cisco

Entre no modo de configuração e execute:

```text
enable
configure terminal

snmp-server community Cisco RO
snmp-server host 172.16.0.100 version 2c Cisco

end
write memory
```

A comunidade `Cisco` deve corresponder à comunidade configurada na interface SNMP do host no Zabbix.

**Nota de segurança:** SNMPv2c utiliza uma comunidade em texto não criptografado. Em ambientes de produção, prefira SNMPv3, com autenticação e criptografia, quando suportado.

### Adicionar o switch ao Zabbix

Crie um host, configure a interface SNMP com o endereço IP de gerenciamento do switch e associe um template compatível com o equipamento.

<img width="1916" height="943" alt="image" src="https://github.com/user-attachments/assets/33c2df81-34d7-4316-836e-1b169a9d6830" />

---

# 12. Resultado esperado

Após concluir as configurações, os dispositivos adicionados deverão aparecer na interface do Zabbix, permitindo acompanhar sua disponibilidade e as métricas coletadas.

<img width="1918" height="942" alt="image" src="https://github.com/user-attachments/assets/86d46663-d313-4d10-bf2f-fd0499204aef" />

---

# 13. Instalação e configuração do Grafana

O Grafana será utilizado para criar dashboards e visualizar os dados coletados pelo Zabbix.

## 13.1. Liberar as portas no firewall

O Grafana utiliza, por padrão, a porta TCP `3000`.

Para permitir o acesso ao Grafana e aos serviços do Zabbix, será utilizado o `firewalld`.

### Instalar e iniciar o firewalld

No servidor AlmaLinux, execute:

```bash
sudo dnf install firewalld -y

sudo systemctl start firewalld
sudo systemctl enable firewalld

sudo systemctl status firewalld
```

### Liberar as portas e os serviços necessários

Execute:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --permanent --add-port=10050/tcp
sudo firewall-cmd --permanent --add-port=10051/tcp
sudo firewall-cmd --permanent --add-port=80/tcp

sudo firewall-cmd --reload
```

### Verificar as portas e os serviços liberados

```bash
sudo firewall-cmd --list-ports
sudo firewall-cmd --list-services
```

## 13.2. Instalar o Grafana

Consulte a documentação oficial:

https://grafana.com/docs/grafana/latest/setup-grafana/installation/redhat-rhel-fedora/

### Importar a chave GPG

Instale o `wget` caso ainda não esteja disponível:

```bash
sudo dnf install wget -y
```

Baixe e importe a chave GPG:

```bash
wget -q -O gpg.key https://rpm.grafana.com/gpg.key
sudo rpm --import gpg.key
```

### Criar o repositório do Grafana

Abra o arquivo:

```bash
sudo vi /etc/yum.repos.d/grafana.repo
```

Pressione `i` para iniciar a edição e adicione:

```ini
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
```

Para salvar, pressione `Esc` e digite:

```text
:wq
```

### Instalar e iniciar o serviço

Execute:

```bash
sudo dnf install grafana -y

sudo systemctl start grafana-server
sudo systemctl enable grafana-server

sudo systemctl status grafana-server
```

## 13.3. Acessar o Grafana

No navegador, acesse:

**URL:** http://172.16.0.100:3000

Credenciais iniciais padrão:

- **Usuário:** `admin`
- **Senha:** `admin`

No primeiro acesso, o Grafana poderá solicitar a alteração da senha.

---

# 14. Configuração do Grafana

## 14.1. Instalar e habilitar o plugin do Zabbix

No Grafana, acesse:

`Administration > Plugins and data > Plugins`

Procure pelo plugin **Zabbix**, clique em `Install` e, quando disponível, habilite-o.

<img width="1903" height="944" alt="image" src="https://github.com/user-attachments/assets/8837c2fa-14e1-42e7-81a0-11b784d270fe" />

## 14.2. Adicionar uma fonte de dados

Acesse:

`Connections > Data sources > Add data source > Zabbix`

<img width="1919" height="942" alt="image" src="https://github.com/user-attachments/assets/63adffc7-5b71-437d-9b11-35f6d44f05b3" />

## 14.3. Configurar a fonte de dados

Preencha os campos conforme a instalação do Zabbix.

| Campo | Valor |
|---|---|
| URL | `http://172.16.0.100/api_jsonrpc.php` |
| Authentication method | `Forward OAuth Identity` |
| Usuário Zabbix | `Admin` |
| Senha Zabbix | `zabbix` |

A URL deve apontar para a API do Zabbix. O método de autenticação deve ser compatível com a configuração do plugin e do servidor; caso a conexão falhe, revise as opções de autenticação disponíveis no plugin.

Após preencher os campos, clique em **Save & Test**.

<img width="1902" height="940" alt="image" src="https://github.com/user-attachments/assets/0a376cf4-81a9-41a8-8533-fa062624a0b0" />

---

# 15. Criação de dashboards

Com a fonte de dados configurada, será possível criar painéis para visualizar as informações coletadas pelo Zabbix.

Acesse:

`Dashboard > New Dashboard > Edit Panel`

Selecione a fonte de dados do Zabbix e configure as consultas e visualizações de acordo com as métricas que deseja acompanhar.

<img width="1917" height="943" alt="image" src="https://github.com/user-attachments/assets/1bfef324-ca9e-4db7-b31c-a4ef3cc3e395" />

---

## Conclusão

Este laboratório permite praticar conceitos de redes, roteamento, endereçamento IP, configuração de servidores Linux, monitoramento por agente e SNMP, além da integração entre Zabbix e Grafana.

Ao concluir o projeto, será possível monitorar dispositivos de diferentes fabricantes em um ambiente virtualizado e utilizar dashboards para acompanhar a disponibilidade e o desempenho da infraestrutura.
