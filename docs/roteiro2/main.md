## <b>Introdução e Objetivo</b>

<p align="justify">
O roteiro 2 contempla uma abordagem nova em relação ao roteiro 1: a utilização de uma nova plataforma de gerenciamento de aplicações distribuídas. Assim, em vez de realizar as instalações de toda a infraestrutura manualmente (conforme foi feito no roteiro anterior), será utilizado o Juju, um outro orquestrador de deploy que integra com o MaaS.
</p>

<p align="justify">
Por conta desta nova abordagem, todas as máquinas contendo as modificações do roteiro 1 foram liberadas, retornando para o status "Ready" no dashboard do MaaS.
</p>

<p align="justify">
Ao final deste roteiro, o objetivo principal é termos, portanto, uma Cloud com um novo gerenciador de deploy instalado.
</p>

## <b>Montagem do Roteiro</b>

<p align="justify">
Todo roteiro apresenta uma primeira parte denonimnada <b>Infra</b> e uma segunda chamada de <b>App</b>.
Os pontos <b>tarefas</b> dentro de cada parte são os passos seguidos para a realização do roteiro. 
Este modelo de organização orientado por partes e tarefas será utilizado em <b>todos os roteiros</b>.
</p>

# <b>Infra</b>

## <b>Parte 0: Instalação do Juju</b>

<p align="justify">
Acessando a máquina main via SSH, foi realizada a instalação do Juju através do seguinte comando:
</p>

``` bash
$ sudo snap install juju --channel 3.6
```

# <b>App</b>

## <b>Conclusão</b>

O que foi possível concluir com a realização do roteiro?