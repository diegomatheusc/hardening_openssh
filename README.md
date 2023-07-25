# Checklist hardening OpenSSH

- [x] Manter o OpenSSH Atualizado
- [x] Implementar Autenticação em Duas Etapas (2FA)
- [x] Alterar a Porta Padrão
- [x] Configurar o Tempo Limite de Inatividade (Idle Timeout) do SSH
- [x] Habilitar o Registro (Logging) do SSH
- [x] Utilizar Chaves SSH com frases de segurança (passphrases)
- [x] Implementar Regras de Firewall
- [x] Monitorar os Registros (Logs) do SSH
- [x] Desabilitar o Acesso SSH baseado em Senhas
- [x] Desabilitar Recursos Não Utilizados do SSH
- [x] Realizar Auditorias Regulares na Configuração do Servidor SSH
- [x] Configurar um Limite para Tentativas de Senha
- [x] Utilizar Autenticação por Chave Pública (public key authentication)


<h3> 1. Automatize atualizações do OpenSSH com o unattended-upgrades </h3>

```bash
sudo apt update
sudo apt install unattended-upgrades
```
Edite o arquivo de configuração **/etc/apt/apt.conf.d/50unattended-upgrades**:

```bash
sudo vim /etc/apt/apt.conf.d/50unattended-upgrades
```

```bash
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}:${distro_codename}-updates";
    // Optionally, you can also add other repositories here.
};
```

Isso instrui o unattended-upgrades a buscar atualizações apenas para os repositórios principais, de segurança e de atualizações. 

Habilite o unattended-upgrades editando o arquivo **/etc/apt/apt.conf.d/20auto-upgrades**:

```bash
sudo vim /etc/apt/apt.conf.d/20auto-upgrades
```
Certifique-se de que a seguinte linha esteja definida como **1**:

```bash  
APT::Periodic::Update-Package-Lists "1";
```

Por último, configure o cronjob para atualizações automáticas editando o arquivo **/etc/apt/apt.conf.d/10periodic**:

```bash
sudo vim /etc/apt/apt.conf.d/10periodic
```

Verifique se as seguintes linhas estão definidas como abaixo:

```bash
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

Isso garantirá que o sistema verifique atualizações, baixe pacotes atualizáveis e realize limpezas regularmente. A última linha **(APT::Periodic::Unattended-Upgrade "1";)** é responsável por habilitar as atualizações automáticas do unattended-upgrades.

##

<h3>2. Autenticação de dois fatores (2FA)</h3>

Até o momento, o OpenSSH não possui suporte nativo para a autenticação de dois fatores, mas podemos realizar essa ação utilizando o Google Authenticator:

```bash
sudo apt update
sudo apt install libpam-google-authenticator
```

Em uma conta de usuário específica (por exemplo, seu usuário), execute o comando google-authenticator para configurar o Google Authenticator:

```bash
google-authenticator
```
Abra o arquivo /etc/pam.d/sshd com um editor de texto com privilégios administrativos:

```bash
sudo vim /etc/pam.d/sshd
```

Adicione a seguinte linha no início do arquivo para habilitar o uso do Google Authenticator:

```bash
auth required pam_google_authenticator.so
```

Reinicie o serviço SSH:

```bash
systemctl restart sshd
```
##

<h3>3. Alterando a porta padrão</h3>

Edite o arquivo de configuração do OpenSSH (geralmente o **sshd_config**):

```bash
sudo vim /etc/ssh/sshd_config
```

Procure a linha que define a porta na qual o servidor OpenSSH está escutando. Geralmente, a linha estará comentada (começa com #). **A porta padrão é 22**.

Remova o caractere # no início da linha e altere o número da porta para a que você deseja usar. Escolha um número de porta acima de 1024, pois as portas abaixo de 1024 são consideradas portas privilegiadas e exigem privilégios de superusuário para serem usadas.

Por exemplo, para usar a porta 4567, altere a linha para:

```bash
Port 4567
```

Reinicie o serviço ssh:

```bash
systemctl restart sshd
```
##

<h3>4. Tempo limite de inatividade</h3>

Para definir um limite de inatividade (idle timeout) no OpenSSH, você pode usar a configuração **ClientAliveInterval** e **ClientAliveCountMax**. Essas opções permitem que o servidor SSH encerre automaticamente uma conexão se não houver atividade durante um determinado período de tempo.

```bash
sudo vim /etc/ssh/sshd_config
```

```bash
ClientAliveInterval 300
ClientAliveCountMax 3
```

- **ClientAliveInterval**: Especifica o intervalo de tempo em segundos após o qual o servidor envia uma mensagem para o cliente para verificar se ele está vivo. Neste exemplo, definimos para 300 segundos (5 minutos).

- **ClientAliveCountMax**: Especifica o número máximo de vezes que o servidor enviará mensagens de verificação ao cliente sem obter resposta antes de encerrar a conexão. Neste exemplo, definimos como 3.

Isso significa que, se o cliente não enviar nenhum dado ao servidor por 5 minutos (300 segundos), o servidor enviará uma mensagem para verificar se o cliente está ativo. Se não houver resposta após 3 tentativas, a conexão será encerrada.

Reinicie o serviço ssh:

```bash
systemctl restart sshd
```
##

<h3>5. Habilitando o registro de log do OpenSSH</h3>

Edite o arquivo de configuração:

```bash
sudo vim /etc/ssh/sshd_config
```

Localize as seguintes linhas no arquivo sshd_config e certifique-se de que não estejam comentadas:

```bash
# Logging
SyslogFacility AUTH
LogLevel INFO

