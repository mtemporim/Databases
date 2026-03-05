# Extensão de disco LVM após aumento no vCenter

*(Exemplo: disco de dump `/mysql/dump` – `sdg`)*

>[!IMPORTANT]
>Esse procedimento deve ser executado **após** o disco ter sido aumentado no vCenter (ou hypervisor equivalente). Nenhuma reinicialização do servidor é necessária.

O exemplo abaixo utiliza o disco **sdg** (`/mysql/dump`) com o Volume Group **disk7-vg1** e o Logical Volume **mysql_dump**, porém o procedimento é idêntico para qualquer disco da arquitetura. Basta substituir os nomes conforme a tabela de referência ao final.

## Índice

---
Arquitetura atual:

* Disco: `/dev/sdg`
* VG: `disk7-vg1`
* LV: `mysql_dump`
* Filesystem: `XFS`
* Mount point: `/mysql/dump`

---

### 1. Verificar o tamanho atual do disco antes do rescan

```shell
lsblk
```

Saída esperada — o SO ainda enxerga o tamanho antigo (ex.: 50G):

```text
sdg   8:96   0   50G  0 disk
└─disk7--vg1-mysql_dump  253:8  0  50G  0 lvm  /mysql/dump
```

### 2. Forçar o rescan do barramento SCSI para o SO reconhecer o novo tamanho

Identificar o número do host SCSI associado ao disco

```shell
ls /sys/class/scsi_disk/
```

Saída esperada (exemplo):

```text
0:0:0:0  0:0:1:0  0:0:2:0  0:0:3:0  0:0:4:0  0:0:5:0  0:0:6:0
```

O comando abaixo é para ser usado para o rescan apenas do dispositivo SCSI (disco) expandido

```shell
echo 1 > /sys/block/sdg/device/rescan
```

ou

```shell

echo 1 > /sys/block/sdg/device/rescan
```

>[!NOTE]
>Alternativamente, é possível fazer o rescan de todos os hosts de uma só vez com o comando abaixo:

```shell
for host in /sys/class/scsi_host/host*/scan; do
  echo "- - -" > $host
done
```

Ou se preferir, poderá executar o rescan para cada host listado

