# CCNA---Projecto-Final 02
## Projeto de Defesa CCNA — Relatório Técnico
## 📌 Introdução
Este projeto foi desenvolvido como parte da minha preparação para a certificação CCNA, com o objetivo de consolidar conhecimentos práticos em roteamento, switching, segurança, NAT e OSPF. A topologia proposta simula um ambiente corporativo com duas filiais conectadas por um link serial entre roteadores, switches de acesso, servidores e estações de trabalho.

Minha missão foi implementar uma infraestrutura de rede funcional, segura e resiliente, aplicando boas práticas de configuração e documentando cada etapa do processo.

### 🗺️ Fase 1: Planejamento e Posicionamento dos Dispositivos
Comecei organizando fisicamente os equipamentos no ambiente de simulação (Packet Tracer/Physical Mode). Posicionei os dois roteadores R1 e R2 no rack principal, seguidos pelos switches S1, S2, S3 e S4, deixando espaço entre eles para facilitar futuras intervenções.

Em seguida, distribui os endpoints:
Na Table(mesa) 1, coloquei PC-C (lado esquerdo) e o servidor DNS/Web (lado direito).

Na Table 2, coloquei PC-A (lado esquerdo) e PC-B (lado direito).

Com os dispositivos posicionados, procedi com as conexões físicas utilizando cabos adequados (coaxial direto para dispositivos do mesmo tipo, cross-over para dispositivos diferentes), seguindo rigorosamente o diagrama lógico fornecido.

### ⚙️ Fase 2: Configuração Básica dos Dispositivos
#### 2.1 Configuração dos PCs
Atribuí manualmente os endereços IPv4 conforme a tabela de endereçamento. Cada PC recebeu seu endereço, máscara e gateway padrão. Essa etapa garante que os hosts tenham conectividade básica com seus respectivos gateways.

#### 2.2 Configuração dos Roteadores R1 e R2
Acessei cada roteador via console e apliquei as configurações iniciais:
Desativei a resolução de nomes de domínio com no ip domain-lookup para evitar atrasos ao digitar comandos errados.
Defini o hostname para identificação clara.
Criei uma senha secreta enable 'ciscoenpass' criptografada e uma senha para acesso ao console ciscoconpass.
Defini o tamanho mínimo de senha para 10 caracteres.
Criptografei todas as senhas em texto claro com service password-encryption.
Configurei um banner MOTD de aviso para acesso não autorizado.

#### 2.3 Configuração das Interfaces dos Roteadores
Para cada interface, apliquei:
Descrição descritiva (ex: Link to S1)
Endereço IP conforme tabela
Comando no shutdown para ativação

#### 2.4 Configuração de SSH
Para acesso remoto seguro:
Defini o domínio 'ccna-lab.com'
Criei um usuário 'admin' com senha 'admin1pass'
Gerei uma chave RSA de 1024 bits
Habilitei SSH versão 2
Configurei as linhas VTY para aceitar apenas conexões SSH e autenticar via banco de dados local

#### 2.5 Configuração dos Switches
Em cada switch:
Defini o hostname conforme tabela
Configurei a SVI da VLAN 1 com o endereço IP de gerenciamento
Defini o gateway padrão para permitir comunicação remota

### 🌐 Fase 3: Configuração do OSPFv2 de Área Única
#### 3.1 Ativação do OSPF
Em ambos os roteadores, criei o processo OSPF com ID 1:
R1 recebeu o router-id 0.0.0.1
R2 recebeu o router-id 0.0.0.2
As redes foram anunciadas na área 0 na ordem solicitada:
R1: rede 64.100.1.0/29 (G0/0/2) e 198.51.100.0/30 (G0/0/0)
R2: rede 209.165.202.128/27 (G0/0/2) e 198.51.100.0/30 (G0/0/0)

#### 3.2 Ajustes OSPF
Para otimizar a operação:
Configurei as interfaces LAN (G0/0/1 em cada roteador) como passivas para não enviar atualizações desnecessárias
Ajustei a largura de banda de referência para 1 Gigabit (auto-cost reference-bandwidth 1000)
Configurei o link entre R1 e R2 como ponto a ponto para eleição de DR/BDR desnecessária
Altere o hello timer para 30 segundos

