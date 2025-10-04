# **Plano de Ação: Projeto "SharkSight"**

**Visão Geral:** Criar uma plataforma de duas partes:
1.  **Framework Preditivo:** Um modelo que utiliza dados de satélite da NASA para gerar mapas de probabilidade de habitats de forrageamento de tubarões em tempo quase real.
2.  **Tag Conceitual "Bio-Tracker":** O design de uma nova geração de tag que não apenas rastreia a localização, mas também infere a atividade de alimentação, enviando dados para aprimorar o framework preditivo.

---

## **Tópicos a Serem Resolvidos e Ideias de Resolução**

### **Tópico 1: Identificar e Processar os Dados Relevantes**

* **Desafio:** Como encontrar, baixar e processar diferentes tipos de dados de satélite e unificá-los em um formato utilizável (mesma projeção, resolução e período de tempo).
* **Ideia de Resolução e Dados Utilizados:**
    1.  **Definir uma Região de Estudo:** Para focar os esforços, escolha uma área conhecida pela atividade de tubarões e pela presença de redemoinhos oceânicos, como o **Mar dos Sargaços / Corrente do Golfo**, conforme sugerido pelos artigos de referência.
    2.  **Dados de Tubarões (Proxy):** O desafio maior é não ter dados de rastreamento de tubarões em tempo real.
        * **Solução A (Ideal):** Encontrar datasets públicos de rastreamento de tubarões (ex: de instituições como OCEARCH ou Movebank). Esses dados (lat/long, data) serão nosso *ground truth*.
        * **Solução B (Plano B):** Digitalizar as localizações de tubarões mostradas nos gráficos dos artigos de referência (Gaube et al., 2018; Braun et al., 2019) para criar um dataset inicial de pontos de presença.
    3.  **Dados Oceanográficos (NASA):**
        * **Fitoplâncton (Base da Cadeia Alimentar):** Utilizar dados de **MODIS-Aqua** para a concentração de Clorofila-a. MODIS tem uma longa série temporal, o que é ótimo para encontrar padrões históricos. *Apesar de PACE ser mais novo, MODIS é mais robusto para um projeto de hackathon.*
        * **Redemoinhos e Correntes (Estruturas Físicas):** Utilizar dados de **SWOT**. Especificamente, dados de anomalia da altura da superfície do mar (Sea Surface Height Anomaly - SSHA). Redemoinhos de núcleo quente (anticiclônicos) têm uma SSHA positiva, e os de núcleo frio (ciclônicos) têm uma SSHA negativa. Essa é a variável chave para identificar os redemoinhos que os tubarões usam.
        * **Temperatura da Superfície do Mar (Variável Comportamental):** Utilizar dados de **MODIS** para a Temperatura da Superfície do Mar (SST - Sea Surface Temperature). A temperatura pode definir os limites do habitat de uma espécie.

### **Tópico 2: Construir o Framework Matemático Preditivo**

* **Desafio:** Como conectar os pontos entre a presença de fitoplâncton, a estrutura dos redemoinhos e a localização dos tubarões, que estão vários níveis tróficos acima?
* **Ideia de Resolução:**
    1.  **Análise Correlacional:** Para cada ponto de localização de um tubarão, extraia os valores correspondentes de Clorofila-a, SSHA e SST dos mapas de satélite da mesma data (ou semana).
    2.  **Construção do Perfil de Habitat:** Analise as distribuições. Os tubarões são encontrados com mais frequência em que faixa de SSHA (ex: dentro de redemoinhos quentes, com SSHA > 10 cm)? Em que faixa de temperatura? Em áreas com gradientes de clorofila (frentes oceânicas)? Isso criará um "perfil de forrageamento".
    3.  **Modelo de Adequabilidade de Habitat (Habitat Suitability Model):**
        * Crie uma grade sobre sua região de estudo.
        * Para cada célula da grade, atribua uma pontuação de 0 a 1 para cada variável, com base no perfil encontrado. Por exemplo, se os tubarões preferem SST entre 20-25°C, uma célula com 22°C recebe pontuação 1, e uma com 15°C recebe 0.
        * Combine as pontuações (ex: por uma média ponderada) para criar um **mapa final de probabilidade de habitat de forrageamento**. Células com pontuação alta são os "hotspots".
    4.  **Visualização:** Apresente o resultado como um mapa interativo. O usuário poderia ver as camadas de dados (SST, SSHA, Clorofila) e o mapa final de probabilidade. Ferramentas como Python com Folium, ou plataformas como o Google Earth Engine, seriam ideais para isso.

    $$
    \text{Índice de Habitat (H)} = w_1 \cdot S_{SST} + w_2 \cdot S_{SSHA} + w_3 \cdot S_{Chl-a}
    $$
    Onde $S$ é a pontuação de adequabilidade para cada variável e $w$ é o peso (importância) de cada variável.