```

- **SyslogFacility**: Especifica a instalação do syslog que será usada para registrar as mensagens do OpenSSH. Neste exemplo, definimos como AUTH para registrar as mensagens relacionadas à autenticação.
- **LogLevel**: Especifica o nível de detalhes do logging. Neste exemplo, definimos como INFO para registrar informações de nível informativo.

Você pode ajustar o nível de log conforme necessário. Os valores válidos para LogLevel são: **QUIET, FATAL, ERROR, INFO, VERBOSE, DEBUG, DEBUG1, DEBUG2 e DEBUG3**.

Salve as configurações e reinicie o serviço:

```bash
systemctl restart sshd
```
##

<h3>6. Chaves SSH com frases de segurança (passphrases)</h3>

Utilizar chaves SSH com frases de segurança (passphrases) é uma prática importante para proteger suas chaves privadas, adicionando uma camada extra de segurança à autenticação. Uma frase de segurança é como uma senha, mas geralmente é mais longa e complexa.

Gerar um par de chaves SSH:
Se você ainda não tem um par de chaves SSH (chave pública e chave privada), você pode gerá-lo usando o comando ssh-keygen. No terminal do seu computador local, execute o seguinte comando:

```bash
ssh-keygen -t rsa
```

Você pode optar por salvar a chave em um diretório específico ou simplesmente pressionar "Enter" para aceitar o local padrão (**~/.ssh/id_rsa**).

Digite uma frase de segurança (passphrase):
Ao gerar a chave, o ssh-keygen irá perguntar se você deseja criar uma frase de segurança (passphrase). É altamente recomendável que você insira uma frase de segurança aqui para proteger sua chave privada. A frase de segurança deve ser uma senha forte e difícil de adivinhar.

A frase de segurança é usada para criptografar sua chave privada, e você precisará inseri-la sempre que usar a chave. Certifique-se de memorizá-la ou armazená-la com segurança, pois não é possível recuperar a chave privada sem ela.

Copiar a chave pública para o servidor remoto:
Após gerar as chaves, você deve copiar a chave pública (id_rsa.pub) para o servidor remoto. Use o comando ssh-copy-id para fazer isso:

```bash
ssh-copy-id username@seu_servidor
```

Substitua username pelo seu nome de usuário no servidor remoto e seu_servidor pelo endereço IP ou nome de domínio do servidor.

O ssh-copy-id copiará a chave pública para o arquivo **~/.ssh/authorized_keys** no servidor remoto, permitindo que você faça login no servidor usando a chave privada protegida por frase de segurança.

Testar o login com chave SSH:
Após copiar a chave pública para o servidor remoto, você pode tentar fazer login no servidor usando a chave SSH:

```bash
ssh username@seu_servidor
```

##

<h3>7. Regras de Firewall (UFW)</h3>
Você precisará configurar o firewall do sistema operacional para permitir o tráfego de SSH (porta 22 por padrão, 4567 no nosso exemplo) e bloquear ou permitir outras portas conforme necessário. 

Instale o UFW (Uncomplicated Firewall) se ainda não estiver instalado:

```bash
sudo apt update
sudo apt install ufw
```

Defina a porta SSH (22) como permitida:

```bash
sudo ufw allow 22
```

Se desejar permitir o SSH em uma porta personalizada você também pode fazer:

```bash
sudo ufw allow 4567
```

Recarregue as regras:

```bash
sudo ufw enable
```

Verifique o status do UFW para garantir que as regras estejam ativadas:

```bash
sudo ufw status
```

##

<h3>8. Monitoramento dos logs</h3>

```bash
vim /var/log/auth.log
```

##

<h3>9. Desabilitar o Acesso SSH baseado em Senhas</h3>

Desabilitar o acesso por senhas no OpenSSH é uma prática recomendada em termos de segurança, pois reforça a autenticação usando chaves públicas, tornando mais difícil para os atacantes tentarem adivinhar ou forçar senhas de acesso ao servidor. Usar chaves públicas com autenticação por senha é uma forma mais segura de proteger o acesso ao seu servidor.

- **Certifique-se de ter configurado a autenticação por chaves SSH**:
Antes de desabilitar o acesso por senhas, é importante garantir que você já configurou a autenticação por chaves SSH para o(s) usuário(s) que precisam acessar o servidor. Se você ainda não configurou as chaves SSH, consulte a seção "Como Utilizar Chaves SSH com frases de segurança (passphrases) no OpenSSH" neste mesmo guia.

- **Edite o arquivo de configuração do OpenSSH**:
Abra o arquivo de configuração do OpenSSH, geralmente chamado de **sshd_config**, usando um editor de texto com privilégios administrativos. O local do arquivo pode variar dependendo do sistema operacional:

```bash
sudo vim /etc/ssh/sshd_config
```

- **Localize a linha que contém PasswordAuthentication** e certifique-se de que ela esteja definida como no. Se a linha não existir, você pode adicioná-la ao arquivo. Certifique-se de remover o caractere # no início da linha para descomentá-la, se necessário.

```bash
PasswordAuthentication no
```

Essa configuração desabilita o acesso por senhas no OpenSSH.

Salve as alterações no arquivo de configuração e feche o editor de texto.

Reinicie o serviço OpenSSH para que as alterações entrem em vigor:

Em distribuições baseadas em Debian (como o Ubuntu):

```bash
sudo systemctl restart sshd
```

##

<h3>10. Desabilitar recursos não utilizados</h3>

Desabilitar recursos não utilizados no OpenSSH é uma prática recomendada em termos de segurança, pois reduz a superfície de ataque do servidor SSH. A menos que você precise de recursos específicos, é uma boa ideia desabilitar aqueles que não estão sendo usados para minimizar o risco de possíveis vulnerabilidades.

- **Desabilitar o encaminhamento X11 (X11 Forwarding)**:
O encaminhamento X11 permite que você execute aplicativos gráficos de um servidor remoto e exiba a interface gráfica em sua máquina local. Em muitos casos, o encaminhamento X11 não é necessário em servidores sem interface gráfica, e desabilitá-lo pode reduzir a exposição a possíveis ataques.

Para desabilitar o encaminhamento X11, edite o arquivo de configuração do OpenSSH (sshd_config) e adicione ou certifique-se de que a seguinte linha esteja definida como no:

```bash
X11Forwarding no
```

- **Desabilitar o encaminhamento da agente SSH (SSH Agent Forwarding)**:
O encaminhamento da agente SSH permite que chaves privadas sejam encaminhadas de sua máquina local para o servidor remoto, o que pode ser conveniente para evitar que você precise armazenar suas chaves em várias máquinas. No entanto, se não for necessário, é mais seguro desabilitá-lo.

Para desabilitar o encaminhamento da agente SSH, adicione ou certifique-se de que a seguinte linha esteja definida como no no arquivo de configuração do OpenSSH (sshd_config):

```bash
AllowAgentForwarding no
```

- **Desabilitar o encaminhamento de porta (Port Forwarding)**:
O encaminhamento de porta permite que você encaminhe portas de um servidor remoto para sua máquina local ou de sua máquina local para o servidor remoto. Embora seja uma funcionalidade útil, é importante desabilitá-la se não for necessária, para evitar o uso indevido e potencial exploração.

Para desabilitar o encaminhamento de porta, adicione ou certifique-se de que a seguinte linha esteja definida como no no arquivo de configuração do OpenSSH (sshd_config):

```bash
AllowTcpForwarding no
```

- **Desabilitar o acesso root via SSH**:
Permitir acesso direto como usuário root via SSH pode ser um risco de segurança significativo, pois os ataques de força bruta geralmente visam contas de usuário "root". É recomendável desabilitar o acesso root via SSH e usar um usuário normal com privilégios de superusuário (sudo) para administrar o servidor.

Para desabilitar o acesso root via SSH, adicione ou certifique-se de que a seguinte linha esteja definida como no no arquivo de configuração do OpenSSH (sshd_config):

```bash
PermitRootLogin no
```

Após fazer as alterações no arquivo de configuração do OpenSSH, salve-o e reinicie o serviço OpenSSH para que as configurações entrem em vigor:

```bash
sudo systemctl restart sshd
```
##

<h3>11. Definir limite de tentativas</h3>

Para configurar um limite de tentativas de senha no servidor OpenSSH e prevenir ataques de força bruta, você pode utilizar a opção **MaxAuthTries** no arquivo de configuração **sshd_config**. Essa opção define o número máximo de tentativas de autenticação (senha ou chave pública) permitidas antes que o servidor encerre a conexão.

```bash
sudo vim /etc/ssh/sshd_config
```

```bash
MaxAuthTries 4
```

Reinicie o serviço do OpenSSH:

```bash
systemctl restart sshd
```
##

<h3>12. Autenticação por chave pública</h3>

A autenticação por chave pública é uma forma segura de fazer login em servidores SSH pois não se faz necessário digitar repetidamente senhas e é mais resistente a ataques de brute-force.

Primeiro verifique se você já tem um par de chaves, caso não crie no formato RSA - 2048 bits:

```bash
ssh-keygen -t rsa -b 2048
```

> -t = type

> -b = bits

Copie a nova chave para o servidor:

```bash
ssh-copy-id usuario@ip
```

Fim.



