# Roteiro de Apresentação — Previsão de Entrega Olist
**T326 · Ciência dos Dados · Prof. Caio Ponte · Turma 16/17**
**Formato:** Roteiro-guia para gravação · 18–20 minutos · 6 integrantes

> **Como usar este roteiro:**
> As "frases sugeridas" são pontos de partida — adapte com suas próprias palavras.
> As "âncoras técnicas" PRECISAM aparecer na fala, mas não precisam ser ditas literalmente.
> Marcações `[CORTE]`, `[ZOOM]`, `[PAUSA]` são orientações para a edição do vídeo.

---

## ─────────────────────────────────────────────
## KAIKE BRITO LEITÃO
### Slides 01–03 · Capa + Definição do Problema · ~2 min
## ─────────────────────────────────────────────

**Slide 01 — Capa**

**Tópicos-guia:**
- Apresentar brevemente a disciplina e o professor
- Nomear o projeto e o tema central
- Mencionar a equipe de forma natural

**Frase de abertura sugerida:**
> "Olá a todos. Meu nome é Kaike, e ao longo dos próximos 20 minutos, a nossa equipe vai apresentar o Projeto 2 da disciplina T326, Ciência dos Dados, do professor Caio Ponte. O projeto se chama Previsão do Tempo de Entrega Olist — um problema de regressão supervisionada usando dados reais de e-commerce brasileiro."

**Âncoras técnicas obrigatórias:**
- "regressão supervisionada"
- "dataset público Olist"
- "95.577 pedidos entregues"

`[CORTE após apresentar os nomes da equipe — pausa de 1 segundo antes de avançar o slide]`

---

**Slide 02 — Contexto Olist + Impacto de Negócio**

`[ZOOM nos três cards: SLA, Cancelamentos, Frete]`

**Tópicos-guia:**
- O que é a Olist e por que esse problema é relevante
- Os três impactos de negócio: SLA, cancelamentos, precificação de frete
- Mostrar a diferença AM vs SP como gancho geográfico

**Frase sugerida para o contexto:**
> "A Olist é o maior marketplace de departamentos do Brasil — ela conecta pequenos lojistas a canais como Mercado Livre e B2W. Com pedidos chegando a todos os 27 estados, o desafio logístico é enorme. Um pedido que sai de São Paulo chega em média em 8 dias... o mesmo pedido com destino ao Amazonas leva quase 25 dias. Isso é uma diferença de três vezes."

**Para os cards de impacto:**
> "Prever esse prazo com precisão não é só acadêmico — tem impacto direto no SLA prometido ao cliente, na taxa de cancelamento por atraso, e na precificação do frete."

`[PAUSA — deixa o gráfico respirar enquanto o ouvinte processa os três cards]`

**Âncoras técnicas:**
- "SLA — cumprimento de prazos"
- "SP: 8,1 dias · AM: 24,8 dias"
- "diferença de três vezes"

---

**Slide 03 — 8 Tabelas + Variável Alvo**

`[ZOOM na tabela — percorrer lentamente com o mouse linha por linha]`

**Tópicos-guia:**
- Apresentar as 8 tabelas e o papel de cada uma
- Destacar que `orders` é o hub central
- Definir claramente a variável alvo

**Frase sugerida para a variável alvo:**
> "Nossa variável alvo é simples: `dias_entrega` — que é simplesmente a diferença em dias entre a data de compra e a data em que o pedido chegou na casa do cliente. Cada linha do nosso dataset representa um pedido entregue, e é exatamente esse tempo que queremos prever."

**Para a tabela de tabelas:**
> "Usamos oito tabelas. A `orders` é o coração do sistema — é onde estão os timestamps de compra e entrega. A `geolocation` é a mais estratégica: ela nos dá as coordenadas de latitude e longitude por CEP, e foi ela que nos permitiu calcular a distância real entre o cliente e o vendedor."

`[DESTACAR a linha 'geolocation' na tabela]`

**Âncoras técnicas:**
- "variável alvo contínua"
- "8 tabelas integradas por order_id"
- "`orders` como hub central"