### 🔒 Fase 4: Configuração de Controle de Acesso e NAT
#### 4.1 Verificação Inicial de Conectividade
Antes das configurações de segurança, validei os problemas relatados:
PC-B não acessava o servidor web
PC-C não pingava PC-A

#### 4.2 Configuração de NAT
Static NAT no R1:
Mapeei o endereço privado de PC-B (192.168.1.5) para o endereço público 64.100.1.7, permitindo que PC-B acesse o servidor web.

PAT no R2:
Criei um pool IPNAT1 com os endereços públicos 209.165.202.140 a 209.165.202.150
Defini uma ACL (1) permitindo os endereços 172.16.2.1 a 172.16.2.15
Apliquei PAT com overload para compartilhamento dos IPs públicos

#### 4.3 Controle de Acesso VTY
Criei ACLs padrão para restringir acesso administrativo:
R1: apenas PC-B (192.168.1.5)
S1: apenas PC-B
R2: apenas PC-C (172.16.2.5)
S3: apenas PC-C

#### 4.4 Extended ACL no R2 (R2-SECURITY)
Implementei uma ACL estendida com quatro regras:
Permitir FTP do IP público de PC-B para o servidor web
Negar qualquer outro FTP vindo da internet
Negar qualquer conexão SSH da internet
Permitir todos os outros tipos de tráfego
Apliquei essa ACL na interface G0/0/2 no sentido inbound.

### 💾 Fase 5: Backup de Configuração e Atualização de IOS
#### 5.1 Backup via TFTP
Utilizei o PC-B como servidor TFTP para realizar backup das configurações em execução:
R1 → R1-Run-Config
S1 → S1-Run-Config
S2 → S2-Run-Config

#### 5.2 Atualização de IOS no S3
O servidor DNS/Web (endereço 209.165.202.131) disponibilizou a imagem c2960-lanbasek9-mz.150-2.SE4.bin. Realizei a cópia via TFTP para a flash do switch, configurei o boot para utilizar essa nova imagem e recarreguei o dispositivo para aplicar a atualização.

## 🧪 Verificações Finais
Após todas as configurações, realizei as seguintes validações:
Objetivo	         Comando	        Resultado Esperado
Vizinhos OSPF	show ip ospf neighbor	R1 e R2 adjacentes
Traduções NAT	show ip nat translations	Mapeamentos estático e dinâmico ativos
ACLs aplicadas	show access-lists	Contadores de matches incrementando
Acesso SSH	ssh -l admin <IP>	Conexão estabelecida
Conectividade	ping entre dispositivos	Resposta bem-sucedida

## 🎯 Motivação para Elaboração do Projeto
A decisão de desenvolver este projeto nasceu da necessidade de consolidar na prática os conceitos teóricos que estudei para a certificação CCNA.

Durante minha trajetória como estudante de redes e segurança, percebi que o conhecimento isolado de comandos não é suficiente para formar um profissional capacitado. É preciso entender como cada peça se conecta literal e logicamente para construir uma infraestrutura funcional, segura e resiliente.

Além disso, este projeto me desafiou a:
Pensar como um engenheiro de redes, planejando antes de executar
Aplicar segurança desde a camada de acesso até o roteamento
Automatizar processos de backup e atualização
Documentar cada etapa como exigido em ambientes corporativos reais.
Acredito que projetos práticos como este são o verdadeiro diferencial entre quem apenas estuda e quem está preparado para atuar no mercado. A segurança da informação e a resiliência de redes não são conceitos abstratos — são construídas linha por linha de configuração, com atenção aos detalhes e compromisso com as melhores práticas.

# Conclusão
Este projeto de defesa CCNA abordou os principais pilares de uma infraestrutura de rede corporativa:
## 1. Endereçamento IPv4 e roteamento OSPF
## 2. NAT e PAT para conectividade com a internet
## 3. Controle de acesso com ACLs padrão e estendidas
## 4. Gerenciamento seguro via SSH
## 5. Backup e atualização de IOS

Cada etapa foi executada com base em boas práticas e documentada de forma clara, simulando um ambiente real de trabalho. Ao final, todos os requisitos foram atendidos, garantindo uma rede funcional, segura e com alta disponibilidade.

Projeto desenvolvido por: [Fernando Lourenço]
Data: Março de 2026
Objetivo: Consolidação prática para certificação CCNA e fortalecimento de competências em redes e segurança.
