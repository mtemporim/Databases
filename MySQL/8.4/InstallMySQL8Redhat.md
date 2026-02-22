# Instalação do MySQL 8.4 no Red Hat Enterprise Linux 9.7


<br><br>

## Índice


### Matriz de compatibilidade 
https://www.mysql.com/support/supportedplatforms/database.html



### Esquema de particionamento do MySQL 8.4

**Disk 1 - 200 GB**
|Mounting Point|Size|
|:--|:--|
|boot     |1 GiB|
|boot efi |1 Gib|
|swap     |16 GiB|
|var      |60 GiB|
|home     |60 GiB|
|/        |63 GiB|
  
**Disk 2 - 200 GB**
|Mounting Point|Size|
|:--|:--|
|/mysql/Data|200 GiB|
  
**Disk 3 - 50 GB**
|Mounting Point|Size|
|:--|:--|
|/mysql/innodb/redo|200 GiB|
|/mysql/innodb/undo|
|/mysql/innodb/doublewrite|

**Disk 4 - 50 GB**
|Mounting Point|Size|
|:--|:--|
|/mysql/innodb/tmp|50 GiB|
  
**Disk 5 - 50 GB**
|Mounting Point|Size|
|:--|:--|
|/mysql/audit|50 GiB| 

**Disk 6 - 50 GB**
|Mounting Point|Size|
|:--|:--|
|/mysql/log (binlog errorlog)|50 GiB| 

**Disk 7 - 50 GB**
|Mounting Point|Size|
|:--|:--|
|/mysql/dump/securefile (dump e securefile)|50 GiB| 
|/mysql/scripts/|



### Registrar na conta Red Hat para utilizar os repositorios oficiais 
subscription-manager register


### Anexar automaticamente a Developer Subscription
subscription-manager attach --auto


### Confirmar que a subscription está ativa
subscription-manager status


### Habilitar os repositórios corretos do RHEL 9
subscription-manager repos \
  --enable=rhel-9-for-x86_64-baseos-rpms \
  --enable=rhel-9-for-x86_64-appstream-rpms


### Confirmar se os repositorios foram referenciados corretamente 
subscription-manager repos --list-enabled

### Limpar cache
dnf clean all


### Atualizar 
dnf update -y















subscription-manager repos --list-enabled





Limpar cache e atualizar



### Atualizar os pacotes 
sudo dnf -y update

### Criação dos usuários administrativos

```shellscript
chmod 770 /home/admindba
sudo useradd -m -d /home/admindba -g admindba <usuário>
sudo passwd <usuário> (informar a senha 123#mudar)
```

### Adicionar como usuários administradores pelo visudo
Invocar o comando visudo
```shellscript
visudo
```
Localizar o seguinite trecho "**Allow root to run any commands anywhere**" e adicionar os  usuários administradores de acordo com o exemplo de  root ja disponível, salvar e sair do visudo"  
root         ALL=(ALL)       ALL  
<usuário1>   ALL=(ALL)       ALL  
<usuário12   ALL=(ALL)       ALL  

Criar o arquivo /etc/sudoers.d/admindba contendo a linha abaixo  
%admindba        ALL=(ALL)       ALL  

### Instalar os pacotes aplicativos para o Epel
sudo dnf -y install yum-utils epel-release 

### Instalar os pacotes de aplicativos opcionais, ferramentas e utilitátios opcionais 
sudo dnf install -y vim-enhanced wget bash-completion tcpdump setroubleshoot setools gcc.x86_64 net-tools tree htop lsof unzip bzip2 ncurses-compat-libs perl


#Configurar o schema de cores padrão do vim para o desert para o root (logado com o root)
echo -e "syntax on\ncolorscheme desert" >> /root/.vimrc


#Configurar o schema de cores padrão do vim para o desert para os usuários administrativo (logado com o usuário administrativo)
sudo echo -e "syntax on\ncolorscheme desert" >> /home/admindba/.vimrc



## Camila Vello Tamburim

##### Ela é muito Zé Roela

# Mas eu amo muito ela ♥