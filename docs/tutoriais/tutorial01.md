# Instalando o Debian no VirtualBox

Neste tutorial criaremos uma máquina virtual o VirtualBox e instalaremos do Debian sem interface gráfica, para uso na disciplina.

O primeiro passo é obter a imagem (ISO) do Debian no site [https://debian.org](https://debian.org).

Na sequência devemos abrir o VirtualBox.

![Fig01](img/t01-01.png)

Na tela inicial clicamos em *Novo*.

![Fig02](img/t01-02.png)

Definimos um nome para nossa máquina virtual. Neste tutorial usaremos `debian`. O VirtualBox já reconhece o nome e o tipo do sistema operacional. Caso ele não reconheça, selecione as opções corretas em *Tipo*, *Subtype* e *Versão*.

![Fig03](img/t01-03.png)

Agora você deve procurar a imagem ISO que foi baixada no início.

![Fig04](img/t01-04.png)

![Fig05](img/t01-05.png)

![Fig06](img/t01-06.png)

É importante **marcar** a opção *Pular Instalação Desassistida*. Caso isso não seja feito, não teremos controle sobre alguns detalhes da instalação que são importantes para a disciplina.

![Fig07](img/t01-07.png)

Aqui definimos a quantidade de memória e de processadores. Para nosso exemplo, vamos manter 2048MB de memória e vamos usar 2 processadores.

![Fig09](img/t01-09.png)

O tamanho do disco de armazenamento pode ser mantido em 20GB. Importante perceber que ele não vai ocupar todo de uma vez, esse é o limite até onde esse disco pode crescer.

![Fig11](img/t01-11.png)

O VirtualBox vai mostrar um resumo e você deve clicar em *Finalizar*.

![Fig12](img/t01-12.png)

Depois de criado, basta clicar em *Iniciar*.

![Fig15](img/t01-15.png)

Na primeira tela de instalação, devemos escolher a opção *Install*. Se não formos rápidos, o sistema entrará automaticamente na opção *Graphical Install* (Instalação gráfica), que **não** é o que queremos. Caso entre na instalação gráfica, basta reiniciar a máquina virtual.

![Fig16](img/t01-16.png)

Vamos manter a máquina virtual em inglês mesmo, já que esse é o padrão nos servidores.

![Fig17](img/t01-17.png)

Já na localização, vamos em *other* para procurar o *Brazil*

![Fig18](img/t01-18.png)

![Fig19](img/t01-19.png)

![Fig20](img/t01-20.png)

O *locale* define as configurações de formato de data, números, etc. Vamos manter em *United States*.

![Fig21](img/t01-21.png)

Já o teclado vamos configurar como *Brazilian*

![Fig22](img/t01-22.png)

O *hostname* é o nome da máquina virtual na rede. Vamos usar *debian*.

![Fig23](img/t01-23.png)

O domínio vamos manter o padrão.

![Fig24](img/t01-24.png)

Agora é a definição da senha de *root*. O *root* é o usuário administrador do sistema. Para as máquinas da disciplina, usaremos a senha `1234`.

![Fig25](img/t01-25.png)

Agora é só confirmar a mesma senha. 

![Fig26](img/t01-26.png)

Escreva o nome do usuário. Esse é o nome real, pode usar espaços, acentos, etc.

![Fig27](img/t01-27.png)

Agora o *username*. Esse é o seu login, não use espaços, acentos, etc.

![Fig29](img/t01-29.png)

Defina a senha do seu usuário. Use `1234`.

![Fig31](img/t01-31.png)

Repita a senha para confirmar.

![Fig32](img/t01-32.png)

Escolha o seu estado para definir o fuso horário.

![Fig33](img/t01-33.png)

Para o processo de escrita no disco, devemos escolher a primeira opção, já que usaremos o disco inteiro da máquina.

![Fig34](img/t01-34.png)

Confirme o disco que vai ser instalado.

![Fig35](img/t01-35.png)

Para o particionamento, vamos usar apenas uma partição com todos os arquivos.

![Fig36](img/t01-36.png)

Agora é só finalizar o particionamento e escrever as mudanças no disco.

![Fig37](img/t01-37.png)

Confirmando.

![Fig38](img/t01-38.png)

Selecione *No*, já que não usaremos discos extras na instalação.

![Fig39](img/t01-39.png)

Escolha o país mais próximo.

![Fig40](img/t01-40.png)

Esses são os servidores que armazenam os arquivos do sistema. Se souber que um é mais rápido, selecione, caso contrário, tanto faz.

![Fig41](img/t01-41.png)

Não usaremos proxy, pode deixar em branco.

![Fig42](img/t01-42.png)

Aqui ele pede participação em uma pesquisa, selecione *No*.

![Fig43](img/t01-43.png)

!!!warning Atenção!
    Esse passo define se o sistema terá interface gráfica. Caso seja selecionada uma interface gráfica, a instalação demorará muito!

Nesse passo, vamos desmarcar as opções de interface gráfica, deixando marcado apenas o *standard system utilities*. Para selecionar use a barra de espaços.

![Fig45](img/t01-45.png)

Essa opção pergunta se deseja instalar o GRUB. Ele é responsável pela inicialização do sistema, logo selecionamos a opção *Yes*.

![Fig46](img/t01-46.png)

Selecionamos o disco (só tem uma opção).

![Fig47](img/t01-47.png)

Clique em *Continue* para que a máquina seja reiniciada.

![Fig48](img/t01-48.png)

A máquina virtual reiniciará e já está pronta para uso!

