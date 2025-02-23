# Projeto de monitoramento de site via notificação no discord

### Tecnologias visadas na utilização:
 - AWS Services
 - Instâncias EC2
 - VPC
 - Security Groups
 - [User Data Script](https://github.com/Belmonte512/Projeto_Monitoramento/blob/main/Script_Inst%C3%A2ncia.txt)
 - NGinx como gerenciador de página WEB
 - Linguagem de Marcação de Hipertexto (HTML)
 - Linux Ubuntu 
 - Conexão SSH
 - Terminal Bash para conexão SSH
 - Protocolo HTTP para requisição do funcionamento do site
 - Script em Bash
 - Systemd
 - [Discord para recebimento de aviso com relação a mal funcionamento ou parada do site (WebHook)](https://support.discord.com/hc/pt-br/articles/228383668-Usando-Webhooks)
 - Git/GitHub para documentação e anotação do processo e seu andamento
 - Claude/BlackBox AI/ChatGPT/Gemini/Copilot/DeepSeek para validação de codigo em bash

### Criação de VPC 
Em sua criação foi prefirivel ser criado uma VPC com poucos IPs disponiveis, apenas para utilização neste projeto, um VPC com mascara /24, mas isso foi em relação a este caso em especifico

Criamos a VPC com 2 Subnets privadas e 2 Subnets públicas e 1 internet gateway, buscado a utilização de 2 zonas de acessibilidade (Availability zones) como padrão. Foi criado tudo de uma vez para poupar tempo em relação as configurações de route tables.

![image](https://github.com/user-attachments/assets/5f211e87-760a-44ad-8bbe-8c000d0734a3)

Finalizada a criação da VPC e suas configurações, a atenção agora é dada ao Security Group

### Criação do Security Group

O Security group deverá ser cridao tendo nas regras de inbound permissões para o acesso via SSH e HTTP apenas para o seu IP. Já na regra de saida (outbound) preferi manter any to any para maior acesso, mas deixo a observação de não seguir o exemplo e buscar ter maior controle das suas regras de saida. OBS: O security group foi criado no momento de criação da instância, mas nada impedia de ter sido criado antes e previamente configurado.

![Documentação11](https://github.com/user-attachments/assets/0020fe19-53aa-4942-8825-64f3e64ab951)

![Documentação12](https://github.com/user-attachments/assets/ae25b10f-6b75-4d79-8c34-ee3f14201983)

**Note que a própria AWS alerta sobre o uso de uma regrade saida Any to Any**

### Criação da Instância EC2 
A criação da Instância EC2 não teve muito segredo, foi criada um instância com uma AMI Ubuntu, por preferência propría, apenas recomendase a utilização de uma AMI com Kernel Linux, e para uso do Script do User Data que seja derivada do Debian

No laboratório do projeto foi criada a instância com o tipo t2.micro, nas configurações de rede utilizamos a VPC anteiormente criada e conectamos a instância a uma das subnets públicas para que possa ter acesso a internet e que possa ser acessada via SSH.

E para acesso via SSH eu configurei a chave de acesso sendo uma que eu mesmo criei anteriormente, mas pode ser criada uma na hora da criação da instâcia também, para isso clique em criar novo par de chaves, selecione o metodo de criptografia que for melhor para você e escolha o formato .pem, coloque o nome que for mais conveniente para você e clique em criar par de chaves, você terá o registro desse par de chaves e será feito o download do arquivo pem, que deverá ser colocado dentro da pasta .ssh do seu usuario, tendo ela sido criada anteiormente ou não, em caso de não ter sido criada, entre na sua pasta de usuario pessoal com o terminal bash do VScode e digite o comando <mkdir /.ssh> assim será gerada a pasta .ssh no seu arquivo de usuario pessoal

### Conexão via SSH

A conexão via SSH foi executada via terminal do VScode, pelo fato de ser mais fácil a utilização do terminal bash via VScode. A conexão é feita pelo metodo que a propría AWS fornece, sendo possível até mesmo a conexão via session manager, mas via SSH foi o preferivil por mim. 

Utilizando o comando <ssh -i "minha_chave.pem" ubuntu@ec2-"minha instância"> para conexão via SSH, deixo ressaltado que este comando deve ser executando estando acessada a pasta .ssh do seu computador e ali contendo a chave .pem que foi utilizada na criação da sua instância, trocando "minha_chave" pelo nome da sua chave anexada a instância e "minha instância" pelo id da sua instância, normalmente este comando vem pronto na parte de conexão via SSH na parte de exemplo.

![Documentação13](https://github.com/user-attachments/assets/ec275e77-0cac-41b7-9005-870ff7c833fb)


### Acessando a Instãncia

Ao acessar a máquina via SSH ele pede confirmação e apenas aceito digitando "yes" e assim já estou dentro da máquina e podendo subir meu usuario para root com o comando `sudo su`.

### Configurando a instância

Nesta parte será criada as configurações para que a página no Nginx esteja dispónivel, funcionando e tendo seu monitoramento e reiniciamento bem configurados.

### Passo a Passo:
 - Adicionar os repostorios oficiais e a chave de acesso a essses repostiorios para ter acesso as versões mais recentes do Nginx, ao invés de utilizar os repositórios padrões do Ubuntu:
```
sudo sh -c 'echo "deb http://nginx.org/packages/ubuntu/ $(lsb_release -cs) nginx" > /etc/apt/sources.list.d/nginx.list'
sudo sh -c 'echo "deb-src http://nginx.org/packages/ubuntu/ $(lsb_release -cs) nginx" >> /etc/apt/sources.list.d/nginx.list'
wget -qO - https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
```
 - Atualizar todo o sistema com < ` sudo apt update && sudo apt upgrade -y `> (Este comando visualiza os repositorios desatualizados e compara com os mais atuais disponiveis e os atualiza imediatamente)
 - Fazer download do Nginx < ` sudo apt install nginx -y ` > (Este comando baixa a versão mais recente do Nginx)
 - Criação do arquivo .html para visualização da página
```bash
sudo cat << 'EOF'> /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meu Site</title>
    <style>
        :root {
            --primary-color: #3498db;
            --secondary-color: #2ecc71;
            --text-color: #2c3e50;
            --background-color: #ecf0f1;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: var(--text-color);
            background-color: var(--background-color);
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }

        header {
            background-color: var(--primary-color);
            color: white;
            padding: 2rem 0;
            margin-bottom: 2rem;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        .header-content {
            text-align: center;
        }

        h1 {
            font-size: 2.5rem;
            margin-bottom: 1rem;
            animation: fadeIn 1s ease-in;
        }

        .info-card {
            background: white;
            border-radius: 10px;
            padding: 2rem;
            margin-bottom: 2rem;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            transition: transform 0.3s ease;
        }

        .info-card:hover {
            transform: translateY(-5px);
        }

        .status {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 1rem;
            margin-top: 1rem;
        }

        .status-indicator {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background-color: var(--secondary-color);
            animation: pulse 2s infinite;
        }

        .button {
            display: inline-block;
            padding: 0.8rem 1.5rem;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
            text-decoration: none;
            margin-top: 1rem;
        }

        .button:hover {
            background-color: #2980b9;
        }

        footer {
            text-align: center;
            padding: 2rem 0;
            margin-top: 2rem;
            border-top: 1px solid #ddd;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        @keyframes pulse {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.2); opacity: 0.7; }
            100% { transform: scale(1); opacity: 1; }
        }

        @media (max-width: 768px) {
            .container {
                padding: 10px;
            }

            h1 {
                font-size: 2rem;
            }
        }
    </style>
</head>
<body>
    <header>
        <div class="container header-content">
            <h1>Deu certo!</h1>
            <p>Seu site está funcionando perfeitamente</p>
        </div>
    </header>

    <main class="container">
        <div class="info-card">
            <h2>Informações do Host</h2>
            <div class="status">
                <div class="status-indicator"></div>
                <p id="hostname">Carregando...</p>
            </div>
            <button class="button" onclick="copyHostname()">Copiar Hostname</button>
        </div>

        <div class="info-card">
            <h2>Informações Adicionais</h2>
            <p id="browserInfo">Carregando informações do navegador...</p>
            <p id="datetime">Carregando data e hora...</p>
        </div>
    </main>

    <footer class="container">
        <p>© <span id="year"></span> - Todos os direitos reservados</p>
    </footer>

    <script>
        // Função para atualizar o hostname
        function updateHostname() {
            let host = location.host || 'Localhost';
            document.getElementById("hostname").innerHTML = host;
        }

        // Função para copiar o hostname
        function copyHostname() {
            const hostname = document.getElementById("hostname").textContent;
            navigator.clipboard.writeText(hostname)
                .then(() => alert('Hostname copiado com sucesso!'))
                .catch(err => console.error('Erro ao copiar:', err));
        }

        // Função para atualizar informações do navegador
        function updateBrowserInfo() {
            const browserInfo = `
                Navegador: ${navigator.userAgent.split(')')[0]})
                Idioma: ${navigator.language}
                Plataforma: ${navigator.platform}
            `;
            document.getElementById("browserInfo").textContent = browserInfo;
        }

        // Função para atualizar data e hora
        function updateDateTime() {
            const now = new Date();
            const options = { 
                weekday: 'long', 
                year: 'numeric', 
                month: 'long', 
                day: 'numeric',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit'
            };
            document.getElementById("datetime").textContent = now.toLocaleDateString('pt-BR', options);
        }

        // Atualizar o ano no footer
        document.getElementById("year").textContent = new Date().getFullYear();

        // Inicializar todas as informações
        updateHostname();
        updateBrowserInfo();
        updateDateTime();

        // Atualizar o datetime a cada segundo
        setInterval(updateDateTime, 1000);
    </script>
</body>
</html>
EOF

```
(Este comando cria um arquivo index.html dentro do diretório destinado a pagina do Nginx, sendo mostrado uma mensagem "Deu certo" e o IP público da instância)
 - Habilitando o serviço do Nginx < ` sudo systemctl enable nginx --now ` > (Este comando serve para que o serviço do Nginx seja habilitado e iniciado)
 - Criação do arquivo de reinicialização do Nginx em caso de desligamento 
~~~bash
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
(Este comando cria o script que será seguido para que o Nginx sejá reiniciado caso ocorra algum erro, é uma verificação via http para verificação do site do nginx, mas para não ter que fazer uma busca do proprio ip da máquina, preferi deixar o endereço como localhost, que funciona tão bem quanto)
 - Garantindo as permissões de execução do restart_nginx.sh < ` sudo chmod +x /usr/local/bin/restart_nginx.sh  `> (Este comando adiciona, caso já não tenha sido adicionada, as permissões de execução do script restart_nginx.sh)
 - Após fornecer as permissões, iremos colocar este script como um serviço do systemd, para que possa ser executado sempre que for iniciada a instância, para isso teremos quecriar um arquivo de serviço deste script, este arquivo segue a extensão .service:
 ~~~bash
sudo cat > /etc/systemd/system/restart_nginx.service << 'EOF'
[Unit]
Description=Monitoramento e reinicialização automática do Nginx
After=network.target

[Service]
ExecStart=/usr/local/bin/restart_nginx.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF
~~~
(Este bloco de codigo fará a criação do arquivo .service que fará a reinicialização do Nginx sempre que o serviço de paginas web cair, algumas informações importantes são referentes as configurações colocadas, como visto a cima no bloco [Service] o script será sempre reiniciado ao ligar a instância e será executado como root)
 - Nesta parte iremos ter que fazer uma reinicialização do daemon do sistema para que as aplicações de serviço sejam salvas ` sudo systemctl daemon-reload ` e logo em seguida devemos habilitar o serviço e inicializa-lo ` sudo systemctl enable restart_nginx ` e `sudo systemctl start restart_nginx `
 - Neste momento iremos criar o monitor do Nginx, que fára o monitoramento e o envio de notificações para o [webhook](https://support.discord.com/hc/pt-br/articles/228383668-Usando-Webhooks), no discord:
~~~bash
sudo cat > /usr/local/bin/monitor_nginx.sh << 'EOF'
#!/bin/bash

# Configurações
URL="http://localhost"
WEBHOOK_URL="https://discordapp.com/api/webhooks/"seu_webhook""
INTERVAL=60
LOG_FILE="/var/log/monitoramento.log"
PREVIOUS_STATUS="unknown"  # rastrear estado anterior

# Função para registrar logs com data e hora 
log() {
    local message="$1"
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    echo "[$timestamp] $message" | tee -a "$LOG_FILE"  
}

# Função para enviar notificação para o Discord
send_notification() {
    local message="$1"
    curl -H "Content-Type: application/json" -X POST -d "{\"content\": \"$message\"}" "$WEBHOOK_URL"
}

# Função para verificar o status do site 
check_site_status() {
    local response_code
    response_code=$(curl -s -o /dev/null -w "%{http_code}" "$URL")

    if [ "$response_code" -ne 200 ]; then
        local message="O site $URL está offline! Código de resposta: $response_code"
        log "$message"
        send_notification "$message"
        PREVIOUS_STATUS="down"  # Atualiza estado para "down"
    else
        log "O site $URL está online. Código de resposta: $response_code"
        
        # Se estava inativo anteriormente, notifica o retorno ao normal
        if [ "$PREVIOUS_STATUS" = "down" ]; then
            local recovery_message="\u2705 O site $URL voltou ao funcionamento normal! Código: $response_code"
            log "$recovery_message"
            send_notification "$recovery_message"
        fi
        
        PREVIOUS_STATUS="up"  # Atualiza estado para "up"
    fi
}

# Loop de monitoramento
while true; do
    check_site_status
    sleep "$INTERVAL"
done
EOF
~~~
(Este código ele cria o arquivo .sh do monitor do site disposto no Nginx fazendo a requisição via http atráves do url localhost. Colocamos o endereço do webhook na variavel correspondente, substituindo "seu webhook" pelo seu codigo do webhook. Criamos a váriavel intervalo para que o codigo que ficará rodando em loop não faça a verificação a todo momento consumindo assim demasiado poder computacional da instância EC2, colocamos também como variavel o caminho absoluto até a pasta /log que onde teremos salvo o monitoramento.log onde ficará salvo os dados de minuto em minuto do funcionamento do site.
A função log cria uma variavel que sevira como base para a mensagem a ser registrada no .log tendo a data com ano, mês, dia, hora, minuto e segundo.
A função send_notification é a que fará o envio da notificação ao servidor discord via webhook, a função em resumo faz uma requisição ao endereço do webhook e posta uma mensagem no mesmo.
A função de verificação do site (check_site_status) ela faz uma requisição ao site e verifica se o codigo http recebido é igual a 200 fazendo também a verificação do estado anterior do site, para que não haja a notificação constante mesmo do site estando online, tal verificação de estado antior é feita utilizando a variavel PREVIOUS_STATE que é atualizada em determinados pontos dos if else.
Por fim temos o loop do monitoramento onde ficara rodando a função check_site_status e após a sua execução ocorrerá o intervalo de 60 segundos.)
 - Finaizando a configuração do .sh temos que adicionar as permissões de execução, para garantir que ele seja executado da maneira que queremos `sudo chmod +x /usr/local/bin/monitor_nginx.sh`
 - Também, precisamos criar o arquivo .service para que o monitor tenha privelegios de execução como root e que seja reiniciado sempre que a instância for reiniciada:
~~~bash
sudo cat > /etc/systemd/system/monitor_nginx.service << 'EOF'
[Unit]
Description=Monitoramento do Nginx com notificações Discord
After=network.target

[Service]
ExecStart=/usr/local/bin/monitor_nginx.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF
~~~
(As configurações são iguais ao do reinicializador do Nginx, mudando apenas o nome do script no ExecStart)
 - Após finalizado a configuração do arquivo .service precisaremos apenas reiniciar novamente o daemon do sistema para que as configurações novas sejam aplicadas e salvas `sudo systemctl daemon-reload` e logo em seguida habilitar e iniciar o monitor `sudo systemctl enable monitor_nginx` e `sudo systemctl start monitor_nginx`

### Validação do script

Essa parte da validação é feita via conexão SSH com a instância EC2

Para validar todos os comando anteriormente inseridos, você pode ir acessando cada uma das pastas para ver se os arquivos foram corretamente inseridos e caso esteja tudo certo você poderá testar se estão sendo executados corretamente, primeiro teste o reinicializador do Nginx com o comando `systemctl stop nginx` e logo em seguida utilize o comando `service --status-all` verifique se o sinal ao lado do Nginx se encontra sendo [-] ou [+] em caso do sinal ser [-] aguarde alguns segundos e verifique novamente, em caso de não haver a mudança para o sinal [+] há algum problema no código do reinicializador, caso a mudança tenha ocorrido corretamente sigamos para a próxima verificação.

![Documentação2](https://github.com/user-attachments/assets/ddc992fe-8987-4056-acae-13ed91db8531)

---------

![Documentação3](https://github.com/user-attachments/assets/4d7f2491-5ab6-4f94-82a5-3436e7f95846)

---------

Agora que o reinicializador já foi devidamente testado e se encontra em funcionamento pleno vamos testar a notificação no servidor discord e o registro dos logs na pasta /var/log/ no arquivo monitoramento.log ou o arquivo que você tiver configurado. Verifique se a verificação ocorre de 1 em 1 minuto e se apresenta a mensagem que foi configurada para aparecer de maneira correta, após isso iremos finalizar os processos do reinicializador e do próprio Nginx usando os comandos `systemctl stop restart_nginx` e `systemctl stop nginx`, após feito isso aguardemos alguns segundos e verificamos nosso servidor discord para averiguarmos se a notificação foi devidamente enviada e se aparece como havia sido anteriormente configurada.



![Documentação4](https://github.com/user-attachments/assets/6e25337c-8fd9-4439-acb2-056b047267e6)

---------

![Documentação5](https://github.com/user-attachments/assets/d1d7cf3b-6142-4a5a-951a-2605b5d7af8d)

---------

![Documentação6](https://github.com/user-attachments/assets/8ef1af06-393b-46f9-89da-8b83a4f13cb6)

---------

para voltar ao monitoramento normal, podemos simplesmente reiniciar o reinicializador do Nginx com o comando `systemctl restart restart_gninx`

![Documentação7](https://github.com/user-attachments/assets/aaae2b4f-a206-4592-8289-614279658d9e)

---------

![Documentação8](https://github.com/user-attachments/assets/6d90e916-b759-4a0f-9241-b34ba545fd8d)

---------

Essa parte da verificação é feita fora da instância EC2

Uma validação simples que podemos fazer também é verificar se ao colocarmos o ip da instância teremos acesso a página em HTML que haviamos configurado e criado, em caso de não ocorrer essas apresentação o erro pode ter relação com a VPC criada, com o Security Group configurado , com a EC2 ou com o próprio Nginx. Para corrigir a parte da VPC analise a estrutura da VPC se está configurada corretamente, verifique se você liberou acesso para o seu IP corretamente no Security Group e a EC2 análise se ela está com acesso a internete e por ultimo verifique se em algum momento os comandos não foram executados corretamente ou sairam com algum erro que não foi visto ou apresentando nesta documentação.

![Documentação10](https://github.com/user-attachments/assets/afb4ced7-c294-4ee8-9b1e-6678435cf07e)

### User Data

Após a instação do servidor do Nginx e seus serviços de monitoramento e reinicializador, iremos criar o user data para automatizar a criação do servidor Nginx, com a página já criada e todos os scripts pré alocados e estruturados da maneira correta, parece comprexo mas será basicamente a criação de um script em Bash que terá a colocação de todos os comandos e scripts citados e apresentados anteriormente juntamente com a ordenação correta de cada um.

segue abaixo do User Data completo, comentado (dentro dos scripts em Português e fora dos scripts em Inglês) e rodando:

```
#!/bin/bash
# Script para automatização de criação de instância com NGINX e monitoramento do 
# funcionamento do site

# Configure repository for nginx (latest released)
sudo sh -c 'echo "deb http://nginx.org/packages/ubuntu/ $(lsb_release -cs) nginx" > /etc/apt/sources.list.d/nginx.list'
sudo sh -c 'echo "deb-src http://nginx.org/packages/ubuntu/ $(lsb_release -cs) nginx" >> /etc/apt/sources.list.d/nginx.list'
wget -qO - https://nginx.org/keys/nginx_signing.key | sudo apt-key add -

# Update system and install nginx
sudo apt update && sudo apt upgrade -y && sudo apt install nginx -y 

# Create sample HTML file
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

# Enable and start nginx service
sudo systemctl enable nginx --now

# Create nginx monitor script
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

# Grant execute permission to the monitor script
sudo chmod +x /usr/local/bin/restart_nginx.sh

# Create a systemd service file for automatic Nginx monitoring and restart
sudo cat > /etc/systemd/system/restart_nginx.service << 'EOF'
[Unit]
Description=Monitoramento e reinicialização automática do Nginx
After=network.target

[Service]
ExecStart=/usr/local/bin/restart_nginx.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd configuration
sudo systemctl daemon-reload

# Enable and start the restart_nginx service
sudo systemctl enable restart_nginx
sudo systemctl start restart_nginx

# Create the monitor for the WEB site
sudo cat > /usr/local/bin/monitor_nginx.sh << 'EOF'
#!/bin/bash

# Configurações
URL="http://localhost"
WEBHOOK_URL="https://discordapp.com/api/webhooks/"seu webhook" "
INTERVAL=60
LOG_FILE="/var/log/monitoramento.log"
PREVIOUS_STATUS="unknown"  # rastrear estado anterior

# Função para registrar logs com data e hora 
log() {
    local message="$1"
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    echo "[$timestamp] $message" | tee -a "$LOG_FILE"  
}

# Função para enviar notificação para o Discord
send_notification() {
    local message="$1"
    curl -H "Content-Type: application/json" -X POST -d "{\"content\": \"$message\"}" "$WEBHOOK_URL"
}

# Função para verificar o status do site
check_site_status() {
    local response_code
    response_code=$(curl -s -o /dev/null -w "%{http_code}" "$URL")

    if [ "$response_code" -ne 200 ]; then
        local message="O site $URL está offline! Código de resposta: $response_code"
        log "$message"
        send_notification "$message"
        PREVIOUS_STATUS="down"  # Atualiza estado para "down"
    else
        log "O site $URL está online. Código de resposta: $response_code"
        
        # Se estava inativo anteriormente, notifica o retorno ao normal
        if [ "$PREVIOUS_STATUS" = "down" ]; then
            local recovery_message="\u2705 O site $URL voltou ao funcionamento normal! Código: $response_code"
            log "$recovery_message"
            send_notification "$recovery_message"
        fi
        
        PREVIOUS_STATUS="up"  # Atualiza estado para "up"
    fi
}

# Loop de monitoramento
while true; do
    check_site_status
    sleep "$INTERVAL"
done
EOF

# Grant execute permission to the monitor script
sudo chmod +x /usr/local/bin/monitor_nginx.sh

# Create a systemd service file for monitoring
sudo cat > /etc/systemd/system/monitor_nginx.service << 'EOF'
[Unit]
Description=Monitoramento do Nginx com notificações Discord
After=network.target

[Service]
ExecStart=/usr/local/bin/monitor_nginx.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd configuration
sudo systemctl daemon-reload

# Enable and start the monitor_nginx service
sudo systemctl enable monitor_nginx
sudo systemctl start monitor_nginx
```

### Conclusão
Em conclusão, podemos verificar que a automação via User Data polpa demasiado tempo em relação a criação dos arquivos, instalação dos softwares e até mesmo na parte de ataualização, reinicialização, habilitação e inicilização dos serviços, pacotes e até mesmo o próprio sistema, no demais houverá algumas dificuldades enfrentadas com relação a conectividade e com o uso ao user data, erros de comando mal interpretado ou mal configurado mas em contrapartida o restante foi demasiadamente tranquilo e sem muitas complicações, na verdade tendo nenhuma complicação nas demais partes.










