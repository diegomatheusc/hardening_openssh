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