### **Tópico 3: Projetar a Tag Conceitual "Bio-Tracker"**

* **Desafio:** Como uma tag pode medir *o que* um tubarão está comendo, e não apenas onde ele está? E como transmitir esses dados em tempo real?
* **Ideia de Resolução (Conceitual):**
    1.  **Detecção de Alimentação (O Quê?):**
        * **Sensores Inerciais Avançados:** Utilizar um acelerômetro e um giroscópio de alta frequência. A assinatura de movimento de um "ataque de predação" (uma arrancada rápida, uma torção corporal, o choque do impacto) é muito diferente da de um nado normal. Um modelo de Machine Learning embarcado (*Edge AI*) na própria tag poderia classificar os movimentos em "Trânsito", "Exploração" e "**Evento de Alimentação**".
        * **Sensores Bioquímicos (Futurista):** Propor um micro-sensor na ponta da tag que mede mudanças bruscas no pH ou detecta a presença de enzimas gástricas, que seriam ativadas durante a digestão.
        * **Micro-Câmera com IA:** Uma pequena câmera no dorso que é ativada quando o acelerômetro detecta um evento de alimentação, capturando alguns segundos de vídeo para identificação da presa.
    2.  **Transmissão de Dados (Tempo Real):**
        * **Sistema Híbrido de Comunicação:** A tag usaria o sistema de satélite **Argos** (baixa largura de banda) para transmitir dados essenciais (localização, profundidade, alertas de "Evento de Alimentação").
        * Quando o tubarão emerge, a tag tentaria se conectar a uma rede de satélites de alta largura de banda (como a Starlink) para fazer o upload de dados mais ricos (como os dados do acelerômetro ou um clipe de vídeo).
    3.  **Fonte de Energia:**
        * Painéis solares finos e flexíveis na barbatana dorsal para recarga na superfície.
        * Um pequeno gerador piezoelétrico que aproveita a energia do movimento da cauda do tubarão.
    4.  **Diagrama e Fluxo de Dados:** Crie um slide ou um vídeo curto mostrando um diagrama da tag, seus componentes, e como o dado flui do tubarão para o satélite e, finalmente, para a plataforma "SharkSight", retroalimentando e validando o modelo preditivo.

### **Tópico 4: Propor uma Nova Medição por Satélite**

* **Desafio:** Ir além dos dados existentes. O que a NASA poderia medir no futuro para facilitar esse tipo de estudo?
* **Ideia de Resolução:**
    * **Satélite de "Biologging Remoto":** Propor um conceito de satélite com um sensor lídar de alta penetração na água, projetado para detectar não apenas a topografia da superfície (como o SWOT), mas também a densidade de biomassa em diferentes profundidades (a "zona crepuscular" mencionada no artigo). Isso permitiria mapear diretamente os cardumes de presas dos quais os tubarões se alimentam, preenchendo a lacuna entre o fitoplâncton e o predador de topo. Esse satélite poderia ser chamado de **"Predator-Prey Ocean Mapper (P-POM)"**.

---

## **Sumário do Plano de Ação para o Hackathon**

1.  **Foco:** Escolher a região da Corrente do Golfo. Usar dados de rastreamento públicos (ou dos artigos) como proxy para a localização de tubarões-brancos.
2.  **Coleta de Dados:**
    * **NASA/SWOT:** Anomalia da Altura da Superfície do Mar (SSHA) para redemoinhos.
    * **NASA/MODIS:** Clorofila-a (fitoplâncton) e Temperatura da Superfície do Mar (SST).
3.  **Execução do Modelo:**
    * Alinhar todos os dados no tempo e espaço.
    * Desenvolver um modelo simples de adequabilidade de habitat que gera um mapa de "hotspots de forrageamento".
    * Visualizar o resultado em um mapa interativo.
4.  **Inovação Conceitual:**
    * Desenhar a **"Bio-Tracker Tag"**, focando na detecção de alimentação via acelerômetro e IA embarcada.
    * Criar um diagrama claro do conceito.
5.  **Apresentação:**
    * Contar uma história:
        > Dos menores plânctons às maiores correntes oceânicas, estamos usando a visão da NASA para proteger os mais importantes predadores do oceano.
    * Apresentar o mapa de hotspots como a principal ferramenta.
    * Mostrar o design da tag como o futuro da pesquisa.