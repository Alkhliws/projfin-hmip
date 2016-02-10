## Introdução ##

Na análise do protocolo MIPv6 será montado um ambiente de testes que fará o uso de máquinas virtuais, devido a facilidade e rapidez para construção dos cenários. Neste trabalho será utilizado a máquina virtual UML (_User Mode Linux_), pois é incorporada ao kernel e apresenta um alto desempenho em execução.

Para testes do protocolo é necessário um kernel Linux compilado com suporte ao MIPv6, uma  instalação do MIPL (_Mobile IPv6 for Linux_) uma implementação do IPv6 Móvel para o Linux, e a instalação do RADVD (_Router ADVertisement Daemon_) necessário em redes IPv6 para gerar _router advertisement_ e escutar _router solicitations_.

## UML ##

_User Mode Linux_ (UML) é uma máquina virtual para sistemas Linux, é uma implementação que permite que o Kernel Linux rode no sistema operacional Linux como um processo normal, diferente de outras tecnologias de virtualização que emulam uma plataforma física. Com UML é possível criar máquinas virtuais para ajudar no desenvolvimento de aplicações, serviços de rede, implementação clusters de rede, testes de segurança.

Para utilizar uma máquina virtual UML somente é necessário uma versão do Kernel Linux compilado para a arquitetura UML e um sistema de arquivos. No endereço http://uml.nagafix.co.uk/ há disponível vários sistemas de arquivos, no ambiente de testes será utilizado um baseado no Linux Debian. Para isso faça o donwload:

```
# wget http://uml.nagafix.co.uk/Debian-4.0/Debian-4.0-x86-root_fs.bz2
# bunzip2 Debian-4.0-x86-root_fs.bz2
# mv Debian-4.0-x86-root_fs DebianFS
```

Um pacote de ferramentas para UML chamado _uml utilities_ deve ser instalado no sistema hospedeiro para permitir interligar as máquinas virtuais em rede, e auxiliar no ambiente de testes. Para isso:

```
# wget http://prdownloads.sourceforge.net/user-mode-linux/uml_utilities_20040406.tar.bz2
# tar jxvf uml_utilities_20040406.tar.bz2 && cd tools
# make && make install 
```

O _uml\_switch_ que acompanha este pacote não permite a simulação de mobilidade, algumas alterações no código do _uml\_switch\_foram feitas pelo desenvolvedor do projeto "Avaliação do Protocolo Fast Handover MIPv6" que permitem a simulação da mobilidade. Será preciso desta versão para o ambiente, para isso:_

```
# wget http://algum.lugar
# tar xvzf uml_switch.tar.gz && cd uml_switch
# make && make install
```

## Kernel ##

Para o ambiente de testes é preciso um kernel compilado para a arquitetura UML com suporte a MIPv6. No kernel o suporte nativo do protocolo MIPv6 está disponível desde a versão 2.6.17, nestes cenários de testes será utilizado a versão 2.6.25.6 do kernel. Para iniciar o trabalho de compilação é necessário fazer o download dos códigos fontes do kernel:

```
# wget http://kernel.org/pub/linux/kernel/v2.6/linux-2.6.25.6.tar.bz2
# tar jxvf linux-2.6.25.6.tar.bz2
# cd linux-2.6.22.6
```

Configurando a arquitetura na qual o kernel será compilado:
```
#  export ARCH=um
```

Configurando o kernel:

```
# make defconfig
# make menuconfig
```

É interessante habilitar a opção _**HOSTFS**_ para as máquinas virtuais terem acesso ao sistema de arquivos do hospedeiro.

```
UML-specific options
-->Host filesystem [HOSTFS]
```

Para configurar o kernel com suporte ao MIPv6 estas são as opções mínimas que devem ser setadas:

```
CONFIG_EXPERIMENTAL=y
CONFIG_SYSVIPC=y
CONFIG_PROC_FS=y
CONFIG_NET=y
CONFIG_INET=y 
CONFIG_IPV6=y
CONFIG_IPV6_MIP6=y
CONFIG_XFRM=y
CONFIG_XFRM_USER=y
CONFIG_XFRM_SUB_POLICY=y
CONFIG_INET6_XFRM_MODE_ROUTEOPTIMIZATION=y
```

