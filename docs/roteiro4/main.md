## <b>Introdução e Objetivo</b>

<p align="justify">
O roteiro 4 contempla uma abordagem nova, mantendo a utlização do Juju para gerenciamento de aplicações distribuídas e o openstack implementado no roteiro anterio, mas explorando agora o conceito de Infraestrutura como código através do Terraform.
</p>

## <b>Montagem do Roteiro</b>

<p align="justify">
Todo roteiro apresenta uma primeira parte denominada <b>Infra</b> e uma segunda chamada de <b>App</b>.
Os pontos <b>tarefas</b> dentro de cada parte são os passos seguidos para a realização do roteiro. 
Este modelo de organização orientado por partes e tarefas será utilizado em <b>todos os roteiros</b>.
</p>

# <b>Infra</b>

<p align="justify">
Até o momento, foram utilizados o dashboard e a interface de linha de comando (CLI) para criar rede, subrede, instâncias, roteadorres e outros recursos. Para criar a infraestrutura necessária agora, conforme será visto mais a frente no App, foi utilizado somente código.
</p>

<p align="justify">
Primeiramente, para não confundir os recursos de cada usuário, foi criada uma separação lógica de dois usuários inseridos em um mesmo domínio (assim como deveria acontecer em nuvem): aluno1 e aluno2. 
</p>

## <b>Parte 1: Criando um único Domain</b>

<p align="justify">
Via Horizon Dashboard, foi criado o domínio AlunosDomain:
</p>

![Novo domínio](./img/criacao_dominio.png)
/// caption
Criação de domínio novo do Horizon Dashboard
///

<p align="justify">
Em seguida, o novo domínio criado foi definido como o novo contexto de uso, conforme mostra a imagem a seguir:
</p>

![Contexto de uso](./img/dominio_contexto.png)
/// caption
Novo domínio como contexto de uso
///

## <b>Parte 2: Criando um projeto para cada Aluno</b>

<p align="justify">
Para separar o aluno1 do aluno2, foram feitos dois projetos que respeitassem o padrão Kit + letra do kit + nome_do_aluno.
</p>

![Criando os projetos](./img/criando_projeto.png)
/// caption
Interface de criação dos projetos para cada usuário aluno
///

<p align="justify">
Por fim, os dois usuários foram criados também utilizando a interface do Horizon Dashboard, dando atenção especial para garantir que ambos tivessem papéis administrativos <b>tanto no momento de criação, quanto na configuração do domínio</b>. O domínio AlunosDomain e os respectivos projetos criados foram informados para cada um.
</p>

![Criando os usuários](./img/criando_usuarios.png)
/// caption
Interface de criação dos usuários
///

![Configurando o domínio](./img/configuracao_dominio.png)
/// caption
Concedendo papel administrativo para ambos os usuários no domínio criado
///

# <b>App</b>

## <b>Parte 1: Criando arquivos do Terraform</b>

<p align="justify">
Utilizando a estrutura de pastas mostrada a seguir, cada aluno realizou a criação de sua própria infraestrutura em uma pasta própria (seguindo o padrão Kit + Letra do Kit + Primeiro Nome) dentro da máquina MAIN.
</p>

![Estrutura pastas Terraform](./img/estrutura_terraform.png)
/// caption
Estrutura de pastas seguida por cada aluno na configuração do terraform
///

Os arquivos utilizados estão detalhados abaixo:

provider.tf
``` bash
# Define required providers
terraform {
required_version = ">= 0.14.0"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.35.0"
    }
  }
}


# Configure the OpenStack Provider

provider "openstack" {
  region              = "RegionOne"
  user_name           = "SEU_USUARIO" # Aqui, alterou-se para aluno1/aluno2
}
```

instance1.tf

``` bash
resource "openstack_compute_instance_v2" "instancia_1" {
  name            = "basic"
  image_name      = "jammy-amd64" # Para saber a imagem adequada, acessamos Project > Compute > Images no Dashboard do Openstack
  flavor_name     = "m1.small"
  key_pair        = "mykey"
  security_groups = ["default"]

  network {
    name = "network_1" # Aqui, o nome da rede mudou de um aluno para outro
  }

  depends_on = [openstack_networking_network_v2.network_1]

}
```

