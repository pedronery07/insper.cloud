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
    - os volumes de disco (Cinder), e;
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

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

<p align="justify">
instalação do nginx -> configuração do arquivo com upstream apontando para as APIs e proxy
</p>

## <b>Configuração das instâncias de API</b>

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

<p align="justify">
instalação docker -> .env -> docker pull -> docker run
</p>

## <b>Configuração da instância de Banco de Dados</b>

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

<p align="justify">
instalação postgres -> instalação docker -> .env -> docker run
</p>

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
Máquina física alocada para o Load Balancer pelo OpenStack
///
![Alocação física da API 1](./img/tarefa4_5_api-1.jpg)
/// caption
Máquina física alocada para a API 1 pelo OpenStack
///
![Alocação física da API 2](./img/tarefa4_5_api-2.jpg)
/// caption
Máquina física alocada para a API 2 pelo OpenStack
///
![Alocação física do Banco de Dados](./img/tarefa4_5_database.jpg)
/// caption
Máquina física alocada para o Banco de dados Postgres pelo OpenStack
///