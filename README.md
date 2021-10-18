
# Montando sem um OFFSET específico
## losetup --partscan

```bash
losetup --partscan --find --show disk.img
#/dev/loop0

lsblk --fs

#NAME      FSTYPE  LABEL
#...
#loop0
#├─loop0p1 hfsplus MacData
#└─loop0p2 exfat   SharedData

mkdir /mnt/MacData
mount /dev/loop0p1 /mnt/MacData
ls /mnt/MacData

#file1 file2 file3 etc...

umount /mnt/MacData

losetup --detach-all
```

## Usando o kpartx (parte da biblioteca multipath-tools)

```bash
apt-get install kpartx
kpartx

#usage : kpartx [-a|-d|-l] [-v] wholedisk
#	-a add partition devmappings
#	-d del partition devmappings
#	-l list partitions devmappings that would be added by -a
#	-p set device name-partition number delimiter
#	-g force GUID partition table (GPT)
#	-v verbose
#Sometimes things will be clear:

kpartx -l winxp.img
#loop0p1 : 0 3326337 /dev/loop0 63
#and other times, a little less so:

kpartx -l os9.img
#loop0p1 : 0 63 /dev/loop0 1
#loop0p2 : 0 54 /dev/loop0 64
#loop0p3 : 0 74 /dev/loop0 118
#loop0p4 : 0 54 /dev/loop0 192
#loop0p5 : 0 74 /dev/loop0 246
#loop0p6 : 0 200 /dev/loop0 320
#loop0p7 : 0 512 /dev/loop0 520
#loop0p8 : 0 512 /dev/loop0 1032
#loop0p9 : 0 3330884 /dev/loop0 1544
#loop0p10 : 0 10 /dev/loop0 3332428
#For additional partition information, use testdisk, parted, mmls, gdisk, sfdisk, or fdisk (more on these below).

kpartx -a -v os9.img
#add map loop0p1 (252:0): 0 63 linear /dev/loop0 1
#add map loop0p2 (252:1): 0 54 linear /dev/loop0 64
#add map loop0p3 (252:2): 0 74 linear /dev/loop0 118
#add map loop0p4 (252:3): 0 54 linear /dev/loop0 192
#add map loop0p5 (252:4): 0 74 linear /dev/loop0 246
#add map loop0p6 (252:5): 0 200 linear /dev/loop0 320
#add map loop0p7 (252:6): 0 512 linear /dev/loop0 520
#add map loop0p8 (252:7): 0 512 linear /dev/loop0 1032
#add map loop0p9 (252:8): 0 3330884 linear /dev/loop0 1544
#add map loop0p10 (252:9): 0 10 linear /dev/loop0 3332428

mount /dev/mapper/loop0p9 /mnt -o ro

ls /mnt
#Applications (Mac OS 9)  Documents                Trash
#Desktop DB               Late Breaking News       VM Storage
#Desktop DF               System Folder
#Desktop Folder           TheVolumeSettingsFolder

umount /mnt
kpartx -d os9.img
#loop deleted : /dev/loop0
```

##  guestfish
libguestfs pode acessar quase qualquer imagem de disco imaginável. 
Ele pode fazer isso com segurança - sem a necessidade de root e com várias camadas de defesa contra imagens de disco nocivas. 
Ele pode acessar imagens de disco em máquinas remotas ou em CDs / pen drives. 
Ele pode acessar sistemas proprietários como VMware e Hyper-V.

```bash
apt install libguestfs-tools
guestfish -ro -a disk.img

#><fs> run
#...
#><fs> list-filesystems
#/dev/sda1: exfat
#><fs> mount /dev/sda1 /
#><fs> ls /
#foo
#bar
#baz
#etc
#><fs> copy-out / .
#><fs> q
```
Para suporte exFAT no momento (outubro de 2016), crie zz-exfat assim:

```bash
echo exfat-fuse > /usr/lib/x86_64-linux-gnu/guestfs/supermin.d/zz-exfat
echo exfat-utils >> /usr/lib/x86_64-linux-gnu/guestfs/supermin.d/zz-exfat
```
ou patch e compilar a partir do código-fonte. Muito obrigado a Richard W.M. Jones 
para ambas as correções. libguestfs também inclui guestmount, que pode 
montar imagens de disco diretamente no sistema de arquivos local.

