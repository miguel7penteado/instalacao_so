Durante a boot, o gerenciador de boot carrega o kernel e um ramdisk chamado initramfs na memória. 
Após algumas inicializações muito básicas, o kernel irá extrair o initramfs para o diretório raiz  /   e então inicializará o programa init que se encontra no disco (geralmente  /init). 
Este programa init fará alguma inicialização adicional (por exemplo, carregar os drivers do sistema de arquivos, do fsck, etc.). 
Então, este programa init montará o diretório raiz real com base nos parâmetros do kernel  root, rootflags, etc. e usa o programa switch_root para alternar o / do initramfs para a nova raiz montada. 
O programa init será iniciado por switch_root para fazer as inicializações finais, como montar entradas em fstab, carregar interface gráfica e assim por diante. 
Muitas distribuições fornecem ferramentas para gerar o initramfs. Essas ferramentas geralmente são modulares e permitem que os usuários adicionem suas próprias custmizações. Por exemplo essa ferramenta na distribuição Arch Linux é chamada de “mkinitcpio”. 
