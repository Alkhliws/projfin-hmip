## Introdução ##

O protocolo IPv6 Móvel (MIPv6) permite que terminais de rede se movimentem e continuem suas comunicações com outros terminais, mesmo estando em outra rede. A mobilidade é transparente as camadas de transporte e de aplicação. Sem suporte ao MIPv6 os pacotes destinados ao terminal móvel serão perdidos, quando ele não estiver em sua rede de origem.

O MIPv6 não resolve todos os problemas de mobilidade, problemas de _handover_, de acessibilidade, controle de acesso são resolvidos na camada de enlace. MIPv6 trata  problemas de mobilidade relacionados a camada de rede.

## Terminologia do MIPv6 ##

Alguns termos definidos pelo MIPv6:

**Nó móvel**: é o terminal de rede que pode trocar seu ponto de acesso a internet sem deixar de ser alcançável via seu endereço domiciliar.

**Endereço domiciliar**: é o endereço estático associado ao nó móvel na rede domiciliar.

**Link domiciliar**: link onde o prefixo de subrede do nó movel é definido.

**Link externo**: algum link que não seja link domiciliar do nó móvel.

_**Care-of-address**_: endereço IP dado ao nó móvel quando estiver em um link externo, o prefixo da rede deste endereço é o da rede externa.

**Nó correspondente**: é o nó com quem o nó móvel esta se correspondendo, pode ser móvel ou fixo.

**Agente domiciliar**: é um roteador no link domiciliar com quem o nó móvel registra seu _care-of-address_ sempre que esta em um link externo.

**Movimento**: é uma troca de ponto de acesso na Internet.

_**Binding**_: é associação do endereço domiciliar do nó móvel com seu _care-of-address_.


## Operação do MIPv6 ##

Um nó móvel sempre está acessível pelo seu endereço domiciliar e enquanto estiver na sua rede domiciliar todos os pacotes destinados a ele serão roteados da forma convencional na Internet.

Quando o nó móvel se movimenta para um link externo adquire um endereço na nova rede o _care-of-address_, que é obtido pelas formas convencionais do IPv6 por _stateless autoconfiguration_ ou _stateful autoconfiguration_. Pacotes para o nó móvel poderão ser endereçados também para o _care-of-address_.

No link externo o nó móvel envia uma mensagem de _Binding Update_ para seu agente domiciliar fazer a associação de seu endereço domiciliar com o seu _care-of-address_. O agente domiciliar registra a mensagem em seu _cache_ e envia um _Binding Acknowledgement_ passando a atuar como um _proxy_ interceptando todos os pacotes com destino ao nó móvel e enviando-lhes via túnel. O nó móvel pode informar também ao nó correspondente sua localização.

A comunicação entre o nó móvel e o correspondente pode ser com tunelamento bidirecional onde todos os pacotes são encaminhados via agente domiciliar, neste caso o nó correspondente não precisa ter suporte ao MIPv6. O outro modo mais eficiente o nó móvel envia um _Biding Update_ para o nó correspondente e este pode endereçar os pacotes diretamente ao _care-of-address_, diminuindo o tráfego na rede.

```
Nó Móvel             Agente Domiciliar        Nó Correspondente(MIPv6)      Nó Correspondente
    |                          |                          |                        |
    |  Binding Update          |                          |                        |
    |------------------------->|                          |                        |
    |                          |                          |                        |
    |  Binding Acknowledgment  |                          |                        |
    |<-------------------------|                          |                        |
    |                          |                          |                        |
    |  Binding Update          |                          |                        |
    |---------------------------------------------------->|                        |
    |                          |                          |                        |
    |                          |  Binding Acknowledgment  |                        |
    |<----------------------------------------------------|                        |
    |                          |                          |                        |
    |      Pacotes (via túnel) |                          | Pacotes (home addres)  | 
    |<-------------------------|<--------------------------------------------------|
    |                          |                          |                        |
    |  Pacotes (Nó Corresp.)   |  Pacotes (via túnel)     |                        |
    |------------------------->|-------------------------------------------------->|
    |                          |                          |                        |
    |                       Pacotes                       |                        |
    |<--------------------------------------------------->|                        |
    |                          |                          |                        |
```

## Detecção de Movimentos ##

Uma das mais complicadas tarefas em mobilidade é a detecção dos movimentos do nó móvel, para realizar esta tarefa o MIPv6 utiliza-se de facilidades do protocolo IPv6 _Neighbor Discovery_, incluindo _Router Discovery_ e _Neighbor Unreachability Detection_.

Para dectetar o movimento deve-se utilizar o protocolo _Neighbor Unreachability Detection_ que é capaz de perceber que o roteador padrão não está mais alcançável, neste caso o nó deve descobrir um novo roteador padrão.

_Router Discovery_ é o protocolo que permite que nós IPv6 descubram roteadores existentes no seu enlace, através das mensagens _Router Advertisement_ e _Router Solicitation_. Um roteador IPv6 periodicamente envia mensagens de _Router Advertisement_ para todo o enlace, desta forma os nós podem configurar seu endereço de rede e roteadores padrão. O nó também pode enviar uma mensagem de _Router Solicitation_ recebendo como resposta um _Router Advertisement_.

## Considerações sobre Segurança ##

No processo de _Binding Update_ com o agente domiciliar pode se utilizar o protocolo IPsec nativo no IPv6 para tornar o processo mais seguro. A troca de mensagens pode ser feita utilizando o protocolo AH do IPsec, para isso é necessário um relacionamento prévio de segurança entre o nó móvel e o agente domiciliar, ou seja, uma chave secreta deve ser configurada nos dois nós.

No processo do registro do nó móvel com o nó correspondente torna-se impossível utilizar o IPsec para prover segurança no processo, pois o nó correspondente pode ser estar localizado em qualquer lugar na _Internet_. Para tornar o processo seguro faz-se necessário o uso de algum mecanismo global de autenticação automática. A solução proposta para esse problema é conhecida como _Return Routability Procedure_.


### Return Routability Procedure ###
Na tentativa de tornar mais seguro o registro do nó móvel com o nó correspondente o MIPv6 introduz uma solução bastante inteligente o _Return Routability Procedure_, a idéia do processo é que o nó correspondente obtenha garantias que o nó móvel seja alcançável pelo seu endereço domiciliar e o _care-of-address_. Somente assim o nó correspondente esta apto para receber _Binding Updates_.

É importante lembrar que o _Return Routability Procedure_ não é totalmente seguro. Pois, um atacante pode capturar as mensagens enviadas pelo nó correspondente e forjar mensagens de _Binding Updates_. Obviamente o mesmo atacante conseguirá fazer o mesmo ataque em uma rede IPv6 sem mobilidade. Podemos concluir que o _Return Routability Procedure_ não introduz riscos adicionais ao protocolo IPv6 básico.