instance2.tf

``` bash
resource "openstack_compute_instance_v2" "instancia_2" {
  name            = "basic2"
  image_name      = "jammy-amd64" # Para saber a imagem adequada, acessamos Project > Compute > Images no Dashboard do Openstack
  flavor_name     = "m1.tiny"
  key_pair        = "mykey"
  security_groups = ["default"]

  network {
    name = "network_1" # Aqui, o nome da rede mudou de um aluno para outro
  }

  depends_on = [openstack_networking_network_v2.network_1]

}
```

network.tf

``` bash
resource "openstack_networking_network_v2" "network_1" {
  name           = "network_1" # Aqui, o nome da rede mudou de um aluno para outro
  admin_state_up = "true"
}

resource "openstack_networking_subnet_v2" "subnet_1" {
  network_id = "${openstack_networking_network_v2.network_1.id}"
  cidr       = "192.167.199.0/24"
}
```

router.tf

``` bash
resource "openstack_networking_router_v2" "router_1" {
  name                = "my_router"
  admin_state_up      = true
  external_network_id = <"ID_EXT_NETWORK"> # Aqui, foi utilizado o ID da rede externa, disponível no Dashboard do Openstack ou via CLI openstack
}

resource "openstack_networking_router_interface_v2" "int_1" {
  router_id = "${openstack_networking_router_v2.router_1.id}"
  subnet_id = "${openstack_networking_subnet_v2.subnet_1.id}"
}
```

<p align="justify">
<b>Importante: </b>
Além da criação desses arquivos, cada aluno precisou criar um par de chaves próprio (denominado de "mykey" nos arquivos relativos às instâncias), acessando o Dashboard do OpenStack em Project > Compute > Key Pairs > Create Key Pair.
</p>

## <b>Parte 2: Credenciais dos respectivos usuários</b>

<p align="justify"> 
Antes de aplicar as mudanças definidas nos arquivos Terraform, era necessário realizar um último passo: adicionar o arquivo de credenciais de cada usuário. Para isso, foi feito o download do arquivo OpenStack RC diretamente pelo Dashboard, no caminho <b>Project → API Access</b>, selecionando a opção <b>"Download OpenStack RC File"</b> para cada usuário (aluno1 e aluno2). 
</p>

<p align="justify"> 
Após o download, o conteúdo de cada arquivo foi copiado e salvo em um novo arquivo dentro do mesmo diretório dos demais, utilizando o padrão <code>openrc.sh</code>. Em seguida, foi atribuído permissão de execução com o comando <code>chmod +x arquivo.sh</code>. Por fim, as variáveis de ambiente foram carregadas executando <code>source arquivo.sh</code>, garantindo que as credenciais daquele usuário estejam ativas na sessão. 
</p>

## <b>Parte 3: Implementação e verificação da infraestrutura</b>

Para fazer a implementação da infraestrutura, foram executados, por fim, os comandos abaixo:

``` bash
terraform plan # Cria o plano de execução antes do Terraform aplicar as configurações feitas

terraform apply # Aplica as mudanças necessárias a partir do plano de execução
``` 

<p align="justify"> 
Com o intuito de verificar que tudo ocorreu normalmente após os comandos acima, ainda foi executado o comando <code>openstack server list</code>, que devolveu as duas instâncias criadas (para cada usuário):
</p>

``` bash
+--------------------------------------+-----------+--------+--------------------------+--------------------------+---------+
| ID                                   | Name      | Status | Networks                 | Image                    | Flavor  |
+--------------------------------------+-----------+--------+--------------------------+--------------------------+---------+
| a99cbd57-eb44-4c0f-a441-6feaf73a16a6 | basic2    | ACTIVE | network_1=192.167.199.11 | jammy-amd64              | m1.tiny |
| cd7a3480-5bd8-4d94-9746-deac0d3324a2 | basic     | ACTIVE | network_1=192.167.199.74 | jammy-amd64              | m1.small|
+--------------------------------------+-----------+--------+---------------------+--------------------------+--------------+
```

## <b>Parte 4: Checkpoint por usuário após a criação das instâncias</b>

