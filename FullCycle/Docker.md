
# Docker

# Começando do zero
Pulei a parte de instalação do WSL, já que estou usando linux.

## - O que é um container??

**Definição:**

    É uma pseudo máquina virtual. É mais leve e transpararente,
    onde eu não preciso de dividir recursos ou configurá-la.


Para construirmos uma definição com fundamentos mais profundos, precisamos antes entender alguns conceitos do Sistema Operacional (SO).

## - O que são os processos de um SO?
Em um ambiente, seja ele Linux, Windows ou macOS, cada processo está relacionado à uma tarefa em execução. Com o amadurecimento dos Sistemas Operacionais, isolou-se cada processo em um Namespace.

**Assim, temos:**

    Namespace X
    {
        Processo PAI 1
        {
            Processo FILHO 1
            Processo FILHO 2
            Processo FILHO 3
        }
    }

    Namespace Y
    {
        Processo PAI 2
        {
            Processo FILHO 4
            Processo FILHO 5
            Processo FILHO 6
            ...
        }
    }

Os namespaces são responsáveis por segregar os processos pais uns dos outros. De modo que fique claro para o sistema quais subprocessos estão ligados entre sí e quais não estão.

## O que é um container?
Desse modo, quando falamos de containers, estamos falando de isolamento de processos (pais) em namespaces distintos.

**CONTAINER == ISOLAMENTO**

**Lembrando que a exclusão de um:**
- Processo pai, namespace ou container ocasiona a destruição de todos os processos filhos.
- Processo filho pode prejudicar ou impedir a execução do processo pai ou dos processos irmãos.

==========================
Passar à limpo:

Devido à essa separação em namespaces cada container vê apenas seus processos internos e está alheio aos demais conteiners e aos outos processos do sistema.

O container é um processo com subprocessos que emulam um sistema operacional.

## Cgroups
Controla os recursos computacionais do container, permite que eu defina quanto de memória e cpu o meu container pode utilizar.

Isso impede (ou pele menos tenta impedir) que um processo consuma todos os recursos do computador em caso de um memory leak ou de uma alta demanda, por exemplo.

Se um container é o isolamento de processos, não faria sentido que ele consumisse todos os recursos disponivéis sendo que eu tenho outros processos rodando que também precisam desses recursos.

## File System
Com container podemos trabalhar com camadas, para isso utilizamos o OFS (Overlay File System). Com o OFS, não precisamos "clonar" todo o sistema ou todas as dependências. Podemos criar uma imagem apenas com as diferenças entre as imagens.

    {
        [App:V1 = 100mb]
        [Dependência 1 = 200mb]
        [Dependência 2 = 300mb]
    }

    ====>>>
    {
        [App:V2 = 150mb]
        [Já tenho as dependências 1 e 2]
        [Dependência 3 = 50mb]
    }

    ====>>>
    {
        [App:V3 = 130mb]
        [Já tenho as dependências 1, 2 e 3]
    }

Antes dos containers, eu precisaria duplicar as dependências 1, 2 e 3 (e todo o sistema operacional) cada vez que eu fosse para uma nova versão do app. Já com containers, como eu já tenho a dependência "instalada" em uma camada, eu posso simplesmente utilizar essa camada. Ou seja, consigo reutilizar a imagem da dependência em diferentes processos.

## Imagens

As imagens muitas vezes são chamadas de snapshots, fotos de como meu sistema operacional está naquela momento.

Mas, quando falamos de docker essas imagens são criadas a partir de camadas.

    |- App1            |- App2
    |                  |
    |- Bash            |- ssh.d
    |                  |
    |- [Ubuntu - Partes que não estão presentes no meu SO]
    |
    |-[Imagem vazia - HD formatado]

Nesse desenho, temos dois apps que compartilham uma imagem do ubuntu (essa imagem contém apenas o que o meu SO não possuí) e cada a app utiliza uma imagem específica (Bash e ssh.d) que roda no ubuntu.

Desse modo, temos imagens isoladas em uma árvore de dependência com outras imagens.

Uma imagem vai afetar toda a àrvore acima dela porquê é utilizada por todas as dependências acima, mas por ser construída em camadas, podemos consertar a camada específica e rebuildar a imagem sem gerar impacto nas demais camadas, já que elas são isoladas.