##  Linux Enhanced Loopback Driver
Disponível no Sourceforge ou no servidor FTP da NASA. Mais antigo e mais complexo de configurar do que as opções acima. A documentação a seguir é um amálgama de USE.txt, readme.txt e INSTALL.txt de Jason Luttgens:
O driver de loopback aprimorado modifica o driver de loopback nativo do kernel do Linux e adiciona funcionalidade que pode fazer o driver emular uma unidade de disco de algumas maneiras. O mais importante para nós é fornecer interpretação automática e mapeamento de partições contidas em um arquivo de imagem de um disco rígido.

Para a maioria das pessoas, aqui está o que você precisa fazer:

1) Baixar binary/vmlinuz-2.4.xx-xfs-enhanced_loop.x.tar.gz
2) Baixar binary/loop-utils-0.0.1-1.i386.rpm
3) Baixar createdev

```bash
./createdev start
rpm --force -ivh /path/to/loop-utils-0.0.1-1.i386.rpm
cd /
tar xvfz /path/to/vmlinuz-2.4.xx-xfs-enhanced_loop.x.tar.gz
```

Então vá e edite seu lilo.conf ou grub.conf (ou qualquer utilitário de inicialização que você use) e adicione outra opção para inicializar o novo kernel. O nome deste kernel é /boot/vmlinuz-2.4.xx-xfs-enhanced_loop. Certifique-se de que está inicializando a partir de um drive SCSI que você recriou e usa um arquivo initrd (a menos que você saiba que o driver SCSI está embutido no kernel).
O script createdev cria os novos nomes de dispositivos de loop (/dev/loopa, /dev/loopb).
Instale a nova configuração de inicialização e reinicie! (selecione o novo kernel)
Para usar o driver de loopback aprimorado, aqui está um exemplo básico:
Você tem um arquivo de imagem, hdb.dd. É uma imagem dd de um disco rígido IDE inteiro. Aqui está um exemplo de sessão de uso do loopback aprimorado:

```bash
losetup -r /dev/loopa hdb.dd (-r means read-only)
sfdisk -l /dev/loopa
#Disk /dev/loopa: cannot get geometry
#
#Disk /dev/loopa: 0 cylinders, 0 heads, 0 sectors/track
#Warning: The first partition looks like it was made
#  for C/H/S=*/255/63 (instead of 0/0/0).
#For this listing I'll assume that geometry.
#Units = cylinders of 8225280 bytes, blocks of 1024 bytes, counting from 0
#
#   Device Boot Start     End   #cyls   #blocks   Id  System
#/dev/loopa1   *      0+   3824-   3825- 30720280+  83  Linux
#/dev/loopa2       3824+   7476-   3652- 29331288    b  Win95 FAT32
#/dev/loopa3          0       -       0         0    0  Empty
#/dev/loopa4          0       -       0         0    0  Empty
mount -o ro /dev/loopa2 /mnt/evid -t vfat
```
Neste ponto, /mnt/evid está montado e pode ser acessado da mesma forma que você normalmente monta e acessa uma partição em um disco rígido.
Quando terminar, desmonte-o e remova a associação losetup:

```bash
umount /mnt/evid/
losetup -d /dev/loopa
```

## Usando uma imagem de partição

### Criar imagem de partição a partir do disco

Se você ainda tem o dispositivo original, pode criar uma imagem de partição em vez de uma imagem de disco completa, uma vez que as imagens de partição não requerem a especificação de um deslocamento durante a montagem. Isso é:

```bash
dd if=/dev/sdb1 of=/images/partition.img
```
Ao invés de:
```bash
dd if=/dev/sdb of=/images/full_disk.img
```
Monte a partição com:
```bash
mount -ro loop /images/partition.img /mnt
```
ou para NTFS:
```bash
ntfs-3g -o ro -o loop /images/partition.img /mnt
```

### Extrair imagem de partição a partir do disco
Se a criação de uma imagem de partição não for uma opção, você pode extrair a partição de uma imagem de disco completa usando dd:

```bash
sfdisk -l -uS winxp.img

#   Device Boot    Start       End   #sectors  Id  System
#winxp.img1   *        63   3326399    3326337   7  HPFS/NTFS
#winxp.img2             0         -          0   0  Empty
#winxp.img3             0         -          0   0  Empty
#winxp.img4             0         -          0   0  Empty

dd if=winxp.img of=extracted.img skip=63 count=3326337

ntfs-3g -o ro -o loop extracted.img /mnt

ls /mnt
#boot.ini                ntldr          RECYCLER
#Documents and Settings  pagefile.sys   System Volume Information
#NTDETECT.COM            Program Files  WINDOWS

umount /mnt
```

