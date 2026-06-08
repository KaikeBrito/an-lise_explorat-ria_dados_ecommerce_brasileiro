# Guia de Apresentação — Projeto 3
## Aprendizado Profundo: CNNs

**T326 - Ciência dos Dados | Professor Caio Ponte | Turma 16/17**
**Dataset:** Intel Image Classification | **Modelo:** MobileNetV2 + Transfer Learning

> Este documento segue exatamente a estrutura exigida no enunciado.
> Use-o como roteiro durante a apresentação, célula por célula no notebook.

---

## Parte 1 — Introdução e Objetivo

> **Célula do notebook:** Seção 1 (markdown de abertura + tabela de dataset)
> **Tempo sugerido:** ~3 minutos

---

### O que apresentar

**Abra o notebook na célula da Seção 1 e fale:**

---

#### 1.1 Dataset escolhido

Escolhemos o **Intel Image Classification Dataset**, publicado originalmente pela Intel para um desafio de visão computacional e disponível no Kaggle.

> 🔗 https://www.kaggle.com/datasets/puneet6060/intel-image-classification

| Característica | Valor |
|---|---|
| Total de imagens | ~25.000 |
| Resolução | 150 × 150 pixels, RGB, formato JPEG |
| Treino rotulado (`seg_train`) | ~14.034 imagens |
| Teste rotulado (`seg_test`) | ~3.000 imagens |
| Predição sem rótulo (`seg_pred`) | ~7.301 imagens |
| Número de classes | **6 categorias de cenas naturais** |

**As 6 classes do dataset:**

| Índice | Classe | O que representa |
|---|---|---|
| 0 | `buildings` | Construções e ambientes urbanos |
| 1 | `forest` | Florestas |
| 2 | `glacier` | Geleiras |
| 3 | `mountain` | Montanhas |
| 4 | `sea` | Mar e oceano |
| 5 | `street` | Ruas e avenidas |

---

#### 1.2 Problema proposto

**Tipo:** Classificação multiclasse supervisionada de imagens.

O objetivo é treinar uma CNN capaz de receber uma foto e identificar automaticamente qual das 6 cenas ela representa.

---

#### 1.3 Relevância do problema

A classificação automática de cenas tem aplicações diretas e relevantes:

- **Google Photos / iCloud** — indexação e busca automática por conteúdo visual
- **Veículos autônomos** — percepção de ambiente para tomada de decisão
- **Monitoramento ambiental** — análise de imagens de satélite e drones
- **Sistemas de busca visual** — encontrar imagens por conteúdo, sem texto

---

## Parte 2 — Processamento de Dados

> **Células do notebook:** Seção 3 (EDA) e Seção 4 (Pré-processamento)
> **Tempo sugerido:** ~3 minutos

---

### O que apresentar

**Abra o notebook nos gráficos gerados e fale:**

---

#### 2.1 Análise Exploratória — Distribuição de classes

*(Mostrar o gráfico `distribuicao_classes.png`)*

O primeiro passo foi verificar se o dataset é **balanceado**:

- Cada classe tem entre **~2.191 e ~2.512 imagens** no treino
- Cada classe tem entre **~437 e ~553 imagens** no teste
- **Conclusão:** o dataset é bem balanceado — não precisamos de técnicas de correção como Class Weights ou oversampling

---

#### 2.2 Análise Exploratória — Amostras visuais

*(Mostrar o gráfico `amostras_dataset.png`)*

Visualizamos 4 imagens aleatórias de cada classe. Isso revelou um ponto importante:

> **Glacier e Mountain são visualmente muito similares** — ambas têm rochas, neve e céu. Essa será a maior fonte de confusão do modelo, como veremos nas métricas.

---

#### 2.3 Normalização

Todos os pixels foram divididos por 255, mapeando os valores de **[0, 255] → [0, 1]**:

```python
rescale = 1.0 / 255
```

**Por quê:** o MobileNetV2 foi treinado com imagens nessa escala. Usar outra escala prejudicaria a convergência e tornaria os pesos pré-treinados ineficazes.

---

#### 2.4 Data Augmentation (aumento de dados)

Aplicamos transformações aleatórias **somente no conjunto de treino**, para aumentar artificialmente a diversidade das imagens e reduzir overfitting:

| Técnica | Parâmetro aplicado | Por que faz sentido |
|---|---|---|
| Rotação | ±20° | Câmeras nem sempre são posicionadas na vertical |
| Deslocamento horizontal | ±15% | O objeto não está sempre centralizado |
| Deslocamento vertical | ±15% | O objeto não está sempre centralizado |
| Cisalhamento (shear) | 10% | Perspectiva levemente inclinada |
| Zoom | ±15% | Variação da distância de captura |
| Espelhamento horizontal | Ativado | Simetria esquerda/direita é válida para cenas naturais |

