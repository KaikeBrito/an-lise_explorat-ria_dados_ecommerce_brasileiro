# Estratégias do Projeto 3 — Aprendizado Profundo: CNNs
> **T326 - Ciência dos Dados | Professor Caio Ponte | Turma 16/17**  
> Documento de engenharia reversa: o que foi feito, por que foi feito e como cada decisão cobre os requisitos do trabalho.

---

## Índice

1. [Cobertura dos requisitos do PDF](#1-cobertura-dos-requisitos-do-pdf)
2. [Dataset escolhido](#2-dataset-escolhido)
3. [Análise Exploratória (EDA)](#3-análise-exploratória-eda)
4. [Pré-processamento de dados](#4-pré-processamento-de-dados)
5. [Arquitetura da CNN](#5-arquitetura-da-cnn)
6. [Transfer Learning — estratégia em 2 fases](#6-transfer-learning--estratégia-em-2-fases)
7. [Procedimentos de treinamento](#7-procedimentos-de-treinamento)
8. [Métricas e avaliação](#8-métricas-e-avaliação)
9. [Decisões técnicas e justificativas](#9-decisões-técnicas-e-justificativas)
10. [Glossário rápido](#10-glossário-rápido)

---

## 1. Cobertura dos requisitos do PDF

A tabela abaixo mapeia cada exigência do enunciado para a seção do notebook onde ela é atendida.

| Requisito do PDF | Atendido? | Onde no notebook |
|---|:---:|---|
| Escolher dataset de imagens | ✅ | Seção 1 — Intel Image Classification |
| Classificação / regressão / detecção | ✅ | Seção 1.3 — Classificação multiclasse |
| Relevância do problema | ✅ | Seção 1.3 — aplicações reais listadas |
| Pré-processamento (resize, normalização, augmentation) | ✅ | Seção 4 completa |
| Arquitetura detalhada da CNN | ✅ | Seção 5 — `build_model()` com `model.summary()` |
| Técnica especial — Transfer Learning | ✅ | Seção 5 e 6 — MobileNetV2 + Fine-Tuning |
| Procedimentos de treinamento (epochs, otimizador, loss) | ✅ | Seção 7 — Fases 1 e 2 |
| Acurácia | ✅ | Seção 8 — `best_model.evaluate()` |
| Precisão | ✅ | Seção 8 — `classification_report` + `precision_score` |
| Recall | ✅ | Seção 8 — `classification_report` + `recall_score` |
| F1-score | ✅ | Seção 8 — `classification_report` + `f1_score` |
| Curva ROC / AUC | ✅ | Seção 8 — `plot_curva_roc()` multiclasse One-vs-Rest |
| Gráficos adequadamente rotulados | ✅ | Todos os plots têm título, eixos e legendas |
| Notebook com textos explicativos | ✅ | Células markdown em todas as seções |
| Apresentação conduzida pelo notebook | ✅ | Estrutura linear e auto-explicativa |
| Dataset diferente de "Dog vs Cat" | ✅ | Intel Image Classification |

---

## 2. Dataset escolhido

**Nome:** Intel Image Classification  
**Fonte:** [Kaggle — puneet6060](https://www.kaggle.com/datasets/puneet6060/intel-image-classification)  
**Origem:** Publicado pela Intel para um desafio de visão computacional na Analytics Vidhya.

### Características

| Atributo | Valor |
|---|---|
| Total de imagens | ~25.000 |
| Resolução | 150 × 150 pixels (RGB) |
| Formato | JPEG |
| Split de treino (`seg_train`) | ~14.034 imagens rotuladas |
| Split de teste (`seg_test`) | ~3.000 imagens rotuladas |
| Split de predição (`seg_pred`) | ~7.301 imagens sem rótulo |
| Número de classes | 6 |

### Classes e mapeamento oficial

| Índice | Nome | Descrição |
|---|---|---|
| 0 | `buildings` | Construções urbanas |
| 1 | `forest` | Florestas |
| 2 | `glacier` | Geleiras |
| 3 | `mountain` | Montanhas |
| 4 | `sea` | Mar / oceano |
| 5 | `street` | Ruas e avenidas |

> A ordem é **alfabética** e garantida pelo Keras ao ler pastas com `flow_from_directory`.  
> Corresponde exatamente ao mapeamento oficial do Kaggle.

### Estrutura de pastas no disco

```
data/intel-image-classification/
├── seg_train/
│   └── seg_train/
│       ├── buildings/   (~2.191 imgs)
│       ├── forest/      (~2.271 imgs)
│       ├── glacier/     (~2.404 imgs)
│       ├── mountain/    (~2.512 imgs)
│       ├── sea/         (~2.274 imgs)
│       └── street/      (~2.382 imgs)
├── seg_test/
│   └── seg_test/
│       ├── buildings/   (~437 imgs)
│       ├── forest/      (~474 imgs)
│       ├── glacier/     (~553 imgs)
│       ├── mountain/    (~525 imgs)
│       ├── sea/         (~510 imgs)
│       └── street/      (~501 imgs)
└── seg_pred/
    └── seg_pred/        (~7.301 imgs sem rótulo)
```

### Por que este dataset?

- **Balanceado:** ~2.200–2.500 imagens por classe — não exige técnicas de re-amostragem como Class Weights.
- **Domínio natural:** fotos de cenas do mundo real, muito próximas do ImageNet — ideal para Transfer Learning.
- **Resolução nativa compatível:** 150×150 px aceitos diretamente pelo MobileNetV2 sem redimensionamento com perda.
- **Relevância prática:** classificação de cenas é usada em Google Photos, navegação autônoma, monitoramento ambiental e busca por conteúdo visual.

---

## 3. Análise Exploratória (EDA)

Duas análises visuais foram realizadas antes de qualquer pré-processamento.

### 3.1 Distribuição de classes

**Função:** `plot_distribuicao_classes()`  
**O que faz:** Gráfico de barras duplas comparando a contagem de imagens por classe no split de treino e no split de teste.

**Por que é importante:** Confirma se o dataset é balanceado. Se uma classe tiver muito mais imagens que outra, o modelo tenderia a favorecer a maioria — o que exigiria técnicas como Class Weights ou oversampling. Neste caso, o dataset é balanceado, então nenhuma correção foi necessária.

**Conclusão do gráfico:** Dataset equilibrado → treino direto sem ajuste de pesos por classe.

### 3.2 Amostras visuais por classe

**Função:** `plot_amostras_por_classe()`  
**O que faz:** Exibe 4 imagens aleatórias de cada uma das 6 classes em uma grade visual.

**Por que é importante:** Permite identificar visualmente o grau de dificuldade do problema. Ao olhar para o grid, percebe-se que `glacier` e `mountain` são as classes mais similares visualmente — o que antecipa que serão as mais confundidas pelo modelo.

---

## 4. Pré-processamento de dados

### 4.1 Normalização

**Técnica:** divisão de todos os pixels por 255.

```python
rescale = 1.0 / 255
```

**Por que:** Os pixels têm valores originais entre 0 e 255 (uint8). Dividir por 255 mapeia para o intervalo [0, 1]. Isso é necessário porque os pesos do MobileNetV2 foram treinados com imagens nessa escala — usar outra escala prejudica a convergência e pode gerar gradientes instáveis.

### 4.2 Data Augmentation

Técnicas aplicadas **somente no conjunto de treino** (nunca no teste ou validação):

| Técnica | Parâmetro usado | Justificativa |
|---|---|---|
| Rotação | `rotation_range=20` | Cenas podem aparecer em ângulos diferentes |
| Deslocamento horizontal | `width_shift_range=0.15` | Objeto não está sempre centrado |
| Deslocamento vertical | `height_shift_range=0.15` | Objeto não está sempre centrado |
| Cisalhamento (shear) | `shear_range=0.10` | Perspectiva levemente inclinada |
| Zoom | `zoom_range=0.15` | Variação de distância da câmera |
| Espelhamento horizontal | `horizontal_flip=True` | Simetria esquerda/direita válida para cenas naturais |
| Preenchimento de borda | `fill_mode="nearest"` | Preenche pixels gerados pelas transformações com o vizinho mais próximo |

**Por que usar augmentation:** Aumenta artificialmente a diversidade do conjunto de treino sem coletar novas imagens. Isso reduz overfitting — o modelo aprende a reconhecer padrões, não a memorizar posições específicas.

**Por que NÃO aplicar no teste:** O conjunto de teste representa dados reais novos. Aplicar augmentation no teste criaria variações artificiais que distorceriam a avaliação real do modelo.

### 4.3 Splits gerados

```
Treino (85% do seg_train) → train_gen   → com augmentation + shuffle
Validação (15% do seg_train) → val_gen  → só normalização, sem shuffle
Teste (seg_test completo) → test_gen    → só normalização, sem shuffle (crítico!)
```

> `shuffle=False` no `test_gen` é crítico para garantir que `y_true` e `y_pred` estejam alinhados na avaliação.

### 4.4 Hiperparâmetros de pré-processamento

```python
IMG_SIZE    = (150, 150)   # Dimensão nativa do dataset
BATCH_SIZE  = 32           # Padrão eficiente para GPU/CPU
VAL_SPLIT   = 0.15         # 15% do treino → validação
SEED        = 42           # Reprodutibilidade global
```

---

## 5. Arquitetura da CNN

### 5.1 Visão geral

```
Input(150, 150, 3)
    ↓
MobileNetV2 (pré-treinada no ImageNet, 154 camadas)
    ↓  [include_top=False — remove o classificador original de 1.000 classes]
GlobalAveragePooling2D      → (batch, 1280)
    ↓
BatchNormalization
    ↓
Dense(256, activation='relu')
    ↓
Dropout(0.40)
    ↓
Dense(6, activation='softmax')   → probabilidade para cada uma das 6 classes
```

### 5.2 Explicação de cada camada do head customizado

**GlobalAveragePooling2D**  
Recebe o mapa de features do MobileNetV2 com shape `(batch, H, W, 1280)` e calcula a média espacial, gerando um vetor `(batch, 1280)`. É preferível ao `Flatten` em redes convolucionais porque captura o contexto global da imagem e reduz a dimensionalidade sem explodir o número de parâmetros.

**BatchNormalization**  
Normaliza as ativações entre lotes, estabilizando a distribuição dos valores e acelerando a convergência. Especialmente útil aqui porque os pesos da base MobileNetV2 já estão em uma escala calibrada.

**Dense(256, relu)**  
Camada totalmente conectada com 256 neurônios e ativação ReLU. Aprende combinações não-lineares dos 1.280 features extraídos pela base. O tamanho 256 é um equilíbrio entre capacidade de aprendizado e risco de overfitting.

**Dropout(0.40)**  
Durante o treino, desativa aleatoriamente 40% dos neurônios em cada passagem. Isso força o modelo a não depender de nenhum neurônio específico, reduzindo co-adaptação e overfitting.

**Dense(6, softmax)**  
Camada de saída com 6 neurônios (um por classe). A ativação softmax garante que as saídas somem 1, produzindo probabilidades interpretáveis. A classe predita é o índice com maior probabilidade.

### 5.3 Contagem de parâmetros

| Grupo | Parâmetros |
|---|---|
| MobileNetV2 base (congelada na Fase 1) | ~2.257.984 |
| Head customizado (treináveis na Fase 1) | ~330.502 |
| Total | ~2.588.486 |

> Na Fase 2, parte dos parâmetros da base é descongelada e também se torna treinável.

### 5.4 Função de perda e otimizador

**Loss: `categorical_crossentropy`**  
Adequada para classificação multiclasse com saída one-hot. Mede a diferença entre a distribuição de probabilidade predita e o rótulo verdadeiro.

**Otimizador: `Adam`**  
Combina momentum e taxa de aprendizado adaptativa por parâmetro. É o padrão atual para a maioria das tarefas de visão computacional por convergir rápido e ser robusto a diferentes escalas de gradiente.

---

## 6. Transfer Learning — estratégia em 2 fases

### O que é Transfer Learning?

Transfer Learning é a técnica de reaproveitar um modelo treinado em uma tarefa grande (ImageNet: 1,2 M imagens, 1.000 classes) para uma tarefa menor e relacionada.

As camadas convolucionais de um modelo treinado no ImageNet aprendem filtros hierárquicos:
- **Camadas iniciais:** detectam bordas, gradientes, cores
- **Camadas intermediárias:** detectam texturas, padrões
- **Camadas finais:** detectam formas complexas, partes de objetos

Esses filtros são **genéricos** e funcionam bem em qualquer domínio de imagens naturais — como é o caso do Intel dataset.

### Por que MobileNetV2 especificamente?

| Critério | MobileNetV2 | VGG16 | ResNet50 |
|---|---|---|---|
| Parâmetros | 3,4 M | 138 M | 25,6 M |
| Top-1 ImageNet | 72,0% | 71,3% | 76,0% |
| Custo computacional | Baixo | Alto | Médio |
| Aceita 150×150? | Nativo | Não (224) | Não (224) |

MobileNetV2 usa **depthwise separable convolutions**, que fatoram uma convolução padrão em duas operações mais baratas, reduzindo o custo em ~8× sem grande perda de acurácia.

### Fase 1 — Feature Extraction

```python
base.trainable = False   # Toda a base MobileNetV2 congelada
LR_FASE1 = 1e-3          # Taxa de aprendizado padrão para head novo
EPOCHS_FASE1 = 10
```

**O que acontece:** apenas as camadas do head customizado (Dense, BatchNorm, Dropout) são atualizadas. A base permanece com os pesos do ImageNet intactos.

**Por que começar assim:** O head recém-inicializado tem pesos aleatórios. Se descongelássemos a base imediatamente, os gradientes aleatórios do head destruiriam os pesos cuidadosamente treinados da base. A Fase 1 "aquece" o head antes de qualquer ajuste fino.

### Fase 2 — Fine-Tuning

```python
FINE_TUNE_AT = 100        # Congela camadas 0..99; libera camadas 100+
LR_FASE2 = 1e-5           # 100× menor que Fase 1
EPOCHS_FASE2 = 10
```

**O que acontece:** as últimas camadas da base MobileNetV2 (as mais próximas da saída, que detectam features de alto nível) são descongeladas e ajustadas com uma taxa de aprendizado muito menor.

**Por que LR tão pequeno:** As camadas pré-treinadas já têm bons pesos. Um LR grande os destruiria (catastrofic forgetting). Com LR=1e-5, fazemos pequenas correções para adaptar as features ao domínio específico de cenas naturais, sem apagar o que foi aprendido no ImageNet.

**Por que descongelar só as últimas camadas:** As primeiras camadas (bordas, texturas básicas) são universais — iguais em qualquer domínio. As últimas camadas (formas complexas, combinações de features) são as que mais se beneficiam da adaptação ao novo domínio.

### Acesso robusto à base no código

```python
# Busca pelo tipo — não pelo nome hardcoded
# O nome varia com a resolução: mobilenetv2_1.00_150 ou mobilenetv2_1.00_224
base_model = next(
    layer for layer in model.layers
    if isinstance(layer, tf.keras.Model) and "mobilenetv2" in layer.name
)
```

> Isso evita o `ValueError: No such layer` que ocorre quando o nome da camada é diferente do esperado.

---

## 7. Procedimentos de treinamento

### 7.1 Hiperparâmetros principais

| Hiperparâmetro | Valor | Justificativa |
|---|---|---|
| `IMG_SIZE` | (150, 150) | Resolução nativa do dataset — sem perda por resize |
| `BATCH_SIZE` | 32 | Equilíbrio padrão entre velocidade e estabilidade do gradiente |
| `EPOCHS_FASE1` | 10 | Suficiente para treinar o head; EarlyStopping corta antes se necessário |
| `EPOCHS_FASE2` | 10 | Refinamento conservador; EarlyStopping controla |
| `LR_FASE1` | 1e-3 | Taxa padrão Adam para camadas novas |
| `LR_FASE2` | 1e-5 | 100× menor para não destruir pesos pré-treinados |
| `FINE_TUNE_AT` | 100 | Libera ~54 das últimas camadas da MobileNetV2 |
| `VAL_SPLIT` | 0.15 | 15% do treino para monitoramento — padrão razoável |
| `SEED` | 42 | Reprodutibilidade em numpy, random e TensorFlow |

### 7.2 Callbacks

**EarlyStopping**
```python
EarlyStopping(
    monitor="val_accuracy",
    patience=5,
    restore_best_weights=True
)
```
Interrompe o treino se a acurácia de validação não melhorar por 5 épocas consecutivas e restaura automaticamente os pesos da melhor época.

**ReduceLROnPlateau**
```python
ReduceLROnPlateau(
    monitor="val_loss",
    factor=0.5,
    patience=3,
    min_lr=1e-7
)
```
Quando a `val_loss` estagna por 3 épocas, reduz o LR pela metade. Permite que o modelo "refine" em regiões planas da superfície de perda sem ultrapassar os mínimos.

**ModelCheckpoint**
```python
ModelCheckpoint(
    filepath="melhor_modelo_fase2.keras",
    monitor="val_accuracy",
    save_best_only=True
)
```
Salva o modelo apenas quando `val_accuracy` melhora. Garante que o modelo final avaliado seja o melhor visto durante todo o treino, não necessariamente o da última época.

### 7.3 Fluxo completo do treinamento

```
1. Construir modelo (base congelada + head aleatório)
2. FASE 1: treinar só o head por até 10 épocas (LR=1e-3)
   └── EarlyStopping monitora val_accuracy
   └── ModelCheckpoint salva melhor_modelo_fase1.keras
3. Descongelar camadas 100+ da MobileNetV2
4. Recompilar com LR=1e-5
5. FASE 2: fine-tuning por até 10 épocas
   └── EarlyStopping monitora val_accuracy
   └── ModelCheckpoint salva melhor_modelo_fase2.keras
6. Carregar melhor_modelo_fase2.keras para avaliação final
```

---

## 8. Métricas e avaliação

Toda a avaliação é feita exclusivamente no `seg_test` (3.000 imagens que o modelo **nunca viu** durante o treino).

### 8.1 Acurácia geral

```python
test_loss, test_acc = best_model.evaluate(test_gen, verbose=0)
```

Proporção de imagens classificadas corretamente sobre o total. Métrica direta, mas pode ser enganosa em datasets desbalanceados — por isso complementamos com F1-score.

### 8.2 Relatório de classificação (`classification_report`)

Exibe por classe:

| Métrica | Definição | Interpretação prática |
|---|---|---|
| **Precision** | TP / (TP + FP) | De todas as imagens classificadas como X, quantas eram realmente X? |
| **Recall** | TP / (TP + FN) | De todas as imagens reais de X, quantas o modelo acertou? |
| **F1-score** | 2 × (P × R) / (P + R) | Média harmônica entre Precision e Recall — melhor métrica única |
| **Support** | Total de imagens reais | Quantidade de imagens da classe no conjunto de teste |

> **macro avg** = média simples entre as 6 classes (trata todas igualmente).  
> **weighted avg** = média ponderada pelo support de cada classe.

### 8.3 Matriz de Confusão

Duas versões geradas lado a lado:

- **Absoluta:** mostra o número exato de acertos e erros por par de classes.
- **Normalizada:** mostra a proporção — mais útil para identificar onde o modelo erra sistematicamente.

**Como interpretar:** a diagonal principal representa acertos. Células fora da diagonal representam confusões. Confusões acima de 10% são destacadas automaticamente no código com um insight textual.

**Confusão esperada neste dataset:** `glacier` ↔ `mountain` — ambas têm cenários rochosos com neve, sendo visualmente muito similares.

### 8.4 Curva ROC / AUC

**Abordagem:** One-vs-Rest (OvR) — para cada classe, trata-se como um problema binário: "é esta classe vs todas as outras".

```python
y_b = label_binarize(y_true, classes=list(range(6)))
fpr, tpr, _ = roc_curve(y_b[:, i], y_prob[:, i])
roc_auc = auc(fpr, tpr)
```

| Valor de AUC | Interpretação |
|---|---|
| 1.0 | Separação perfeita |
| 0.9–1.0 | Excelente |
| 0.8–0.9 | Bom |
| 0.7–0.8 | Razoável |
| 0.5 | Equivalente a chute aleatório |

**O que o gráfico mostra:** cada classe tem sua própria curva e AUC. A classe com menor AUC é a mais desafiadora para o modelo separar.

### 8.5 Grade de predições visuais

```python
plot_predicoes_amostra(test_gen, y_pred, y_true, class_labels)
```

Exibe 12 imagens do conjunto de teste com o rótulo real e o predito. Acertos em verde, erros em vermelho. Serve para tornar o resultado concreto e interpretável na apresentação.

### 8.6 Resumo executivo final

Imprime no console as métricas globais consolidadas:

```
Acurácia        : XX.XX%
F1-Score macro  : X.XXXX
Precisão macro  : X.XXXX
Recall macro    : X.XXXX
```

---

## 9. Decisões técnicas e justificativas

### Por que não treinar do zero?

O dataset tem ~14.000 imagens de treino. Para treinar uma CNN do zero com boa acurácia em imagens naturais, seriam necessárias centenas de milhares de imagens. Treinar do zero com 14k imagens levaria a overfitting severo e acurácia inferior. O Transfer Learning resolve esse problema reaproveitando features já aprendidas.

### Por que não usar Class Weights?

Class Weights são usados quando o dataset é desbalanceado (ex: 90% de uma classe, 1% de outra). O Intel dataset tem ~2.200–2.500 imagens por classe — praticamente uniforme. Aplicar Class Weights num dataset balanceado não melhora e pode degradar o treino.

### Por que `GlobalAveragePooling2D` em vez de `Flatten`?

`Flatten` converteria o mapa de features `(H, W, 1280)` em um vetor gigante, criando um número explosivo de parâmetros na camada Dense seguinte e aumentando o risco de overfitting. `GlobalAveragePooling2D` calcula a média espacial, produzindo um vetor compacto de 1.280 valores que captura o contexto global da imagem.

### Por que `shuffle=False` no test generator?

O gerador de teste não deve embaralhar as imagens porque precisamos que `y_pred[i]` corresponda a `y_true[i]`. Se o gerador embaralhasse, as predições ficariam desalinhadas com os rótulos verdadeiros e todas as métricas estariam erradas.

### Por que salvar o modelo com `ModelCheckpoint`?

Ao final do treino, `restore_best_weights=True` no EarlyStopping já restaura os pesos da melhor época. O `ModelCheckpoint` garante adicionalmente que o melhor modelo seja persistido em disco — útil para carregar depois sem re-treinar, e para o caso de o kernel cair durante o treino.

### Por que usar `training=False` na chamada da base?

```python
x = base(inputs, training=False)
```

A MobileNetV2 contém camadas de BatchNormalization internas. `training=False` garante que essas camadas usem as estatísticas fixas do pré-treino (média e variância calculadas no ImageNet) em vez de recalcular a partir do batch atual. Isso é essencial na Fase 1 para manter o comportamento estável da base congelada.

---

## 10. Glossário rápido

| Termo | Definição |
|---|---|
| **CNN** | Rede Neural Convolucional — arquitetura especializada em processar dados com estrutura espacial (imagens) |
| **Transfer Learning** | Reutilização de pesos de um modelo pré-treinado em uma nova tarefa |
| **Fine-Tuning** | Ajuste fino de um modelo pré-treinado com taxa de aprendizado reduzida |
| **Feature Extraction** | Usar a base pré-treinada como extrator de características, treinando apenas o classificador final |
| **MobileNetV2** | Arquitetura CNN eficiente que usa depthwise separable convolutions |
| **ImageNet** | Dataset com 1,2 M imagens e 1.000 classes, base do pré-treino |
| **Depthwise Separable Convolution** | Operação que fatoriza uma convolução padrão em duas etapas, reduzindo custo computacional |
| **GlobalAveragePooling2D** | Operação que reduz um mapa de features 3D para um vetor pela média espacial |
| **Dropout** | Técnica de regularização que desativa neurônios aleatoriamente durante o treino |
| **BatchNormalization** | Normaliza as ativações de cada camada por lote, estabilizando o treino |
| **Softmax** | Função de ativação que converte logits em probabilidades somando 1 |
| **Categorical Cross-Entropy** | Função de perda para classificação multiclasse com saída one-hot |
| **Adam** | Otimizador adaptativo que combina momentum e RMSprop |
| **EarlyStopping** | Para o treino automaticamente quando a métrica de validação para de melhorar |
| **ReduceLROnPlateau** | Reduz o learning rate quando a validação estagna |
| **ModelCheckpoint** | Salva o melhor modelo em disco durante o treino |
| **AUC** | Área sob a curva ROC — mede a capacidade discriminativa do modelo |
| **ROC** | Curva que plota TPR vs FPR para diferentes limiares de decisão |
| **F1-score** | Média harmônica entre Precision e Recall |
| **Precision** | Proporção de predições positivas que eram realmente positivas |
| **Recall** | Proporção de casos positivos reais que foram detectados |
| **Overfitting** | Quando o modelo memoriza o treino mas generaliza mal para dados novos |
| **Catastrofic Forgetting** | Destruição dos pesos pré-treinados por um LR excessivo durante fine-tuning |
| **One-vs-Rest (OvR)** | Estratégia multiclasse que trata cada classe como binária contra todas as outras |
| **Data Augmentation** | Aumento artificial da diversidade do treino por transformações aleatórias |
| **Learning Rate (LR)** | Taxa que controla o tamanho do passo de atualização dos pesos |

---

*Documento gerado a partir do notebook `Projeto3_CNN_IntelImageClassification.ipynb`*  
*T326 - Ciência dos Dados | Professor Caio Ponte*