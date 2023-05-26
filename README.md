#  Objetivo:

O objetivo deste projeto é criar um ambiente na [AWS (Amazon Web Services)](https://aws.amazon.com/pt/ "AWS") usando a instância EC2 com o sistema operacional Amazon Linux 2 e configurar um servidor Apache. Além disso, será implementado um script para verificar o status do serviço Apache e registrar os resultados em um diretório NFS.

# Requisitos AWS
- 	Gerar uma chave pública para acesso ao ambiente;
- 	Criar 1 instância EC2 com o sistema operacional Amazon Linux 2 (Família t3.small, 16 GB SSD);
- 	Gerar 1 elastic IP e anexar à instância EC2;
- 	Liberar as portas de comunicação para acesso público: (22/TCP, 111/TCP e UDP, 2049/TCP/UDP, 80/TCP, 443/TCP).

# Requisitos Linux
- Configurar o NFS entregue.
- 	Criar um diretorio dentro do filesystem do NFS com seu nome.
- 	Subir um apache no servidor - o apache deve estar online e rodando.
- 	Criar um script que valide se o serviço esta online e envie o resultado da validação para o seu diretorio no nfs.
- 	O script deve conter - Data HORA + nome do serviço + Status + mensagem personalizada de ONLINE ou offline.
- 	O script deve gerar 2 arquivos de saida: 1 para o serviço online e 1 para o serviço OFFLINE.
- 	Preparar a execução automatizada do script a cada 5 minutos.
- 	Fazer o versionamento da atividade.
- 	Fazer a documentação explicando o processo de instalação do Linux.


# Primeiros Passos na AWS
#### Criação do Security Group:
1. Clique em "Security Groups".
2. Clique em "Create Security Group".
3. Forneça um nome e uma descrição para o SG.
4. Selecione a VPC correta.
5. Adicione as regras de entrada para as portas desejadas.
6. Clique em "Create" para criar o SG.
7. Permita o acesso público às seguintes portas de comunicação: 22/TCP, 111/TCP e UDP, 2049/TCP/UDP, 80/TCP, 443/TCP. Essas portas serão utilizadas para comunicação externa da instância.


##### A abertura de portas ficará conforme a tabela abaixo:
| Porta  | Protocolo | Descrição                            | Origem      |
|--------|-----------|--------------------------------------|-------------|
| 22     | TCP       | Acesso remoto (SSH)                  | 0.0.0.0/0    |
| 111    | TCP e UDP | RPC                                  | 0.0.0.0/0     |
| 2049   | TCP e UDP | Network File System (NFS)            | 0.0.0.0/0     |
| 80     | TCP       | Acesso HTTP                          | 0.0.0.0/0     |
| 443    | TCP       | Acesso HTTPS                         | 0.0.0.0/0    |


#### Criação da Instância EC2:

1. 	Clique em "Painel EC2".
2. 	Clique em "Instâncias".
3. 	Clique em "Executar Instância".
4. 	Escolha um nome para sua instância.
5. 	Selecione a imagem "Amazon Linux 2".
6. 	Selecione o tipo de instância "t3.small".
7. 	Crie um par de chaves, que posteriormente será usada para conectar-se a nossa instância.
8. 	Selecione o Security Group que foi criado anteriormente, com as portas devidamente abertas para comunicação externa.
9. 	Selecione 16GB de armazenamento SSD (gp3)
10. Clique em "Executar instância".

#### Geração do Elastic IP:

1. 	Clique na opção "IPs elásticos"
2. 	Depois em "Alocar endereço IP elástico"
3. 	Selecione a mesma região que a sua instância EC-2 está rodando.
4. 	Clique em "Alocar".
5. 	Depois de criado o IP, selecione a caixinha a esquerda dele.
6. 	Clique em "Ações" e selecione a opção "Associar endereço IP elástico"
7. 	Selecione a instância criada anteriormente e confirme a associação clicando em "Associar"

#### Configurando nosso EFS
1. Digite *EFS* no campo de busca.
2. Clique na opção para criar um sistema de arquivos.
3. Escolha um nome para o seu sistema de arquivos e selecione uma VPC.
4. Clique sobre o EFS recem criado e selecione a opção "ver detalhes"
5. Depois selecione a opção "rede" e logo em seguido "editar"
6. Logo após, selecione o SG da instância que estará sendo usada para o trabalho.
1. Clique em "Anexar" e copie o comando que está abaixo da opção "Usando o cliente NFS"

# Configuração no Ambiente Linux

#### Configuração do Apache
1. Primeiro vamos fazer a instalação dos pacotes, executando o comando `sudo yum install httpd`.
2. Agora vamos inicar o servidor apache, utilizando o comando `sudo systemctl start httpd`.
3. Nessa etapa, iremos automatizar a inicilização do apache, que será feito no momento que a nossa instância for iniciada, para isso utilize o comando `sudo systemctl enable httpd`.
4. Agora para verificar a situação do servidor, utilize o comando `sudo systemctl status httpd`.

### Configuração do EFS na instância EC-2

1. Primeiramente, devemos fazer a instalação dos pacote *amazon-efs-utils* , que irá fazer a manipulação do nosso EFS. Para isso, execute o comando `sudo yum install amazon-efs-utils` e espere a instalação ser realizada.
2. Após, realizar a execução do comando `sudo mkdir mnt/EFS`.
3. Logo após, iremos rodar o comando que copiamos, do EFS. Deverá ser um comando assim `sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport id_do_seu_efs:/ mnt/EFS `, esse comando tem como finalidade montar o EFS no diretório que criamos anteriormente.
1. Para mantermos essas configurações caso a nossa instância venha a sofrer um reboot, devemos editar o arquivo presente *fstab*, que fica localizada no caminho */etc/fstab*. Para fazermos essa edição devemos utilizar o comando `sudo nano /etc/fstab` e acrescentar a seguinte linha de código `id_do_seu-EFS	/mnt/EFS  efs  defaults,_netdev 0     0`
1. Usar o comando `mkdir /mnt/EFS/<seu-nome>` para criarmos um diretório conforme a atividade exige.

#### Criação do Script

1. Criar um arquivo utilizando o comando `nano script.sh`
2. Adicionar o seguinte código ao nosso arquivo .sh 

        #!/bin/bash
        SERVICE=httpd
        STATUS=$(systemctl is-active $SERVICE)
        DATE=$(date +"%d/%m/%Y %H:%M:%S")
        if [ $STATUS == "active" ]; then
          MESSAGE="O apache $SERVICE está ONLINE"
          echo "$DATE - $MESSAGE" >> /mnt/EFS/<seu-nome>/online.log
        else
          MESSAGE="O apache $SERVICE está OFFLINE"
          echo "$DATE - $MESSAGE" >> /mnt/EFS/<seu-nome>/offline.log
        fi
1. Alterar as permissões de do script usando o comando `chmod +x script.sh`, esse comando concede permissões de execução do script na nossa instância.

#### Automatizando o Script

1. Executar o comando `sudo crontab -e`, com ele iremos dar instruções ao nosso script.sh para ser executado de 5 em 5 minutos, item proposto pela atividade.
2. Adicione a seguinte linha `*/5 * * * * sudo bash /script.sh`, pressione *CTRL +C* e digite *:q* para sair do editor e salvar as configurações.
3. Após executar `sudo crontab -l` para checarmos se o arquivo foi salvo corretamente.