>[!WARNING]
>
>Esse comando só deverá ser executado após o levantamento do endereço SCSI pelo comando **ls /sys/class/scsi_disk/**

```shell
echo 1 > /sys/class/scsi_disk/0\:0\:0\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:1\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:2\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:3\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:4\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:5\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:6\:0/device/rescan
```

Verificar se o SO passou a enxergar o novo tamanho do disco

```shell
lsblk
```

O disco **sdg** deverá refletir o novo tamanho (ex.: 200G):

```text
sdg   8:96   0  200G  0 disk
└─disk7--vg1-mysql_dump  253:x  0    50G  0 lvm  /mysql/dump
```

>[!NOTE]
>Neste momento o LV ainda exibe o tamanho antigo. Os passos seguintes irão propagá-lo até o filesystem.

---

### 3. Redimensionar o Physical Volume (PV) para absorver o espaço novo do disco

Verificar:

```shell
pvs
```

Saída esperada — a coluna **PSize** reflete o tamanho e **PFree** mostra o espaço disponível para extensão:

```text
PV         VG        Fmt  Attr PSize    PFree
/dev/sdg   disk7-vg1 lvm2 a--   <50.00g    0
```

Atualizar o LVM para reconhecer o novo espaço do disco:

```shell
pvresize /dev/sdg
```

Verificar o redimensionamento:

```shell
pvs
```

Saída esperada — a coluna **PSize** reflete o novo tamanho e **PFree** mostra o espaço disponível para extensão:

```text
PV         VG          Fmt  Attr PSize    PFree
/dev/sdg   disk7-vg1   lvm2 a--  <200.00g <150.00g
```

> 🔹 Agora o **Volume Group passou a enxergar espaço livre disponível**, proveniente do PV redimensionado.**.

### 4. Verificar espaço livre no Volume Group

```shell
vgs disk7-vg1
```

Exemplo:

```text
VG        #PV #LV Attr   VSize    VFree
disk7-vg1  1   1  wz--n- <200.00g <150.00g
```

### 5. Estender o Logical Volume (LV) utilizando todo o espaço livre disponível

Verificar:

```shell
lvs /dev/disk7-vg1/mysql_dump
```

Saída esperada:

```text
LV           VG          Attr       LSize
mysql_dump   disk7-vg1   -wi-ao---- <50.00g
```

>[!NOTE]
>
> Utilizar **todo o espaço livre disponível**:

```shell
lvextend -l +100%FREE /dev/disk7-vg1/mysql_dump
```

O que isso faz?

* `-l +100%FREE` → usa todo o espaço livre disponível no VG
* Não desmonta o filesystem — a operação é online

Verificar:

```shell
lvs /dev/disk7-vg1/mysql_dump
```

Saída esperada:

```text
LV           VG          Attr       LSize
mysql_dump   disk7-vg1   -wi-ao---- <200.00g
```

### 6. Expandir o filesystem XFS para ocupar o novo tamanho do LV (online)

Verificar o tamanho:

```shell
df -hT /mysql/dump
```

Saída esperada:

```text
Filesystem                          Type  Size  Used Avail Use% Mounted on
/dev/mapper/disk7--vg1-mysql_dump   xfs   50G    xxG   xxG  xx% /mysql/dump
```

⚠️ **Não desmontar o filesystem** (XFS cresce online).

>[!IMPORTANT]
>O XFS **só cresce**, nunca diminui. A operação é **online** — o ponto de montagem `/mysql/dump` **não precisa ser desmontado**.

```shell
xfs_growfs /mysql/dump
```

Saída esperada:

```text
meta-data=/dev/mapper/disk7--vg1-mysql_dump isize=512    agcount=4, agsize=3276544 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=13106176, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13106176 to 52427776
```

>[!NOTE]
>A última linha confirma a extensão: **data blocks changed from X to Y**.

### 7. Validar o novo tamanho disponível no ponto de montagem

Verificar o novo tamanho:

```shell
df -hT /mysql/dump
```

Saída esperada:

```text
Filesystem                          Type  Size  Used Avail Use% Mounted on
/dev/mapper/disk7--vg1-mysql_dump   xfs   200G   xxG   xxG  xx% /mysql/dump
```

```shell
lsblk
```

```shell
mount | grep mysql/dump
```

Tudo deve refletir o novo tamanho do disco.

#### Tabela de referência para extensão dos demais discos

| Disco | Device  | VG          | LV                | Ponto de montagem   |
|:------|:--------|:------------|:------------------|:--------------------|
| 2     | /dev/sdb| disk2-vg1   | mysql_data        | /mysql/data         |
| 3     | /dev/sdc| disk3-vg1   | mysql_innodb      | /mysql/innodb       |
| 4     | /dev/sdd| disk4-vg1   | mysql_innodb_temp | /mysql/innodb/temp  |
| 5     | /dev/sde| disk5-vg1   | mysql_audit       | /mysql/audit        |
| 6     | /dev/sdf| disk6-vg1   | mysql_log         | /mysql/log          |
| 7     | /dev/sdg| disk7-vg1   | mysql_dump        | /mysql/dump         |

>[!NOTE]
>Para qualquer disco da tabela, substitua o **Device**, **VG**, **LV** e o **Ponto de montagem** nos comandos `pvresize`, `lvextend` e `xfs_growfs` conforme a linha correspondente.

#### Troubleshooting comum

* Disco não rescaneado: Rode o comando abaixo para checar logs. Se falhar, utilizar o loop de hosts.

```shell
dmesg | grep sd 
```

* pvresize falha: Confirme novo tamanho com lsblk. Se PV tem partições, utilizar growpart/parted antes.

```shell
lsblk
```

* xfs_growfs falha: Verifique se é XFS (df -T) e se o LV foi estendido.

```shell
df -hT
```

## Extensão de disco LVM após aumento no vCenter para discos com mais de uma partição

### Verificar o tamanho atual do disco antes do rescan

```shell
lsblk
```

Saída esperada — o SO ainda enxerga o tamanho antigo (ex.: 500G):

```text
sdb                          8:16   0  500G  0 disk
└─sdb1                       8:17   0  498G  0 part
  ├─Disk2-mysql_3307_data  252:4    0  249G  0 lvm  /mysql/3307/data
  └─Disk2-mysql_3306_data  252:6    0  249G  0 lvm  /mysql/3306/data
```

### Forçar o rescan do barramento SCSI para o SO reconhecer o novo tamanho

Identificar o número do host SCSI associado ao disco

```shell
ls /sys/class/scsi_disk/
```

Saída esperada (exemplo):

```text
0:0:0:0  0:0:1:0  0:0:2:0  0:0:3:0  0:0:4:0
```

O comando abaixo é para ser usado para o rescan apenas do dispositivo SCSI (disco) expandido

```shell
echo 1 > /sys/block/sdb/device/rescan
```

>[!NOTE]
>Alternativamente, é possível fazer o rescan de todos os hosts de uma só vez com o comando abaixo:

```shell
for host in /sys/class/scsi_host/host*/scan; do
  echo "- - -" > $host
done
```

Ou se preferir, poderá executar o rescan para cada host listado

>[!WARNING]
>
>Esse comando só deverá ser executado após o levantamento do endereço SCSI pelo comando **ls /sys/class/scsi_disk/**

```shell
echo 1 > /sys/class/scsi_disk/0\:0\:0\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:1\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:2\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:3\:0/device/rescan
echo 1 > /sys/class/scsi_disk/0\:0\:4\:0/device/rescan
```

Verificar se o SO passou a enxergar o novo tamanho do disco

```shell
lsblk
```

O disco **sdb** deverá refletir o novo tamanho (ex.: 600G), porém a partição sdb1 ainda estará no tamanho original

```text
sdb                          8:16   0  600G  0 disk
└─sdb1                       8:17   0  498G  0 part
  ├─Disk2-mysql_3307_data  252:4    0  249G  0 lvm  /mysql/3307/data
  └─Disk2-mysql_3306_data  252:6    0  249G  0 lvm  /mysql/3306/data