**Frase de passagem para o Enrico:**
> "Com o problema definido e as tabelas mapeadas, a próxima etapa foi transformar esses dados brutos em variáveis que o modelo consegue aprender. Vou passar para o Enrico, que vai explicar como construímos essas 38 features."

`[CORTE — troca de apresentador]`

---

## ─────────────────────────────────────────────
## LUIZ TILLIO
### Slides 04–05 · Feature Engineering + Haversine + Limpeza · ~2 min
## ─────────────────────────────────────────────

**Slide 04 — 38 Features em 6 Domínios**

`[ZOOM nos cards coloridos dos 6 domínios]`

**Tópicos-guia:**
- Apresentar os 6 domínios e a lógica por trás de cada grupo
- Destacar as features geográficas como o maior diferencial
- Mencionar que features com ★ são novas vs. baseline

**Frase sugerida:**
> "Construímos 38 variáveis a partir das 8 tabelas, organizadas em seis domínios. As temporais capturam quando a compra foi feita. As financeiras capturam o valor e o frete. As de produto capturam o peso e o volume. Mas o domínio que mais trouxe resultado foi o geográfico — criamos 13 features novas que transformaram a forma como o modelo enxerga o espaço."

**Para a feature `mesma_uf`:**
> "A mais simples e ao mesmo tempo a mais poderosa: `mesma_uf`. Uma flag binária — zero ou um — que diz se o cliente e o vendedor estão no mesmo estado. Pedidos intra-estaduais chegam em média em 7,3 dias. Interestaduais: 14 dias. Uma diferença de quase 100%."

`[DESTACAR o domínio Geográficas no slide]`

**Âncoras técnicas:**
- "38 features em 6 domínios"
- "`mesma_uf`: 7,3d intra-UF vs 14,1d inter-UF"
- "features geográficas como maior diferencial"

---

**Slide 05 — Feature Haversine + Filtragem**

`[ZOOM na fórmula de Haversine]`

**Tópicos-guia:**
- Explicar intuitivamente a fórmula de Haversine
- Por que não usar distância euclidiana em graus
- As 4 regras de limpeza e a justificativa do P99

**Para a Haversine — intuição primeiro, matemática depois:**
> "Para calcular a distância real entre o cliente e o vendedor, usamos a fórmula de Haversine. A intuição é simples: a Terra é redonda, então a distância em linha reta num mapa plano não é a mesma que a distância real. Usando latitude e longitude em graus, o erro euclidiano pode chegar a 15% em rotas longas como SP para Manaus. A Haversine corrige isso com um custo computacional praticamente zero."

**Para a limpeza:**
> "Aplicamos quatro regras de filtragem. A mais importante: removemos entregas acima do percentil 99, que corresponde a 46 dias. Acima disso, são casos excepcionais — greves, desastres, erros de rota — que distorceriam o modelo. Perdemos menos de 1% dos dados e ganhamos muito em qualidade."

`[ZOOM na tabela de limpeza — percorrer as 4 regras]`
`[PAUSA após mostrar o resultado: 95.577 pedidos no rodapé]`

**Âncoras técnicas:**
- "fórmula de Haversine"
- "erro euclidiano de até 15%"
- "P99 = 46 dias"
- "menos de 1% dos dados removidos"

**Frase de passagem para o Mario:**
> "Agora temos nossas 38 features calculadas e o dataset limpo. Mas antes de entrar nos modelos, precisamos preparar os dados: tratar os valores ausentes, normalizar as escalas e codificar as variáveis categóricas. Isso é a parte do Mario."

`[CORTE — troca de apresentador]`

---

## ─────────────────────────────────────────────
## MARIO
### Slides 06–07 · Nulos + Scaler + Encoding + Pipeline · ~2 min
## ─────────────────────────────────────────────

**Slide 06 — KNNImputer · StandardScaler · OrdinalEncoder**

`[ZOOM nos três cards lado a lado]`

**Tópicos-guia:**
- Explicar cada técnica com uma analogia quando possível
- Destacar a diferença KNN vs SimpleImputer
- Deixar clara a escolha OrdinalEncoder vs OHE

