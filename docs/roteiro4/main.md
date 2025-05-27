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

## <b>Questionário, Projeto ou Plano</b>

Esse seção deve ser preenchida apenas se houver demanda do roteiro.

## <b>Discussões</b>

Quais as dificuldades encontradas? O que foi mais fácil? O que foi mais difícil?

## <b>Conclusão</b>

O que foi possível concluir com a realização do roteiro?