# 🛰️ Estação de Solo - CubeSat Ground Station V3

**Sistema de Monitoramento e Telemetria para Missões CubeSat**

Este software atua como o "Mission Control" para operações de satélites, projetado para receber dados de telemetria via rádio (LoRa32), processá-los em tempo real e exibi-los em um dashboard de alta performance.

A versão V3 conta com uma arquitetura modular, visualização de atitude 3D baseada em WebGL (Three.js) e integração robusta com bancos de dados temporais.

## 🚀 Funcionalidades

**Comunicação Serial Real-Time:** Conexão direta com antenas de solo (Baseada em LoRa32/ESP32) para recepção de pacotes brutos.

**Parser Inteligente:** Tratamento de dados, conversão de tipos e verificação de integridade (Checksum) dos pacotes de telemetria.

**Visualização 3D (Digital Twin):** Renderização da atitude do satélite em tempo real no navegador usando Three.js e WebSockets, sem dependência de plugins legados.

**Dashboard Grafana:** Interface visual completa com:
- Gráficos de Aceleração, Giroscópio e RSSI.
- Mapa GPS com rastreamento de rota.
- Status de Bateria, Temperatura e Pressão.
- Logs de sistema e diagnósticos.

**Persistência de Dados:**
- **InfluxDB:** Armazenamento temporal para gráficos históricos.
- **CSV:** Backup local automático de todos os voos na pasta data/launches.

**Automação:** Script de lançamento (launch.bat) que gerencia todo o ambiente (Docker, Python, Servidor Web).

## 📂 Arquitetura do Projeto

O código foi refatorado para uma estrutura modular, facilitando a manutenção e a escalabilidade:

```
MeuProjeto/
│
├── config/                  # Arquivos de Configuração
│   ├── config.json          # Configuração da Missão, Porta Serial e InfluxDB
│   └── influxDBtoken.txt    # Token de segurança do banco (opcional)
│
├── data/                    # Armazenamento de Dados
│   └── launches/            # Logs CSV gerados automaticamente por missão
│
├── src/                     # Código Fonte Modular
│   ├── serial_handler.py    # Gerencia a conexão com o LoRa32 e leitura de blocos
│   ├── parser.py            # Traduz os dados brutos para o formato do banco
│   ├── database.py          # Gerencia a conexão e envio para o InfluxDB
│   ├── file_manager.py      # Gerencia a escrita dos arquivos CSV
│   ├── Atitude.py           # Servidor WebSocket para o Dashboard 3D
│   └── Atitude.html         # Código fonte da visualização Three.js (para referência)
│
├── docker-compose.yml       # Orquestração dos containers (InfluxDB + Grafana)
├── launch.bat               # Script de inicialização automática (Windows)
├── main.py                  # Núcleo do sistema (Orquestrador)
└── requirements.txt         # Dependências do Python
```

## 🛠️ Pré-requisitos