**Para o KNNImputer:**
> "Para os valores ausentes, usamos o KNNImputer com cinco vizinhos. A ideia é simples: se um produto tem o peso cadastrado mas o volume está faltando, a gente imputa o volume com base em produtos com peso similar. É muito mais inteligente do que usar a mediana global, que ignora completamente a relação entre as variáveis."

**Para o StandardScaler:**
> "O StandardScaler transforma cada feature para ter média zero e desvio padrão um. Isso é essencial para o Ridge — que usa regularização L2, sensível à escala. Para as árvores e o XGBoost não faz diferença, mas aplicamos universalmente para manter o pipeline consistente."

**Para o OrdinalEncoder:**
> "Para as variáveis categóricas — como o estado do cliente e a categoria do produto — usamos o OrdinalEncoder. A alternativa seria o OneHotEncoder, mas com 71 categorias de produto, ele geraria 71 colunas extras. O OrdinalEncoder mantém tudo em uma coluna, e os modelos de árvore lidam muito bem com isso."

`[DESTACAR o box 'ColumnTransformer' no rodapé do slide]`

**Âncoras técnicas:**
- "KNNImputer com 5 vizinhos"
- "OrdinalEncoder vs OneHotEncoder"
- "71 categorias de produto"
- "ColumnTransformer"

---

**Slide 07 — Pipeline Completo + Split 80/20**

`[ZOOM no fluxo de pipeline — percorrer da esquerda para a direita]`

**Tópicos-guia:**
- Mostrar o fluxo completo de ponta a ponta
- Explicar o conceito de data leakage de forma simples
- Justificar o split 80/20

**Para o pipeline:**
> "Tudo isso está encapsulado dentro de um ColumnTransformer no scikit-learn. O motivo é fundamental: garantir que os parâmetros de imputação e normalização sejam estimados APENAS no conjunto de treino. Se calcularmos a mediana no dataset completo, o modelo 'vê' dados do futuro durante o treino — isso é data leakage, e invalida a avaliação."

**Para o split:**
> "Dividimos 80% para treino e 20% para teste, com seed fixo 42 para garantir reprodutibilidade. São 76 mil pedidos para treinar e 19 mil para avaliar — mais do que suficiente para os dois conjuntos serem representativos."

`[CORTE — troca de apresentador após mostrar a tabela de nulos]`

**Âncoras técnicas:**
- "data leakage"
- "parâmetros estimados APENAS no treino"
- "80% treino · 20% teste · seed 42"

**Frase de passagem para o Pedro:**
> "Com o pipeline pronto, antes de qualquer modelo, precisamos entender o que os dados nos contam. A análise exploratória é o próximo passo — e quem fez isso foi o Pedro."

`[CORTE — troca de apresentador]`

---

## ─────────────────────────────────────────────
## PEDRO CHAVES
### Slides 08–10 · EDA Completa · ~3 min
## ─────────────────────────────────────────────

**Slide 08 — Distribuição do Target + KPIs**

`[ZOOM nos 4 cards de KPI: Média 11,6d · Mediana 10d · Skew 1,43 · Std 7,81d]`

**Tópicos-guia:**
- Apresentar as estatísticas descritivas de forma intuitiva
- Explicar o skewness e o que ele implica
- Apontar a sazonalidade no boxplot mensal

**Frase sugerida:**
> "Antes de qualquer modelo, a gente sempre olha para o que queremos prever. A distribuição do `dias_entrega` tem quatro características importantes: a média é 11,6 dias, a mediana é 10 dias, o desvio padrão é 7,8, e o skewness é 1,43. Esse skewness positivo significa que a distribuição puxa para a direita — a maioria dos pedidos chega rápido, mas existe uma cauda longa de pedidos que demoram muito mais."

`[ZOOM no histograma — apontar a cauda direita]`

**Para o boxplot mensal:**
> "No boxplot ao lado, vemos que junho, julho e agosto têm prazos consistentemente menores. Janeiro e fevereiro são os meses mais lentos. Isso provavelmente reflete o efeito do Carnaval e das festas de fim de ano na cadeia logística."

