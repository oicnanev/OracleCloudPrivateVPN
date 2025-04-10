# Tutorial: Como Montar uma VPN na Oracle Cloud Free Tier com WireGuard e DynDNS

Neste tutorial completo, vou guiá-lo passo a passo para criar sua própria VPN segura e rápida usando a Oracle Cloud Infrastructure (OCI) no plano Always Free, configurar o WireGuard como tecnologia VPN e associar um serviço DynDNS (como No-IP) para acesso simplificado.

## Pré-requisitos

1. *Conta Oracle Cloud*: Você precisará se registrar no [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/) 
2. *Cartão de crédito válido*: Necessário para verificação (sem cobranças no plano Always Free) 
3. *Acesso SSH*: Conhecimento básico de linha de comando
4. *Conta No-IP*: Para configurar o serviço DynDNS 

## Passo 1: Criar uma Conta na Oracle Cloud

1. Acesse [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
2. Preencha seus dados pessoais e selecione sua região (escolha uma próxima a você para melhor latência) 
3. Forneça os detalhes do cartão de crédito para verificação (não será cobrado)
4. Após confirmação por e-mail, acesse o console da OCI

## Passo 2: Criar uma Rede Virtual (VCN)

1. No console OCI, navegue para "Networking" > "Virtual Cloud Networks"
2. Clique em "Start VCN Wizard" e selecione "Create VCN with Internet Connectivity" 
3. Configure com:
   - Nome: "VPN-VCN" (ou outro de sua preferência)
   - CIDR Block: 10.0.0.0/16
   - Public Subnet CIDR: 10.0.0.0/24
   - Private Subnet CIDR: 10.0.1.0/24
4. Clique em "Create" e depois "View Virtual Cloud Network"

## Passo 3: Criar uma Instância de Computação

1. Navegue para "Compute" > "Instances"
2. Clique em "Create Instance" e configure:
   - Nome: "VPN-Server"
   - Imagem: Ubuntu 22.04 (ou versão mais recente disponível) 
   - Shape: VM.Standard.A1.Flex (ARM, 4 vCPUs, 24GB RAM - parte do Always Free) 
3. Em "Networking":
   - Selecione a VCN criada anteriormente
   - Subnet: Public Subnet
   - Marque "Assign a public IPv4 address"
4. Em "Add SSH keys":
   - Selecione "Generate a key pair for me"
   - Baixe as chaves SSH privada e pública
5. Clique em "Create" e aguarde a instância ficar disponível (status "Running") 

## Passo 4: Conectar à Instância via SSH

1. No seu computador local, abra um terminal
2. Altere as permissões da chave privada:
   bash
   chmod 600 caminho/para/sua/chave_privada.key
   
3. Conecte-se à instância:
   bash
   ssh -i caminho/para/sua/chave_privada.key ubuntu@<IP_PUBLICO_DA_INSTANCIA>
   
   Substitua <IP_PUBLICO_DA_INSTANCIA> pelo IP mostrado no console da OCI 

## Passo 5: Atualizar o Sistema

Dentro da instância, execute:
bash
sudo apt update && sudo apt upgrade -y
sudo reboot

Aguarde alguns minutos e reconecte-se via SSH 

## Passo 6: Instalar o WireGuard usando PiVPN

1. Execute o instalador PiVPN:
   bash
   curl -L https://install.pivpn.io | bash
   
2. Siga as instruções na tela:
   - Selecione "WireGuard" como tecnologia VPN 
   - Use a porta padrão 51820 (ou outra de sua preferência)
   - Para DNS, selecione "Cloudflare" ou outro provedor
   - Selecione o IP estático da sua instância quando perguntado
   - Aceite as configurações padrão para o resto
3. Após instalação, reinicie o serviço:
   bash
   sudo systemctl restart wg-quick@wg0
   

## Passo 7: Criar Usuários VPN

1. Para cada dispositivo que deseja conectar, crie um perfil:
   bash
   pivpn add
   
   - Digite um nome para o cliente (ex: "casa-pc", "trabalho-notebook")
   - Pressione Enter para usar configurações padrão 
2. Os arquivos de configuração serão salvos em ~/configs/

## Passo 8: Baixar as Configurações para Seus Dispositivos

Do seu computador local (não da instância Oracle), baixe os arquivos de configuração:
bash
scp -i caminho/para/sua/chave_privada.key ubuntu@<IP_PUBLICO_DA_INSTANCIA>:~/configs/*.conf .

Isso copiará todos os arquivos de configuração para seu diretório atual 

## Passo 9: Configurar Regras de Firewall na Oracle

1. No console OCI, navegue para "Networking" > "Virtual Cloud Networks"
2. Selecione sua VCN e depois "Security Lists"
3. Edite a "Default Security List"
4. Adicione uma regra de entrada (Ingress):
   - CIDR de origem: 0.0.0.0/0
   - Protocolo IP: UDP
   - Porta de destino: 51820 (ou a porta que escolheu)
   - Descrição: "Permitir WireGuard VPN" 

## Passo 10: Configurar DynDNS com No-IP

1. Acesse [noip.com](https://www.noip.com/) e crie uma conta se ainda não tiver
2. Crie um hostname (ex: "seuservidor.ddns.net")
3. Na sua instância Oracle, instale o cliente No-IP:
   bash
   sudo apt install noip2
   
4. Execute o configurador:
   bash
   sudo noip2 -C
   
   - Digite suas credenciais No-IP
   - Selecione a interface de rede (geralmente eth0)
   - Atualize a cada 5 minutos (ou intervalo desejado)
5. Inicie o serviço e configure para iniciar automaticamente:
   bash
   sudo noip2
   sudo systemctl enable noip2
   

## Passo 11: Configurar Seus Dispositivos para Conectar à VPN

1. Instale o cliente WireGuard em seus dispositivos:
   - Windows/macOS/Linux: [https://www.wireguard.com/install/](https://www.wireguard.com/install/)
   - Android/iOS: Loja de aplicativos oficial 
2. Importe o arquivo de configuração (.conf) baixado anteriormente
3. Conecte-se usando o hostname DynDNS (seuservidor.ddns.net) em vez do IP

## Passo 12: Testar e Ajustar

1. Verifique a conexão VPN em cada dispositivo
2. Teste a velocidade e latência
3. Para melhorar a segurança, considere:
   - Alterar a porta do WireGuard para uma não padrão
   - Configurar firewall adicional na instância (UFW)
   - Implementar autenticação de dois fatores 

## Dicas Adicionais

1. *Monitoramento*: A Oracle oferece 10TB de tráfego de saída por mês no Always Free 
2. *Backup*: Regularmente exporte suas configurações WireGuard
3. *Atualizações*: Mantenha o sistema atualizado com:
   bash
   sudo apt update && sudo apt upgrade -y
   
4. *Desempenho*: O shape ARM (Ampere A1) oferece melhor desempenho que as instâncias x86 no free tier 

## Solução de Problemas Comuns

- *Conexão recusada*: Verifique as regras de segurança na OCI e se o WireGuard está rodando (sudo systemctl status wg-quick@wg0)
- *Velocidade lenta*: Experimente outras regiões da Oracle ou ajuste as configurações de MTU no WireGuard
- *IP dinâmico*: Verifique se o cliente No-IP está atualizando corretamente (sudo noip2 -S)

Com esta configuração, você terá uma VPN privada, segura e de alta velocidade conectando seus dispositivos domésticos e de trabalho, sem depender de serviços VPN comerciais e com custo zero usando o Oracle Cloud Free Tier.
