# Albatross - Software

```Text
Data: 27/04/22
Programa: V2.0 (Albatross_v2.0)
Python: V3.9.2
```

Esse documento descreve todas as etapas necessárias para preparar o software Albatross.

## Funcionamento do código

O programa funciona com módulos de controle. Dentre esses módulos, existe um responsável por gerenciar todos os outros. Esse módulo é o "Controle_Geral.py", que é executado durante a inicialização.  

**Controle_Geral:** Este é o arquivo responsável por gerenciar os módulos e fazer as verificações.  

**Conexao_DB:** Módulo responsável pelo acesso ao banco de dados.  

**Encoder:** Esse módulo é responsável pela leitura do encoder e travamento da catraca.
  
**Oled_Display:** Módulo responsável pelo controle do Display OLED.

**Registro_atividade:** Módulo responsável pela criação dos logs. No programa atual temos dois tipos logs:  

- **LOG DE ACESSO:**
O log de acesso é criado toda vez que um usuário aproxima o cartão no sensor RFID da catraca. Dentro do arquivo log, fica armazenado o Id, data, hora e se o usuário foi permitido (nome do arquivo é “log.txt”).
- **LOG DO SISTEMA:**
O log de sistema é criado toda vez que o programa apresenta algum erro (nome do arquivo é “logsistem.txt”).

  > **_Nota:_** É necessário declarar o caminho completo do arquivo que irá armazenar o log. Essa declaração é feita no módulo Registro_atividade, linha: arquivo = open(r'Caminho do arquivo', "r+") .

**Leitura_RFID:** Responsável pela comunicação com a porta serial. Sendo assim, é ele que retorna a leitura do RFID.

**Ativacao_led:** Esse módulo faz o controle de todos os LEDs e também verifica o tempo da passagem.

**CRONOS:** Essa biblioteca verifica e atualiza a data ea hora dos dispositivos(RPi e RTC). Além disso, ela também verifica se o programa pode iniciar.
> **_Nota:_** Dentro da pasta code, tem um arquivo chamado teste_bibliotecas. Esse arquivo é só para debug.

> <font size="3"> <span style="color:red">**_IMPORTANTE:_**</span> Esses módulos devem estar todos na mesma pasta.</font>

## Itens mínimos

- Monitor
- Teclado e mouse
- Fonte de alimentação
- Raspberry Pi
- Cartão Micro SD de 8 GB
- Adaptador de cartão Micro SD
- Conexão com a internet
- Pendrive

## Preparando a Raspberry Pi

### Instalando o sistema Raspberry Pi OS

Prepare um cartão Micro SD com a imagem "Raspberry Pi OS Lite".