*(Mostrar o gráfico `augmentacao.png` — original vs versões augmentadas)*

> **Importante:** augmentation **nunca** é aplicada no teste ou validação. Eles recebem apenas normalização.

---

#### 2.5 Divisão dos dados

```
seg_train  →  85% treino     → ~11.929 imagens  (com augmentation)
           →  15% validação  →  ~2.105 imagens  (só normalização)
seg_test   →  100% teste     →  ~3.000 imagens  (só normalização)
```

---

## Parte 3 — Treinamento e Avaliação

> **Células do notebook:** Seções 5, 6, 7 e 8
> **Tempo sugerido:** ~7–9 minutos

---

### 3.1 Arquitetura da CNN — Transfer Learning com MobileNetV2

> **Célula:** Seção 5 (markdown de arquitetura + `model.summary()`)

---

#### O que é Transfer Learning?

Em vez de treinar uma rede do zero — o que exigiria centenas de milhares de imagens — reutilizamos o **MobileNetV2**, um modelo já treinado no **ImageNet** (1,2 milhão de imagens, 1.000 classes).

As camadas convolucionais desse modelo já aprenderam filtros genéricos e muito úteis:

```
Camadas iniciais  →  bordas, gradientes, cores
Camadas médias    →  texturas, padrões
Camadas finais    →  formas complexas, partes de objetos
```

Esses filtros funcionam para qualquer imagem natural — incluindo as cenas do nosso dataset.

---

#### Por que MobileNetV2?

| Modelo | Parâmetros | Acurácia ImageNet | Custo computacional |
|---|---|---|---|
| **MobileNetV2** | **3,4 M** | **72,0%** | **Baixo** |
| ResNet50 | 25,6 M | 76,0% | Médio |
| VGG16 | 138 M | 71,3% | Alto |

MobileNetV2 usa **depthwise separable convolutions** — uma técnica que reduz o custo computacional em ~8× sem grande perda de acurácia. É ideal para ambientes com CPU ou GPU modesta.

---

#### Arquitetura completa do modelo

```
Entrada: imagem 150 × 150 × 3 (RGB)
         ↓
MobileNetV2  (154 camadas, pesos do ImageNet, include_top=False)
         ↓  extrai mapa de features: shape (4, 4, 1280)
GlobalAveragePooling2D  →  vetor (1280,)
         ↓
BatchNormalization       →  estabiliza distribuição
         ↓
Dense(256, relu)         →  aprende combinações não-lineares
         ↓
Dropout(0.40)            →  regularização: desativa 40% dos neurônios no treino
         ↓
Dense(6, softmax)        →  saída: probabilidade para cada uma das 6 classes
```

**Hiperparâmetros da arquitetura:**

| Parâmetro | Valor |
|---|---|
| Entrada | 150 × 150 × 3 |
| Base (MobileNetV2) | Pré-treinada no ImageNet, `include_top=False` |
| Pooling | GlobalAveragePooling2D |
| Neurônios Dense | 256 |
| Ativação interna | ReLU |
| Dropout | 0.40 (40%) |
| Neurônios de saída | 6 (uma por classe) |
| Ativação de saída | Softmax |

---

#### Função de perda e otimizador

| Configuração | Valor | Por quê |
|---|---|---|
| **Loss** | `categorical_crossentropy` | Adequada para classificação multiclasse com saída one-hot |
| **Otimizador** | `Adam` | Adaptativo, converge rápido, robusto a diferentes escalas de gradiente |
| **Métrica monitorada** | `accuracy` | Intuitiva e direta para classificação balanceada |

---

### 3.2 Técnica especial — Estratégia em 2 Fases

> **Célula:** Seção 6 (Fase 1 e Fase 2)

O treinamento foi dividido em duas fases para proteger os pesos pré-treinados e maximizar a adaptação ao domínio.

---

#### FASE 1 — Feature Extraction (base congelada)

```
Base MobileNetV2  →  CONGELADA  (pesos fixos do ImageNet)
Head customizado  →  TREINÁVEL

Learning Rate  =  1e-3  (padrão para camadas novas)
Épocas         =  até 10  (EarlyStopping controla)
```

**Por que começar com a base congelada:**
O head recém-criado tem pesos aleatórios. Se descongelarmos a base imediatamente, os gradientes caóticos do head destruiriam os pesos valiosos do ImageNet. A Fase 1 "aquece" o head primeiro.

---

#### FASE 2 — Fine-Tuning (últimas camadas descongeladas)

```
Camadas 0..99   →  CONGELADAS  (features genéricas: bordas, texturas)
Camadas 100+    →  DESCONGELADAS  (features de alto nível: formas específicas)
Head customizado →  TREINÁVEL

Learning Rate  =  1e-5  (100× menor que Fase 1)
Épocas         =  até 10  (EarlyStopping controla)
```

