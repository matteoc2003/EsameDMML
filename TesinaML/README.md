# 💳 Credit Card Fraud Detection

> **Elaborato di Machine Learning** — Università  
> *Matteo Caruso, Elia Corti, Federico Jarach, Tommaso Zanni*

---

## 📋 Case of Study

Il progetto affronta il problema del **rilevamento delle frodi nelle transazioni con carta di credito**, un task di classificazione binaria di rilevanza critica nel settore bancario e finanziario.

Il dataset contiene transazioni reali anonimizzate, suddivise in training e test set, con l'obiettivo di costruire un modello in grado di classificare ogni transazione come **fraudolenta** (`is_fraud = 1`) o **legittima** (`is_fraud = 0`).

L'analisi ha incluso un confronto sistematico tra **9 algoritmi di Machine Learning** differenti, valutati tramite curve ROC e AUC, con una fase finale di calibrazione della soglia decisionale basata su un'analisi del **profitto atteso** (impatto economico reale delle decisioni).

---

## 📊 Dataset

> ⚠️ **I file CSV originali non sono inclusi nella repository** per via delle loro dimensioni (~500 MB totali).  
> Puoi scaricarli direttamente da Kaggle: **[Credit Card Fraud Detection Dataset](https://www.kaggle.com/datasets/kartik2112/fraud-detection)**

| Attributo         | Dettaglio                                                           |
|-------------------|---------------------------------------------------------------------|
| **File Training** | `fraudTrain.csv` — ~1.000.000 di transazioni                      |
| **File Test**     | `fraudTest.csv` — ~500.000 di transazioni                         |
| **Sample**        | `fraudSample.csv` ✅ — 10 righe di esempio (7 legittime + 3 frodi) incluso nel repo |
| **Target**        | `is_fraud` — frode (1) vs. legittima (0), variabile binaria        |
| **Classe di minoranza** | ~0.57% di transazioni fraudolente (dataset fortemente sbilanciato) |
| **Features chiave** | `amt` (importo), `category` (categoria merceologica), `lat`, `long`, `merch_lat`, `merch_long`, `unix_time`, `city_pop`, `gender` |

---

## 🔬 Metodologia

### 1. Analisi Esplorativa e Bilanciamento
- Il dataset originale è **fortemente sbilanciato** (~0.57% frodi): rilevato tramite analisi della distribuzione della variabile target
- Selezione delle feature informative: `category`, `amt`, `gender`, `lat`, `long`, `city_pop`, `unix_time`, `merch_lat`, `merch_long`
- **Bilanciamento** tramite `ROSE` + `ovun.sample` (metodo "both"):
  - Training set bilanciato: **200.000 osservazioni**, 30% frodi
  - Test set bilanciato: **100.000 osservazioni**, 30% frodi
- Analisi Zero-Variance con `nearZeroVar` → nessuna variabile eliminata

### 2. Feature Importance tramite Albero Decisionale
- Utilizzo di un albero CART come strumento di **model selection** (cross-validation 5-fold, metrica ROC)
- **Variabili più importanti identificate**: `amt` e `category`
- Insight chiave: le transazioni fraudolente hanno **importo mediano più alto** rispetto alle legittime, ma non raggiungono i picchi massimi → i truffatori operano in una fascia di importo specifica

### 3. Modelli Addestrati e Confrontati

| Modello               | Note principali                                                    |
|-----------------------|--------------------------------------------------------------------|
| **GLM (Logistico)**   | Baseline; buona AUC, ma alta misclassificazione delle frodi       |
| **Naive Bayes**       | Robusto, ma ~4.600 falsi allarmi in più rispetto al GLM           |
| **LASSO**             | Regressione logistica con regolarizzazione L1 (lambda ottimale via CV) |
| **KNN (k=10)**        | Richiede normalizzazione; applicato su subset delle feature       |
| **Decision Tree**     | Interpretabile; usato per feature selection                       |
| **Random Forest**     | 100 alberi; migliore bilanciamento bias-varianza                  |
| **Gradient Boosting** | 100 iterazioni, interaction.depth=3; alta capacità predittiva     |
| **PLS**               | Partial Least Squares con 5-fold CV; riduzione dimensionalità     |
| **LDA**               | Linear Discriminant Analysis; assunzione di normalità             |
| **Neural Network**    | MLP via `nnet`, 5-fold CV, preprocessing: corr + nzv + range     |

### 4. Valutazione dei Modelli
- Curve **ROC** su Training e Test set per ogni modello
- Metrica principale: **AUC (Area Under the Curve)**
- Analisi del **gap Train-Test AUC** per rilevare overfitting
- **Matrice di confusione** per analisi operativa (falsi positivi/negativi)

### 5. Calibrazione della Soglia e Analisi del Profitto
- Il modello migliore (Random Forest) opera su un dataset bilanciato (30% frodi) ≠ distribuzione reale (~0.57%)
- **Ricalibrazione Bayesiana** delle probabilità per allinearle alla distribuzione reale
- **Ottimizzazione della soglia decisionale** massimizzando il profitto atteso:
  - **Costo di una frode non rilevata**: importo della transazione fraudolenta
  - **Costo di un falso allarme**: perdita della commissione su transazione legittima bloccata
- Validazione della soglia ottimale su dataset originale (~500k transazioni)

### 6. Scoring su Nuovi Dati
- Pipeline di scoring su transazioni inedite:
  1. Preprocessing e allineamento dei livelli dei fattori
  2. Predizione probabilità con Random Forest
  3. Ricalibrazione Bayesiana
  4. Applicazione soglia ottimale → decisione BLOCCO / OK
  5. Report finale ordinato per rischio decrescente

---

## 📈 Risultati Principali

### Confronto AUC Modelli (Test Set)

| Modello               | AUC (approx.) | Note                              |
|-----------------------|---------------|-----------------------------------|
| **Random Forest**     | 🥇 Migliore   | Best performer, scelto per scoring|
| **Gradient Boosting** | 🥈 Alto       | Leggermente inferiore a RF        |
| **Neural Network**    | Alto          | Buona generalizzazione            |
| **LASSO**             | Medio-alto    | Nessun overfitting rilevato       |
| **LDA / PLS**         | Medio         | Lineari, meno flessibili          |
| **Naive Bayes**       | Medio         | ~4.600 falsi allarmi extra vs GLM |
| **KNN (k=10)**        | Variabile     | Sensibile alla normalizzazione    |
| **GLM**               | Solido        | ASE Test < ASE Train → no overfitting |
| **Decision Tree**     | Più basso     | Usato principalmente per feature selection |

> Il **Random Forest** risulta il modello vincitore per bilanciamento AUC, stabilità e assenza di overfitting.

### Insight Operativi
- Le frodi tendono a concentrarsi in **categorie merceologiche specifiche** con importi nella fascia media
- La soglia ottimale calcolata tramite analisi del profitto è **inferiore al classico 0.5**, per massimizzare la detection delle frodi reali (a bassa prevalenza)
- La ricalibrazione Bayesiana è **fondamentale**: senza di essa le probabilità sarebbero sistematicamente sovrastimate di ~50x rispetto alla realtà

---

## 🗂️ Struttura dei File

```
ML/
├── fraudTrain.csv           # Dataset di training originale (~1M righe)
├── fraudTest.csv            # Dataset di test originale (~500k righe)
├── train.csv                # Training set bilanciato (200k righe, 30% frodi)
├── test.csv                 # Test set bilanciato (100k righe, 30% frodi)
├── fraudTestCompatto.csv    # Subset compatto del test set
├── Tesina.Rmd               # Report principale (sorgente R Markdown)
├── Tesina copy.Rmd          # Versione estesa con scoring e analisi profitto
├── Tesina.nb.html           # Report compilato (HTML notebook)
├── Tesina copy.nb.html      # Report esteso compilato (HTML)
├── Tesina-copy.docx         # Report esteso compilato (Word)
├── Tuning NNET.docx         # Analisi tuning della rete neurale
├── newScript.R              # Script R aggiuntivo per esperimenti
├── createScoreData.R        # Script per generare dati di scoring
└── dispensa.md              # Dispensa teorica sui modelli ML utilizzati
```

---

## 🛠️ Tecnologie e Librerie R

| Libreria       | Utilizzo                                               |
|----------------|--------------------------------------------------------|
| `ROSE`         | Bilanciamento classi (ROSE + ovun.sample)             |
| `caret`        | Training unificato, CV, preprocessing, feature import.|
| `rpart`        | Decision Tree (CART)                                  |
| `randomForest` | Random Forest                                         |
| `gbm`          | Gradient Boosting Machine                             |
| `glmnet`       | LASSO/Ridge logistico con cross-validation            |
| `e1071`        | Naive Bayes                                           |
| `MASS`         | LDA (Linear Discriminant Analysis)                    |
| `pls`          | Partial Least Squares                                 |
| `nnet`         | Neural Network (MLP)                                  |
| `pROC`         | Curve ROC e calcolo AUC                               |
| `ggplot2`      | Visualizzazioni grafiche                              |
| `dplyr`        | Data manipulation                                     |

---

## ▶️ Come Riprodurre l'Analisi

1. Clonare il repository e aprire il progetto in **RStudio**
2. Installare le librerie necessarie:
   ```r
   install.packages(c("ROSE", "caret", "rpart", "randomForest", "gbm",
                      "glmnet", "e1071", "MASS", "pls", "nnet", "pROC",
                      "ggplot2", "dplyr"))
   ```
3. Impostare la working directory sulla cartella `ML/`
4. Aprire `Tesina copy.Rmd` per l'analisi completa (con scoring e profitto) oppure `Tesina.Rmd` per l'analisi base
5. Premere **Knit** in RStudio

> ⚠️ **Nota**: `fraudTrain.csv` (~350 MB) e `fraudTest.csv` (~150 MB) sono file di grandi dimensioni. I dataset bilanciati pre-processati (`train.csv` e `test.csv`) sono inclusi per velocizzare la riproduzione.

---

## 👥 Autori

*Matteo Caruso · Elia Corti · Federico Jarach · Tommaso Zanni*

*Progetto realizzato nell'ambito del corso di **Machine Learning** — A.A. 2024/2025*