Opções que o agente domiciliar e o nó móvel necessitam:

```
CONFIG_IPV6_TUNNEL=y
CONFIG_IPV6_MULTIPLE_TABLES=y

```

Opção que o nó móvel também necessita:
```
CONFIG_IPV6_SUBTREES=y
```

Para alguns indicadores de movimento do nó móvel, pode ser setado:
```
CONFIG_ARPD=y
```

Para suporte ao IPsec é necessário pelo menos:
```
CONFIG_INET6_ESP=y
```

Para utilizar IPsec nos túneis:
```
CONFIG_NET_KEY=y
CONFIG_NET_KEY_MIGRATE=y
```

No menu de configuração estas são as opções que devem ser setadas:
```
Code maturity level options
--> Prompt for development and/or incomplete code/drivers [CONFIG_EXPERIMENTAL]

General setup 
--> System V IPC [CONFIG_SYSVIPC]

Networking
--> Networking support [CONFIG_NET]
--> Networking options
    --> Transformation user configuration interface [CONFIG_XFRM_USER]
    --> Transformation sub policy support [CONFIG_XFRM_SUB_POLICY]
    --> Transformation migrate database [CONFIG_XFRM_MIGRATE]
    --> PF_KEY sockets [CONFIG_NET_KEY]
    --> PF_KEY MIGRATE [CONFIG_NET_KEY_MIGRATE]
    --> TCP/IP networking [CONFIG_INET]
    --> The IPv6 protocol [CONFIG_IPV6]
    --> IPv6: AH transformation [CONFIG_INET6_AH]
    --> IPv6: ESP transformation [CONFIG_INET6_ESP]
    --> IPv6: IPComp transformation [CONFIG_INET6_IPCOMP]
    --> IPv6: Mobility [CONFIG_IPV6_MIP6]
    --> IPv6: IPsec transport mode [CONFIG_INET6_XFRM_MODE_TRANSPORT]
    --> IPv6: IPsec tunnel mode [CONFIG_INET6_XFRM_MODE_TUNNEL]
    --> IPv6: MIPv6 route optimization mode [CONFIG_INET6_XFRM_MODE_ROUTEOPTIMIZATION]
    --> IPv6: IPv6-in-IPv6 tunnel [CONFIG_IPV6_TUNNEL]
    --> IPv6: Multiple Routing Tables [CONFIG_IPV6_MULTIPLE_TABLES]
    --> IPv6: source address based routing [CONFIG_IPV6_SUBTREES]
File systems
--> Pseudo filesystems
    --> /proc file system support [CONFIG_PROC_FS]
```

Para verificar se o kernel está configurado corretamente para MIPv6 existe o shell script _**chkconf\_kernel.sh**_ que acompanha o pacote do MIPL.

Após a configuração o kernel deve ser compilado, os seus módulos devem ser copiados para o sistema de arquivos usado pela máquina virtual, por isso serão instalados no diretório uml-modules para depois serem copiados.

```
# make
# strip linux
# make modules_install INSTALL_MOD_PATH=uml-modules
# unset arch
```

Para copiar os módulos do kernel para o sistema de arquivos:

```
# mkdir loop
# mount -o loop DebianFS loop
# cp -rf linux-2.6.25.6/uml-modules/lib/modules/* loop/lib/modules
```

É preciso copiar os arquivos fontes do kernel para o sistema de arquivos, pois o MIPL precisa para a sua compilação.
```
# cp -r linux-2.6.25.6.tar.bz2 loop/usr/src/
# cd loop/usr/src
# tar jxvf linux-2.6.25.6.tar.bz2
# ln -s linux-2.6.25.6 linux
# rm linux-2.6.25.6.tar.bz2
```

## Preparando o sistema de arquivos ##

Alguns pacotes de desenvolvimento, bibliotecas deverão ser instalados no sistema de arquivos, pois os programas MIPL e RADVD serão compilados na máquina virtual. E para auxiliar na analise dos cenários serão instaladas algumas ferramentas de rede.

