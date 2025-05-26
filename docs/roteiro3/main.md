## <b>Introdução e Objetivo</b>

<p align="justify">
O roteiro 3 contempla uma abordagem nova, mantendo a utlização do Juju para gerenciamento de aplicações distribuídas mas introduzindo agora o openstack.
</p>

<p align="justify">
Ao final deste roteiro, o objetivo principal é termos, portanto, uma Cloud com um novo gerenciador de deploy instalado, utilizando o OpenStack como plataforma de nuvem privada. 
</p> 

<p align="justify"> 
O roteiro é dividido em três etapas: criação da infraestrutura (deploy do OpenStack), configuração dos serviços e redes da nuvem, e, por fim, a utilização da infraestrutura com o deployment de aplicações dentro de máquinas virtuais gerenciadas pela nuvem OpenStack.
</p>

## <b>Montagem do Roteiro</b>

<p align="justify">
Todo roteiro apresenta uma primeira parte denonimnada <b>Infra</b> e uma segunda chamada de <b>App</b>.
Os pontos <b>tarefas</b> dentro de cada parte são os passos seguidos para a realização do roteiro. 
Este modelo de organização orientado por partes e tarefas será utilizado em <b>todos os roteiros</b>.
</p>

# <b>Infra</b>

## <b>Parte 0: Detalhamento dos serviços implantados na cloud</b>

<p align="justify">
Foi implantada uma infraestrutura de nuvem privada utilizando OpenStack, orquestrada com MAAS e Juju, e com armazenamento distribuído via Ceph. Antes de tudo, foi necessário garantir que todas as bridges da nossa nuvem estavam corretamente configuradas com a denominação "br-ex", já que esta é a interface de rede que conectará o OpenStack à rede externa.
</p>

<p align="justify"> 
Também, foi utilizada a interface do MaaS para garantir as especificações necessárias para as nossas máquinas físicas que estão listadas a seguir:
</p>

| Servidor | Tags | CPUs | NICs | RAM | Disks | Storage |
|----------|------|------|------|-----|-------|---------|
| server1  | controller, juju | 2 | 1 | 12 | 1 | 80
| server2  | reserva | 2 | 1 | 16 | 2 | 80
| server3  | compute | 2 | 1 | 32 | 2 | 80
| server4  | compute | 2 | 1 | 32 | 2 | 80
| server5  | compute | 2 | 1 | 32 | 2 | 80

<p align="justify"> 
Por fim, foi definido o modelo do deploy e seguido <a href="https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/install-openstack.html" target="_blank">o tutorial oficial do OpenStack</a> para instalar as seguintes dependências: 
</p>

``` bash
juju add-model --config default-series=jammy openstack

juju switch maas-controller:openstack
```

- Ceph OSD
- Nova Compute
- MySQL InnoDB Cluster
- Vault
- Neutron Networking
- Keystone 
- RabbitMQ
- Nova Cloud Controller
- Placement
- Horizon - OpenStack Dashboard
- Glance 
- Ceph Monitor
- Cinder
- Ceph RADOS Gateway

<p>Com as devidas integrações realizadas, foi executado um último comando de finalização da Infra:</p>

``` bash
juju config ceph-osd osd-devices='/dev/sdb'
``` 

# <b>Setup</b>

No setup, foi feita a configuração dos serviços que controlam:

- as VMs (Nova);
- os volumes de disco (Cinder);
- a estrutura de rede virtual (Neutron).

## <b>Parte 1: Autenticação</b>

## <b>Parte 2: Horizon</b>

## <b>Parte 3: Imagens e Flavors</b>

## <b>Parte 4: Rede Externa</b>

## <b>Parte 5: Rede Interna e Roteador</b>

## <b>Parte 6: Conexão</b>

## <b>Parte 7: Instância</b>

# <b>App</b>

## <b>Preparação da arquitetura</b>

<p align="justify">
A principal tarefa a ser completada no App foi a utilização da infraestrutura configurada até o momento para colocar o projeto da disciplina na cloud criada. 
</p>

