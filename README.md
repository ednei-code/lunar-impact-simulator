# 🌙 Simulador de Impactos Lunares Baseado em Dados Observacionais

> Simulador estatístico de flashes de impacto lunar treinado com dados reais do NELIOTA/ESA — geração de eventos sintéticos, simulação temporal e validação formal.

![Simulação de impactos lunares](outputs/animations/simulacao_lunar.gif)

*Um ano sintético de bombardeamento lunar em ~56 segundos: cada flash aparece na posição, brilho e duração sorteados pelo gerador estatístico — fiel aos padrões aprendidos de 295 eventos reais.*

---

## 🚀 Sobre o projeto

A Lua é bombardeada continuamente por meteoroides, e muitos desses impactos produzem **flashes óticos** visíveis por frações de segundo. Este projeto usa o catálogo público do **[NELIOTA](https://neliota.astro.noa.gr/)** — projeto da **ESA** operado pelo Observatório Nacional de Atenas, no telescópio de 1,2 m de Kryoneri — para responder a uma pergunta:

> **É possível caracterizar estatisticamente os impactos lunares observados, construir um gerador sintético que preserve suas propriedades, simular sua evolução e explorar cenários hipotéticos de forma estatisticamente consistente?**

**Resposta: sim — validada por cinco testes formais, todos com p > 0.05.** Diferente de uma simulação puramente visual, cada elemento aqui é orientado por dados: o simulador *aprende* as distribuições reais e gera eventos sintéticos estatisticamente indistinguíveis dos originais.

> 📖 Este README é o resumo executivo. Para a documentação completa da metodologia — passo a passo, modelos, decisões e justificativas — veja **[docs/SOBRE_O_PROJETO.md](docs/SOBRE_O_PROJETO.md)**. A fonte da verdade, com código e gráficos, é o [notebook](notebooks/simulacao_lunar.ipynb).

## 🔬 O que foi feito

| Etapa | Resultado-chave |
|---|---|
| **1. Limpeza** | 313 → **295 eventos confirmados**; tratamento de placeholders e padronização de mais de 30 variações de rótulo |
| **2. EDA visual** | 4 padrões identificados: brilho quase-normal, duração **quantizada pela câmera**, espacial bimodal (artefato observacional), detecções em rajadas |
| **3. Modelagem** | Skew-normal para brilho (KS p = 0.31) · bootstrap empírico para duração · KDE 2D para posição · Poisson não-homogêneo mensal (≈ 36.9 impactos/ano) |
| **4. Gerador sintético** | `gerar_impactos_sinteticos()` integra os 4 modelos; 315 eventos sintéticos vs. 295 reais em 8 anos |
| **5. Engine temporal** | Eventos discretos (tick = 6h) com contabilidade completa: gerados = inseridos = encerrados |
| **6. Visualização** | Vídeo de 1.349 quadros (~56s): posição exata, brilho proporcional à magnitude, duração herdada dos dados |
| **7. Validação formal** | **Aprovado em todos os testes** (tabela abaixo) |
| **8. Cenários hipotéticos** | Taxa dobrada, chuva de meteoros e população mais energética — assinaturas isoladas confirmadas |

## ✅ Validação formal

Real (295 eventos) × Sintético (315 eventos), com três métricas complementares:

| Componente | Teste de hipótese | Jensen-Shannon | Wasserstein |
|---|---|---|---|
| Brilho (i_mag) | KS p = 0.2101 ✅ | 0.1687 | 0.0895 mag |
| Duração (n_frames) | χ² p = 0.8240 ✅ | 0.0745 | 0.1108 frames |
| Posição (Longitude) | KS p = 0.8303 ✅ | 0.1969 | 3.23° |
| Posição (Latitude) | KS p = 0.5880 ✅ | 0.1289 | 1.87° |
| Sazonalidade (mês) | χ² p = 0.7883 ✅ | — | — |

Em nenhum teste foi possível distinguir os dados sintéticos dos reais. As distâncias são fisicamente desprezíveis: o "erro" médio de brilho (0.09 mag) é **menor que a incerteza de medição do próprio telescópio**.

## 🧪 Cenários hipotéticos

Validado, o simulador vira um laboratório — comparando cada intervenção ao caso-base com a **mesma seed**, para isolar seu efeito:

| Cenário | Intervenção | Assinatura observada |
|---|---|---|
| **Taxa dobrada** | λ mensal × 2 | ≈ 78.8 impactos/ano (vs. ≈ 37) |
| **Chuva de meteoros** | +15 impactos/ano em dezembro | Dezembro concentra ~37% dos eventos |
| **Mais energéticos** | Magnitudes deslocadas em −1.0 | População ~1 mag mais brilhante, mesma forma |

Como o modelo aprende as *detecções* do NELIOTA (que dependem do telescópio, não só da Lua), os cenários respondem *"o que o instrumento veria se..."* — interpretação fiel ao que o modelo realmente sabe.

## 💡 Destaques metodológicos

- **Descoberta da natureza discreta da duração** — os 295 eventos assumem só 14 valores, todos múltiplos de ≈ 0.033s (cadência da câmera a 30 fps). Modelos contínuos (log-normal, gama) foram testados e corretamente descartados: atribuiriam probabilidade a durações fisicamente impossíveis.
- **Respeito às premissas dos testes** — KS é inválido para discretas com empates; duração e sazonalidade foram validadas com qui-quadrado.
- **Debug guiado por testes de consistência** — a engine passou por duas iterações de correção, ambas detectadas automaticamente pela contabilidade gerados = inseridos = encerrados (documentadas no notebook).
- **Reprodutibilidade total** — seed fixa (42) em todas as simulações.
- **Limitações declaradas, não escondidas** — cada suposição está registrada junto da sua justificativa.

## ▶️ Como executar

Desenvolvido no **Google Colab** (roda também em Jupyter local, Python 3.10+):

```bash
git clone https://github.com/SEU-USUARIO/lunar-impact-simulator.git
cd lunar-impact-simulator
pip install -r requirements.txt
```

Abra `notebooks/simulacao_lunar.ipynb` e execute as células em ordem (no Colab: **Ambiente de execução → Reiniciar e executar tudo**). Se necessário, ajuste o caminho do CSV para `data/impacto_lunar.csv`.

## 📁 Estrutura do repositório

```
lunar-impact-simulator/
├── README.md                       # este resumo executivo
├── requirements.txt
├── .gitignore
├── docs/
│   └── SOBRE_O_PROJETO.md          # documentação completa da metodologia
├── notebooks/
│   └── simulacao_lunar.ipynb       # notebook completo (passos 1–8 + conclusão)
├── data/
│   └── impacto_lunar.csv           # catálogo de eventos do NELIOTA
└── outputs/
    ├── videos/
    │   └── simulacao_lunar_h264.mp4
    └── animations/
        └── simulacao_lunar.gif
```

## ⚠️ Limitações e trabalhos futuros

**Limitações:** independência entre variáveis assumida (brilho × duração × posição — refinamento prioritário); viés observacional do telescópio embutido de propósito (o modelo simula o que o detector vê); amostra de 295 eventos limita a complexidade validável; sem física de impacto (massa, energia, crateras).

**Próximos passos:** estrutura conjunta entre variáveis (cópulas/KDE multivariado) · física simplificada de crateras · modelos generativos de IA · dashboard interativo 3D.

## 🛠️ Tecnologias

Python 3.10+ · pandas · numpy · scipy (skew-normal, KDE, KS, χ², Jensen-Shannon, Wasserstein) · matplotlib · seaborn · plotly · OpenCV · ffmpeg · Google Colab

## 🙏 Créditos e dados

Dados públicos do [NELIOTA — NEO Lunar Impacts and Optical Transients](https://neliota.astro.noa.gr/) (ESA / Observatório Nacional de Atenas). Análise independente, com fins educacionais e de portfólio, sem vínculo com a equipe NELIOTA/ESA.

## 📫 Contato

**Ednei Cunha Vicente** — Cientista de Dados

- 📧 Email: [ednei.adgpo@gmail.com](mailto:ednei.adgpo@gmail.com)
- 💼 LinkedIn: [linkedin.com/in/ednei-cunha-vicente-551b64187](https://www.linkedin.com/in/ednei-cunha-vicente-551b64187/)

Dúvidas, sugestões ou vontade de conversar sobre o projeto? Será um prazer! ⭐ Se o projeto foi útil ou interessante, considere deixar uma estrela.
