# Assistente di Refertazione NGS — Pannello 50 Geni

**S.C. Anatomia Patologica · ASST FBF Sacco · 2026**

Strumento operativo per la compilazione della Parte B del referto NGS su pannello targeted 50 geni (Diatech Pharmacogenetics — Myriapod NGS). Funziona come file HTML standalone — nessuna installazione, nessun server, nessun dato inviato all'esterno.

---

## Cosa fa

- Menu a cascata **Sede → Gene → Variante** (solo geni rilevanti per quella sede)
- Motore VAF: confronta il VAF inserito con la cellularità stimata su H&E e classifica il dato (coerente / subclonale / LOH / artefatto)
- Genera la **frase pronta** da copiare in Parte B su Pathox
- Gestisce varianti senza VAF (fusioni, CNV, MSI, imbalance, delezioni, wild-type) con template dedicati — il blocco VAF si disabilita automaticamente
- **Accumulatore multi-gene normalizzato:** chiave `sede|gene|variante` previene duplicati; salvare lo stesso gene due volte sostituisce la versione precedente
- **Pannello negativo:** per NSCLC, CRC, Gastrico/GEJ e Melanoma, un click genera la frase completa per tutti i geni negativi; rimuove automaticamente dall'accumulatore eventuali geni del pannello già presenti per evitare contraddizioni
- **Blocco su VAF sotto soglia:** casi con VAF < 5% o non conclusivi non entrano mai nell'accumulatore né nel referto finale
- Reflex: segnala quando è richiesta conferma IHC/FISH prima di firmare
- Note pratiche sede-specifiche per ogni variante
- Glossario integrato con badge Tier I/II/III, VAF, CNV, MSI-H, Reflex, ALK imbalance

---

## Flusso operativo

