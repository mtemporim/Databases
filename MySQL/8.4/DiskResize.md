# Extensão de disco LVM após aumento no vCenter

*(Exemplo: disco de dump `/mysql/dump` – `sdg`)*

## Cenário

O disco virtual foi **aumentado no vCenter**, porém o sistema operacional **ainda não reconhece o novo tamanho** ou o espaço adicional **não está disponível no filesystem**.

Arquitetura atual:

* Disco: `/dev/sdg`
* VG: `disk7-vg1`
* LV: `mysql_dump`
* Filesystem: `XFS`
* Mount point: `/mysql/dump`

---

## 1️⃣ Reescaneamento do disco no SO (sem reboot)

Após o aumento do disco no vCenter, execute:

```shell
echo 1 > /sys/class/block/sdg/device/rescan
```

Verificar se o novo tamanho foi reconhecido:

```shell
lsblk /dev/sdg
```

Exemplo esperado:

```text
sdg   8:96   0  100G  0 disk
└─disk7--vg1-mysql_dump 253:8 0 50G 0 lvm /mysql/dump
```

> 🔹 Aqui o **disco já cresceu**, mas o LV ainda não.

---

## 2️⃣ Redimensionar o Physical Volume (PV)

Atualizar o LVM para reconhecer o novo espaço do disco:

```shell
pvresize /dev/sdg
```

Verificar:

```shell
pvs
```

Saída esperada (exemplo):

```text
/dev/sdg  lvm2  <100.00g  <50.00g
```

> 🔹 Agora o **Volume Group tem espaço livre disponível**.

---

## 3️⃣ Verificar espaço livre no Volume Group

```shell
vgs disk7-vg1
```

Exemplo:

```text
VG        #PV #LV Attr   VSize    VFree
disk7-vg1  1   1  wz--n- <100.00g <50.00g
```

---

## 4️⃣ Estender o Logical Volume (LV)

Utilizar **todo o espaço livre disponível**:

```shell
lvextend -l +100%FREE /dev/disk7-vg1/mysql_dump
```

Verificar:

```shell
lvs /dev/disk7-vg1/mysql_dump
```

---

## 5️⃣ Expandir o filesystem XFS (online)

⚠️ **Não desmontar o filesystem** (XFS cresce online).

```shell
xfs_growfs /mysql/dump
```

Verificar o novo tamanho:

```shell
df -h /mysql/dump
```

---

## 6️⃣ Validação final

```shell
lsblk
df -h /mysql/dump
mount | grep mysql/dump
```

Tudo deve refletir o novo tamanho do disco.