**Uma imagem é um conjunto de dependências isoladas e encadeadas.** Nesse casso, a imagem do meu App1, seria construída assim:

    App1Image
    [
        |- App1            
        |                  
        |- Bash            
        |                  
        |- [Ubuntu - Partes que não estão presentes no meu SO]
        |
        |-[Imagem vazia - HD formatado]
    ]


## Dockerfile
Declara e define como vai ser a imagem a ser buildada.

Normalmente, não começamos a partir de uma imagem vazia, como no exemplo acima. Na prática pegamos uma imagem pronta e partimos dela.

Para dar pull nessa imagem, utilizamos:
    FROM: ImageName {ex: Ubuntu}

Depois, caso eu queira rodar um comando:
    RUN: Commando {ex: apt-get install x}

Exponha a porta 8000:
    EXPOSE: 8000

**O Dockerfile só é utilizado na construção de uma imagem, se você não for alterar absolutamente nada na imagem, você não precisa de um Dockerfile.**

## Como um container funciona??

Se dermos um zoom no container poderemos que dentro dele temos uma **Imagem em Estado Imutável**.

O container é muito leve e rápido porquê ele roda em um processo em tempo real.

    Processo
    [
        Container
        [
            Imagem - Imutável
            Camada Read/Write
        ]
    ]

A gente só consegue fazer alterações no container enquanto ele roda, por causa da camada de Read/Write, é nessa layer que fazemos as alterações e não na imagem em si.


**Dockerfile + Build ==> Nova Imagem1**

ou
**Imagem1 + Commit(no read/Write) ===> Nova Imagem2**


## Onde ficam as imagens?
As imagens ficam guardadas em um Image Registry, uma espécie de repositório.

E sempre que rodamos um Dockerfile ou criamos um container, a imagem desse container vai para esse registro e quando precisamos pegamos a imagem pronta lá.
    Dockerfile                      Image Registry
    From: ImageName <<<----Pull---- ImageName

    ======== BUILD ========

    Container                  Image Registry
    ImageName2 ----Push---->>> ImageName2


## Como o Docker funciona?
Integrando os 3 fundamentos de containers (Namespace, Cgroups e File System) o Docker criou o conceito de Docker Host.

O Docker ele tem um host onde ele vai rodar. E, esse host na verdade ele fica rodando um processo que é uma daemon que disponibiliza sempre  uma API.

Pra conseguirmos falar com o Docker Host, vamos precisar de um Docker Client que vai se comunicar com a API disponibilizada pelo daemon lá no host.

Os comandos do Docker, nada mais são que uma invocação do Client do Docker para se comunicar com a API do Deamon do host.

**Docker Client pode:**
- Criar containers
- Run, Push, Pull
- Volumes
- Network

**Docker Host tem:**
- Daemon - API
- Cache (Pull/Push do Registry)
- Gerenciamento de volumes (Persistência de dados)
- Gerenciamento de Network (Conexão entre containers)

O legal do gerenciamento de volumes é a capacidade dele permitir ao container persistir dados no Sistema Operacional. Desse modo, mesmo que eu mate e rode novamente, meus dados continuarão disponíveis.



# Iniciando com Docker

## Comandos
    sudo systemctl start docker
Starta o docker engine

    docker ps
Lista os containers ativos

    docker run hello-world 
Testa se o docker foi instalado corretamente.

**Notas:**
- hello-world:latest == hello-world, quando você omite a tag ele pega a mais recente

- Caso você rode o docker ps depois do hello-world, vai perceber que nenhum container está ativo. Isso ocorre porquê o entrypoint define que o container deve ser destruído logo após a sua execução.


    docker ps -a
Lista todos os containers (ativos e inativos)


    docker run -it ubuntu bash

- **-i** --> modo interativo, manter o container ativo e faz um attach no nosso terminal
- **-t** --> tty, permite que eu rode comandos no terminal


Um container é um processo, então se ele finalizar o que tinha pra fazer o processo morre e por consequência o container também.

- **--rm** --> deleta o container depois que ele for stoppado


##NGINX - Engine X

Servidor, proxy reverso, web server

    docker run nginx