1. L'**Oncologo** invia la richiesta NGS
2. Il **Patologo** riceve il caso in Pathox, valuta l'H&E e stima la cellularità tumorale prima dell'invio in BioMol
3. Il **Biologo Molecolare** estrae DNA+RNA da FFPE, sequenzia e compila **Parte A** in Pathox (kit, campione, copertura, VAF grezzi — senza interpretazione clinica)
4. Il **Patologo** apre il tool nel browser, seleziona sede → gene → variante, inserisce cellularità e VAF, preme **Genera**
5. Per ogni gene aggiuntivo preme **+ Aggiungi altro gene** (la frase viene salvata nell'accumulatore)
6. Se tutto il resto è negativo, preme **⊘ Tutto il resto negativo?** per aggiungere in un click la frase del pannello negativo completo
7. Preme **Copia referto completo** e incolla in **Parte B** su Pathox, quindi firma

**Note:**
- La sede rimane fissa tra un gene e l'altro
- I geni si possono inserire in qualsiasi ordine
- "Nuovo caso" azzera completamente l'accumulatore — nessun rischio di contaminazione tra casi
- Casi bloccati non vengono mai salvati nell'accumulatore, indipendentemente da quale bottone si preme

---

## File del repository

| File | Descrizione |
|---|---|
| `NGS_Refertazione_v2.html` | Tool di refertazione — si apre nel browser |
| `NGS_master.xlsx` | Database sorgente (sede · gene · variante · reflex · note · vafMode · kind) |
| `NGS_Doc1_Operativo.pptx` | Briefing operativo per i patologi (workflow + casi clinici) |
| `NGS_Doc2_Riferimento.pptx` | Guida tecnica + morfologia per sede |
| `NGS_Workflow_Bigino.pdf` | Schema operativo in una pagina |

---

## Architettura del database

Il file `NGS_master.xlsx` contiene una riga per ogni combinazione sede/gene/variante.

| Campo | Descrizione |
|---|---|
| `SEDE` | Sede tumorale |
| `GENE` | Nome del gene |
| `RISULTATO` | Label della variante nel menu |
| `TIPO` | Standard / STEP |
| `TIER` | Tier AMP/ASCO/CAP |
| `REFLEX` | Testo reflex se richiesto, "No" altrimenti |
| `NOTE` | Nota pratica sede-specifica |
| `VAF_MODE` | `required` / `not_applicable` |
| `KIND` | Tipo molecolare (snv, fusion, cnv, del_hom, del, splice, indel, msi, imbalance, wildtype, structural) |

**Statistiche DB correnti:** 21 sedi · 162 voci

---

## Pannello coperto

**SNV/InsDel (49 geni):** AKT1, ALK, AR, BRAF, CDK4, CDKN2A, CTNNB1, DDR2, EGFR, ERBB2, ERBB3, ERBB4, ESR1, FGFR1, FGFR2, FGFR3, FGFR4, GNA11, GNAQ, GNAS, HRAS, IDH1, IDH2, KEAP1, KIT, KRAS, MAP2K1, MET, MTOR, NF1, NRAS, NTRK1, NTRK2, NTRK3, PDGFRA, PIK3CA, POLE, PTEN, RAF1, RB1, RET, ROS1, SMAD4, SMO, STK11, TERT, TP53, TSC1

**Fusioni RNA (12 geni):** ALK, FGFR1, FGFR2, FGFR3, MET, NRG1, NTRK1, NTRK2, NTRK3, PPARG, RET, ROS1

**CNV validati:** EGFR · ERBB2 · MET

**MSI:** 126 marcatori (pannello Bethesda esteso)

---

## Logica VAF

| Condizione | Stato | Colore | Azione |
|---|---|---|---|
| VAF < 5% | Artefatto | 🔴 Rosso | Blocco — NON FIRMARE, verifica tecnica |
| VAF > cellularità × 0.85 | LOH | 🟠 Arancio | Firma con nota obbligatoria |
| VAF < atteso × 0.4 | Subclonale | 🟡 Giallo | Firma con nota obbligatoria |
| VAF nel range ±30% | Coerente | 🟢 Verde | Firma |
| VAF fuori range >30% | Discordante | 🟡 Giallo | Firma con nota |

VAF atteso per mutazione eterozigote = cellularità ÷ 2

---

## Template per tipo molecolare

| Kind | Template frase |
|---|---|
| `snv` / `splice` / `indel` | Motore VAF → frase con VAF e cellularità |
| `fusion` con `::` | "Si documenta la fusione X::Y nel contesto di [sede]." |
| `fusion` senza partner | "Si documenta il riarrangiamento di [gene]..." |
| `cnv` | "Si rileva segnale di amplificazione... richiede conferma IHC/FISH." |
| `msi` | "Il profilo molecolare è compatibile con MSI-H." |
| `imbalance` | "Si rileva sbilanciamento di espressione... conferma con IHC D5F3." |
| `del_hom` | "Si documenta perdita biallelica di segnale... conferma con FISH." |
| `wildtype` | "Non si evidenziano alterazioni di [gene] tra quelle indagate." |
| `structural` | "Si documenta la variante strutturale [label]..." |

---

## Pannelli negativi per sede

| Sede | Geni coperti |
|---|---|
| NSCLC | EGFR, ALK, ROS1, MET, KRAS, BRAF, NTRK1/2/3, RET, ERBB2, MSI |
| CRC | KRAS, NRAS, BRAF, MSI, ERBB2, NTRK1/2/3 |
| Gastrico/GEJ | ERBB2, MSI, FGFR2, MET |
| Melanoma | BRAF, NRAS, KIT, GNA11, GNAQ |

Il pannello negativo è sostitutivo: rimuove dall'accumulatore i geni del pannello già presenti prima di aggiungere la frase negativa completa.

---

## Sicurezza e gestione errori

- **Casi bloccati:** VAF < 5% o test non conclusivo → non entrano mai nell'accumulatore né nel referto copiato
- **Duplicati:** accumulatore normalizzato per chiave `sede|gene|variante` — la versione più recente sostituisce sempre la precedente
- **Nuovo caso:** azzera completamente l'accumulatore
- **Note interne:** visibili nell'accumulatore come promemoria, ma escluse dal testo copiato in Parte B
- **Pannello negativo + gene positivo:** il pannello rimuove automaticamente i geni in conflitto prima di aggiungere la frase negativa

---

## Limiti

- Non calcola TMB (pannello 50 geni non sufficiente per cutoff terapeutici)
- CNV validati solo per EGFR, ERBB2, MET
- Non distingue variante somatica da germinale
- Fusioni richiedono RNA di qualità sufficiente
- ALK expression imbalance ≠ fusione confermata — richiede sempre IHC D5F3
- Tool non validato come dispositivo IVD — la responsabilità è del patologo che firma

---

## Classificazione varianti

Sistema **AMP/ASCO/CAP** (Li et al., *Journal of Molecular Diagnostics* 2017) · Database: OncoKB · ClinVar · COSMIC

---

## Versioning

`Versione 2.1 — Marzo 2026`

**Modifiche v2.1:**
- WT e NV non richiedono VAF
- Nuovo caso azzera l'accumulatore
- Casi bloccati esclusi da accumulatore e copia in ogni scenario
- Pannello negativo sostitutivo (rimuove conflitti)
- Accumulatore normalizzato per chiave sede|gene|variante
- Note interne escluse dalla copia in Parte B

---

*Documento interno ad uso diagnostico · S.C. Anatomia Patologica · ASST FBF Sacco · Classificazione varianti secondo AMP/ASCO/CAP (Li et al., JMD 2017)*