`[PAUSA — deixa o boxplot na tela por 3 segundos]`

**Âncoras técnicas:**
- "skewness de 1,43 — assimetria positiva"
- "média 11,6d vs mediana 10d"
- "cauda longa nos meses de verão"

---

**Slide 09 — Correlações + Outliers**

`[ZOOM no gráfico de barras horizontais — começar pelas barras mais longas]`

**Tópicos-guia:**
- Apresentar o ranking de correlação de Pearson
- Explicar a correlação negativa de `mesma_uf`
- Conectar os outliers às escolhas de pré-processamento

**Para as correlações:**
> "Esse gráfico mostra a correlação de Pearson de cada feature com o target. Azul significa correlação positiva — quanto maior a feature, maior o prazo. Vermelho significa o oposto. As quatro features mais correlacionadas são todas geográficas: faixa de distância, média da UF, distância em km e a estimativa do prazo da própria Olist."

`[DESTACAR as barras vermelhas de mesma_uf e mesma_regiao]`

**Para as barras vermelhas:**
> "Duas features têm correlação negativa: `mesma_uf` e `mesma_regiao`. Isso faz todo o sentido — se o cliente e o vendedor estão no mesmo estado, o prazo é menor. Correlação negativa não é ruim para o modelo, é um sinal forte."

**Para os outliers:**
> "No boxplot ao lado, identificamos outliers pelo critério IQR em 9 features. O peso do produto tem quase 14 mil outliers — mas eles não foram removidos. São valores reais: produtos muito pesados têm mais variação no prazo por conta do modal logístico. Remover seria perder informação."

**Âncoras técnicas:**
- "correlação de Pearson"
- "`faixa_dist_km`: r = +0,47"
- "`mesma_uf`: correlação negativa de -0,41"
- "critério IQR · outliers não removidos"

---

**Slide 10 — Mapa Geográfico + Distância Haversine**

`[ZOOM no mapa scatter — apontar as cores vermelhas no Norte]`

**Tópicos-guia:**
- Descrever o que o olho vê no mapa antes de interpretar
- Mostrar a progressão do prazo por faixa de distância
- Reforçar o insight AM vs SP

**Para o mapa:**
> "Esse scatter mostra cada pedido no mapa do Brasil, colorido pela quantidade de dias de entrega. Verde escuro é rápido, rosa é lento. O Sudeste e o Sul concentram as entregas mais rápidas — onde estão a maioria dos vendedores. O Norte e Nordeste aparecem em rosa porque a distância até os centros de distribuição é muito maior."

`[ZOOM no gráfico de barras por faixa de distância]`

**Para as faixas de distância:**
> "O gráfico da direita torna isso quantitativo. Pedidos locais, abaixo de 100 quilômetros, chegam em média em 5,9 dias. Pedidos de 2000 quilômetros ou mais chegam em 19 dias — mais de três vezes o prazo. E essa relação é quase linear, como mostra a linha de tendência com correlação de 0,44."

**Âncoras técnicas:**
- "AM: 24,8 dias · SP: 8,1 dias"
- "dist_km: r = +0,44"
- "local <100km: 5,9d · extrema >2000km: 19,3d"

**Frase de passagem para o Ian:**
> "A análise exploratória nos deu muita clareza sobre o que importa no dado. Agora a pergunta é: qual modelo aprende melhor esses padrões? O Ian vai apresentar como treinamos e comparamos cinco abordagens diferentes."

`[CORTE — troca de apresentador]`

---

## ─────────────────────────────────────────────
## IAN SANTOS
### Slides 11–13 · Treinamento + Benchmark · ~5 min
## ─────────────────────────────────────────────

**Slide 11 — 5 Modelos + Split**

`[ZOOM na tabela de modelos — percorrer linha por linha]`

**Tópicos-guia:**
- Apresentar cada família de modelo e sua justificativa
- Enfatizar a diversidade de abordagens: linear, árvore, bagging, boosting
- Explicar o split e o KFold