O fato do container estar com a porta 80 ativa, não significa que eu tenho acesso à ela.


    [Mundo Externo]
           |
           X   // O Docker NÃO liberou acesso
           |   // à porta 80, ela não existe externamente
        [Docker]
           |
           80  // O NGINX está liberando a porta
           |   // 80 para o Docker, só o Docker    
           |   // tem acesso à ela.
        [NGINX]

A porta do container ativa não obriga o Host a liberar o acesso à essa porta

    docker run -p 8080:80 nginx
- **-p 8080:80** --> Publica a porta 8080 e mapeia as conexeções do mundo externo (8080) para a porta do container (80).
Assim, quando eu acesso o localhost:8080, o docker me redireciona para a porta 80 do container que está rodando o nginx.

    docker run -d -p 8080:80 nginx
- **-d** --> Detached, des attachar o terminal do container. Possibilita que eu continue usando o terminal, não bloqueia com o docker rodando.

Como Matar o container?

    docker ps
    docker stop {CONTAINER ID}
    
Como startar o container?

    docker ps
    docker start {CONTEINER ID}

Como Remover um container?

    docker rm {CONTEINER ID}
    //OU
    docker rm {NAME} -f

- **-f** --> Force

    docker run -d --name {container_name} nginx

- **--name** --> Posso dar um nome pro container

## Como entrar no container que já está rodando?

    docker exec {container_name} {COMMANDO}
Executa um comando dentro do container


    docker exec -it {container_name} {COMMANDO}
Executa o comando dentro do container e impede que ele seja finalizado automaticamente após rodar (segura o container)(-i), além de permitir que eu interaja com o terminal (-t).

Caso eu queira instalar pacotes dentro de um container (rodando ubuntu, por exemplo) antes é necessário dar um apt-get update. Uma vez que a imagem do container é a mais básica possível e não contém cache com o objetivo de aumentar a performance.

Tudo que for gravado dentro de um container será perdido quando o container for deletado, exceto quando utiliza um bind mount.

O **bind mount** adiciona um volume de dentro do computador para dentro do container, tornando possível manter os dados mesmo após a exclusão do container.

## Como adicionar um arquivo do meu computador no container?

    docker run -d --name nginx -p 8080:80 -v ~/pasta/no/computador:diretorio/no/container

- **-v** --> Volume
- **dir/PC:dir/Container** --> Mapeia a minha pasta para dentro do container, quando o container morre o arquivo continua no meu pc.


    docker run -d --name nginx -p 8080:80 --mount type=bind, source="$(pwd)"/,target=/dir/do/container

- **$(pwd)** --> atalho para pegar o diretório em que eu estou



- **-v x --mount** --> O -v consegue criar a pasta se necessário.


## Se eu subir a POC no docker compose e o gateway no docker run passando o -v com o diretório do projeto do gateway eu consigo alterar as coisas, salvar e elas são modificadas na aplicação????



## Como usar o docker sem root
https://docs.docker.com/engine/install/linux-postinstall/

https://docs.docker.com/engine/security/rootless/




- **-i** --> Mantém o container rodando, não deixa ele finalizar automaticamente.
- **-t** --> tty, é só um terminal. O Docker simula um terminal quando você roda -t.


## Volumes

    docker volume create meuvolume
    docker volume inspect meuvolume

Cria volumes.
Retorna todas as propriedades do volume.

    docker run --name nginx -d --mount **type=volume, source=meuvolume, target=/app" meucontainer

    docker run --name nginx2 -d -v meuvolume/app meucontainer2

Dois jeitos de conseguir startar um container e linká-lo ao meuvolume na pasta /app.

 ## Imagens

 As imagens do docker ficam guardadas no DockerHub.

 ### Como criar imagens?

 Criar um arquivo chamado Dockerfile sem extensão (.txt, .md).

 O Dockerfile permite que a gente construa uma imagem específica, um ubuntu com o VIM instalado, por exemplo.

 Dockerfile
    // Você precisa de uma imagem base, já que toda imagem é criada com base em outra.
    FROM nginx:latest

    // Executa os comandos
    RUN apt-get update
    RUN apt-get install vim -y

- **-y** --> confirma automaticamente

Para rodar a imagem

    docker build -t nomeNoDockerHub/nginx-com-vim:latest .
    
    docker run -it nomeNoDockerHub/nginx-com-vim:latest 
    vim oi