<p align="justify"> 
Desenvolvido em paralelo, o projeto da disciplina — disponível <a href="https://github.com/antoniolma/insper.cloud-projeto" target="_blank">neste repositório</a> — consistiu em uma aplicação FastAPI composta por três endpoints simples. Para simular uma nuvem mais próxima da realidade, foi adotada a seguinte topologia: 
</p>

- 2 instâncias com a API desenvolvida;
- 1 instância com banco de dados Postgres;
- 1 instância com LoadBalancer, Nginx.

<p align="justify"> 
Em ambientes de produção, utilizar duas instâncias da API é considerado uma boa prática, pois permite garantir balanceamento de carga, tolerância a falhas e alta disponibilidade. Dessa forma, em vez de a requisição do cliente chegar diretamente a uma instância da API, ela é encaminhada primeiro ao Load Balancer, que distribui as requisições entre as instâncias disponíveis, as quais estão conectadas ao banco de dados da aplicação. A seguir, apresenta-se um diagrama que ilustra essa arquitetura: 
</p>

![App do Roteiro 3](./img/app_roteiro3.png)
/// caption
Topologia de uso da infraestrutura
///

<p align="justify"> 
Para iniciar a implementação, o dashboard do Horizon foi acessado (via NAT) e, nele, foram criadas as quatro instâncias com os seguintes nomes: load-balancer, api-1, api-2 e database.
</p>

![Instâncias- Horizon](./img/horizon_instances.png)
/// caption
Local de visualização e criação de instâncias no dashboard do Horizon
///

<p align="justify"> 
Após criadas as instâncias, foi necessário atribuir a cada uma um IP Flutuante para que fosse possível atingir todas por meio da NUC Main. Afinal, conforme mostrado no diagrama de topologia da rede anteriormente, as instâncias se encontram em uma subrede e possuem IPs que não são úteis para conversar com máquinas que se encontram na rede externa.
</p>

<p align="justify">
Para isso, foram atribuídos IPs flutuantes para cada instância (uma por vez), reutilizando os comandos vistos no Setup: 
</p>

``` bash
FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)

openstack server add floating ip nome-da-instancia $FLOATING_IP
```

<p align="justify">
Além disso, vale ressaltar que, a fim de baratear o custo de manutenção da arquitetura ao máximo sem prejudicar o desempenho do cliente, optou-se pelo flavor m1.tiny para todas as instâncias. 
</p>

<p align="justify">
Contudo, para aplicações com maior demanda e um volume de dados maior para armazenamento, seria provável a necessidade de escolher flavors maiores do que uma aplicação em ambiente controlado de aprendizado. Afinal, em uma cloud comercial, o custo é proporcional ao tamanho do flavor e seu tempo de uso.
</p>

## <b>Configuração da instância LoadBalancer</b>

<p align="justify">
Primeiramente, foi instalado o Nginx por meio do seguinte comandos:
</p>

``` bash
sudo apt-get install nginx
```

<p align="justify">
Em seguida, foi necessário editar o arquivo que permitiria o nginx a enxergar as duas instâncias para as quais o Load Balancer apontaria, conforme o diagrama ilustrado no início do relatório:
</p>

``` bash
# Acessando arquivo de configuração do nginx
sudo nano /etc/nginx/sites-available/default
```

<p align="justify">
Dentro do arquivo, as alterações feitas foram as seguintes:
</p>

``` bash
upstream backend {
        server [IP de subrede da instância da API 1]:8080;
        server [IP de subrede da instância da API 2]:8080;
}

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        location / { proxy_pass http://backend; 
}
```

<p align="justify">
Por fim, o serviço do nginx foi devidamente reinicializado para salvar as alterações feitas no arquivo:
</p>

``` bash
sudo service nginx restart
```

## <b>Instalação do Docker</b>

