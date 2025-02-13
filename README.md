#Projeto de monitoramento de site via notificação no discord

### Tecnologias visadas na utilização:
 - AWS Services
 - Instâncias EC2
 - VPC
 - Security Groups
 - NGinx como gerenciador de página WEB
 - Linguagem de Marcação de Hipertexto (HTML)
 - Linux
 - Conexão SSH
 - Protocolo HTTP
 - Script em Bash ou Python
 - CRON
 - Telegram,Discord ou Slack para recebimento de aviso com
relação a mal funcionamento ou parada do site
 - Git/GitHub para documentação e anotação do processo e seu andamento

### Criação de VPC 
A criação da VPC não demonstrou tamanha dificuldade após a orientação fornecida pelo nosso instrutor pudemos seguir sem muitas dificuldades

Criamos a VPC com 2 Subnets privadas e 2 Subnets públicas e 1 internet gateway, criamos tudo de uma vez para poupar tempo em relação as configurações de route tables

Finalizada a criação da VPC e suas configurações, a atenção agora é dada a instância EC2 

### Criação da Instância EC2 
A criaçãoda Instância EC2 não teve muito segredo, foi criada um instância com uma AMI com Kernel Linux (Ubuntu,Debian,Amazon Linux)

No laboratório do projeto foi criada a instância com o tipo t2.micro, nas configurações de rede utilizamos a VPC anteiormente criada e conectamos a instância a uma das subnets públicas.

### Criação do Security Group
O Security group deverá ser cridao tendo nas regras de inbound permissões para o acesso via SSH e HTTP apenas para o seu IP. Já na regra de saida (outbound) preferi manter any to any para maior acesso, mas deixo a observação de não seguir o exemplo e buscar ter maior controle das suas regras de saida. OBS: O security group foi criado no momento de criação da instância, mas nada impedia de ter sido criado antes e previamente configurado.

### Conexão via SSH


