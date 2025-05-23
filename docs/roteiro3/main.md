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
Por fim, foi definido o modelo do deploy e seguido o tutorial oficial do OpenStack para instalar as seguintes dependências: 
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

# <b>App</b>

## <b>Questionário, Projeto ou Plano</b>

Esse seção deve ser preenchida apenas se houver demanda do roteiro.

## <b>Discussões</b>

Quais as dificuldades encontradas? O que foi mais fácil? O que foi mais difícil?

## <b>Conclusão</b>

O que foi possível concluir com a realização do roteiro?