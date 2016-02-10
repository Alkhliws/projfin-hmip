# Justificativas #

A mobilidade em redes IP é ainda objeto de pesquisa na comunidade científica. Quando um terminal móvel muda de subrede, ele recebe um novo endereçamento IP que pode causar sérios efeitos colaterais em conexões TCP e mesmo em comunicações com o UDP. Uma das propostas de solução para este problema é o protocolo IPv6 Móvel da IETF [RFC3775, 2004].

O protocolo IPv6 Móvel (MIPv6) permite a mobilidade na internet, que um terminal móvel migre da sua rede de origem (_"home network"_), para outras redes (_"foreign networks"_) e continuem se comunicando com outros terminais de maneira transparente as camadas superiores a IP.

No MIPv6 cada nó em sua rede de origem é identificado pelo _"home address"_ que é um endereço estático e recebe diretamente todos os pacotes destinados a ele da forma convencional de roteamento na internet.

Quando o dispositivo se move para uma rede externa ele adquire um novo endereço o _"care-of-address"_. O nó pode adquirir o _"care-of-address"_ por _"stateless autoconfiguration"_ ou _"stateful autoconfiguration"_. O nó móvel então informa ao seu _"home agent"_, um roteador da sua rede de origem, o seu novo endereço. A partir de então todos os pacotes com destino ao _"home address"_ são interceptados pelo _"home agent"_ e redirecionados via túnel ao _"care-of-address"_.

O protocolo IPv6 Móvel possui alguns problemas de desempenho no que diz respeito à mobilidade localizada (micro mobilidade) em uma rede visitada. Enquanto estiver em uma região de micro mobilidade, um nó móvel está sempre atualizando sua localização, o que acarreta vários problemas tais como: (a) tempo na detecção do movimento, (b) tempo de configuração do endereço na nova rede, (c) tempo de atualização da nova localização junto ao _"home agent"_, (d) pacotes enviados ao _"care-of-address"_ antigo são perdidos. A mudança freqüente gera muito tráfego com sinalizações e numa rede com excessivo numero de unidades móveis, a tendência é a queda na qualidade de serviço e aumento do _"delay"_ na entrega dos pacotes.

Para solucionar parte dos problemas do MIPv6 foi proposto o IP Móvel Hierárquico (HMIPv6), onde a mobilidade global (inter domínio) e a mobilidade local (intra domínio) são tratadas de formas distintas. A principal mudança foi à introdução de um novo agente, o _"Mobility Anchor Point"_ (MAP), que é um roteador que gerencia a mobilidade para minimizar o tempo de latência entre os _"handoffs"_ e perda de pacotes.

O modelo hierárquico é baseado no conceito de redes de mobilidade. Uma rede de mobilidade é um espaço de endereçamentos para nós móveis um domínio, que deve possuir um MAP. A mobilidade dos nós móveis pode ser local, no mesmo domínio, ou global quando visita um novo domínio.

Na mobilidade inter domínio quando o nó móvel visita um novo domínio, ele recebe dois CoA: o _"Private Care of Address"_ (PCoA) é o endereço referente a rede local, e o _"Virtual Care of Address"_ (VCoA) endereço associado a rede de mobilidade (domínio).

Na mobilidade intra domínio o nó móvel apenas troca o seu PCoA o seu VCoA permanece o mesmo.

Como resultado, quando um nó externo envia pacotes ao nó móvel estes são endereçados ao VCoA e enviados a rede de mobilidade, são interceptados pelo MAP e enviados via túnel ao PCoA . E quando um nó local envia pacotes ao nó móvel estes são endereçados ao PCoA e são entregues diretamente. Este modelo pode ser estendido a vários níveis de hierarquia.

Atualmente não existe uma implementação recente do HMIPv6 embora exista uma versão estável do MIPv6. O presente trabalho se enquadra dentro do contexto apresentado.