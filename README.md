# 🌙 Simulador de Impactos Lunares — Documentação Completa

> Simulador estatístico de flashes de impacto lunar treinado com dados reais do NELIOTA/ESA — geração de eventos sintéticos, simulação temporal e validação formal.

📄 *Esta é a documentação aprofundada da metodologia. Para o resumo executivo, veja o [README](../README.md); para o código e os gráficos, o [notebook](../simulacao_lunar.ipynb).*

![Simulação de impactos lunares](../simulacao_lunar.gif)

*Um ano sintético de bombardeamento lunar em ~56 segundos: cada flash aparece na posição, brilho e duração sorteados pelo gerador estatístico — fiel aos padrões aprendidos de 295 eventos reais.*

---

## 📌 Sumário

- [Sobre o projeto](#-sobre-o-projeto)
- [Os dados: o que é o NELIOTA](#-os-dados-o-que-é-o-neliota)
- [Pergunta central](#-pergunta-central)
- [Pipeline do projeto](#-pipeline-do-projeto)
- [Principais descobertas da análise exploratória](#-principais-descobertas-da-análise-exploratória)
- [Modelagem estatística](#-modelagem-estatística)
- [O gerador sintético](#-o-gerador-sintético)
- [Engine de simulação temporal](#-engine-de-simulação-temporal)
- [Visualização científica](#-visualização-científica)
- [Validação formal](#-validação-formal)
- [Cenários hipotéticos](#-cenários-hipotéticos)
- [Destaques metodológicos](#-destaques-metodológicos)
- [Como executar](#-como-executar)
- [Estrutura do repositório](#-estrutura-do-repositório)
- [Limitações conhecidas](#-limitações-conhecidas)
- [Trabalhos futuros](#-trabalhos-futuros)
- [Tecnologias utilizadas](#-tecnologias-utilizadas)
- [Créditos e dados](#-créditos-e-dados)
- [Contato](#-contato)

---

## 🚀 Sobre o projeto

A superfície da Lua é bombardeada continuamente por meteoroides. Muitos desses impactos produzem **flashes óticos** — clarões visíveis por frações de segundo — que carregam informação estatística sobre a frequência, a energia e a distribuição espacial desses eventos.

Diferente de uma simulação puramente visual ou "artística", este projeto constrói um **simulador orientado por dados reais**: um sistema que *aprende* os padrões estatísticos de um conjunto de impactos observados e gera eventos sintéticos consistentes com essas observações — com validação estatística formal de que os dados inventados são indistinguíveis dos reais.

O projeto cobre o ciclo completo de ciência de dados aplicada:

**análise exploratória → modelagem probabilística → geração sintética → simulação temporal → visualização científica → validação formal → exploração de cenários hipotéticos**

## 🔭 Os dados: o que é o NELIOTA

O **[NELIOTA](https://neliota.astro.noa.gr/)** (NEO Lunar Impacts and Optical Transients) é um projeto de monitoramento lunar financiado pela **ESA** (Agência Espacial Europeia), cujo objetivo é determinar a distribuição de tamanho e frequência de pequenos objetos próximos à Terra (NEOs) através da detecção de flashes de impacto na superfície da Lua.

O sistema opera no **telescópio de 1,2 m de Kryoneri**, do Observatório Nacional de Atenas, com um instrumento de câmera dupla que observa simultaneamente em duas bandas fotométricas (R e I), permitindo detectar e validar automaticamente flashes causados por micrometeoroides colidindo com a superfície lunar.

Para cada evento validado, o catálogo público reporta: data e hora, coordenadas de longitude e latitude lunares, magnitudes calibradas nas bandas R e I, número de frames em cada banda, duração máxima e classificação dos especialistas. Desde 2017, o projeto acumulou centenas de horas de observação e mais de 300 eventos catalogados.

## ❓ Pergunta central

> **É possível caracterizar estatisticamente os padrões espaciais, temporais e energéticos dos impactos lunares observados pelo NELIOTA, construir um gerador sintético que preserve essas propriedades, simular sua evolução e explorar cenários hipotéticos de forma estatisticamente consistente?**

**Resposta: sim — validada formalmente por cinco testes estatísticos, todos com p > 0.05.**

## 🔬 Pipeline do projeto

| Etapa | O que foi feito | Resultado-chave |
|---|---|---|
| **1. Carregamento e limpeza** | Tratamento de placeholders, padronização de rótulos, filtro de confiabilidade | 313 → **295 eventos confirmados** |
| **2. EDA visual** | Distribuições de brilho, duração, posição e tempo | 4 padrões distintos identificados |
| **3. Modelagem estatística** | Ajuste e teste de modelos para cada variável | 4 modelos validados |
| **4. Gerador sintético** | Integração dos modelos em uma única função geradora | Eventos sintéticos estruturalmente idênticos aos reais |
| **5. Engine de simulação** | Laço de eventos discretos com relógio simulado | Contabilidade completa: 41 gerados = 41 inseridos = 41 encerrados |
| **6. Visualização científica** | Renderização quadro a quadro da Lua sendo bombardeada | Vídeo de 1.349 quadros (~56s, 24 fps) |
| **7. Validação formal** | KS, qui-quadrado, Jensen-Shannon e Wasserstein | **Aprovado em todos os testes** |
| **8. Cenários hipotéticos** | Intervenções controladas no gerador validado | 3 cenários com assinaturas isoladas confirmadas |

## 📊 Principais descobertas da análise exploratória

**Brilho (magnitude I):** distribuição em formato de sino com leve assimetria negativa (skew ≈ −0.77) — existem um pouco mais de eventos muito brilhantes do que uma curva simétrica preveria.

**Duração do flash:** fortemente assimétrica (skew ≈ 3.91)... e, na investigação, revelou-se **discreta, não contínua**: os 295 eventos assumem apenas 14 valores únicos, todos múltiplos de ≈ 0.033s — a cadência de captura das câmeras a 30 fps. A câmera só mede durações em frames inteiros (1, 2, 3...), nunca frações.

**Distribuição espacial:** impactos espalhados por toda a face visível, sem viés hemisférico (lat média ≈ 1.8°, lon média ≈ −0.01°) — consistente com bombardeamento isotrópico. O mapa de densidade revela um padrão bimodal (núcleos em lon ≈ −50° e +65°), interpretado como **artefato observacional** da geometria de apontamento do telescópio, não fenômeno físico.

**Distribuição temporal:** os impactos chegam em **rajadas** — mediana de 0.89 dias entre eventos, mas média de 10.92 dias (skew ≈ 13.9), refletindo campanhas de observação intercaladas com longos hiatos (incluindo um ano inteiro sem registros em 2024). A sazonalidade (pico em jun–ago e out) é operacional, ligada à disponibilidade de noites observáveis.

## 📐 Modelagem estatística

Cada variável recebeu a técnica mais adequada ao seu comportamento real — testada, não assumida:

| Variável | Modelo adotado | Justificativa |
|---|---|---|
| Brilho (`i_mag`) | **Skew-normal** | KS não rejeita o ajuste (p = 0.3067); normal pura é rejeitada (p = 0.0137) |
| Duração (`n_frames`) | **Bootstrap da distribuição empírica** | Variável discreta com poucos valores únicos; modelos paramétricos não são validáveis com n = 295 |
| Posição (`lat`/`lon`) | **KDE 2D** (bw = 0.3) | Preserva o padrão bimodal observado sem assumir causa física |
| Taxa temporal | **Poisson não-homogêneo** (λ mensal) | Captura a sazonalidade operacional: λ varia de 0.88 a 7.38 impactos/mês (≈ 36.9/ano) |

Modelos contínuos para a duração (log-normal, gama) foram testados e **corretamente descartados**: atribuiriam probabilidade positiva a durações fisicamente impossíveis para o instrumento.

## ⚙️ O gerador sintético

A função `gerar_impactos_sinteticos()` integra os quatro modelos em uma única "fábrica" de impactos:

1. O **número de eventos** de cada mês é sorteado de uma Poisson com o λ mensal aprendido
2. Cada evento recebe **brilho** da skew-normal ajustada
3. **Duração** reconstruída como `n_frames × 0.033s` via bootstrap — preservando a granularidade física do instrumento
4. **Posição** amostrada do KDE 2D (com clip na face visível)

A saída é um `DataFrame` com a mesma estrutura da base real, pronto para comparação direta. Em 8 anos simulados: **315 eventos sintéticos vs. 295 reais** — volumes estatisticamente equivalentes.

## ⏱️ Engine de simulação temporal

Para dar vida ao gerador, uma engine de **eventos discretos** percorre a linha do tempo em passos fixos (1 tick = 6 horas simuladas). A cada tick, ela:

1. **Insere** os impactos cuja data chegou → o flash "nasce"
2. **Mantém** acesos os flashes ainda dentro da sua vida útil (herdada de `n_frames`)
3. **Encerra** os que terminaram → o flash "morre"

A condição de parada — rodar enquanto houver eventos por inserir **ou** flashes ativos — foi refinada em duas iterações de depuração, ambas detectadas por um **teste de consistência** (gerados = inseridos = encerrados). O processo de debug está documentado no notebook: bugs encontrados por testes automáticos, não por sorte.

## 🎬 Visualização científica

A regra do jogo: **científica, não apenas ilustrativa.** Cada flash aparece:

- na **posição exata** sorteada pelo gerador (projeção ortográfica lat/lon → disco lunar)
- com **tamanho e brilho proporcionais à magnitude real** — respeitando a convenção astronômica (menor magnitude = mais brilhante)
- pelo **tempo herdado** da simulação

A única licença visual — um rastro de desvanecimento de 6 ticks após cada flash apagar, para que eventos de 1 frame não passem despercebidos — está explicitamente documentada no notebook.

**Saídas:** vídeo mp4 H.264 (1.349 quadros, 24 fps, ~56s, um ano sintético completo), painel estático de quadros-chave e GIF para visualização no GitHub.

## ✅ Validação formal

A prova final: os dados sintéticos são estatisticamente indistinguíveis dos reais? Três "réguas" complementares — testes de hipótese (KS/χ²), divergência de Jensen-Shannon e distância de Wasserstein — aplicadas a cada componente:

| Componente | Teste de hipótese | Jensen-Shannon | Wasserstein |
|---|---|---|---|
| Brilho (i_mag) | KS p = 0.2101 ✅ | 0.1687 | 0.0895 mag |
| Duração (n_frames) | χ² p = 0.8240 ✅ | 0.0745 | 0.1108 frames |
| Posição (Longitude) | KS p = 0.8303 ✅ | 0.1969 | 3.23° |
| Posição (Latitude) | KS p = 0.5880 ✅ | 0.1289 | 1.87° |
| Sazonalidade (mês) | χ² p = 0.7883 ✅ | — | — |

**Leitura:** p > 0.05 em todos os testes = sem evidência de diferença entre real e sintético. As distâncias são fisicamente desprezíveis: o "erro" médio de brilho (0.09 mag) é **menor que a incerteza de medição do próprio telescópio**, e o desvio de posição (2–3°) é mínimo num disco de 180°.

## 🧪 Cenários hipotéticos

Validado, o simulador vira um **laboratório**: alteramos as condições e observamos o que aconteceria — algo impossível de fazer com a Lua de verdade. Três cenários, sempre comparados ao caso-base com a **mesma seed** (isolando o efeito da intervenção):

| Cenário | Intervenção | Assinatura observada |
|---|---|---|
| **Taxa dobrada** | λ mensal × 2 | ≈ 78.8 impactos/ano (vs. ≈ 37 no base) |
| **Chuva de meteoros** | +15 impactos/ano em dezembro | Dezembro concentra ~37% dos eventos do ano |
| **Mais energéticos** | Magnitudes deslocadas em −1.0 | População inteira ~1 mag mais brilhante, mesma forma |

Cada intervenção produziu **exatamente a assinatura esperada, sem efeitos colaterais** nas demais variáveis — comportamento consistente de um simulador bem construído.

**Interpretação honesta:** como o modelo aprendeu a taxa de *detecções* do NELIOTA (que depende do telescópio, não só da Lua), os cenários respondem perguntas do tipo *"o que o instrumento veria se..."* — mantendo as conclusões fiéis ao que o modelo realmente sabe.

## 💡 Destaques metodológicos

- **Descoberta da natureza discreta da duração** — a investigação dos valores únicos revelou a quantização pela cadência da câmera, mudando toda a estratégia de modelagem dessa variável.
- **Respeito às premissas dos testes** — o KS é inválido para variáveis discretas com empates; duração e sazonalidade foram validadas com qui-quadrado.
- **Reprodutibilidade total** — seeds fixas (42) em todas as simulações: mesmos resultados a cada execução.
- **Cenários com a mesma seed do base** — a única diferença entre os "mundos" é a intervenção.
- **Debug guiado por testes de consistência** — a engine passou por duas iterações de correção, ambas detectadas automaticamente pela contabilidade gerados = inseridos = encerrados.
- **Limitações declaradas, não escondidas** — cada suposição do modelo está registrada no notebook junto da sua justificativa.

## ▶️ Como executar

Desenvolvido no **Google Colab** (também roda em Jupyter local, Python 3.10+).

1. Clone o repositório:
   ```bash
   git clone https://github.com/ednei-code/lunar-impact-simulator.git
   cd lunar-impact-simulator
   ```

2. Instale as dependências:
   ```bash
   pip install -r requirements.txt
   ```

3. Abra `simulacao_lunar.ipynb` e execute as células em ordem — ou, no Colab, use **Ambiente de execução → Reiniciar e executar tudo**.

4. Se necessário, ajuste o caminho do CSV na célula de carregamento para `impacto_lunar.csv`.

> 🔁 Todas as simulações usam **seed fixa (42)** — os resultados são totalmente reprodutíveis.

## 📁 Estrutura do repositório

```
lunar-impact-simulator/
├── README.md                       # resumo executivo (porta de entrada)
├── LICENSE                         # licença MIT
├── requirements.txt                # dependências do projeto
├── .gitignore
├── simulacao_lunar.ipynb           # notebook completo (passos 1–8 + conclusão)
├── impacto_lunar.csv               # catálogo de eventos do NELIOTA
├── simulacao_lunar.gif             # animação da simulação (1 ano sintético)
├── simulacao_lunar_h264.mp4        # vídeo em qualidade cheia (~56s, 24 fps)
└── docs/
    └── SOBRE_O_PROJETO.md          # este documento
```

## ⚠️ Limitações conhecidas

- **Independência entre variáveis:** brilho, duração e posição são amostrados de forma independente. Possíveis correlações reais (ex.: flashes mais brilhantes durarem mais frames) não foram modeladas — este é o refinamento futuro prioritário.
- **Viés observacional embutido:** o padrão espacial bimodal e a sazonalidade mensal refletem a geometria de apontamento e a disponibilidade operacional do telescópio de Kryoneri, não fenômenos físicos da Lua. O modelo replica esse viés **de propósito** — ele simula o que o detector vê — e essa escolha está documentada.
- **Amostra limitada:** 295 eventos confirmados restringem a complexidade dos modelos que os dados conseguem sustentar com confiança estatística.
- **Sem física de impacto:** o simulador é estatístico; não modela massa, velocidade ou energia dos meteoroides, nem formação de crateras.

## 🔮 Trabalhos futuros

- Modelagem da **estrutura conjunta** entre variáveis (cópulas ou KDE multivariado) — eliminando a suposição de independência
- **Física simplificada de crateras**: da magnitude do flash a estimativas de energia e diâmetro de cratera
- **Modelos de IA**: clusterização de regimes de impacto, redes generativas para amostragem conjunta
- **Dashboard interativo 3D** da simulação em tempo real
- Extração das funções do notebook para módulos reutilizáveis em `src/`

## 🛠️ Tecnologias utilizadas

| Categoria | Ferramentas |
|---|---|
| Linguagem | Python 3.10+ |
| Manipulação de dados | pandas, numpy |
| Estatística e modelagem | scipy (skew-normal, KDE, KS, χ², Jensen-Shannon, Wasserstein) |
| Visualização | matplotlib, seaborn, plotly |
| Vídeo | OpenCV, ffmpeg (H.264) |
| Ambiente | Google Colab / Jupyter |

## 🙏 Créditos e dados

- **Dados:** [NELIOTA — NEO Lunar Impacts and Optical Transients](https://neliota.astro.noa.gr/), projeto financiado pela **ESA** e operado pelo **Observatório Nacional de Atenas** no telescópio de 1,2 m de Kryoneri. Os dados de impactos são públicos e disponibilizados pela equipe do projeto no portal oficial.
- Este projeto é uma análise independente com fins educacionais e de portfólio, sem vínculo com a equipe NELIOTA/ESA.

## 📫 Contato

**Ednei Cunha Vicente** — Cientista de Dados

- 📧 Email: [ednei.adgpo@gmail.com](mailto:ednei.adgpo@gmail.com)
- 💼 LinkedIn: [linkedin.com/in/ednei-cunha-vicente-551b64187](https://www.linkedin.com/in/ednei-cunha-vicente-551b64187/)

Ficou com alguma dúvida, tem sugestões ou quer conversar sobre o projeto? Será um prazer trocar ideias! ⭐ Se este projeto foi útil ou interessante para você, considere deixar uma estrela no repositório.
