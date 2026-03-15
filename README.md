# Assistente di Refertazione NGS — Pannello 50 Geni

**S.C. Anatomia Patologica · ASST FBF Sacco · 2026**

Strumento operativo per la compilazione della Parte B del referto NGS su pannello targeted 50 geni (Diatech Pharmacogenetics). Funziona come file HTML standalone — nessuna installazione, nessun server, nessun dato inviato all'esterno.

---

## Cosa fa

- Menu a cascata **Sede → Gene → Variante** (solo geni rilevanti per quella sede)
- Motore VAF: confronta VAF inserito con cellularità stimata su H&E, classifica il dato (coerente / subclonale / LOH / artefatto)
- Genera la **frase pronta** da copiare in Parte B su Pathox
- Gestisce varianti senza VAF (fusioni, CNV, MSI, imbalance, delezioni) con template dedicati per tipo molecolare
- Accumulatore multi-gene: genera più geni in sequenza, copia tutto il referto in un colpo solo
- Reflex: segnala quando è richiesta conferma IHC/FISH prima di firmare
- Note pratiche sede-specifiche per ogni variante
- Glossario integrato (Tier I/II/III, VAF, CNV, MSI-H, Reflex, ALK imbalance)

---

## File inclusi

| File | Descrizione |
|---|---|
| `NGS_Refertazione_v2.html` | Tool di refertazione — si apre nel browser |
| `NGS_master.xlsx` | Database sorgente (sede · gene · variante · reflex · note · vafMode · kind) |
| `genera_output.py` | Script per rigenerare l'HTML dal master Excel |
| `NGS_Doc1_Operativo.pptx` | Briefing operativo per i patologi (workflow + 9 casi clinici) |
| `NGS_Doc2_Riferimento.pptx` | Guida tecnica + morfologia per sede |
| `NGS_Presentazione_Oncologi.pptx` | Presentazione per i clinici |
| `NGS_Workflow_Bigino.pdf` | Schema operativo in una pagina |

---

## Come si usa

**Flusso operativo:**

1. L\'**Oncologo** invia la richiesta NGS tramite il sistema informatico ospedaliero
2. Il **Patologo** riceve il caso in Pathox, valuta l\'H&E e stima la cellularità tumorale prima dell\'invio in BioMol
3. Il **Biologo Molecolare** estrae DNA+RNA da FFPE, sequenzia e compila **Parte A** in Pathox (kit, campione, copertura, VAF grezzi — senza interpretazione clinica)
4. Il **Patologo** apre `NGS_Refertazione_v2.html` nel browser, seleziona sede → gene → variante, inserisce cellularità e VAF, preme **Genera**
5. Per ogni gene aggiuntivo preme **+ Aggiungi altro gene**
6. Quando ha finito tutti i geni preme **Copia referto completo** e incolla in **Parte B** su Pathox, quindi firma

**Note:**
- La sede rimane fissa tra un gene e l'altro
- I geni si possono inserire in qualsiasi ordine
- Per varianti senza VAF (fusioni, MSI, CNV, delezioni omozigoti) il blocco VAF si disabilita automaticamente

---

## Architettura del database

Il file `NGS_master.xlsx` contiene una riga per ogni combinazione sede/gene/variante con i seguenti campi:

| Campo | Descrizione |
|---|---|
| `SEDE` | Sede tumorale (CRC, NSCLC, Gastrico/GEJ, ...) |
| `GENE` | Nome del gene |
| `RISULTATO` | Label della variante nel menu |
| `TIPO` | Standard / STEP |
| `TIER` | Tier AMP/ASCO/CAP |
| `FRASE` | Frase base (non usata direttamente dal tool v2) |
| `REFLEX` | Testo reflex se richiesto, "No" altrimenti |
| `NOTE` | Nota pratica sede-specifica |
| `VAF_MODE` | `required` / `not_applicable` |
| `KIND` | Tipo molecolare: `snv` · `fusion` · `cnv` · `del_hom` · `del` · `splice` · `indel` · `msi` · `imbalance` · `wildtype` · `structural` |