**Por que LR 100× menor na Fase 2:**
As camadas pré-treinadas já têm bons pesos. Um LR alto os destruiria completamente (catastrofic forgetting). Com LR=1e-5, fazemos pequenas correções para adaptar as features de alto nível ao domínio de cenas naturais, sem apagar o que foi aprendido.

---

### 3.3 Procedimentos de treinamento

> **Célula:** Seção 6 e 7

---

#### Hiperparâmetros de treinamento

| Hiperparâmetro | Valor | Justificativa |
|---|---|---|
| `IMG_SIZE` | 150 × 150 | Resolução nativa — sem perda por redimensionamento |
| `BATCH_SIZE` | 32 | Equilíbrio padrão entre velocidade e estabilidade |
| `EPOCHS_FASE1` | 10 | Treino do head; EarlyStopping corta antes se necessário |
| `EPOCHS_FASE2` | 10 | Fine-tuning conservador |
| `LR_FASE1` | 0,001 (1e-3) | Taxa padrão Adam para camadas novas |
| `LR_FASE2` | 0,00001 (1e-5) | 100× menor para não destruir pesos pré-treinados |
| `VAL_SPLIT` | 15% | Reserva de validação a partir do treino |
| `FINE_TUNE_AT` | camada 100 | Libera ~54 das últimas camadas do MobileNetV2 |
| `SEED` | 42 | Reprodutibilidade em numpy, random e TensorFlow |

---

#### Callbacks utilizados

Três callbacks garantiram um treino robusto e eficiente:

**1. EarlyStopping**
```
Monitor: val_accuracy
Patience: 5 épocas
Ação: interrompe o treino e restaura os pesos da melhor época
```
Evita desperdício de tempo em épocas que não melhoram o modelo.

**2. ReduceLROnPlateau**
```
Monitor: val_loss
Patience: 3 épocas
Fator: 0.5 (reduz LR pela metade)
LR mínimo: 1e-7
```
Quando o aprendizado estagna, o LR é reduzido automaticamente para refinar sem ultrapassar mínimos.

**3. ModelCheckpoint**
```
Monitor: val_accuracy
Salva: apenas quando melhora (save_best_only=True)
Arquivo: melhor_modelo_fase2.keras
```
Garante que o modelo avaliado seja o melhor visto durante todo o treino.

---

#### Curvas de treinamento

*(Mostrar o gráfico `curvas_treinamento.png`)*

O gráfico mostra acurácia e perda para treino e validação ao longo de todas as épocas. A linha vertical pontilhada marca a transição da Fase 1 para a Fase 2.

**O que observar:**
- Na **Fase 1**, a acurácia de validação sobe rapidamente — o head aprende depressa com features prontas
- Na **Fase 2**, há um ganho adicional mais gradual — o fine-tuning adapta as features de alto nível
- O **gap treino–validação** indica o nível de overfitting (gap < 5% = boa generalização)

---

### 3.4 Avaliação — Métricas de qualidade

> **Célula:** Seção 8 (todas as métricas e gráficos)

Toda a avaliação é feita exclusivamente no `seg_test` — 3.000 imagens que o modelo **nunca viu** durante o treino ou validação.

---

#### Acurácia geral

```python
test_loss, test_acc = best_model.evaluate(test_gen)
```

Proporção de imagens classificadas corretamente sobre o total de imagens de teste.

> Falar o valor obtido: **"O modelo atingiu X% de acurácia no conjunto de teste."**

---

#### Relatório de classificação por classe

*(Mostrar o `classification_report` no notebook)*

Para cada uma das 6 classes, o relatório exibe:

| Métrica | O que mede | Como interpretar |
|---|---|---|
| **Precision** | Qualidade das predições positivas | De tudo que o modelo disse ser "floresta", quantos eram realmente florestas? |
| **Recall** | Cobertura das classes reais | De todas as imagens de "floresta", quantas o modelo encontrou? |
| **F1-score** | Equilíbrio entre Precision e Recall | Métrica mais completa — ideal para comparar classes |
| **Support** | Quantidade de imagens por classe no teste | Contexto para interpretar os outros valores |

**Macro avg** = média simples entre as 6 classes (trata todas igualmente)

---

#### Matriz de Confusão

*(Mostrar o gráfico `matriz_confusao.png` — duas versões lado a lado)*

- **Versão absoluta (esquerda):** número exato de acertos e erros por par de classes
- **Versão normalizada (direita):** proporção — mais fácil de identificar padrões de erro

**Como ler:** a diagonal principal (azul escuro) são os acertos. Qualquer célula fora da diagonal é um erro — o modelo confundiu a linha (classe real) com a coluna (classe predita).