**Frase sugerida:**
> "Para esse problema, implementamos cinco modelos de famílias completamente diferentes. Começamos pelo Ridge — um baseline linear com regularização L2. É o modelo mais simples, e serve como referência: se os modelos complexos não forem significativamente melhores, não vale a complexidade."

**Percorrendo os modelos:**
> "A Decision Tree é nossa referência não-linear — interpretável por regras de negócio. O Random Forest aplica bagging: treina centenas de árvores em subsets diferentes dos dados e agrega por média, reduzindo variância. O Gradient Boosting é sequencial: cada nova árvore tenta corrigir os erros da anterior. E o XGBoost é a versão otimizada do boosting, com regularização L1 e L2 nativa e tratamento eficiente de valores ausentes."

`[DESTACAR a linha XGBoost com o símbolo ★]`
`[RETAKE SE NECESSÁRIO — essa parte tem muita informação técnica]`

**Para o split:**
> "Todos os modelos foram treinados no mesmo split: 80% para treino — 76 mil pedidos — e 20% reservados para teste final. Para o ajuste de hiperparâmetros, usamos KFold com 5 dobras dentro do conjunto de treino. Isso garante que o conjunto de teste nunca contamina a busca."

**Âncoras técnicas:**
- "cinco famílias de modelos"
- "Ridge como baseline linear"
- "bagging vs boosting"
- "KFold 5 dobras"

---

**Slide 12 — RandomizedSearchCV + Hiperparâmetros**

`[ZOOM na equação n_iter × cv = 100 fits]`

**Tópicos-guia:**
- Explicar por que RandomizedSearch e não GridSearch
- Percorrer os 8 hiperparâmetros e o efeito de cada
- Mostrar o resultado do RMSE CV

**Para o RandomizedSearch:**
> "Para otimizar os hiperparâmetros do XGBoost, usamos o RandomizedSearchCV. A alternativa seria o GridSearchCV, que testa todas as combinações possíveis. O problema: com 8 hiperparâmetros e dezenas de valores cada, isso seriam milhares de treinamentos. O RandomizedSearch amostra 20 combinações aleatórias. Com 5 dobras de validação cruzada, são 100 treinamentos no total — e estudos mostram que isso é suficiente para encontrar um ótimo próximo do global."

`[ZOOM na tabela de hiperparâmetros]`

**Para os hiperparâmetros:**
> "Os mais importantes: `learning_rate` controla o quanto cada árvore contribui — menor é mais robusto mas mais lento. `max_depth` limita a complexidade de cada árvore. `subsample` e `colsample_bytree` fazem amostragem de linhas e colunas — análogo ao bagging do Random Forest, mas dentro do boosting. E `reg_alpha` e `reg_lambda` são as regularizações L1 e L2."

**Âncoras técnicas:**
- "RandomizedSearchCV"
- "n_iter=20 × cv=5 = 100 fits"
- "100× mais eficiente que GridSearchCV"
- "RMSE CV ≈ 5,55 dias"

---

**Slide 13 — Benchmark Final**

`[ZOOM nos 4 KPI cards primeiro, depois nos gráficos]`

**Tópicos-guia:**
- Apresentar os KPIs do XGBoost em destaque
- Percorrer o gráfico de barras horizontais
- Destacar a progressão dos modelos

**Para os KPIs:**
> "Os números do nosso modelo final: RMSE de 5,62 dias — isso é o erro médio quadrático. R² de 0,4806 — 48% da variância do prazo é explicada. MAE de 3,94 dias — em média, o modelo erra menos de 4 dias. E Bias de 0,014 dia — quase zero, o que mostra que o modelo não subestima nem superestima sistematicamente."

`[PAUSA — deixa os números na tela por 3 segundos]`

**Para os gráficos:**
> "Olhando o benchmark completo: Ridge, nosso baseline linear, tem RMSE de 6,16 e R² de 0,38. Decision Tree: 6,01 e 0,41. Random Forest: 5,73 e 0,46. Gradient Boosting: 5,67 e 0,47. E o XGBoost vence com 5,62 e 0,48. A progressão é clara: cada família mais complexa melhora sobre a anterior."

