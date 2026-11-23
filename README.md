#  Uma Proposta de Interação Remota com Robôs Móveis (ROS 2 + micro-ROS + ESP32)

> Pesquisa de Iniciação Tecnológica (**PIBIT** – Edital PROPPG nº 02/2025) vinculada ao projeto *"Robótica Autônoma Móvel em Nuvens Computacionais"*[cite: 39, 40].  
> **Instituição:** Universidade Tecnológica Federal do Paraná (UTFPR) – Câmpus Apucarana  
> **Autor:** Matheus Oliveira Pardinho | **Orientador:** Prof. Dr. Lucio Agostinho Rocha

---

## Sobre o Projeto:

Este repositório documenta a arquitetura, os experimentos didáticos e o firmware desenvolvidos para viabilizar a comunicação em tempo real entre microcontroladores de baixo custo e o ecossistema **Robot Operating System 2 (ROS 2 Jazzy Jalisco)**[cite: 42, 46].

O objetivo é estabelecer uma infraestrutura replicável e determinística baseada na arquitetura **Client-Agent** (protocolo **Micro XRCE-DDS**), permitindo que sistemas embarcados atuem como nós nativos no grafo ROS 2 para futura integração com processamento em nuvem[cite: 42, 45].

###  Arquitetura de Comunicação:

```text
+-----------------------------------+       Serial USB        +-----------------------------------------------+
|         ESP-WROOM-32 (ESP32)      |   (/dev/ttyUSB0 @115200)|       VM Ubuntu 24.04 LTS (VirtualBox)        |
|                                   | ----------------------> |                                               |
|  - FreeRTOS + Biblioteca rclc     |    Micro XRCE-DDS       |  +-----------------------------------------+  |
|  - Nó: /micro_ros_arduino_node    |                         |  | Docker: microros/micro-ros-agent:jazzy  |  |
|  - Tópico:                        |                         |  +--------------------+--------------------+  |
|    /micro_ros_arduino_node_       |                         |                       | DDS Nativo            |
|    publisher (std_msgs/Int32)     |                         |                       v                       |
+-----------------------------------+                         |  +-----------------------------------------+  |
                                                              |  |     Grafo ROS 2 Jazzy Jalisco (Host)    |  |
                                                              |  |  - ros2 topic list / echo / rqt_graph   |  |
                                                              |  +-----------------------------------------+  |
                                                              +-----------------------------------------------+
```

---

##  Evolução Metodológica e Decisões Técnicas:

Durante o desenvolvimento experimental, gargalos de memória, ausência de suporte nativo a RTOS e falhas de repasse de portas seriais motivaram a reestruturação do ambiente inicial[cite: 42, 43]:

| Parâmetro | Abordagem Inicial | Solução Consolidada | Justificativa Técnica |
| :--- | :--- | :--- | :--- |
| **Ambiente Virtual** | WSL 2 (Ubuntu 22.04 LTS)[cite: 51] | VirtualBox (Ubuntu 24.04 LTS)[cite: 51] | Eliminação de falhas no reconhecimento e repasse direto de portas seriais USB/COM (`/dev/ttyUSB0`). |
| **Distribuição ROS** | ROS 2 Humble[cite: 51] | ROS 2 Jazzy Jalisco + micro-ROS[cite: 51] | Suporte contínuo (LTS) e alinhamento com a imagem oficial `microros/micro-ros-agent:jazzy`[cite: 42, 75]. |
| **Hardware** | NodeMCU ESP8266[cite: 51] | ESP-WROOM-32 (ESP32)[cite: 51] | Arquitetura dual-core Xtensa LX6, 520 KB SRAM e suporte nativo ao **FreeRTOS** exigido pelo middleware[cite: 43, 45]. |
| **IDE / Firmware** | PlatformIO (exploratório)[cite: 47, 51] | Arduino IDE + `micro_ros_arduino`[cite: 51] | Compilação direta e maior replicabilidade para experimentos didáticos[cite: 45, 52]. |
| **Ponte DDS** | Compilação manual no host[cite: 48] | Container Docker oficial[cite: 48] | Isolamento de dependências e inicialização rápida do Agent serial. |

---

##  Como Executar o Projeto:

### Pré-requisitos
* **Hardware:** Placa ESP32 (`ESP-WROOM-32`) + cabo USB de dados[cite: 42, 49].
* **Ambiente Linux:** Ubuntu 24.04 LTS (nativo ou VM VirtualBox com filtro USB habilitado)[cite: 46].
* **Software:** ROS 2 Jazzy Jalisco instalado, Docker Engine e Arduino IDE 2.x com pacote `esp32 by Espressif Systems` e biblioteca `micro_ros_arduino` (release `jazzy`)[cite: 47, 48, 74, 75].

### 1. Gravar o Firmware no ESP32
1. Abra o arquivo `firmware/esp32_microros_publisher/esp32_microros_publisher.ino` na Arduino IDE[cite: 74].
2. Selecione a placa **ESP32 Dev Module** e a porta serial correspondente[cite: 75].
3. Compile e faça o upload do código (`115200` baud)[cite: 48, 49].

### 2. Iniciar o micro-ROS Agent via Docker
Com o ESP32 conectado na porta `/dev/ttyUSB0` da máquina Ubuntu, execute o script ou o comando Docker abaixo[cite: 48, 75]:

```bash
sudo docker run -it --rm --net host -v /dev:/dev --privileged \
  microros/micro-ros-agent:jazzy serial --dev /dev/ttyUSB0 -b 115200 -v6
```
*(Pressione o botão **EN/RST** no ESP32 caso a sessão Client-Agent não inicie automaticamente)[cite: 50].*

### 3. Validar a Publicação no Grafo ROS 2
Em um novo terminal na VM Ubuntu, carregue o ambiente do ROS 2 e inspecione os tópicos ativos[cite: 76]:

```bash
source /opt/ros/jazzy/setup.bash
ros2 topic list
ros2 topic echo /micro_ros_arduino_node_publisher
```

---

##  Roteiros dos Experimentos Didáticos:

A pasta `/experimentos` contém o passo a passo documentado de cada etapa validada na pesquisa[cite: 49, 54]:
1. **[Experimento 01](./experimentos/01-instalacao-ros2-jazzy):** Configuração do VirtualBox, Ubuntu 24.04 LTS, instalação do ROS 2 Jazzy e validação do grafo DDS com `talker`/`listener` e `turtlesim`[cite: 48, 54, 57, 63].
2. **[Experimento 02](./experimentos/02-setup-microros-ping-pong):** Estruturação do workspace `microros_ws`, compilação com `colcon` e validação do protocolo Micro XRCE-DDS com o nó `ping_pong`[cite: 48, 64, 66, 71].
3. **[Experimento 03](./experimentos/03-integracao-esp32-docker-agent):** Integração física do ESP-WROOM-32 via Arduino IDE e ponte serial containerizada com Docker.

---

##  Documentação Científica:
* **Relatório Final PIBIT (2026):** disponível em [`docs/Relatorio_Final_PIBIT_Matheus_Pardinho.pdf`](./docs/Relatorio_Final_PIBIT_Matheus_Pardinho.pdf)[cite: 39, 40].
* **Resumo Expandido (XXXI SICITE 2026):** disponível em [`docs/Resumo_Expandido_SICITE_2026.pdf`](./docs/Resumo_Expandido_SICITE_2026.pdf)[cite: 13].