- **Python 3.8+:** [Download](https://python.org)
- **Docker Desktop:** Necessário para rodar o banco de dados e o Grafana. [Download](https://docker.com)
- **Drivers CP210x:** Para reconhecer o LoRa32/ESP32 na porta USB. [Download](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)

## ⚙️ Instalação e Configuração

### Clone o Repositório:
```bash
git clone [https://github.com/seu-usuario/solografana.git](https://github.com/RhuanV/Dashboard_Cubesat_SoloGrafana.git)
cd solografana
```

### Crie o Ambiente Virtual (Recomendado):
```bash
python -m venv venv
# Ativar no Windows:
.\venv\Scripts\activate
```

### Instale as Dependências:
```bash
pip install -r requirements.txt
```

### Configure o Docker:
1. O código só funciona com o docker RODANDO no windows. Sempre inicie o Docker.
2. inicie os containers rodando `docker-compose up -d` na pasta do projeto. Isso inicia Grafana e InfluxDB.

### Configure o InfluxDB:
1. Abra o banco de dados em `http://localhost:8086/`
2. Crie um login e configure o banco de dados localmente
3. A organização deve ser "ITACUBE" e o bucket "telemetria"

### Configure o Grafana:
1. Abra o Grafana em `http://localhost:3000/`.
2. Crie login e configure o usuário
3. Conecte ao banco InfluxDB configurado anteriormente
4. Comece um dashboard e selecione a opção de importar por .Json, 
    o código está em `./GrafanaDashboard/dashboard_cubesat_v2.json`
5. Crie o dashboard com nome "mission-control-final-v3-blue"

### Configure o Sistema:
1. Abra o arquivo `config/config.json`.
2. Ajuste a porta serial (`"port": "COM3"` ou `/dev/ttyUSB0`) para corresponder à sua antena LoRa32.
3. Defina o nome da missão (`"name": "Missao_Alpha_01"`).

Essas configurações só precisam ser realizadas uma vez!


## 🛰️ Como Iniciar uma Missão
A maneira mais simples é utilizar o script de automação incluído:

1. **Conecte o LoRa32** na porta USB.
2. **Execute o arquivo** `launch.bat` (clique duplo).

O script irá automaticamente:
- Verificar se o Docker está rodando.
- Subir os containers do InfluxDB e Grafana.
- Iniciar um servidor web local para os modelos 3D.
- Abrir o Dashboard do Grafana no seu navegador em modo tela cheia.
- Iniciar o `main.py` para começar a capturar e transmitir dados.

**Nota:** Para encerrar, feche a janela do terminal do Python ou execute o `stop.bat` (se criado).

## 📊 Visualização no Grafana

O sistema já inclui um dashboard pré-configurado (**Mission Control Final V3**).

- **URL Padrão:** http://localhost:3000
- **Login:** admin / admin (padrão do Docker)
- **Visualização 3D:** O painel central utiliza HTML/JS nativo para renderizar o satélite. Certifique-se de que o script `launch.bat` rodou o servidor de assets na porta 8000.

## 🖥️ Instalação do Dashboard

Este projeto inclui um arquivo `.json` pré-configurado contendo todo o layout da Estação de Solo (CubeSat V2). Siga os passos abaixo para importá-lo no seu Grafana.

### Pré-requisitos
- Grafana instalado e rodando (geralmente em http://localhost:3000).
- Fonte de dados (ex: InfluxDB) já configurada no Grafana.

### Passo a Passo

#### **Obter o Código:**
1. No repositório, abra o arquivo `dashboard_cubesat_v2.json`.
2. Copie todo o conteúdo do arquivo (ou faça o download dele).

#### **Acessar a Área de Importação:**
1. Abra o Grafana no seu navegador.
2. No menu lateral esquerdo, clique em **Dashboards** (ícone de quatro quadrados) > **New** > **Import**.

#### **Carregar o JSON:**
- **Opção A (Upload):** Clique no botão "Upload dashboard JSON file" e selecione o arquivo baixado.
- **Opção B (Colar):** Cole o código JSON copiado na caixa de texto "Import via panel json" e clique no botão **Load**.

#### **Configurar e Confirmar:**
- **Name:** Você pode renomear o dashboard se desejar.
- **Folder:** Escolha a pasta onde ele será salvo (ex: General).
- **Select a data source:** ⚠️ **Muito Importante:** O Grafana pedirá para você mapear a fonte de dados usada no JSON. Selecione o seu banco de dados (InfluxDB) no menu suspenso.

#### **Finalizar:**
Clique em **Import**. O dashboard deve carregar imediatamente.


## 🔧 Troubleshooting (Problemas Comuns)

**Erro de Conexão Serial:** Verifique se o LoRa32 está conectado e se a porta no `config.json` está correta. Feche outros programas (como Arduino IDE) que possam estar usando a porta.

**Dados não aparecem no Gráfico:** Verifique se o filtro de "Missão" no topo do Grafana corresponde ao nome configurado no `config.json`.

**Visualização 3D travada em "CONNECTING":** Certifique-se de que o `main.py` está rodando (ele sobe o servidor WebSocket na porta 8765) e que o navegador não está bloqueando conexões locais.

* **Logo da Equipe:** No painel de cabeçalho (HTML), localize a tag `<img>` e altere o atributo `src` para o caminho da sua imagem (ex: `/public/img/logo.png`) ou cole o código Base64 gerado.