`[ZOOM na barra verde do XGBoost nos dois gráficos]`

**Âncoras técnicas:**
- "RMSE = 5,620 · R² = 0,4806 · MAE = 3,94"
- "progressão Ridge → XGBoost"
- "Bias = 0,014 — modelo bem calibrado"

**Frase de passagem para o Luiz:**
> "O XGBoost venceu o benchmark. Mas para confiar num modelo, não basta olhar o número — precisamos analisar os resíduos, entender o que o modelo aprendeu, e ser honestos sobre os limites. Isso fica com o Luiz."

`[CORTE — troca de apresentador]`

---

## ─────────────────────────────────────────────
## ENRICO SANTOS NAVAJAS
### Slides 14–17 · Avaliação + Feature Importance + Conclusão · ~4 min
## ─────────────────────────────────────────────

**Slide 14 — Métricas RMSE / MAE / R²**

`[ZOOM nos três cards de métricas]`

**Tópicos-guia:**
- Dar a intuição de cada métrica antes da fórmula
- Explicar por que usar RMSE em vez de MAE como principal
- Contextualizar o R²=0,48

**Para as métricas:**
> "Antes de analisar os resultados, é importante entender o que cada métrica mede. O RMSE é a raiz do erro quadrático médio — ele penaliza erros grandes mais do que erros pequenos. Numa distribuição com skewness positivo como a nossa, isso é desejado: não queremos errar muito nos pedidos que já vão demorar."

**Para o MAE:**
> "O MAE, erro médio absoluto, é mais intuitivo: em média, o modelo erra 3,94 dias para mais ou para menos. Esse é o número que você explicaria para o gestor de logística."

**Para o R²:**
> "O R² de 0,48 significa que o modelo explica 48% da variância do prazo de entrega. E os outros 52%? São fatores que não estão no nosso dataset: condições climáticas, volume de pedidos na transportadora naquele dia, trânsito, feriados regionais. Não é uma limitação do modelo — é uma limitação dos dados disponíveis."

**Âncoras técnicas:**
- "RMSE penaliza erros grandes quadraticamente"
- "MAE = 3,94 dias"
- "R² = 0,4806 — 48% da variância explicada"

---

**Slide 15 — Análise de Resíduos**

`[ZOOM nos 3 painéis]`

**Tópicos-guia:**
- Descrever o que o olho vê em cada painel
- Interpretar o Bias de 0,014
- Discutir a cauda direita nos resíduos

**Para os painéis:**
> "A análise de resíduos tem três painéis. No primeiro, Real vs Predito, os pontos se alinham razoavelmente à diagonal — se o modelo fosse perfeito, todos estariam exatamente nela. A dispersão cresce para entregas mais longas: pedidos de 30 dias têm mais incerteza do que pedidos de 10 dias, o que é esperado."

`[ZOOM no painel central: resíduos vs predito]`

**Para os resíduos:**
> "No painel central, os resíduos estão centrados em zero sem nenhum padrão sistemático. O Bias é 0,014 dia — praticamente zero. Isso significa que o modelo não subestima nem superestima o prazo de forma consistente. É um modelo bem calibrado."

`[ZOOM no histograma de resíduos]`

**Para o histograma:**
> "A distribuição dos resíduos é aproximadamente normal com uma leve cauda à direita — coerente com o skewness do próprio target. Não há nada alarmante aqui."

`[PAUSA — 2 segundos]`

**Âncoras técnicas:**
- "resíduos centrados em zero"
- "Bias = 0,014 dia"
- "dispersão maior em entregas longas"

---

**Slide 16 — Feature Importance + Melhorias**

`[ZOOM no gráfico de barras — destacar a dominância de mesma_uf]`

**Tópicos-guia:**
- Apresentar a dominância de `mesma_uf`
- Explicar por que o XGBoost venceu tecnicamente
- Apresentar o roadmap de melhorias com honestidade