**Tarefa 1) Prints do Dashboard do OpenStack para o usuário aluno1**

![Tarefa 1.1 - Aluno 1](./img/aluno1/tarefa1_1_aluno1.jpg)
/// caption
Aba Identity (Identidade) > Projects (Projetos) do Dashboard do OpenStack para o aluno1 
///

![Tarefa 1.2 - Aluno 1](./img/aluno1/tarefa1_2_aluno1.jpg)
/// caption
Aba Identity (Identidade) > Users (Usuários) do Dashboard do OpenStack para o aluno1
///

![Tarefa 1.3 - Aluno 1](./img/aluno1/tarefa1_3_aluno1.jpg)
/// caption
Aba Compute (Computação) > Overview (Visão geral) do Dashboard do OpenStack para o aluno1
///

![Tarefa 1.4 - Aluno 1](./img/aluno1/tarefa1_4_aluno1.jpg)
/// caption
Aba Compute (Computação) > Instances (Instâncias) do Dashboard do OpenStack para o aluno1
///

![Tarefa 1.5 - Aluno 1](./img/aluno1/tarefa1_5_aluno1.jpg)
/// caption
Aba de Topologia de Rede (Network Topology) do Dashboard do OpenStack para o aluno1
///

----------------------------------------------------------------

**Tarefa 2) Prints do Dashboard do OpenStack para o usuário aluno2**

![Tarefa 1.1 - Aluno 2](./img/aluno2/tarefa1_1_aluno2.jpg)
/// caption
Aba Identity (Identidade) > Projects (Projetos) do Dashboard do OpenStack para o aluno2 
///

![Tarefa 1.2 - Aluno 2](./img/aluno2/tarefa1_2_aluno2.jpg)
/// caption
Aba Identity (Identidade) > Users (Usuários) do Dashboard do OpenStack para o aluno2
///

![Tarefa 1.3 - Aluno 2](./img/aluno2/tarefa1_3_aluno2.jpg)
/// caption
Aba Compute (Computação) > Overview (Visão geral) do Dashboard do OpenStack para o aluno2
///

![Tarefa 1.4 - Aluno 2](./img/aluno2/tarefa1_4_aluno2.jpg)
/// caption
Aba Compute (Computação) > Instances (Instâncias) do Dashboard do OpenStack para o aluno2
///

![Tarefa 1.5 - Aluno 2](./img/aluno2/tarefa1_5_aluno2.jpg)
/// caption
Aba de Topologia de Rede (Network Topology) do Dashboard do OpenStack para o aluno2
///

## <b>Parte 5: Criando um plano de Disaster Recovery e SLA (QUESTÕES)</b>

!!! note "Exercise"
    <p align="jusitfy">
    Você é o CTO (Chief Technology Officer) de uma grande empresa com sede em várias capitais no Brasil e precisa implantar um sistema crítico, de baixo custo e com dados sigilosos para a área operacional.
    </p>

    a) Você escolheria Public Cloud ou Private Cloud?

    b) Agora explique para ao RH por que você precisa de um time de DevOps.

    <p align="jusitfy"> 
    c) Considerando o mesmo sistema crítico, agora sua equipe deverá planejar e implementar um ambiente resiliente e capaz de mitigar possíveis interrupções/indisponibilidades. Para isso, identifiquem quais são as principais ameaças que podem colocar sua infraestrutura em risco, e descreva as principais ações que possibilitem o restabelecimento de todas as aplicações de forma rápida e organizada caso algum evento cause uma interrupção ou incidente de segurança. Para isso monte um plano de DR e HA que considere entre as ações:
    </p>

    - Mapeamento das principais ameaças que podem colocar em riscos o seu ambiente.
    - Elenque e priorize as ações para a recuperação de seu ambiente em uma possível interrupção/desastre.
    - Como sua equipe irá tratar a política de backup?
    - Considerando possíveis instabilidades e problemas, descreva como alta disponibilidade será implementada em sua infraestrutura.

