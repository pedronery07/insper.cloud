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

## <b>Parte 1: Instalação do Juju</b>

<p align="justify">
A instalação do Juju foi realizada na máquina <b>main</b>, acessada via SSH. O seguinte comando foi utilizado:
</p>

``` bash
$ sudo snap install juju --channel 3.6
```

## <b>Parte 2: Arquivos de definição de cloud</b>

<p align="justify">
Como o Juju utilizará o MaaS como provedor de máquinas e sistema operacional, inicialmente foi necessário garantir que o Juju reconhecesse o MaaS como um provedor de recursos válido.
</p>

Após essa verificação, criou-se um arquivo de configuração chamado *maas-cloud.yaml* com o seguinte conteúdo:

``` bash
  clouds:
    maas-one:
      type: maas
      auth-types: [oauth1]
      endpoint: http://192.168.0.3:5240/MAAS/
```

Em seguida, foi adicionada a cloud, utilizando o seguinte comando:

``` bash
$ juju add-cloud --client -f maas-cloud.yaml maas-one
```

Por fim, foi necessário adicionar as credenciais MaaS para que o Juju pudesse interagir com a nova cloud adicionada. Um novo arquivo, denominado como *maas-creds.yaml*, foi criado com esta finalidade:

``` bash
credentials:
  maas-one:
  anyuser:
    auth-type: oauth1
    maas-oauth: <API KEY gerado no menu do usuário do MaaS>
```

Essas credenciais foram aplicadas com o comando:

``` bash
$ juju add-credential --client -f maas-creds.yaml maas-one
```

## <b>Parte 3: Criação do controlador</b>

Para finalizar a infraestrutura necessária, foi criado um controlador no **server 1** da nossa rede privada.

<p align="justify">
Para que o Juju saiba em qual servidor o controlador irá ficar, criou-se a tag 'juju' no <b>server 1</b> através dashboard do Maas e, em seguida, foi executado o comando a seguir:
</p>

``` bash
$ juju bootstrap --bootstrap-series=jammy --constraints tags=juju maas-one maas-controller
```

<p align="justify">
Esse comando utiliza a série <b>jammy</b> e define o nome do controlador como <b>maas-controller</b>, vinculado à cloud <b>maas-one</b>.
</p>

# <b>App</b>

## <b>Conclusão</b>

O que foi possível concluir com a realização do roteiro?