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
- [x] Desabilitar Pedidos de Conexão SSH sem Senha para Usuários
- [x] Configurar um Limite para Tentativas de Senha
- [x] Desabilitar o Encaminhamento (Forwarding) X11
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
sudo nano /etc/apt/apt.conf.d/10periodic
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