<!-- a) -->
<p align="jusitfy">
<b>a)</b> Para um <b>sistema crítico</b>, de <b>baixo custo</b> e que lida com <b>dados sigilosos</b> na área operacional, a escolha mais adequada é: <b>Private Cloud</b>. Justamente por conta de se tratar de <b>dados sensíveis</b> e haver a necessidade de estabelecer <b>medidas de segurança</b> específicas ao contexto da empresa, a opção mais segura seria optar pela Cloud privada.
</p>

<!-- b) -->
<p align="jusitfy">
<b>b)</b> E-mail para a equipe de RH
</p>

!!! note "E-mail para a equipe de RH"
    **Assunto:** Solicitação de equipe DevOps
    
    *Olá, equipe de RH, tudo bem?*

    *No momento, nós da equipe de implementação do principal projeto da área de 
    tecnologia e infraestrutura da empresa estivemos criando o plano de ação para 
    como realizar podemos realizar este projeto da melhor forma possível, porém
    utilizando somente os recursos estritamente necessários para a implementação
    e chegamos a conclusão de que, para entregar um sistema crítico, seguro e de qualidade,
    precisamos formar um time de <b>DevOps</b>.*

    *Dado a escala do projeto, um time de DevOps seria essencial para podermos melhorar
    a qualidade da entrega, eliminando gargalos e criando um fluxo contínuo de desenvolvimento,
    reduzindo drasticamente o tempo de entrega de novas funcionalidades e correções críticas.
    Isso traria um ambiente de trabalho mais eficiente e resultaria em uma maior segurança do sistema.*
    
    *Vocês poderiam abrir novas vagas ou realocar profissionais para esse
    time o quanto antes?*

    *Desde já agradeço!*  
    *Atenciosamente,*
    *CTO – Empresa TechnoCloud*


<!-- c) -->
<p align="justify">
<b>c)</b> Para garantir que nosso sistema crítico sobreviva a falhas e se recupere rapidamente, vamos estruturar um <b>plano de recuperação de desastres (DR)</b> e <b>alta disponibilidade (HA)</b>. Vamos dividir este plano em quatro principais etapas:s
</p>

<h3>1. Mapeamento das Principais Ameaças</h3>
<ul>
  <li><b>Falhas de Hardware:</b> Pane em servidores (CPU, memória, disco) ou componentes de rede.</li>
  <li><b>Interrupções de Energia/Climatéricas:</b> Quedas de energia, picos de tensão, enchentes, tempestades ou incêndios no datacenter.</li>
  <li><b>Falhas de Software:</b> Bugs em atualizações do SO ou módulos críticos; corrupção de banco de dados.</li>
  <li><b>Cyberataques:</b> Ransomware, DDoS, exploração de vulnerabilidades, acesso não autorizado.</li>
  <li><b>Erro Humano:</b> Configuração incorreta de equipamentos, deploy incorreto de código, remoção acidental de dados.</li>
  <li><b>Problemas de Rede/Telco:</b> Latência ou queda de links de Internet/MPLS entre capitais.</li>
</ul>

<h3>2. Plano de Ações para Recuperação (Prioridades)</h3>
<ul>
  <li><b>Prioridade 1 – Infraestrutura Básica (RTO curto):</b>
    <ul>
      <li>Fontes redundantes (UPS+gerador) e links de Internet múltiplos.</li>
      <li>Datacenter primário + réplica ativa/stand-by em outra capital.</li>
      <li>Sincronização contínua de VMs/containers (Storage Replication).</li>
    </ul>
  </li>
  <li><b>Prioridade 2 – Camada de Aplicação e Banco (RTO médio):</b>
    <ul>
      <li>Cluster de banco primário com réplica (síncrona/semissíncrona) em DC secundário.</li>
      <li>Load Balancer em modo ativo-ativo ou ativo-passivo; servidores de aplicação em múltiplas zonas.</li>
    </ul>
  </li>
  <li><b>Prioridade 3 – Integrações e Serviços Externos (RTO estendido):</b>
    <ul>
      <li>Contêineres orquestrados replicados (Kubernetes/OpenShift) em ambas localidades.</li>
      <li>Circuit Breakers para degradar funcionalidades não críticas em alta latência.</li>
      <li>Fila de mensageria (Kafka/RabbitMQ) replicada; políticas de retry e DLQ.</li>
    </ul>
  </li>
  <li><b>Prioridade 4 – Serviços de Suporte (RTO menos crítico):</b>
    <ul>
      <li>Elastic Stack (Elasticsearch+Kibana) espelhado entre DCs; Prometheus+Grafana replicados.</li>
      <li>Active Directory/LDAP redundante; failover automático de DNS.</li>
    </ul>
  </li>
