## Definição dos Cenários ##

Após os estudos bibliográficos sobre o protocolo MIPv6 e a preparação de um ambiente de testes que permite a simulação de cenários. Iniciou-se a etapa para definir os cenários que permitiriam analisar o comportamento e testar a funcionalidade do protocolo.

Para analisar alguns mecanismos do protocolo e testar seus modos de operação, dois cenários base foram definidos. No primeiro cenário uma comunicação entre um dispositivo móvel e um dispositivo estático, no segundo comunicação entre dois dispositivos móveis. Nos dois cenários serão testados os dois modos de operação do protocolo MIPv6, o primeiro com tunelamento bidirecional e o segundo com otimização de roteamento.

## Cenário 1 ##

No primeiro cenário será apresentado o movimento do nó móvel da sua rede domiciliar para uma rede estrangeira e depois o retorno para sua rede, e durante o decorrer deste processo uma comunicação estará estabelecida entre o nó móvel e um nó correspondente. A topologia da rede pode ser observada na figura abaixo.

Este cenário mostra três redes, na rede com o prefixo 2000:a:: está presente um roteador que  oferece o serviço de agente domiciliar e o nó móvel, na rede com prefixo 2000:c:: está o nó correspondente com que o nó móvel está se comunicando. E para rede com prefixo 2000:d:: que o nó móvel se move.

Para simulação da rede foi construído um script que inicia as máquinas virtuais e as conecta nos _uml\_switches_ gerando a topologia da rede do cenário 1. Segue em anexo ao trabalho.

Segue os arquivos de configuração das máquinas virtuais: