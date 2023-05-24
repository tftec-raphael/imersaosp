# Imersão TFTEC Azure ao vivo em SP

Esse projeto descreve os principais passos utilizados no nosso treinamento presencial de SP.
Temos como objetivo construir uma infraestrutura completa, simulando um cenário real de uma empresa que tem a necessidade de utilizar diversos recursos do Azure para rodar o seu negócio.

# Estrutura
O desenho de arquitetura informado abaixo mostra alguns detalhes de como está configurada a estrutura da empresa que iremos construir.

![TFTEC Cloud](https://github.com/raphasi/partiunuvem/blob/master/EstruturaApp_IaaS.png "Semana Partikunuvem")


## STEP01 - Criar a topologia de rede HUB-SPOKE
Iremos utilizar pelo menos, 4 regiões diferentes do Azure, com o objetivo de atendermos as restrições de vCpus existentes em contas trial do Azure.

1 - Criar a estrutura da VNET-HUB:

```cmd
   # VNET-HUB
   Nome: vnet-hub
   Região: East-US
   Adress Space: 10.10.0.0/16
   
   # Subnets
   Subnet: AzureBastionSubnet
   Address Space: 10.10.250.0.0/24
   
   Subnet: sub-srv
   Address Space: 10.10.1.0.0/24
   
   Subnet: sub-db
   Address Space: 10.10.2.0.0/24
   
   Subnet: GatewaySubnet
   Address Space: 10.10.251.0.0/24
   
```

2 - Criar a estrutura da VNET-SPOKE01:

```cmd
   # VNET-SPOKE01
   Nome: vnet-spoke01
   Região: UK South
   Adress Space: 10.20.0.0/16
   
   # Subnets
   Subnet: sub-intra
   Address Space: 10.10.1.0.0/24
     
```

3 - Criar a estrutura da VNET-SPOKE02:

```cmd
   # VNET-SPOKE02
   Nome: vnet-spoke02
   Região: Japan East
   Adress Space: 10.30.0.0/16
   
   # Subnets
   Subnet: sub-web
   Address Space: 10.30.1.0.0/24
   
   Subnet: sub-appgw
   Address Space: 10.30.250.0.0/24
     
```





2 - Após instalar seu servidor Windows Server 2022, execute o comando abaixo para realizar a instalação do IIS:


```cmd
   Install-WindowsFeature -name Web-Server -IncludeManagementTools

```
3 - Faça download da pasta do projeto utilizando o seguinte link:


   [https://github.com/raphasi/semanapartiunuvem/archive/refs/heads/master.zip](https://github.com/raphasi/partiunuvem/archive/refs/heads/master.zip)

4 - Copie a pasta "partiunuvem" para dentro do diretório c:\inetpub\wwwroot:
```cmd
  C:\inetpub\wwwroot\partiunuvem 
  
```
5 - Para os demais passos da configuração você deve acompanhar a Aula 02.


# Segunda Aula - Modernizando sua aplicação com Cloud

Modernizando a estrutura de aplicação e banco de dados utilizando serviços Cloud Azure PaaS.

![TFTEC Cloud](https://github.com/raphasi/partiunuvem/blob/master/EstruturaApp_PaaS.png "Semana Partikunuvem")

## Etapa de Modernização do Banco de dados
1 - Donwload e instalação do Net .Framework 4.8 na VM-DB:

https://dotnet.microsoft.com/en-us/download/dotnet-framework/thank-you/net48-offline-installer

2 - Download e instalaçao do DMA (Data Migration Assistant) na VM-DB:

https://www.microsoft.com/en-us/download/confirmation.aspx?id=53595

3 - Demais etapas detalhadas na Live:
- Criar um uma instância de Server SQL para hospedar o Azure SQL Database
- Criar uma instância de Azure SQL Database
- Utilizar o DMA para rodar um Assessment de compatilbilidade do banco antes da migração
- Parar a aplicação (iisreset /stop)
- Utilizar o DMA para rodar a migração do Schema do banco
- Utilizar o DMA para rodar a migração dos dados para o Azure SQL Database
- Conferir os dados migrados
- Desabilitar o serviço do SQL no servidor de oriem (IaaS)
- Alterar o endereço na string de conexão do servidor de aplicação
- Desligar a VM-DB

## Etapa de Modernização da Aplicação
1 - Donwload e instalação do App Service migration assistant na VM-APP:

https://azure.microsoft.com/en-au/products/app-service/migration-tools/

2 - Demais etapas detalhadas na Live:
- Criar um App Service Plan
- Fazer Upgrade do App Service Plan para uma instância S2 (Somente quem estiver com conta trial)
- Criar um Projeto no Azure Migrate
- Utilizar o App Service migration assistant para migração do site
- Ajustar a string de conexão no seu Web APP
- Desligar a VM-APP


## Para todas as instruções referentes a como montar este ambiente, acompanhe as aulas do evento.

Aula01 - https://www.youtube.com/watch?v=MWdrCNYmDGw

Aula02 - https://www.youtube.com/watch?v=o0tqdyFVtGI

Aula03 - https://www.youtube.com/watch?v=GXe13QbW8wk
