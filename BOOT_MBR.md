
Independentemente do computador ou sistema operacional, desktops e laptops padrão ("compatíveis com IBM") são ligados e inicializados usando uma de duas maneiras: o método BIOS-MBR tradicional e o método UEFI-GPT mais recente, usado pelos mais recentes versões do Windows, Linux e Mac OS X em PCs, laptops e tablets mais recentes. Este artigo resume o processo pelo qual os PCs BIOS tradicionais carregam um sistema operacional, cobrindo os fundamentos e detalhes do BIOS, MBR e setor de inicialização.

### Visão geral do processo de inicialização do BIOS / MBR
No diagrama abaixo, a sequência de inicialização para todos os computadores e sistemas operacionais padrão é mostrada:
![](fotos/1-MBR-Boot-Sequence.png)
Como você pode ver, o processo de inicialização é dividido em vários componentes principais, cada um dos quais é um subsistema completamente separado com muitas opções e variações diferentes. As implementações de cada componente podem diferir muito dependendo do seu hardware e sistema operacional, mas as regras que eles seguem e o processo pelo qual funcionam são sempre os mesmos.

## Componentes do processo de inicialização
### O BIOS
O BIOS é onde o hardware encontra o software pela primeira vez e onde começa toda a mágica do boot. O  código BIOS é embutido na placa-mãe do seu PC, geralmente armazenado no que é chamado de EEPROM  1 e é consideravelmente específico do hardware. O BIOS é o software de nível mais baixo que faz interface com o hardware como um todo, 2 e é a interface por meio da qual o carregador de inicialização e o kernel do sistema operacional podem se comunicar e controlar o hardware. Por meio de chamadas padronizadas para o BIOS (“interrupções” no jargão de computador), o sistema operacional pode acionar o BIOS para ler e gravar no disco e fazer interface com outros componentes de hardware.

Quando o seu PC é ligado pela primeira vez, muita coisa acontece. Os componentes elétricos do PC são inicialmente responsáveis ​​por dar vida ao seu computador, pois os circuitos de neutralização levam o toque do botão liga / desliga e acionam um interruptor que ativa a fonte de alimentação e direciona a corrente da fonte de alimentação para a placa-mãe e, principalmente através dela, para todos os vários componentes do seu PC. À medida que cada componente individual recebe eletricidade vital, ele é ligado e colocado online em seu estado inicial. As rotinas de inicialização e a funcionalidade geral dos componentes mais simples, como RAM e PSU, estão embutidos neles como uma série de circuitos lógicos (portas AND / NAND e OR / NOR), enquanto as partes mais complicadas, como a placa de vídeo, têm seus próprios microcontroladores que atuam como mini-CPUs,controlar o hardware e interagir com o resto do seu PC para delegar e supervisionar o trabalho.

### O Processo POST
Uma vez que seu PC tenha sido ligado, o BIOS começa seu trabalho como parte do processo POST (Power-On Self Test). Ele une todas as várias partes do seu PC e faz a interface entre elas conforme necessário, configurando seu monitor de vídeo para aceitar VGA básico e exibi-lo na tela, inicializando os bancos de memória e dando à CPU acesso a todo o hardware. Ele verifica os barramentos IO em busca de hardware conectado e identifica e mapeia o acesso aos discos rígidos que você conectou ao PC. O BIOS em placas-mãe mais novas é inteligente o suficiente para até mesmo reconhecer e identificar dispositivos USB, como unidades externas e mouses USB, permitindo que você inicialize a partir de dispositivos USB e use o mouse em software legado.

Durante o procedimento POST, testes rápidos são realizados sempre que possível e erros causados ​​por hardware incompatível, dispositivos desconectados ou componentes com falha são freqüentemente detectados.  É a BIOS responsável por uma variedade de mensagens de erro, como “erro de teclado ou nenhum teclado presente ”ou avisos sobre memória incompatível / não reconhecida. Neste ponto, a maior parte do trabalho do BIOS foi concluída e está quase pronto para passar para o próximo estágio do processo de inicialização. A única coisa que resta é executar o que é chamado de “Add-On ROMs”: algum hardware conectado à placa-mãe pode exigir a intervenção do usuário para completar sua inicialização e o BIOS realmente entrega o controle de todo o PC para rotinas de software codificadas em hardware como o placa de vídeo ou controladores RAID. Eles assumem o controle do computador e de sua tela e permitem que você faça coisas como configurar matrizes RAID ou definir as configurações de tela antes mesmo que o PC tenha realmente terminado de ligar. Quando terminam a execução, eles passam o controle do computador de volta para o BIOS e o PC entra em um estado básico e utilizável e está pronto para começar.

### BIOS Boot Handoff
Depois de configurar os dispositivos básicos de entrada e saída do seu PC, o BIOS agora entra nos estágios finais onde ainda está no controle do seu computador. Neste ponto, você normalmente terá a opção de apertar rapidamente uma tecla para entrar na configuração do BIOS, de onde você pode definir as configurações de hardware e controlar como o seu PC é inicializado. Se você não escolher nada, o BIOS iniciará a primeira etapa para realmente “inicializar” o seu PC usando as configurações padrão.

Anteriormente, mencionamos que uma parte importante do trabalho do BIOS é detectar e mapear os discos rígidos conectados. Esta lista agora é útil, pois a BIOS carregará um programa muito pequeno do primeiro disco rígido para a memória e dirá à CPU para executar seu conteúdo, transferindo o controle do computador para o que quer que esteja no disco rígido e encerrando sua atividade papel no carregamento de seu PC. Este disco rígido é conhecido como “dispositivo de inicialização”, “disco de inicialização” ou “unidade 0” e geralmente pode ser selecionado ou definido na configuração do BIOS.

### O dispositivo de inicialização
Independentemente de o BIOS ter sido configurado para inicializar a partir de um disco rígido local ou de um stick USB removível, a sequência de transferência é a mesma. Uma vez que os procedimentos do BIOS POST e AddOn ROM foram concluídos, o BIOS carrega os primeiros 512 bytes do disco rígido do dispositivo de inicialização selecionado - esses 512 bytes são comumente conhecidos como MBR ou Master Boot Record.

### O Master Boot Record (MBR)
O MBR é o primeiro e mais importante componente do lado do software no procedimento de inicialização em máquinas baseadas em BIOS. Cada disco rígido possui um MBR e contém várias informações importantes.













