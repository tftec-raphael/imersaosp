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
   # Criar um Resource Group
   Nome: rg-azure
   Região: East US
   
   # VNET-HUB
   Nome: vnet-hub
   Região: East-US
   Adress Space: 10.10.0.0/16
      
   # Subnets   
   Subnet: sub-srv
   Address Space: 10.10.1.0.0/24
   
   Subnet: sub-db
   Address Space: 10.10.2.0.0/24
   
   Subnet: sub-pvtendp
   Address Space: 10.10.3.0.0/24
   
   Subnet: AzureBastionSubnet
   Address Space: 10.10.250.0.0/24
   
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

## STEP02 - Deploy dos NSGs
Criar 3 NSGs de acordo com o modelo abaixo:
```cmd
   Nome: nsg-hub
   Região: east-us
   Associar Subnet: sub-srv e sub-db
```

```cmd
   Nome: nsg-intra
   Região: uk-south
   Associar Subnet: sub-intra
```

```cmd
   Nome: nsg-web
   Região: japan-east
   Associar Subnet: sub-web
```

## STEP03 - Deploy das VMs
Para todas as VMs iremos o tamanho B2S e utilizar o sistema operacional Windows Server 2022.

```cmd
   # EAST US
   Nome: vm-adds
   Região: east-us
   Vnet: vnet-hub
   Subnet: sub-srv
```

```cmd
   # UK SOUTH
   Nome: vm-intra01
   Região: uk-south
   Zone: Zone 1
   Vnet: vnet-spoke01
   Subnet: sub-intra
   
   Nome: vm-intra02
   Região: uk-south
   Zone: Zone 2
   Vnet: vnet-spoke01
   Subnet: sub-intra
```

```cmd 
   # JAPAN EAST
   Nome: vm-web01
   Região: japan-east
   Zone: Zone 1
   Vnet: vnet-spoke02
   Subnet: sub-web
   
   Nome: vm-web02
   Região: japan-east
   Zone: Zone 1
   Vnet: vnet-spoke02
   Subnet: sub-web
 ```  
 
   
2 - Deploy Azure Bastion

```cmd
   Nome: bastion01
   Região: East US
   SKU: Standard
```

3 - Desabilitar o firewall de todas as VMs via Run Command, usando o seguinte comando:

```cmd
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

4 - Acessar as VMs via Bastion e realizar teste de ping entre VMs de vnets diferentes.

5 - Criar peering entre as seguintes vnets:
```cmd
vnet-hub/vnet-spoke01 - vnet-spoke01/vnet-hub
vnet-hub/vnet-spoke02 - vnet-spoke02/vnet-hub
```
4 - Realizar teste de ping entre VMs de vnets diferentes novamente.

## STEP04 - Configuração de domínios e certificados
1 - Criar um DNS Zone (Zona de DNS pública no Azure).

2 - Apontar registros NS (name server) do provedor público para o Azure.
Testar a validação do DNS com o seguinte comando:
```cmd
nslookup -type=SOA tftec.cloud
```
3 - Gerar um certificado digital válido:
Opções:
OPÇÃO01: https://app.zerossl.com
OPÇÃO02: https://punchsalad.com/ssl-certificate-generator/

4 - Converter o certificado para PFX:

https://www.sslshopper.com/ssl-converter.html


## STEP05 - Deploy Azure Key Vault
1- Deploy Azure Key Vault:
```cmd
   Nome: kvault-certs
   Região: east-us
   Usar pvt enpdoint na vnet-hub e subnet sub-pvtendp 
```

2- Fazer upload do certificado PFX no Key Vault

3- Criar um managed identify e conceder permissão no Key Vault como:
```cmd
   Secret: Get
```

## STEP06 - Deploy estrutura INTRANET
1- Instalar IIS nas VMS vm-intra01 e vm-intra02:
```cmd
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```
2- Instalar .net core:

https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/runtime-aspnetcore-7.0.5-windows-hosting-bundle-installer

3- Baixar código da aplicação aplicação do seguinte link do Github:

https://github.com/tftec-raphael


## STEP07 - Deploy Azure Load Balancer
1- Deploy Load Balancer 
```cmd
   Nome: lb-intra
   Tipo: Interno
   Região: uk-south
   Sku: Standard 
   Backend: bepool-intra
```

## STEP08 - Deploy Nat Gateway
1- Deploy Nat Gateway
```cmd
   Nome: nat-gw01
   Região: uk-south
   Tipo: Public IP Address
```
2- Associar a subnet sub-intra


## STEP08 - Application Gateway
1- Deplou Apg Gateway
```cmd
   Nome: appgw-site
   Região: japan-east
   Tier: Standard 2
   Tipo: Public IP Address
   Backend: bepool-web
   
   Rule01: rule-https-web
   Listener: lst-443
   Protocol: HTTPS - Escolher o certificado do Key Vault
   Backend Setting: bset80
   
   
   Rule01: rule-http-web
   Listener: lst-80
   Protocol: HTTPS
   Backend Setting: bset80
   Target type: Redirection
```

2- Criar um ASG
```cmd
   Nome: asg-web
   Região: japan-east   
```
3- Associar a placa de rede das seguintes VMs ao ASG:
```cmd
  vm-web01
  vm-web02
```
4- Liberar regra no NSG nsg-web
```cmd
Criar regra, liberando qualquer origem, setar o destino com o Application Security Group e usar portas 80 e 443.
```
5- Ajustar registro DNS externo
```cmd
Acessar a zona de DNS público e criar um registro do A.
Usar Alias Record Set e apontar para o IP público do App Gateway.
```

## STEP08 - Deploy Storage Account
1- Deploy Storage Account
```cmd
   Nome: tfteclabsp01 (usar seu nome exclusivo)
   Região: east-us
   Performance: Standard
   Redundancy: GRS - Marcar opção para  Read Access
   Usar pvt enpdoint na vnet-hub e subnet sub-pvtendp
   Storage Sub-Resource: Blob
```
2- Criar um Container blob
```cmd
   Nome: images
```
3- Criar um segundo private endpoint
```cmd
   Usar pvt enpdoint na vnet-hub e subnet sub-pvtendp
   Storage Sub-Resource: File
```
4- Mapear Azure Files
```cmd
   Mapear o Azure Files nas seguintes VMs:
   vm-intra01
   vm-intra02
```
Validar se a conexão está apontando para ip interno

## STEP09 - Deploy Aplicação Imagens (WebApp)
1- Criar um App Service Plan
```cmd
   Nome: appplanimages
   Operating System: Windows
   Região: east-us
   Pricing plan: Standard S1
   vm-intra02
```
2- Criar um WebApp
```cmd
   Nome: tftecimages01 (usar seu nome exclusivo)
   Publish: Code
   Runtime Stack: .NET6
   Região: east-us
   Pricing plan: Standard S1
   Escolher AppServicePlan já criado
```




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