```

>[!NOTE]
>Neste momento o LV ainda exibe o tamanho antigo. Os passos seguintes irão propagá-lo até o filesystem.

Verificar a partição atual

```shell
parted /dev/sdb print
```

Saída esperada

```text
Number  Start   End    Size   Type     File system  Flags
 1      1049kB  535GB  535GB  primary               lvm
```

### Expandir a partição com parted

```shell
parted /dev/sdb
```

Dentro do parted, execute:

```shell
(parted) print                  # veja o número da partição e o fim do disco
(parted) resizepart 1 100%      # expande a partição 1 até o fim do disco
(parted) quit                   # sair do parted 
```

Verificar se o resize foi realizado

```shell
parted /dev/sdb print
```

```text
Number  Start   End    Size   Type     File system  Flags
 1      1049kB  644GB  644GB  primary               lvm
```

Nesse momento o lsblk ja consegue exibir os valores de disco e partição com o novo tamanho

Verificar se o SO passou a enxergar o novo tamanho do disco para a partição também

```shell
lsblk
```

O disco **sdb** deverá refletir o novo tamanho (ex.: 600G), e a partição sdb1 também

```text
sdb                          8:16   0  600G  0 disk
└─sdb1                       8:17   0  600G  0 part
  ├─Disk2-mysql_3307_data  252:4    0  249G  0 lvm  /mysql/3307/data
  └─Disk2-mysql_3306_data  252:6    0  249G  0 lvm  /mysql/3306/data
```

### Verificação da propagação para PV, VG e LV

Executar os comandos para validar o ajuste de tamanho

Verificar os Physical Volume (PV)

```shell
pvs
```

Saida esperada

```shell
PV         VG    Fmt  Attr PSize   PFree
.........  ..... .... ...  ....... .....
/dev/sdb1  Disk2 lvm2 a--  498.00g 4.00m
.........  ..... .... ...  ....... .....
.........  ..... .... ...  ....... .....
```

Verificar os Volume Group (VG)

```shell
vgs
```

Saida esperada

```shell
VG    #PV #LV #SN Attr   VSize   VFree
.....   .   .   . ...... ....... .....
Disk2   1   2   0 wz--n- 498.00g 4.00m
.....   .   .   . ...... ....... .....
.....   .   .   . ...... ....... .....
```

Verificar os Logical Voluma (LV)

```shell
lvs
```

Saida esperada

```shell
LV               VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
...............  ..... .......... .......
mysql_3306_data  Disk2 -wi-ao---- 249.00g
...............  ..... .......... .......
...............  ..... .......... .......
```

### Informar o kernel sobre a mudança (sem reboot)

```shell
partprobe /dev/sdb
```

ou se não funcionar:

```shell
kpartx -u /dev/sdb
```

### Redimensionar o Physical Volume

Verificar o PV

```shell
pvresize /dev/sdb1
```

Saida esperada

```shell
PV         VG    Fmt  Attr PSize   PFree
.........  ..... .... ...  ....... .....
/dev/sdb1  Disk2 lvm2 a--   <600.00g 101.99g
.........  ..... .... ...  ....... .....
.........  ..... .... ...  ....... .....
```

Verificar o VG

```shell
vgs
```

Saida esperada

```shell
VG    #PV #LV #SN Attr   VSize   VFree
.....   .   .   . ...... ....... .....
Disk2   1   2   0 wz--n- <600.00g 101.99g
.....   .   .   . ...... ....... .....
.....   .   .   . ...... ....... .....
```

### Expandir o Logical Volume

Ver o nome exato do VG

```shell
vgs
```

Expandir usando todo espaço livre

Exemplo lvextend -l +100%FREE /dev/<VG_NAME>/<LV_NAME>

```shell
lvextend -l +100%FREE /dev/Disk2/mysql_3306_data
```

Verificar o novo tamanho do LV

```shell
lvs
```

Saida esperada

```shell
LV               VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
...............  ..... .......... .......
mysql_3306_data  Disk2 -wi-ao---- <351.00g
...............  ..... .......... .......
...............  ..... .......... .......
```

### Expandir no filesystem (online)

Verificar tipo do FS

```shell
df -Th /mysql/3306/data
```

```text
Filesystem                         Type      Size  Used Avail Use% Mounted on
.................................  .......   ....  ....  ....  ... .................
/dev/mapper/Disk2-mysql_3306_data  xfs       249G  249G   78M 100% /mysql/3306/data
.................................  .......   ....  ....  ....  ... .................
.................................  .......   ....  ....  ....  ... .................
```

Se XFS:

```shell
xfs_growfs /mysql/3306/data
```

Saida esperada

```shell
meta-data=/dev/mapper/Disk2-mysql_3306_data isize=512    agcount=4, agsize=16318464 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=65273856, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=31872, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 65273856 to 92011520
```

Se EXT4:

```shell
resize2fs /dev/<VG_NAME>/Disk2-mysql_3306_data
```

### Validar

```shell
df -h /mysql/3306/data
pvs && lvs
```