**Confusão esperada:** `glacier` ↔ `mountain` — ambas têm paisagens rochosas com neve e são intrinsecamente similares. Mesmo humanos frequentemente confundem essas duas categorias.

---

#### Curva ROC / AUC — por classe

*(Mostrar o gráfico `curva_roc.png`)*

A curva ROC mostra a capacidade do modelo de separar cada classe das demais (**One-vs-Rest**).

| AUC | Interpretação |
|---|---|
| 1,00 | Separação perfeita |
| 0,90 – 1,00 | Excelente |
| 0,80 – 0,90 | Bom |
| 0,50 | Equivalente a chute aleatório |

Cada classe tem sua própria curva e valor de AUC. A classe com menor AUC é a mais desafiadora para o modelo distinguir das outras.

> Falar: **"O AUC médio (macro) obtido foi de X,XXX, indicando [excelente/bom] poder discriminativo do modelo."**

---

#### Grade de predições visuais

*(Mostrar o gráfico `predicoes_amostra.png`)*

12 imagens do conjunto de teste com rótulo real e predito:
- **Verde** = acerto
- **Vermelho** = erro

Serve para tornar o resultado concreto e intuitivo para a audiência.

---

#### Resumo executivo — métricas finais

*(Mostrar a célula da Seção 9 com o print do resumo)*

```
Dataset         : Intel Image Classification (Kaggle)
Modelo          : MobileNetV2 + Transfer Learning
Imagens treino  : ~11.929 (+~2.105 validação)
Imagens teste   : ~3.000
Classes (6)     : buildings, forest, glacier, mountain, sea, street
------
Acurácia        : XX.XX%
F1-Score macro  : X.XXXX
Precisão macro  : X.XXXX
Recall macro    : X.XXXX
```

---

## Conclusão

> **Célula do notebook:** Seção 10
> **Tempo sugerido:** ~1 minuto

---

**Resumo do que foi feito:**

1. Escolhemos o **Intel Image Classification Dataset** — 6 classes de cenas naturais, ~17k imagens rotuladas, 150×150 px
2. Realizamos **EDA** — confirmamos que o dataset é balanceado e identificamos as classes mais similares (glacier × mountain)
3. Aplicamos **pré-processamento** — normalização [0,1] e Data Augmentation (5 técnicas de transformação)
4. Construímos a **arquitetura** — MobileNetV2 pré-treinada + GlobalAveragePooling + Dense(256) + Dropout(0.4) + Softmax(6)
5. Treinamos em **2 fases** — Feature Extraction (LR=1e-3) seguida de Fine-Tuning (LR=1e-5)
6. **Avaliamos** com acurácia, F1-score, precisão, recall, matriz de confusão (absoluta + normalizada) e curva ROC/AUC por classe

**Principais conclusões:**

- Transfer Learning foi essencial — 14k imagens seriam insuficientes para treinar do zero com boa acurácia
- Fine-Tuning com LR reduzido adicionou ganho real sem destruir os pesos do ImageNet
- Data Augmentation controlou o overfitting (gap treino–validação baixo)
- Glacier e Mountain são intrinsecamente difíceis de distinguir — mesmo para humanos

---

## Checklist de apresentação

Use antes de começar para verificar se está tudo pronto:

- [ ] Notebook executado do início ao fim (todos os outputs visíveis)
- [ ] Gráficos gerados: `distribuicao_classes.png`, `amostras_dataset.png`, `augmentacao.png`
- [ ] Gráficos gerados: `curvas_treinamento.png`, `matriz_confusao.png`, `curva_roc.png`, `predicoes_amostra.png`
- [ ] Modelo salvo: `melhor_modelo_fase2.keras`
- [ ] Seção 9 executada com o resumo de métricas impresso
- [ ] Todos os membros sabem qual parte vão apresentar
- [ ] Tempo total estimado: 10–13 minutos

---

## Divisão sugerida por membro (equipe de 5)

| Membro | Parte | Tempo |
|---|---|---|
| Membro 1 | Introdução e Dataset (Seções 1 e 2 do notebook) | ~2 min |
| Membro 2 | EDA + Pré-processamento (Seções 3 e 4) | ~3 min |
| Membro 3 | Arquitetura + Transfer Learning (Seção 5) | ~3 min |
| Membro 4 | Treinamento + Callbacks + Curvas (Seções 6 e 7) | ~3 min |
| Membro 5 | Métricas + Conclusão (Seções 8, 9 e 10) | ~3 min |

> Total: ~14 minutos (dentro do limite de 10–15 minutos)

---

*Projeto 3 — T326 Ciência dos Dados | Professor Caio Ponte | Turma 16/17*