# Docker Portifólio
Projeto criado para praticar o uso do docker. Se trata de cluster para exemplificar como criar um ambiente altamente escalável com balanceamento de carga e gerenciamento de containers (principalmente disponibilidade) com o docker swarm.
## Conteúdo no ambiente de Produção:
- Quatro nós criados a partir da AWS EC2 com uma VPC, sub-rede (para conversarem entre si), um grupo de segurança e um NFS para formar o `swarm`, sendo:
  - O `leader` com disponibilidade `drain`, resposável apenas para o controle do swarm e como servidor dos volumes utilizando NFS.
  - Dois nós gerenciadores/trabalhadores que podem receber containers e servem de backup caso o `leader` fique indisponível.
  - Um nó apenas para trabalho, responsável pelos bancos de dados com volume fixado.
- Um `registry` para guardar imagens personalizadas.
- Um proxy reverso utilizando o nginx para balanceamento de carga.
- O [swarmpit](https://swarmpit.io/) para monitoramento do swarm.
- Algumas imagens como [apache](https://hub.docker.com/_/httpd), [postgres](https://hub.docker.com/_/postgres) e o [php-apache](https://hub.docker.com/r/webdevops/php-apache) para servir algumas aplicações que em um futuro próximo serão migradas.
- Route 53 para gerenciamento de domínios.
## Instalação:
- Adicionar labels em nós que receberão container cujo o volume é fixado
  ```bash
  $ docker node ls
  $ docker node update --label-add gerenciador=2 nome-ou-id-de-um-no-gerenciador-exceto-leader
  $ docker node update --label-add trabalhador=1 nome-ou-id-de-um-no-apenas-trabalhador```
- Instalando o `registry`
  ``` bash
  $ docker stack deploy -c registry/docker-compose.yml
  ```
- Criando o build do proxy
  ```bash
  $ cd proxy-portifolio && docker build -t localhost:5000/proxy-portifolio:1.0 .
  ```
- Instalando o swarmpit
  ```bash
  $ git clone https://github.com/swarmpit/swarmpit -b master 
  $ docker stack deploy -c swarmpit/docker-compose.yml swarmpit 
  ```
- Instalando os demais projetos (proxy, apache, php, banco de dados e etc)
  ```bash
  $ docker stack deploy -c portifolio/docker-compose.yml portifolio
  ```
- Alterando a disponibilidade do nó `leader`
  ```bash
  $ docker node update --availability drain nome-ou-id
  ```
- Instalar e configurar o NFS servidor no nó `leader`
  ```bash
  $ apt-get -y install nfs-server
  $ nano /etc/exports
  ```
  - Adicione estas duas linhas no final do arquivo e salve o arquivo:
    ```bash
    /var/lib/docker/volumes/apache/_data *(rw,sync,no_subtree_check)
    /var/lib/docker/volumes/abraco-quentinho/_data *(rw,sync,no_subtree_check)
    ```
  - Após adicionar as linhas deve-se exportar os diretórios
    ```bash
    $ exportfs -ar
    ```
- Instalar e configurar o NFS cliente nos demais nós
  ```bash
  $ apt-get -y install nfs-common
  $ mount ip-no-leader:/var/lib/docker/volumes/apache/_data /var/lib/docker/volumes/apache/_data
  $ mount ip-no-leader:/var/lib/docker/volumes/abraco-quentinho/_data /var/lib/docker volumes/abraco-quentinho/_data
  ```
 - Adicionar os arquivos nos volumes pelo nó `leader`
 ```bash
 $ git clone https://github.com/SamuelNunesDev/SamuelNunesDev.git && mv SamuelNunesDev/* /var/lib/docker/volumes/apache/_data
 $ git clone https://github.com/SamuelNunesDev/abraco-quentinho.git && mb abraco-quentinho/* /var/lib/docker/volumes/abraco-quentinho/_data
 ```

## Créditos
- [Samuel Nunes](https://github.com/SamuelNunesDev)
## Licença
 - MIT License (MIT). Por favor olhe [LICENSE](LICENSE.md) para mais informações.
