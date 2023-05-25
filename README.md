# ** Objetivo:**

O objetivo deste projeto é criar um ambiente na [AWS (Amazon Web Services)](https://aws.amazon.com/pt/ "AWS") usando a instância EC2 com o sistema operacional Amazon Linux 2 e configurar um servidor Apache. Além disso, será implementado um script para verificar o status do serviço Apache e registrar os resultados em um diretório NFS.

# **Primeiros Passos:**
1. Criar uma instancia  EC2 na AWS com as seguintes configurações:
> - Sistema Operacional (SO): Amazon Linux 2
>- Família: t3.small
>- Armazenamento: 16GB SSD 

2.  Liberar as seguintes portas para comunicação externa:
> 22 - TCP 
> 111 - TCP e UDP
> 2049 - TCP e UDP
> 80 - TCP
> 443 - TCP.

1. 
