# Projeto de monitoramento de site via notificação no discord

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
A criação da VPC não demonstrou tamanha dificuldade após a orientação fornecida pelo nosso instrutor pudemos seguir sem muitas dificuldades. Em sua criação foi prefirivel ser criado uma VPC com poucos IPs disponiveis, apenas para utilização neste projeto, um VPC com mascara /24, mas isso foi em relação a este caso em especifico

Criamos a VPC com 2 Subnets privadas e 2 Subnets públicas e 1 internet gateway, buscado a utilização de 2 zonas de acessibilidade (Availability zones) como padrão.Foi criado tudo de uma vez para poupar tempo em relação as configurações de route tables.

Finalizada a criação da VPC e suas configurações, a atenção agora é dada a instância EC2 

### Criação da Instância EC2 
A criação da Instância EC2 não teve muito segredo, foi criada um instância com uma AMI Ubuntu, por preferência propría, apenas recomendase a utilização de uma AMI com Kernel Linux, e para uso do Script do User Data que seja derivada do Debian

No laboratório do projeto foi criada a instância com o tipo t2.micro, nas configurações de rede utilizamos a VPC anteiormente criada e conectamos a instância a uma das subnets públicas para que possa ter acesso a internet e que possa ser acessada via SSH.

E para acesso via SSH eu configurei a chave de acesso sendo uma que eu mesmo criei anteriormente, mas pode ser criada uma na hora da criação da instâcia também, para isso clique em criar novo par de chaves, selecione o metodo de criptografia que for melhor para você e escolha o formato .pem, coloque o nome que for mais conveniente para você e clique em criar par de chaves, você terá o registro desse par de chaves e será feito o download do arquivo pem, que deverá ser colocado dentro da pasta .ssh do seu usuario, tendo ela sido criada anteiormente ou não, em caso de não ter sido criada, entre na sua pasta de usuario pessoal com o terminal bash do VScode e digite o comando <mkdir /.ssh> assim será gerada a pasta .ssh no seu arquivo de usuario pessoal

### Criação do Security Group

O Security group deverá ser cridao tendo nas regras de inbound permissões para o acesso via SSH e HTTP apenas para o seu IP. Já na regra de saida (outbound) preferi manter any to any para maior acesso, mas deixo a observação de não seguir o exemplo e buscar ter maior controle das suas regras de saida. OBS: O security group foi criado no momento de criação da instância, mas nada impedia de ter sido criado antes e previamente configurado.

### Conexão via SSH

A conexão via SSH foi executada via terminal do VScode, pelo fato de ser mais fácil a utilização do terminal bash via VScode. A conexão é feita pelo metodo que a propría AWS fornece, sendo possível até mesmo a conexão via session manager, mas via SSH foi o preferivil por mim. 

Utilizando o comando <ssh -i "minha_chave.pem" ubuntu@ec2-"minha instância"> para conexão via SSH, deixo ressaltado que este comando deve ser executando estando acessada a pasta .ssh do seu computador e ali contendo a chave .pem que foi utilizada na criação da sua instância, trocando "minha_chave" pelo nome da sua chave anexada a instância e "minha instância" pelo id da sua instância, normalmente este comando vem pronto na parte de conexão via SSH na parte de exempo.

### Acessando a Instãncia

Ao acessar a máquina via SSH ele pede confirmação e apenas aceito digitando "yes" e assim já esotu dentro da máquina.

### Configurando a instância

Nesta parte será criada as configurações para que a página no Nginx esteja dispónivel, funcionando e tendo seu monitoramento e reiniciamento bem configurados.

### 1º Passo:
 - Atualizar todo o sistema com < ` sudo apt update && sudo apt upgrade -y `> (Este comando visualiza os repositorios desatualizados com os mais atuais e os atualizada imediatamente)
 - Fazer download do Nginx < ` sudo apt install nginx -y ` > (Este comando baixa a versão mais recente do Nginx)
 - Criação do arquivo .html para visualização da página
```
sudo cat << 'EOF'> /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<body>

<h1>Deu certo!</h1>
<h2><p id="hostname"></p></h2>

<script>
let host = location.host;
document.getElementById("hostname").innerHTML = host;
</script>
</body>
</html>
EOF
```
(Este comando cria um arquivo index.html dentro do diretório destinado a pagina do Nginx, sendo mostrado uma mensagem "Deu certo" e o IP público da instância)
 - Habilitando o serviço do Nginx < ` sudo systemctl enable nginx --now ` > (Este comando serve para que o serviço do Nginx seja habilitado e iniciado)
 - Criação do arquivo de reinicialização do Nginx em caso de desligamento 
~~~
sudo cat > /usr/local/bin/restart_nginx.sh << 'EOF'
#!/bin/bash

# URL do site a ser monitorado
URL="http://localhost"  # Substitua pelo endereço do seu site

while true; do
    # Faz uma requisição HTTP ao site
    if curl --output /dev/null --silent --head --fail "$URL"; then
        echo "Site está online: $URL"
    else
        echo "Site está offline: $URL"
        # Reinicia o Nginx se o site estiver offline
        sudo systemctl restart nginx
    fi
    # Aguarda 5 segundos antes de verificar novamente
    sleep 5
done
EOF
~~~
(Este comando cria o script que será seguido para que o Nginx sejá reiniciado caso ocorra algum erro, é uma verificação via http para verificação do site em si)
 - Garantido as permissões de execução do restart_nginx.sh < ` sudo chmod +x /usr/local/bin/restart_nginx.sh  `> (Este comando adiciona, caso já não tenha sido adicionada, as permissões de execução do script restart_nginx.sh)
 