<p align="justify">
Tanto para as instâncias de API quanto para o banco de dados, foi necessário realizar a instalação do docker. Afinal, puxaremos a mesma imagem para as duas instâncias de API do projeto pelo docker hub. Para isso, foi seguido o tutorial oficial no <a href="https://docs.docker.com/engine/install/ubuntu/" target="_blank">site do docker</a>. Os comandos executados, portanto, foram:
</p>

``` bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
```

## <b>Configuração das instâncias de API</b>

<p align="justify">
Primeiramente, foram configuradas as variáveis de ambiente necessárias no arquivo .env. Aqui, além da URL para a interação com o banco de dados, também foram informadas as chaves necessárias para fazer as requisições para a API de cotação do dólar e do euro (um dos endpoints da aplicação):
</p>

``` bash
AWESOME_API_KEY="API_KEY_AQUI"
SECRET_KEY="SECRET_KEY_AQUI"
ALGORITHM=HS256

POSTGRES_USER=cloud
POSTGRES_PASSWORD=senha
POSTGRES_DB=cloud
DATABASE_URL=postgresql://cloud:senha@[IP de subrede do Banco de Dados]:5432/cloud
```

<p align="justify">
Em seguida, a imagem do projeto publicada no docker hub foi puxada e o container foi executada nas duas instâncias.
</p>

``` bash
sudo docker pull antoniolma/app
sudo docker run -p 8080:80 --env-file .env -d antoniolma/app

# Verificando se o container está sendo executado na máquina
sudo docker ps -a
```

## <b>Configuração da instância do Banco de Dados</b>

<p align="justify">
Primeiramente, foi instalado o PostgreSQL por meio dos seguintes comandos:
</p>

``` bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

<p align="justify">
Em seguida, assim como feito nas APIs, foram configuradas as variáveis de ambiente necessárias no arquivo .env:
</p>

``` bash
POSTGRES_USER=cloud
POSTGRES_PASSWORD=senha
POSTGRES_DB=cloud
```

<p align="justify">
Por fim, foi executado o docker na porta padrão do postgres para que as instâncias de API possam enxergar o banco de dados:
</p>

``` bash
sudo docker run -p 5432:5432 --env-file .env -d postgres
```

## <b>Verificação final</b>

<p align="justify">
Para conferir o devido funcionamento da infraestrutura, foi criado um túnel SSH que conectasse o computador local a uma das instâncias de API criada. Para isso, foi executado o mesmo comando de túnel utilizado no roteiro 1, com as devidas adaptações necessárias:
</p>

``` bash
$ ssh cloud@10.103.1.10 -L 8080:[IP flutuante load-balancer]:80 
``` 

**Tarefa 4.2) Arquitetura de rede da infraestrutura dentro do Dashboard do OpenStack**

![Arquitetura de rede](./img/tarefa4_2.jpg)
/// caption
Arquitetura de rede final
///

**Tarefa 4.3) Lista de VMs utilizadas com nome e IPs alocados,**

![Lista de VMs](./img/tarefa4_3.jpg)
/// caption
Visualização dos nomes e IPs alocados via dashboard do Horizon
///

**Tarefa 4.4) Dashboard do FastAPI conectado via máquina Nginx/LB.**

![Dashboard FastAPI](./img/tarefa4_4.jpg)
/// caption
Dashboard do FastAPI acessado após a aplicação do túnel SSH via máquina Nginx
///

**Tarefa 4.5) Servidores (máquina fisica) alocados para cada instância pelo OpenStack.**

![Alocação física do Load Balancer](./img/tarefa4_5_load-balancer.jpg)
/// caption
Máquina física alocada para o Load Balancer pelo OpenStack: Server 5
///
![Alocação física da API 1](./img/tarefa4_5_api-1.jpg)
/// caption
Máquina física alocada para a API 1 pelo OpenStack: Server 4
///
![Alocação física da API 2](./img/tarefa4_5_api-2.jpg)
/// caption
Máquina física alocada para a API 2 pelo OpenStack: Server 2
///
![Alocação física do Banco de Dados](./img/tarefa4_5_database.jpg)
/// caption
Máquina física alocada para o Banco de dados Postgres pelo OpenStack: Server 3
///