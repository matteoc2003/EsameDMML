# Analisi sullo Smaltimento dei Rifiuti Urbani

> **Elaborato di Data Mining** — Università  
> *Matteo Caruso, Elia Corti, Federico Jarach, Tommaso Zanni*

---

## Case of Study

Il progetto analizza il fenomeno dello **smaltimento dei rifiuti urbani** a livello comunale in Italia, sfruttando un dataset pubblico (`public_data_waste_fee.csv`) contenente **4.341 osservazioni** e **39 variabili** relative a caratteristiche demografiche, geografiche, economiche e di gestione dei rifiuti di ogni comune.

L'analisi si articola in **due macro-obiettivi**:

1. **Modello con target continuo** — Previsione della percentuale di raccolta differenziata (`sor`) per ogni comune.
2. **Modello con target dicotomico** — Classificazione del tipo di tariffa applicata dal comune (`d_fee`): tariffa puntuale di tipo **PAYT** (*Pay As You Throw*) vs. tariffa standard.

---

## Dataset

| Attributo         | Dettaglio                                                              |
|-------------------|------------------------------------------------------------------------|
| **Fonte**         | Dataset pubblico open-data — `public_data_waste_fee.csv`              |
| **Osservazioni**  | 4.341 comuni italiani                                                  |
| **Variabili**     | 39 (demografiche, geografiche, economiche, composizione rifiuti)       |
| **Target 1**      | `sor` — percentuale di raccolta differenziata (variabile continua)    |
| **Target 2**      | `d_fee` — tipo di tariffa: PAYT (1) vs Standard (0) (variabile binaria)|

---

## Metodologia

### 1. Pre-processing & Missing Data
- Analisi esplorativa dei valori mancanti con la libreria `VIM`
- Rimozione delle variabili con oltre il 20% di NA (es. `wood`, `texile`, `province`, `finance`)
- **Imputazione multipla MICE** con metodo `CART` per le variabili restanti, verificando l'assunzione **MAR** (*Missing At Random*)

### 2. Gestione della Collinearità
- Calcolo di **TOL (Tolerance)** e **VIF (Variance Inflation Factor)** per le variabili quantitative
- Rimozione iterativa delle variabili collineari (`pop`, `wden`, `msw_so`)
- Analisi di connessione **χ²_norm** per le variabili qualitative → rimozione di `region` (χ²_norm = 0.96 con `geo`)

### 3. Model Selection
- **Stepwise selection** bidirezionale basata su **AIC** per individuare il modello più parsimonioso

### 4. Trasformazioni (Target Continuo)
- **Box-Cox** sulla variabile dipendente `sor` → λ ≈ 1.64, trasformazione: ŷ' = ŷ²/2
- **GAM (Generalized Additive Model)** per valutare le non-linearità → applicazione di trasformazioni **logaritmiche** a `tc`, `cres`, `csor`, `area`, `alt`, `raee`
- **RESET test** post-trasformazione: p-value = 0.47 (nessuna misspecificazione)

### 5. Analisi degli Outlier
- Identificazione tramite **DFFITS** (soglia: 2√p/n)
- Confronto modello con/senza outlier → gli outlier migliorano il fit, vengono mantenuti

### 6. Eteroschedasticità
- Rilevata con **ncvTest** → varianza non costante
- Correzione tramite **errori standard robusti di White**
- **Bootstrap** (B = 1999 repliche) per stimare intervalli di confidenza al 95%

### 7. Modello Logistico (Target Dicotomico)
- Stessa pipeline di gestione della collinearità
- Studio delle interazioni tra variabili: interazione `sor × alt` significativa
- Calcolo degli **Odds Ratio** con intervalli di confidenza (forest plot)
- Valutazione con **R² di McFadden** e matrice di confusione

---

## Risultati Principali

### Modello Lineare (Target: `sor` — Raccolta Differenziata)

| Metrica               | Valore                                              |
|-----------------------|-----------------------------------------------------|
| **Trasformazione target** | Box-Cox λ ≈ 1.64 → ŷ' = ŷ²/2              |
| **RESET test**        | p-value = 0.47 (forma funzionale corretta)          |
| **Variabili finali**  | 22 predittori selezionati via stepwise              |
| **Inferenza robusta** | Errori standard White + Bootstrap 95% CI           |

**Variabili più rilevanti**: raccolta organica, carta, vetro, metallo, plastica, rifiuti ingombranti, area geografica (`geo`), tipologia tariffaria (`d_fee`), salario medio (`wage`), PIL pro capite (`gdp`).

### Modello Logistico (Target: `d_fee` — Tipo di Tariffa)

| Metrica                  | Valore    |
|--------------------------|-----------|
| **R² McFadden**          | ≈ 0.27    |
| **Accuratezza**          | 88.3%     |
| **Tasso di errore**      | 11.7%     |
| **Specificity Standard** | 97.9%     |
| **Sensitivity PAYT**     | 23.6%     |

Il modello classifica molto bene i comuni con tariffa **standard** (97.9% corretto), ma fatica a identificare i comuni con tariffa **PAYT**, probabilmente per lo sbilanciamento delle classi nel dataset.

**Interazione chiave**: `sor × alt` — la relazione tra raccolta differenziata e altitudine risulta statisticamente significativa nel determinare il tipo di tariffa.

---

## Struttura dei File

```
TesinaDM/
├── public_data_waste_fee.csv   # Dataset originale
├── tesinaDM.Rmd                # Report completo (sorgente R Markdown)
└── tesinaDM.docx               # Report compilato (Word)
```

---

## Librerie R Utilizzate

| Libreria      | Utilizzo                                    |
|---------------|---------------------------------------------|
| `VIM`         | Visualizzazione valori mancanti             |
| `mice`        | Imputazione multipla MICE                   |
| `mctest`      | Calcolo TOL e VIF                           |
| `corrgram`    | Matrice di correlazione grafica             |
| `MASS`        | Box-Cox transformation                      |
| `gam`         | Generalized Additive Models                 |
| `lmtest`      | RESET test, test eteroschedasticità         |
| `sandwich`    | Errori standard robusti di White            |
| `car`         | Bootstrap, ncvTest, DFFITS                  |
| `forestmodel` | Forest plot degli Odds Ratio                |
| `dplyr`       | Data manipulation                           |

---

## Come Riprodurre l'Analisi

1. Clonare il repository e aprire il progetto in **RStudio**
2. Installare le librerie necessarie:
   ```r
   install.packages(c("VIM", "mice", "mctest", "corrgram", "MASS", "gam",
                      "lmtest", "sandwich", "car", "forestmodel", "dplyr",
                      "gvlma", "plyr", "psych"))
   ```
3. Impostare la working directory sulla cartella `TesinaDM/`
4. Aprire `tesinaDM.Rmd` in RStudio e premere **Knit** per generare il report

> **Nota**: L'imputazione MICE è computazionalmente intensa. Il file `data_imputed.csv` già pre-computato è incluso per saltare questo step.

---

## Autori

*Matteo Caruso · Elia Corti · Federico Jarach · Tommaso Zanni*

*Progetto realizzato nell'ambito del corso di Data Mining — A.A. 2024/2025*