Para poder fazer o download dos pacotes é necessário a criação de uma rede entre o hospedeiro e a máquina virtual. Neste caso deve-se criar uma interface virtual no hospedeiro, o módulo _**tun**_ deve estar carregado no sistema.
```
# modprobe tun
# tunctl -u <nome do usuário>
```

Configurando a interface virtual do hospedeiro, habilitando o roteamento de pacotes e o _nat_ para os pacotes da máquina virtual:
```
# ifconfig tap0 192.168.1.1 netmask 255.255.255.0
# echo 1 > /proc/sys/net/ipv4/ip_forward
# iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
# iptables -I FORWARD -i tap0 -j ACCEPT
# iptables -I FORWARD -o tap0 -j ACCEPT
```

Iniciando a máquina virtual:
```
# ./linux ubda=DebianFS mem=128M eth0=tuntap,tap0
```

Configurando a rede na maquina virtual:
```
uml# ifconfig eth0 192.168.1.2 netmask 255.255.255.0
uml# route add default gw 192.168.1.1
uml# echo nameserver <seu_dns> > /etc/resolv.conf
```

Na distribuição Linux Debian há um gerenciador de pacotes o _**apt**_ que facilita a instalação de programas, ele permite procurar e instalar pacotes. Para utilizar é necessário configurar um repositório de pacotes:
```
uml# echo deb http://security.debian.org/ etch/updates main contrib > /etc/apt/source.list
```

Os seguinte pacotes serão instalados:
  * gcc-4.1
  * libc6-dev
  * make
  * libtool
  * automake1.9
  * autoconf2.13
  * bison
  * telnet
  * indent
  * flex
  * patch

```
uml# apt-get update
uml# apt-get install gcc-4.1 libc6-dev make libtool automake1.9 autoconf2.13 bison telnet indent flex patch
```

## MIPL ##

O MIPL (_Mobile IPv6 for Linux_) é uma implementação de Suporte a Mobilidade no IPv6 (RFC 3775), desenvolvida na Universidade Tecnólogica de Helsinki. É um programa em _user space_ que trabalha junto com MIPv6 habilitado no kernel.

Faça o download da ultima versão estável do MIPL:
```
uml# wget ftp://ftp.linux-ipv6.org/pub/usagi/patch/mipv6/umip-0.4/daemon/tarball/mipv6-daemon-umip-0.4.tar.gz
uml# tar xvzf mipv6-daemon-umip-0.4.tar.gz && cd mipv6-daemon-umip-0.4
```

```
uml# CPPFLAGS='-isystem /usr/src/linux/include' ./configure --enable-vt 
```

Compilando e instalando:
```
uml# make
uml# make install
```

## RADVD ##

O RADVD (_Router ADVertisement Daemon_)é um serviço para roteadores IPv6. Ele envia mensagens _Router Advertisement_ e responde a _Router Solicitation_ especificadas na RFC 2641. Essas mensagens são utilizadas para _stateless autoconfiguration_. Faça o donwload do pacote:
```
uml# wget http://www.litech.org/radvd/dist/radvd-1.1.tar.gz
uml# tar xvzf radvd-1.1.tar.gz && cd radvd-1.1
```

## Gerador de trafego ##

Para auxiliar nos testes do protocolo MIPv6 foi criado uma simples aplicação chamada **gen** que permite gerar um trafego do tipo UDP na rede. O programa foi desenvolvido utilizando a linguagem C e a API de _sockets_, é semelhante a ferramenta _ping_, porém utiliza as camadas de aplicação e de transporte. Com o **gen** permite-se observar a transparecia do protocolo com as camadas superiores a de rede.

O programa encontra-se disponível em http://. Para compila-lo basta:
```
# gcc -o gen gen.c
# mv gen /bin
```

Para utilizar o programa, é preciso que um nó da rede inicie o **gen** e fique esperando pacotes, desta forma:
```
mn# gen
```

E outro nó deve iniciar o **gen** passando como parâmetro o endereço IP do outro nó, desta forma:
```
cn# gen 2000:a::1
```