Isso pode ser feito usando o Software [rpi-imager](https://www.raspberrypi.com/software/).

### Configurando o Raspberry Pi OS

#### Definir usuário e senha

```TEXT
Usuário: pi
Senha: robocore321
```

#### Mudando o nome do usuário

```SH
# Ativando o root e mudando a senha.
sudo passwd root 
# Mude a senha para robocore321.
````

Feito isso, vá em <span style="color:blue"> Configurações > Preferências > Raspberry pi Configuration</span> e desative o `auto login` e reinicie o sistema operacional.

```SH
# Reiniciando o OS
sudo reboot
```

Após isso, entre como usuário root e mude o usuário `pi` para `robocore`.

```SH
# Mudando o nome do usuário para robocore
usermod -l robocore pi -md /home/robocore
```

Feito isso, reinicie o sistema operacional e entre com o usuário robocore.

```sh
# Desativando o usuário root
sudo passwd –l root
```

Removendo a senha do terminal.

```SH
# Abrindo o arquivo com o editor nano
sudo nano /etc/sudoers.d/010_pi-nopasswd

# Deixe o arquivo assim: 
robocore ALL=(ALL) NOPASSWD: ALL
```

>**_Nota:_** Antes de seguir, reative o `auto login`.

#### Ativar a Porta Serial

```TEXT
sudo raspi-config
## Interface Setting
## Serial port
## Enable
```

>**_Nota:_** O login shell não deve ser ativado.

#### Ativar o VNC

```TEXT
sudo raspi-config
## Interface Setting
## VNC
## Enable
```

#### Ativar SSH

```TEXT
sudo raspi-config
## Interface Setting
## SSH
## Enable
```

#### Ativar I2C

```TEXT
sudo raspi-config
## Interface Setting
## I2C
## Enable
```

### Atualizando os pacotes

Essa etapa serve para garantir que os pacotes estão atualizado.

```SH
# Atualizando os pacotes.
sudo apt update && sudo apt upgrade -y
```

### Instalando e configurando o Banco de dados

Neste projeto foi usado o mariadb. Sendo assim, devemos instalá-lo na máquina.

```SH
# Instalando o mariadb
sudo apt install mariadb-server php-mysql -y
 
# Configurando
sudo mysql_secure_installation
```

> **_Nota:_** Primeiro coloque a senha do usuário root (a mesma do sistema). Feito isso, mude a para: `rcat321;`

### Instalando a ferramenta para gerenciar o banco de dados

Essa etapa não é necessária para o funcionamento do programa, porém o phpmyadmin é uma ótima ferramenta para gerenciar o banco de dados.

```sh
# Instalando o servidor apache
sudo apt install apache2 -y
 
# Instalando o php
cd /var/www/html
sudo apt install php -y

# Instalando o phpmyadmin
cd ~
sudo apt install phpmyadmin -y

# Configurando
sudo phpenmod mysqli
sudo service apache2 restart
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
sudo mysql --user=root --password
create user albatross@localhost identified by 'rcat321';
grant all privileges on *.* to albatross@localhost;
FLUSH PRIVILEGES;
exit;
```

### Copiando o programa

Prepare um pendrive com uma cópia da pasta "Albatross".

A pasta pode ser encontrado no [drive de projetos](https://Aqui_vai_link_do_drive) da robocore.

```SH
# Copiando o pasta para a área de trabalho da Raspberry Pi
cp -r /media/robocore/nome do pendrive/Albatross /home/robocore/Desktop
```

### Criando o ambiente virtual

Um ambiente virtual proporciona um melhor controle das bibliotecas instaladas.
Para instalá-lo:

```SH
# Instalando o ambiente virtual
sudo apt install git openocd virtualenv python3-pip

# Criando o ambiente virtual
cd /home/robocore/Desktop/Albatross/
virtualenv venv -p python3

# Ativando o ambiente
source venv/bin/activate
```

### Instalando as bibliotecas
Para essa etapa o ambiente virtual deve estar ativo. Exemplo de como deve estar:  
<span style="color:green">(venv) robocore@raspberrypi: ~ $</span>
```SH
# Instalando todas as biblioteca necessárias
pip3 install -r /home/robocore/Desktop/Albatross/bibliotecas
```

### Testando o código

O código deve ser testado antes de prosseguir com o passo a passo.

```sh
# Executando o código
python3 /home/robocore/Desktop/Albatross/Controle_Geral.py
```

> **_Nota:_** Esse comando só irá funcionar se o ambiente estiver ativo. Exemplo de como deve estar:  
<span style="color:green">(venv) robocore@raspberrypi: ~ $</span>

### Preparando o programa para iniciar automaticamente

Com tudo configurado vamos criar um serviço para executar o programa.

```SH
# Criando o serviço de update
sudo cp /home/robocore/Desktop/Albatross/servicos/servico_update.service /lib/systemd/system

# Dando a permissão 644(RW/R/R) para o arquivo "servico_update.service"
sudo chmod 644 /lib/systemd/system/servico_update.service

# Criando o serviço Albatross
sudo cp /home/robocore/Desktop/Albatross/servicos/albatross_service.service /lib/systemd/system

# Dando a permissão 644(RW/R/R) para o arquivo "albatross_service.service"
sudo chmod 644 /lib/systemd/system/albatross_service.service

# Permitindo que o arquivo "Controle_Geral.py" seja executado
chmod +x /home/robocore/Desktop/Albatross/code/Controle_Geral.py

# Permitindo que o arquivo "update_actions.py" seja executado
chmod +x /home/robocore/Desktop/Albatross/code/update_actions/update_actions.py

# Recarregando o daemon
sudo systemctl daemon-reload

# Habilitando o serviço
sudo systemctl enable albatross_service.service

# Iniciando o programa
sudo systemctl start albatross_service.service
```

> **_Nota:_** O serviço update será iniciado dentro do programa principal.

### Comandos para controlar os serviços

```sh
# Status do serviço
sudo systemctl status nome_servico.service
# Verificar o estado de início do serviço
journalctl -u nome_servico.service -f
# Iniciando serviço
sudo systemctl start nome_servico.service
# Parando o serviço
sudo systemctl stop nome_servico.service
# Desativando o serviço
sudo systemctl disabled nome_servico.service
# Habilitando o serviço
sudo systemctl enable nome_servico.service
# Reiniciando o serviço
sudo systemctl restart nome_servico.service
```

> **_Nota:_** Não deve ser necessário usar esses comandos normalmente, mas é importante conhecê-los.

### Programa funcionando

Com todas as etapas anteriores completas o programa deve funcionar corretamente.

#### Debug

O debug pode ser feito por VNC ou SSH.

> **_Nota:_** É importante sempre olhar o display OLED, pois ele mostra as principais informações para debug.

#### LEDs durante a execução do programa

LED Central Vermelho = Acesso negado.  
LED Central Laranja Estático = Desconectado do servidor.  
LED Central Laranja Fade = Conectado ao servidor.  
LEDs Setas Ligado = Acesso permitido.  

> **_Nota:_** Se ao iniciar o programa as setas ficarem ativas, quer dizer que há algum problema com o Horário/Data da Raspberry Pi.