**Para a feature importance:**
> "O gráfico de importância de features do XGBoost conta uma história clara. A feature `mesma_uf` tem score de 0,77 — ela sozinha contribui com 77% do Gain médio das árvores. Isso confirma o que a análise exploratória já sugeria: a fronteira estadual é o maior determinante do prazo de entrega. É um resultado geograficamente intuitivo."

`[DESTACAR as 5 features principais]`

**Para os 52%:**
> "Quero ser honesto sobre o que não conseguimos prever. Os 52% de variância não explicada refletem fatores reais: o estado do tráfego no dia da entrega, o volume de pedidos na transportadora, se havia uma greve, se o endereço era de difícil acesso. Para capturar isso, precisaríamos de dados externos que não estão no dataset."

`[ZOOM na tabela de melhorias]`

**Âncoras técnicas:**
- "`mesma_uf` score 0,77"
- "52% de variância não explicada"
- "fatores externos: clima, trânsito, volume da transportadora"

---

**Slide 17 — Conclusão Final**

`[ZOOM nos três cards: Melhor Modelo · Principal Insight · Próximos Passos]`

**Tópicos-guia:**
- Resumir em três bullets objetivos
- Reforçar o impacto prático
- Agradecer e encerrar com elegância

**Frase sugerida:**
> "Para fechar, três pontos. O melhor modelo foi o XGBoost: RMSE de 5,62 dias, R² de 0,48, e Bias de praticamente zero. O principal insight foi que a variável `mesma_uf` domina o modelo — a fronteira estadual é mais determinante do que qualquer característica do produto ou do pagamento. E como próximos passos, incorporar dados externos como clima e rotas de CD pode aumentar o R² em até 8 pontos percentuais."

**Encerramento:**
> "Esse projeto foi construído do zero: desde a engenharia das 38 features até o tuning do XGBoost com validação cruzada. Agradecemos ao professor Caio Ponte pela condução da disciplina, e ficamos à disposição para perguntas."

`[PAUSA FINAL — deixa a tela de conclusão por 5 segundos antes de encerrar a gravação]`
`[CORTE FINAL]`

**Âncoras técnicas:**
- "RMSE = 5,62 · R² = 0,48 · Bias = 0,014"
- "`mesma_uf` domina com score 0,77"
- "potencial R² +0,05 a +0,08 com dados externos"

---

## ═══════════════════════════════════════════════════
## RESPOSTAS PARA PERGUNTAS PROVÁVEIS DO PROFESSOR
## ═══════════════════════════════════════════════════

### KAIKE — sobre Definição do Problema

**P: "Por que vocês escolheram regressão e não transformar em classificação (atrasado/no prazo)?"**
> R: "A variável alvo é contínua — é um número de dias, não uma categoria. Transformar em binário (atrasado/no prazo) jogaria fora a informação sobre quanto tempo exatamente o pedido leva. Com regressão, o modelo consegue dizer 'esse pedido vai levar 15 dias', o que é mais útil operacionalmente do que 'esse pedido vai atrasar'."

**P: "Como vocês garantiram que o dataset é representativo?"**
> R: "Filtramos apenas pedidos com status 'delivered' e removemos menos de 1% dos dados por regras de limpeza. Os 95.577 pedidos cobrem pedidos de setembro de 2016 a setembro de 2018, com representação de todos os 27 estados brasileiros."

---

### ENRICO — sobre Feature Engineering e Haversine

**P: "Por que a Haversine e não a distância rodoviária real?"**
> R: "A distância rodoviária exigiria uma API externa como Google Maps, com custo e latência. A Haversine é uma boa aproximação para a distância em linha reta, e na prática ela tem correlação r=+0,44 com o target — o que mostra que captura bem a lógica logística. A limitação é reconhecida no roadmap de melhorias."

**P: "Por que o P99 de 46 dias e não P95 ou outro critério?"**
> R: "P95 corresponderia a 38 dias — seria conservador demais e descartaria entregas legítimas para estados do Norte que naturalmente demoram mais. P99 preserva 99% dos dados reais e remove apenas os casos verdadeiramente excepcionais: 880 pedidos de 95.577."