Per aggiornare il database: modificare `NGS_master.xlsx`, poi eseguire:

```bash
python genera_output.py
```

---

## Pannello coperto

**SNV/InsDel (49 geni):** AKT1, ALK, AR, BRAF, CDK4, CDKN2A, CTNNB1, DDR2, EGFR, ERBB2, ERBB3, ERBB4, ESR1, FGFR1, FGFR2, FGFR3, FGFR4, GNA11, GNAQ, GNAS, HRAS, IDH1, IDH2, KEAP1, KIT, KRAS, MAP2K1, MET, MTOR, NF1, NRAS, NTRK1, NTRK2, NTRK3, PDGFRA, PIK3CA, POLE, PTEN, RAF1, RB1, RET, ROS1, SMAD4, SMO, STK11, TERT, TP53, TSC1

**Fusioni RNA (12 geni):** ALK, FGFR1, FGFR2, FGFR3, MET, NRG1, NTRK1, NTRK2, NTRK3, PPARG, RET, ROS1

**CNV validati:** EGFR · ERBB2 · MET

**MSI:** 126 marcatori (8 Bethesda + 118 mononucleotidici)

---

## Logica VAF

| Condizione | Stato | Colore | Azione |
|---|---|---|---|
| VAF < 5% | Artefatto | 🔴 Rosso | NON FIRMARE — verifica tecnica |
| VAF > cellularità × 0.85 | LOH | 🟠 Arancio | Firma con nota obbligatoria |
| VAF < atteso × 0.4 | Subclonale | 🟡 Giallo | Firma con nota obbligatoria |
| VAF nel range ±30% | Coerente | 🟢 Verde | Firma |
| VAF fuori range >30% | Discordante | 🟡 Giallo | Firma con nota |

VAF atteso eterozigote = cellularità ÷ 2

Varianti con `vafMode: not_applicable` (fusioni, CNV, MSI, delezioni omozigoti, imbalance) bypassano il motore VAF e generano template dedicati.

---

## Template per tipo molecolare

| Kind | Template frase |
|---|---|
| `snv` / `splice` / `indel` | Motore VAF → frase con VAF e cellularità |
| `fusion` con `::` | "Si documenta la fusione X::Y nel contesto di [sede]." |
| `fusion` senza partner | "Si documenta il riarrangiamento di [gene]..." |
| `cnv` | "Si rileva segnale di amplificazione... richiede conferma IHC/FISH." |
| `msi` | "Il profilo molecolare è compatibile con instabilità microsatellitare elevata (MSI-H)." |
| `imbalance` | "Si rileva sbilanciamento di espressione... conferma con IHC D5F3." |
| `del_hom` | "Si documenta perdita biallelica di segnale... conferma con FISH." |
| `wildtype` | "Non si evidenziano alterazioni di [gene] tra quelle indagate nel campione analizzato." |
| `structural` | "Si documenta la variante strutturale [label]..." |

---

## Classificazione varianti

Sistema **AMP/ASCO/CAP** (Li et al., *Journal of Molecular Diagnostics* 2017).

Database di riferimento: OncoKB · ClinVar · COSMIC

---

## Limiti

- Non calcola TMB (pannello 50 geni non sufficiente per cutoff terapeutici)
- CNV validati solo per EGFR, ERBB2, MET — gli altri geni producono segnale non refertabile
- Non distingue variante somatica da germinale (analisi solo su FFPE tumorale)
- Fusioni richiedono RNA di qualità sufficiente — blocchi >5 anni o malfissati possono dare falsi negativi
- ALK expression imbalance ≠ fusione confermata — richiede sempre IHC D5F3

---

## Versioning

`Versione 2.0 — Marzo 2026`

Per segnalare errori o aggiornamenti clinici: aprire una issue o contattare la S.C. Anatomia Patologica ASST FBF Sacco.

---

*Documento interno ad uso diagnostico · S.C. Anatomia Patologica · ASST FBF Sacco · Classificazione varianti secondo AMP/ASCO/CAP (Li et al., JMD 2017)*
