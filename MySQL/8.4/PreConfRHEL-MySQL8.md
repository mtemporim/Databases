# Pré-configuração para a instalação do MySQL 8 e 8.4 no Red Hat Enterprise Linux 9.7

## Índice

## Matriz de compatibilidade

[https://www.mysql.com/support/supportedplatforms/database.html]

## Esquema de particionamento do MySQL 8.4

**Disk 1 - 200 GB**  

|Mounting Point|Size|
|:--|:--|
|boot|1 GiB|
|boot efi|1 Gib|
|swap|16 GiB|
|var|60 GiB|
|home|60 GiB|
|/|63 GiB|
  
**Disk 2 - 200 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/data|200 GiB|
  
**Disk 3 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/innodb/  (redo, undo, doublewrite)|50 GiB|

**Disk 4 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/innodb/temp|50 GiB|
  
**Disk 5 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/audit|50 GiB|

**Disk 6 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/log (binlog, errorlog)|50 GiB|

**Disk 7 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/dump/securefile (diretorios dump e seu sub diretorio securefile, e diretório scripts)|50 GiB|

### Registrar na conta Red Hat para utilizar os repositorios oficiais

```shell
subscription-manager register
```

### Anexar automaticamente a Developer Subscription

```shell
subscription-manager attach --auto
```

### Confirmar que a subscription está ativa

```shell
subscription-manager status
```

### Habilitar os repositórios corretos do RHEL 9

```shell
subscription-manager repos \
  --enable=rhel-9-for-x86_64-baseos-rpms \
  --enable=rhel-9-for-x86_64-appstream-rpms
```

### Confirmar se os repositorios foram referenciados corretamente

```shell
subscription-manager repos --list-enabled
```

### Limpar cache

```shell
dnf clean all
```

### Atualizar os pacotes

```shell
sudo dnf -y update
```

### Preparar o ambiente para o permissionamento de grupo para o usuário Alescdba

Aplicar as permissões corretas, o 2 no 2770 garante que tudo criado dentro herda o grupo dbadmin, e o "setfacl" evita “permissão quebrada”

```shell
chmod 2770 /home/dbadmin
setfacl -m g:dbadmin:rwx /home/dbadmin
setfacl -d -m g:dbadmin:rwx /home/dbadmin
```

Criar o permissionamento do grupo para dbadmin

```shell
cat <<'EOF' | tee /etc/sudoers.d/dbadmin >/dev/null
%dbadmin ALL=(ALL) ALL
EOF
```

Ajustar as permissões minimas

```shell
chmod 440 /etc/sudoers.d/dbadmin
```

### Criação dos usuários administrativos

Criar o usuário

```shell
useradd -m <usuario>
usermod -aG dbadmin <usuario>
passwd <usuario>
passwd -e <usuario>
```

### Configurar o vim para exibir o tema desert como default para usuários logados com o root

```shell
cat > /root/.vimrc <<'EOF'
syntax on
colorscheme desert
EOF

chmod 600 /root/.vimrc
```

### Configurar o vim para exibir o tema desert como default para usuários DBAs logados pertencentes ao grupo dbadmin (logado com o usuário administrativo)

```shell
mkdir -p /etc/vimrc.d

cat > /etc/vimrc.d/dbadmin.vim <<'EOF'
syntax on
colorscheme desert
EOF

chmod 644 /etc/vimrc.d/dbadmin.vim
```

### Criar o profile para aplicar padronização dos usuários pertencentes ao grupo dbadmin e a aplicação do tema desert para os respectivos

>[!IMPORTANT]
>Esse script roda no login bash e só afeta quem está no grupo dbadmin.

```shell
cat > /etc/profile.d/dbadmin.sh <<'EOF'
# Aplica somente para membros do grupo dbadmin
if id -nG "$USER" | grep -qw "dbadmin"; then
  export PS1='[\u@DBA \W]\$ '
  alias ll='ls -lh'
  export VIMINIT='source /etc/vimrc.d/dbadmin.vim'
fi
EOF

chmod 644 /etc/profile.d/dbadmin.sh
```

### Instalar os pacotes de aplicativos, ferramentas e utilitários opcionais

```shell
dnf -y install dnf-utils setroubleshoot setools policycoreutils-python-utils 
```

### Verifique se o sistema está registrado (obrigatório)

```shell
subscription-manager status
```

Deve aparecer:  
**Overall Status: Registered**  

### Habilitar o CodeReady Builder (CRB) para instalar o pacote aplicativo para o Epel (Extra Packages for Enterprise Linux)

```shell
subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
```

### Instalar o EPEL Release (URL oficial Fedora)

```shell
dnf install -y \
https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

Atualizar o cache

```shell
dnf clean all
dnf makecache
```

Instalar o htop

```shell
dnf install -y htop
```

### Configuração do NTP e o fuso (offset)

Verificar se o serviço do NTP está ativo  

```shell
systemctl status chronyd
```

Checar se esta recebendo do ntp a atualização

```shell
chronyc sources
```

>[!NOTE]
>Caso a saida seja igual a essa, siginifca que o NTP esta configurado ok e por IP

```text
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 172.23.1.10                   2   6   177    38  +7226ns[  +47us] +/-   12ms
```

Se não estiver instalado, instalar com o comando abaixo e seguir os passos seguintes para configurar

```shell
dnf -y install chrony
```

Se não estiver habilitado, habilitar com o comando abaixo e seguir os passos seguintes para configurar

```shell
systemctl enable chronyd
```

Se estiver habilitado, executar o comando abaixo para parar o serviço do Chrony e seguir os passos seguintes para configurar

```shell
systemctl stop chronyd
```

Editar o arquivo  de configuração do chrony e adicionar o servidor ntp da Alesc

```shell
vi /etc/chrony.conf
```

Colocar um comentário na seguinte linha

**pool 2.pool.ntp.org iburst**  
ficará assim #pool 2.pool.ntp.org iburst**

Adicione a seguinte linha  
**pool ntp.alesc.sc.gov.br iburst**

Caso for em homologação ou desenvolvimento, apontar para o IP  
**pool 172.23.1.10 iburst**

Remover o comentário do segunite trecho para permitir os logs  
**#log measurements statistics tracking**  
ficará assim:  
log measurements statistics tracking  

Reiniciar o serviço do Chrony

```shell
systemctl restart chronyd
```

Checar se esta recebendo do ntp a atualização

```shell
chronyc sources
```

```text
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 172.23.1.10                   2   6   177    38  +7226ns[  +47us] +/-   12ms
```

Conferir se Time Zone esta configurado para o fuso (offset) correto pelo "Time zone" e se a sincronização automática esta ativa pelo "System clock synchronized"

```shell
timedatectl 
```

```text
Local time: Wed 2023-07-05 12:28:31 -03
Universal time: Wed 2023-07-05 15:28:31 UTC
RTC time: Wed 2023-07-05 15:28:31
Time zone: America/Sao_Paulo (-03, -0300)
System clock synchronized: yes
NTP service: active
RTC in local TZ: no
```

Caso necessite ajustar o timezone, execute o seguinte comando

```shell
timedatectl set-timezone "America/Sao_Paulo"
```

E caso necessite habilitar a sincronização automática do ntp

```shell
timedatectl set-ntp true
```

Verificar se data/hora estão corretos

```shell
timedatectl
```

### Remoção da instalação do MariaDB e MySQL nativas caso exista

```shell
dnf -y remove mysql-community-server mariadb mariadb-connector-c-config-3.2.6-1.el9_0.noarch
```

### Adicionando os discos para o MySQL

Após atachar o disco que terá o ponto de montagem unico em /mysql/data executar o seguinte comando

```shell
lsblk
```

Ele deverá paracer algo com **sdb    8:16   0 200G  0 disk**

#### Criar o Physical Volume (PV)

```shell
pvcreate /dev/sdb
```

O que isso faz?  

- Inicializa o disco para uso pelo LVM
- Escreve metadados LVM no início do disco
- Não cria filesystem ainda

Verificar:

```shell
pvs
```

Saída esperada  
/dev/sdax......  
.  
.  
.  
**/dev/sdb   lvm2   200.00g**

#### Criar o Volume Group (VG)

```shell
vgcreate disk2-vg1 /dev/sdb
```

O que é um VG?

- É um pool de espaço
- Agrupa um ou mais discos (PVs)
- De onde saem os Logical Volumes

Verificar

```shell
vgs
```

Saida esperada  
disk1-vg1  
.  
.  
.  
**disk2-vg1   1   0   0 wz--n- <200.00g <200.00g**  

#### Criar o Logical Volume (LV)

```shell
lvcreate -n mysql_data -l 100%FREE disk2-vg1
```

O que isso faz?

- -n mysql_data → nome do LV
- -l 100%FREE → usa todo o espaço disponível do VG
- disk2-vg1 → VG de origem

Verificar

```shell
lvs
```

Saída esperada  
swap  
.  
.  
.  
mysql_data  -wi-a----- <200.00g  

O caminho do dispositivo agora é: **/dev/disk2-vg1/mysql_data**

#### Criar o filesystem (XFS)

```shell
mkfs.xfs /dev/disk2-vg1/mysql_data
```

Por que XFS?

- Padrão do RHEL
- Excelente para I/O grande
- Recomendado para MySQL
- Suporta crescimento online

>[!IMPORTANT]
>⚠️ Esse comando apaga qualquer dado no LV (aqui está vazio, ok).

#### Criar o diretório para o ponto de montagem para o disco 2

```shell
mkdir -p /mysql/data
```

#### Persistir no /etc/fstab (forma correta)

1. Obter o UUID

```shell
blkid /dev/disk2-vg1/mysql_data
```

Saída esperada
/dev/disk2-vg1/mysql_data: **UUID="1a095ba2-2ec2-4ef8-9268-d7334f4a7875" TYPE="xfs"**
2. Editar /etc/fstab

```shell
vim /etc/fstab
```

Adicionar:

UUID=1a095ba2-2ec2-4ef8-9268-d7334f4a7875 /mysql/data  xfs  defaults,noatime  0 0

Por que noatime?

- Evita escrita desnecessária
- Bom para banco de dados

#### Testar o fstab (OBRIGATÓRIO)

```shell
umount /mysql/data
mount -a
```

>[!NOTE]
>Se não der erro, todas as configurações foram realizadas corretamente.

### Realizar a configuração com os demais discos

Baseado no procedimento efetuado para o disco 2 do ponto de montagem no /mysql/data sera realizada a configuração para os próximos discos para atendimento da arquitetura customizada do MySQL - Alesc

Identificar os discos

```shell
lsblk
```

```text
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0  200G  0 disk
├─sda1                    8:1    0    1G  0 part /boot/efi
├─sda2                    8:2    0    1G  0 part /boot
├─sda3                    8:3    0   60G  0 part
│ └─disk1--vg3-home     253:2    0   60G  0 lvm  /home
├─sda4                    8:4    0   60G  0 part
│ └─disk1--vg2-var      253:3    0   60G  0 lvm  /var
├─sda5                    8:5    0   16G  0 part
│ └─disk1--vg1-swap     253:1    0   16G  0 lvm  [SWAP]
└─sda6                    8:6    0   62G  0 part
  └─disk1--vg4-root     253:0    0   62G  0 lvm  /
sdb                       8:16   0  200G  0 disk
└─disk2--vg1-mysql_data 253:4    0  200G  0 lvm  /mysql/data
sdc                       8:32   0   50G  0 disk
sdd                       8:48   0   50G  0 disk
sde                       8:64   0   50G  0 disk
sdf                       8:80   0   50G  0 disk
sdg                       8:96   0   50G  0 disk
sr0                      11:0    1 1024M  0 rom
```

Criar os Physicals Volumes

```shell
pvcreate /dev/sdc
pvcreate /dev/sdd
pvcreate /dev/sde
pvcreate /dev/sdf
pvcreate /dev/sdg
```

Criar os Volumes Groups

```shell
vgcreate disk3-vg1 /dev/sdc
vgcreate disk4-vg1 /dev/sdd
vgcreate disk5-vg1 /dev/sde
vgcreate disk6-vg1 /dev/sdf
vgcreate disk7-vg1 /dev/sdg
```

Criar os Logicals Volumes

```shell
lvcreate -n mysql_innodb -l 100%FREE disk3-vg1
lvcreate -n mysql_innodb_temp -l 100%FREE disk4-vg1
lvcreate -n mysql_audit -l 100%FREE disk5-vg1
lvcreate -n mysql_log -l 100%FREE disk6-vg1
lvcreate -n mysql_dump -l 100%FREE disk7-vg1
```

Criar os filesystens em XFS

```shell
mkfs.xfs /dev/disk3-vg1/mysql_innodb
mkfs.xfs /dev/disk4-vg1/mysql_innodb_temp
mkfs.xfs /dev/disk5-vg1/mysql_audit
mkfs.xfs /dev/disk6-vg1/mysql_log
mkfs.xfs /dev/disk7-vg1/mysql_dump
```

Criar os diretórios para o ponto de montagem para os demais discos

```shell
mkdir -p /mysql/innodb/
mkdir -p /mysql/audit
mkdir -p /mysql/log
mkdir -p /mysql/dump 
```

Montar individualmente os discos com a observação da criação do diretório entre as as montagens para evitar erro na primeira vez

```shell
mount /dev/disk3-vg1/mysql_innodb /mysql/innodb/
mkdir -p /mysql/innodb/temp
mount /dev/disk4-vg1/mysql_innodb_temp /mysql/innodb/temp
mount /dev/disk5-vg1/mysql_audit /mysql/audit/
mount /dev/disk6-vg1/mysql_log /mysql/log/
mount /dev/disk7-vg1/mysql_dump /mysql/dump/
```


Persistir no /etc/fstab  
Obter o UUID

```shell
blkid /dev/disk3-vg1/mysql_innodb
blkid /dev/disk4-vg1/mysql_innodb_temp
blkid /dev/disk5-vg1/mysql_audit
blkid /dev/disk6-vg1/mysql_log
blkid /dev/disk7-vg1/mysql_dump
```

Montar as entradas, editar o fstab e adicionar

```shell
vim /etc/fstab
```

>[!WARNING]
>
>Os discos deverão ser montados seguindo a ordem conforme o modelo abaixo

```text
/dev/mapper/disk1--vg4-root /                       xfs     defaults        0 0
UUID=a1a1a1a1-a1a1-a1a1-a1a1-a1a1a1a1a1a1 /boot                   ext4    defaults        1 2
UUID=a2a2-a2a2          /boot/efi               vfat    umask=0077,shortname=winnt 0 2
/dev/mapper/disk1--vg3-home /home                   xfs     defaults        0 0
/dev/mapper/disk1--vg2-var /var                    xfs     defaults        0 0
/dev/mapper/disk1--vg1-swap none                    swap    defaults        0 0
UUID=a3a3a3a3-a3a3-a3a3-a3a3-a3a3a3a3a3a3 /mysql/data  xfs  defaults,noatime  0 0
UUID=a4a4a4a4-a4a4-a4a4-a4a4-a4a4a4a4a4a4 /mysql/innodb  xfs  defaults,noatime  0 0
UUID=a5a5a5a5-a5a5-a5a5-a5a5-a5a5a5a5a5a5 /mysql/innodb/temp  xfs  defaults,noatime  0 0
UUID=a6a6a6a6-a6a6-a6a6-a6a6-a6a6a6a6a6a6 /mysql/audit  xfs  defaults,noatime  0 0
UUID=a7a7a7a7-a7a7-a7a7-a7a7-a7a7a7a7a7a7 /mysql/log  xfs  defaults,noatime  0 0
UUID=a8a8a8a8-a8a8-a8a8-a8a8-a8a8a8a8a8a8 /mysql/dump  xfs  defaults,noatime  0 0
```