---

### MARIO — sobre Pré-processamento

**P: "O ColumnTransformer realmente evita data leakage?"**
> R: "Sim. O `fit()` do ColumnTransformer é chamado apenas nos dados de treino. Isso significa que a mediana para imputação e o μ e σ para o scaler são calculados exclusivamente com os 76.461 pedidos de treino. O `transform()` aplica esses mesmos parâmetros ao conjunto de teste sem 'ver' os dados de teste."

**P: "Por que KNNImputer para features numéricas e não SimpleImputer?"**
> R: "O SimpleImputer usa a mediana global, que ignora as relações entre variáveis. Um produto pesado com volume ausente vai receber a mediana global de volume — que provavelmente está errada. O KNNImputer encontra os 5 produtos com peso mais similar e usa o volume deles como referência. É mais caro computacionalmente, mas muito mais preciso."

---

### PEDRO — sobre EDA

**P: "A correlação de 0,47 para `faixa_dist_km` não é data leakage, já que ela foi calculada com o target?"**
> R: "A correlação de Pearson é calculada pós-construção do dataset mas pré-split. É uma medida de análise, não um parâmetro do modelo. O que configura data leakage seria usar o target para calcular um encoding e depois usar esse encoding no teste — que não é o caso aqui."

**P: "O skewness de 1,43 indica que precisariam de uma transformação logarítmica no target?"**
> R: "Testamos a transformação log no target. O ganho foi marginal — menos de 0,2 ponto de RMSE — e adiciona complexidade na interpretação (os resíduos ficam em log, não em dias). Decidimos manter o target original para preservar a interpretabilidade das métricas."

---

### IAN — sobre Treinamento

**P: "20 iterações no RandomizedSearch é suficiente para cobrir bem o espaço de hiperparâmetros?"**
> R: "Bergstra e Bengio, em 2012, demonstraram que 20 a 50 iterações aleatórias cobrem o espaço de hiperparâmetros tão bem quanto o grid completo, porque a função de perda tende a ser suave e as combinações aleatórias amostram regiões diferentes. Testamos com 30 iterações também e a diferença de RMSE foi inferior a 0,1 dia."

**P: "Por que não incluíram LightGBM ou CatBoost na comparação?"**
> R: "A escolha foi pedagógica: cobrir famílias distintas — linear, árvore, bagging, boosting. LightGBM e CatBoost são variações de boosting, como o XGBoost. Incluí-los aumentaria o tempo de treinamento sem acrescentar diversidade de abordagem ao benchmark."

---

### LUIZ — sobre Avaliação

**P: "R²=0,48 é considerado bom para esse tipo de problema?"**
> R: "Para previsão de prazo logístico sem dados de rastreamento — apenas com características do pedido no momento da compra — sim. O estado da arte em competições similares fica entre 0,45 e 0,55. O que não conseguimos capturar são fatores externos como clima e volume da transportadora, que não estão no dataset."

**P: "O Bias de 0,014 indica que o modelo está subestimando ou superestimando?"**
> R: "Um Bias positivo de 0,014 dia significa que o modelo superestima marginalmente — em média, prevê 0,014 dia a mais do que o real. É estatisticamente insignificante. Para referência, isso é equivalente a 20 minutos num horizonte de 10 dias. O modelo está muito bem calibrado."

---

## RESUMO DE TIMING

| Integrante | Slides | Tempo |
|---|---|---|
| Kaike  | 01–03 | ~2 min |
| Enrico | 04–05 | ~2 min |
| Mario  | 06–07 | ~2 min |
| Pedro  | 08–10 | ~3 min |
| Ian    | 11–13 | ~5 min |
| Luiz   | 14–17 | ~4 min |
| **Total** | **17 slides** | **~18 min** |

> **Dica de gravação:** grave cada integrante separadamente e edite em seguida.
> Use as marcações `[CORTE]` como pontos de edição.
> Deixe 1–2 segundos de silêncio no início e no fim de cada fala para facilitar o corte.