</ul>

<h3>3. Política de Backup</h3>
<ul>
  <li><b>Escopo e Frequência:</b>
    <ul>
      <li><em>Backup Diário (RPO ≤ 24h):</em> Dump full nos bancos relacionais + incremental a cada 4h; arquivos críticos (certificados, scripts) em repositório versionado + snapshot diário.</li>
      <li><em>Backup Semanal (RPO ≥ 1 semana):</em> Arquivos de auditoria e relatórios em backup full semanal; retenção de 8 semanas.</li>
      <li><em>Backup Mensal (RPO ≥ 1 mês):</em> Snapshots de VMs (templates/golden images) em fita virtual ou storage de arquivamento.</li>
    </ul>
  </li>
  <li><b>Local de Armazenamento:</b>
    <ul>
      <li><em>On-Site:</em> NAS com RAID + VLAN exclusiva para tráfego de backup.</li>
      <li><em>Off-Site:</em> Cópias diárias criptografadas (AES-256) por VPN para DR site em outra capital; opcionalmente, terceiro datacenter geograficamente distante (~200 km).</li>
    </ul>
  </li>
  <li><b>Armazenamento e Retenção:</b>
    <ul>
      <li>Full mensal com retenção de 12 meses; backups semanais retidos por 8 semanas; diários retidos por 14 dias.</li>
      <li>RPO máximo de 4 horas para dados críticos; 24 h para configurações.</li>
    </ul>
  </li>
  <li><b>Testes de Restauração:</b>
    <ul>
      <li>Restauração completa em homologação a cada 3 meses.</li>
      <li>Simulação de perda total do datacenter primário ao menos uma vez ao ano.</li>
    </ul>
  </li>
</ul>

<h3>4. Implementação de Alta Disponibilidade (HA)</h3>
<ul>
  <li><b>Camada de Rede e Balanceamento:</b>
    <ul>
      <li>Load Balancers redundantes (ativo-passivo ou ativo-ativo) em DCs diferentes com health checks a cada 30 s e failover de VIP.</li>
      <li>Multipath Routing via BGP multipath entre ISPs distintos.</li>
    </ul>
  </li>
  <li><b>Camada de Aplicação:</b>
    <ul>
      <li>Cluster de contêineres/VMs autoescaláveis em duas ou mais capitais, orquestrados por K8s/OpenShift/VMware Tanzu.</li>
      <li>Autoscaling baseado em métricas (CPU, memória, tempo de resposta) para escalar horizontalmente.</li>
    </ul>
  </li>
  <li><b>Camada de Banco de Dados e Armazenamento:</b>
    <ul>
      <li>Cluster primário com réplica síncrona em DC secundário; promoção automática de réplica se o primário falhar.</li>
      <li>Storage compartilhado (NAS/SAN) replicado (GlusterFS, Ceph ou soluções dedicadas) com acesso em múltiplos hosts.</li>
    </ul>
  </li>
  <li><b>Camada de Serviços de Suporte:</b>
    <ul>
      <li>DNS dinâmico apontando para múltiplos IPs de balanceadores com TTL baixo (30-60 s) para failover rápido.</li>
      <li>Identity Provider redundante (LDAP/AD) sincronizando contas em tempo real; controlador de domínio secundário assume automaticamente.</li>
    </ul>
  </li>
  <li><b>Monitoramento e Orquestração de Failover:</b>
    <ul>
      <li>Monitoramento 24×7 (Zabbix/Prometheus) com alertas e dashboards (Grafana).</li>
      <li>Playbooks de resposta a incidentes (runbooks) para cada tipo de falha; drills trimestrais.</li>
      <li>Orquestração de failover automático (Pacemaker/Corosync, Keepalived, K8s Operators) para cenários de falha simples.</li>
    </ul>
  </li>
</ul>