Observe que se você usar fdisk em vez de sfdisk:

```bash
# fdisk -lu winxp.img

    Device Boot      Start         End      Blocks   Id  System
winxp.img1   *          63     3326399     1663168+   7  HPFS/NTFS
```
você precisará subtrair o setor final (3326399) do setor inicial (63) e adicionar 1 para obter o tamanho da partição em setores (3326337).

Você também pode usar os mmls de Brian Carrier (parte do The Sleuth Kit) ou o disco de teste de Christophe Grenier para encontrar facilmente o tamanho dos setores, mas você também precisará especificar o tipo de partição:

```bash
mmls winxp.img
# Não é possível determinar o tipo de partição (Mac ou DOS em 0)

mmls -t dos winxp.img
#DOS Partition Table
#Offset Sector: 0
#Units are in 512-byte sectors
#
#     Slot    Start        End          Length       Description
#00:  -----   0000000000   0000000000   0000000001   Primary Table (#0)
#01:  -----   0000000001   0000000062   0000000062   Unallocated
#02:  00:00   0000000063   0003326399   0003326337   NTFS (0x07)
#03:  -----   0003326400   0003332447   0000006048   Unallocated

testdisk winxp.img

#Proceed > Intel > Advanced
#
#Disk winxp.img - 1706 MB / 1627 MiB - CHS 827 64 63
#
#     Partition                  Start        End    Size in sectors
# 1 * HPFS - NTFS              0   1  1   824  63 63    3326337

```

# Montando a partição com um OFFSET especificado

## Encontrando o OFFSET
Para encontrar o deslocamento da partição, simplesmente 
multiplicamos o deslocamento inicial por bytes por setor. 
Ambos podem ser facilmente encontrados com as ferramentas mencionadas 
na seçãoacima. Vamos usar mmls para este exemplo:

```bash
mmls -t dos winxp.img
#DOS Partition Table
#Offset Sector: 0
#Units are in 512-byte sectors
#
#     Slot    Start        End          Length       Description
#00:  -----   0000000000   0000000000   0000000001   Primary Table (#0)
#01:  -----   0000000001   0000000062   0000000062   Unallocated
#02:  00:00   0000000063   0003326399   0003326337   NTFS (0x07)
#03:  -----   0003326400   0003332447   0000006048   Unallocated
#63 * 512 = 32256
```

Podemos evitar até a necessidade de multiplicar usando parted para encontrar o deslocamento:

```bash
parted winxp.img
#(parted) unit
#Unit?  [compact]? B
#(parted) print
#Model:  (file)
#Disk winxp.img: 1706213376B
#Sector size (logical/physical): 512B/512B
#Partition Table: msdos
#
#Number  Start   End          Size         Type     File system  Flags
# 1      32256B  1703116799B  1703084544B  primary  ntfs         boot
#
#(parted) quit
```

## Montando a partição:

```bash
mount -ro loop,offset=32256 -t ntfs winxp.img /mnt
ls /mnt
#boot.ini                ntldr          RECYCLER
#Documents and Settings  pagefile.sys   System Volume Information
#NTDETECT.COM            Program Files  WINDOWS
sudo umount /mnt
```

Ou se você preferir:

```bash
losetup -o 32256 /dev/loop1 winxp.img
mount -r -t ntfs /dev/loop1 /mnt
ls /mnt
#boot.ini                ntldr          RECYCLER
#Documents and Settings  pagefile.sys   System Volume Information
#NTDETECT.COM            Program Files  WINDOWS
umount /mnt
losetup -d /dev/loop1
```

# Notas

## Tipos de arquivos errado, opção errônea ou SuperBlock ruim

Se você receber um "tipo de fs errado" ao tentar montar uma partição ext3, pode ser devido a um diário sujo:

```bash
mount -o loop,ro -t ext3 linux.img /mnt
# mount: wrong fs type, bad option, bad superblock on /dev/loop0
```

Verifique com o arquivo:

```bash
file linux.img
#linux.img: Linux rev 1.0 ext3 filesystem data (needs journal recovery)
```
Se a imagem não precisa ser mantida em formato forense, você pode querer reparar o sistema de arquivos:

```bash
fsck.ext3 linux.img
```
Caso contrário, Hal Pomeranz descreveu várias soluções alternativas:
[Montagem de imagens usando superblocos alternativos](http://blogs.sans.org/computer-forensics/2008/12/18/mounting-images-using-alternate-superblocks/): como substituída pelas técnicas abaixo, essa abordagem envolve encontrar e especificar um endereço de superbloco alternativo para usar na montagem.
[Montando Imagens Usando Superblocos Alternativos (Acompanhamento)](http://blogs.sans.org/computer-forensics/2009/10/05/mounting-images-using-alternate-superblocks-follow-up/): Para imagens ext3, especifique "-t ext2" ao montar para ignorar o diário.
[Como montar sistemas de arquivos EXT4 sujos](https://digital-forensics.sans.org/blog/2011/06/14/digital-forensics-mounting-dirty-ext4-filesystems): Para imagens ext4, monte com a opção "noload" para ignorar a recuperação de diário. Para imagens ext3, você também precisa especificar "-t ext4".

## Montando uma partição HFS+ contida em um volume [Core Storage](https://en.wikipedia.org/wiki/Core_Storage) 

### Determine o tamanho dos setores lógicos (geralmente 512 ou 4096 bytes):

```bash
fdisk -l disk.img
#...
#Sector size (logical/physical): 512 bytes / 512 bytes
#Disklabel type: gpt
#...
#Device        Start       End   Sectors   Size Type
#disk.img1        40    409639    409600   200M EFI System
#disk.img2    409640 975503591 975093952   465G Apple Core storage
#disk.img3 975503592 976773127   1269536 619.9M Apple boot
```

### Determine o deslocamento e o tamanho nos setores:

```bash
testdisk disk.img
```
Selecione Proceed > EFI GPT > Analyse > Quick Search, que vai gerar uma saída como esta:

```bash
#   Partition                  Start        End    Size in sectors
# P EFI System                    40     409639     409600 [EFI]
# P Mac HFS                   409640  974778407  974368768
# P Mac HFS                975503592  976773127    1269536
```

Aperte Q quatro vezes para sair do testdisk

### Monte e verifique o conteúdo:

```bash
mount disk.img -t hfsplus -o ro,loop,offset=$((409640*512)),sizelimit=$((974368768*512)) /mnt
ls /mnt
#Applications  cores  etc   installer.failurerequests  net      private  System  User Information  usr  Volumes
#bin           dev    home  Library                    Network  sbin     tmp     Users             var
umount /mnt
```

# Notas sobre a montagem do Core Storage
Para partições em discos físicos, especifique apenas o limite de tamanho, por exemplo,
```bash
mount / dev / sdx2 -t hfsplus -o ro, sizelimit = $ ((974368768 * 512)) / mnt
```
Descobri que executar fsck.hfsplus em uma partição HFS+ dentro de um volume de Core Storage (por exemplo, losetup -o $ ((409640 * 512)) / dev / loop0 disk.img && fsck.hfsplus / dev / loop0 para uma imagem ou fsck.hfsplus -f / dev / sdx2 para um disco) permitiu que a partição fosse montada normalmente no gerenciador de arquivos. No entanto, embora o disco tenha inicializado normalmente depois, esse método envolve pequenos reparos no cabeçalho do volume, então é melhor usar o processo descrito em 3.2.1-3.
Se você não conseguir montar a imagem ou o disco, experimente HFS+ Rescue ", um programa para recuperar arquivos de uma partição formatada em HFS +. Você pode restaurar seus arquivos e diretórios mesmo quando não é possível montá-los com seu sistema operacional. Como efeito colateral, o programa também restaura arquivos excluídos. "
Para montar um volume FileVault Drive Encrypted (FVDE), você pode usar fvdemount
A tentativa de montar normalmente uma partição HFS+ envolvida em um contêiner do Core Storage retornará erros como estes:
tipo de fs incorreto, opção incorreta, superbloco incorreto em / dev / loop0, página de código ausente ou programa auxiliar ou outro erro
hfsplus: incapaz de encontrar superbloco HFS +
hfsplus: cabeçalho de volume secundário inválido

## HFS vs HFS+

