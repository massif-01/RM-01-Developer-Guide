# Guida per Sviluppatori RM-01

**Da leggere prima dell'uso:**

L'RM-01 è composto da un **Modulo di Inferenza**, un **Modulo di Applicazione** e un **Chip di Crittografia e Gestione** (di seguito denominato Modulo di Gestione), interconnessi tramite un **chip di switch Ethernet integrato**, che forma una sottorete LAN interna. Quando un utente collega un host (ad esempio, PC, smartphone, iPad) tramite l'interfaccia **USB Type-C**, l'RM-01 virtualizza un'interfaccia Ethernet per l'host tramite la funzionalità USB Ethernet. L'host ottiene quindi un indirizzo IP e si unisce automaticamente alla sottorete per lo scambio di dati.

Dopo l'accensione del dispositivo e la connessione all'host tramite l'interfaccia **USB Type-C**, il sistema configurerà automaticamente la sottorete di rete locale. All'**host dell'utente** verrà assegnato un indirizzo IP statico `10.10.99.100`. Il **Modulo di Inferenza** (IP: `10.10.99.98`) e il **Modulo di Applicazione** (IP: `10.10.99.99`)—entrambi implementano servizi SSH indipendenti, consentendo agli utenti di accedervi direttamente tramite client SSH standard (ad esempio, OpenSSH, PuTTY). Il Modulo di Gestione, invece, richiede l'accesso tramite uno strumento per porta seriale.

---

## Come fornire l'accesso a Internet all'RM-01 dall'host (usando macOS come esempio)

Dopo aver collegato l'**host dell'utente** tramite USB Type-C, l'RM-01 apparirà nella lista delle interfacce di rete come:  
- **`AX88179A`** (Versione per Sviluppatori)  
- **`RMinte RM-01`** (Versione di Rilascio Commerciale)

Segui questi passaggi per configurare la condivisione di Internet:

1. Apri **Impostazioni di Sistema** (System Settings)  
2. Vai su **Rete** (Network) → **Condivisione** (Sharing)  
3. Abilita **Condivisione Internet** (Internet Sharing)  
4. Fai clic sull'icona **“i”** accanto alle impostazioni di condivisione per accedere all'interfaccia di configurazione:  
   - Imposta **“Condividi la tua connessione da”** su: **Wi-Fi**  
   - In **“Ai computer che usano”**, seleziona: **AX88179A** o **RMinte RM-01** (a seconda del modello del dispositivo)  
5. Fai clic su **Fine** (Done)

Successivamente, torna alla pagina delle impostazioni **Rete** (Network) e configura manualmente l'interfaccia di rete dell'RM-01:  
- **Indirizzo IP**: `10.10.99.100`  
- **Maschera di Sottorete**: `255.255.255.0`  
- **Router**: `10.10.99.100` (cioè l'IP dell'host stesso)

> **Nota**:  
> Questa configurazione imposta l'host come gateway, fornendo accesso alla rete NAT per l'RM-01. Il gateway predefinito e il DNS dell'RM-01 sono assegnati automaticamente dall'host tramite il servizio DHCP. Impostare manualmente l'IP assicura che rimanga all'interno della sottorete `10.10.99.0/24`, coerente con la comunicazione dei servizi interni del dispositivo.

---

## Informazioni sulla scheda di memoria **CFexpress Type-B**

La scheda di memoria **CFexpress Type-B** è uno dei componenti principali del dispositivo RM-01, responsabile dell'avvio del sistema, del dispiegamento del framework di inferenza e di funzioni chiave come la distribuzione del software ISV/SV e l'autenticazione di autorizzazione.

La scheda di memoria è divisa in tre partizioni indipendenti, ovvero:

`rm01sys` `rm01app` `rm01models`

`rm01sys` — Partizione di Sistema  
Il sistema operativo e l'ambiente di esecuzione principale del **Modulo di Inferenza** sono installati in questa partizione. È severamente vietato agli utenti o agli sviluppatori accedere, modificare o eliminare il contenuto di questa partizione. Qualsiasi modifica non autorizzata può causare il mancato avvio del Modulo di Inferenza o il malfunzionamento delle funzioni di inferenza, e qualsiasi danno hardware o software risultante non è coperto dai servizi di garanzia.

`rm01app` — Partizione di Applicazione  
Questa partizione viene utilizzata per archiviare temporaneamente i file di immagine `Docker` inviati dagli utenti o dagli sviluppatori. Una volta che l'immagine viene scritta in `rm01app`, il sistema RM-01 la migra automaticamente nello storage **NVMe SSD** integrato dell'host e completa il dispiegamento containerizzato. Non eseguire o modificare direttamente i file delle applicazioni in questa partizione.

`rm01models` — Partizione dei Modelli  
Questa partizione è dedicata all'archiviazione di modelli di intelligenza artificiale su larga scala (ad esempio, LLM, modelli multimodali, ecc.) caricati da utenti o sviluppatori. Per dettagli sui formati dei modelli, limitazioni di dimensione, procedure di caricamento e requisiti di compatibilità, consulta la sezione “Informazioni sui Modelli” qui sotto.

---

## Informazioni sul Modulo di Applicazione

Indirizzo IP: `10.10.99.99`  
Intervallo di Porte: `59000-59299`

Specifiche Hardware del Modulo di Applicazione:

```
Processore: Intel Core i3-N305 (8 core, 8 thread, frequenza base 1.8 GHz, frequenza turbo massima 3.8 GHz)  
Memoria: 16 GB / 24 GB LPDDR5-4800MT/s (integrata, non espandibile)  
Archiviazione: 512 GB / 1 TB / 2 TB (opzionale) NVMe SSD  
```

Credenziali di Accesso SSH del Modulo di Applicazione:

```
Nome utente predefinito: rm01  
Password predefinita: rm01 (preimpostata in fabbrica, solo per il primo accesso)  
```

**Avviso di Sicurezza:**  
Per garantire la sicurezza del sistema, utilizza immediatamente il comando `passwd` per modificare la password predefinita dopo il primo accesso SSH. La password predefinita è solo per la configurazione iniziale e non deve essere utilizzata in ambienti di produzione o distribuzione.

---

## Informazioni sul Modulo di Inferenza

Indirizzo IP: `10.10.99.98`  
Intervallo di Porte di Servizio: `58000–58999`

Il **Modulo di Inferenza** è l'unità di calcolo principale dell'RM-01, che supporta varie configurazioni di inferenza IA ad alte prestazioni. Gli utenti possono selezionare il modello appropriato in base alla scala del modello e ai requisiti di prestazioni. Le quattro configurazioni hardware preimpostate in fabbrica sono le seguenti:

| Memoria | Larghezza di Banda della Memoria | Potenza di Calcolo | Numero di Tensor Core |
|---------|---------------------------------|:------------------|:----------------------|
| 32 GB   | 204.8 GB/s                      | 200 TOPS (INT8)   | 56                    |
| 64 GB   | 204.8 GB/s                      | 275 TOPS (INT8)   | 64                    |
| 64 GB   | 273 GB/s                        | 1,200 TFLOPS (FP4)| 64                    |
| 128 GB  | 273 GB/s                        | 2,070 TFLOPS (FP4)| 96                    |

L'RM-01 viene fornito con i seguenti due framework di inferenza preinstallati sulla scheda di memoria **CFexpress Type-B**, entrambi eseguiti sul Modulo di Inferenza:

`vLLM`: Si avvia automaticamente, ascolta di default la porta **58000**, fornisce interfacce API compatibili con OpenAI e supporta richieste standard come POST /v1/chat/completions.  
`TEI` (Text Embedding Inference): Richiede un avvio manuale, utilizzato per servizi di incorporamento di testo.

Metodo di Accesso API:  
Dopo aver caricato con successo un modello, il servizio di inferenza **vLLM** può essere accessibile tramite il seguente indirizzo:  
`http://10.10.99.98:58000/v1/chat/completions`  
Supporta chiamate dirette utilizzando client OpenAI standard (ad esempio, openai-python, curl, Postman).

**Avviso di Sicurezza:**  
Per garantire la sicurezza e la stabilità del sistema, il Modulo di Inferenza non fornisce permessi di accesso SSH. Gli utenti e gli sviluppatori non possono accedere direttamente o interagire con il sistema operativo sottostante di questo modulo. Qualsiasi tentativo di aggirare le politiche di sicurezza o accedere direttamente al Modulo di Inferenza può causare anomalie del sistema, corruzione dei dati o interruzioni del servizio, non coperti dai servizi di garanzia.

---

## Informazioni sui Modelli

L'RM-01 supporta l'inferenza di vari modelli di intelligenza artificiale, inclusi, ma non limitati a: **Modelli di Linguaggio su Larga Scala (LLM)**, **Modelli Multimodali (MLM)**, **Modelli di Visione-Linguaggio (VLM)**, **Modelli di Incorporamento di Testo (Embedding)** e **Modelli di Riclassificazione (Reranker)**. Tutti i file dei modelli devono essere archiviati sulla scheda di memoria **CFexpress Type-B** integrata nel dispositivo, e gli utenti devono utilizzare un lettore di schede **CFexpress Type-B** compatibile per caricare, gestire e aggiornare i modelli dal lato dell'host.

Quando la scheda di memoria **CFexpress Type-B** viene collegata all'RM-01, il sistema la monta come un volume di dati di sola lettura chiamato `models` al percorso `/home/rm01/models`. La sua struttura di file standard è la seguente:

```bash
models/
├── auto/                  # Directory per il caricamento automatico dei modelli (distribuzione di livello produzione)
│   ├── embedding/         # Modelli di incorporamento (non caricati automaticamente, vedi sotto)
│   ├── llm/               # Modelli di linguaggio su larga scala (file di pesi archiviati direttamente, vedi sotto)
│   └── reranker/          # Modelli di riclassificazione (non caricati automaticamente, vedi sotto)
└── dev/                   # Directory dei modelli definiti dagli sviluppatori (maggiore priorità)
    ├── embedding/
    ├── llm/
    └── reranker/
```

> **Nota**:  
> - La directory `auto/` è utilizzata per **distribuzione di modelli standardizzata e leggera**, riconosciuta automaticamente dal sistema.  
> - La directory `dev/` è utilizzata per **controllo dettagliato del comportamento dei modelli da parte degli sviluppatori**, con priorità maggiore rispetto a `auto/`. Il sistema ignorerà i modelli in `auto/` se viene utilizzata `dev/`.

---

### 1. Modalità Automatica (auto)

#### Uso:  
Posiziona i **file di pesi completi** del modello (ad esempio, `.safetensors`, `.bin`, `.pt`, `.awq`, ecc.) **direttamente nella directory `auto/llm/`**, **senza annidarli in sottocartelle**.

> Esempio errato:  
> `auto/llm/Qwen3-30B-A3B-Instruct-2507-AWQ/model.safetensors`  
>  
> Esempio corretto:  
> `auto/llm/model-001-of-006.safetensors`  
> `auto/llm/config.json`  
> `auto/llm/tokenizer.json`  
> `auto/llm/vocab.json`

#### Comportamento del Sistema:  
- All'avvio del dispositivo, il sistema scansiona la directory `auto/llm/` e carica automaticamente i modelli in formati compatibili.  
- **Non è supportato il caricamento automatico di modelli di incorporamento o riclassificazione**, solo LLM.  
- Dopo il caricamento, il modello abilita **capacità di inferenza di base per impostazione predefinita** e **non abilita le seguenti funzionalità avanzate**:  
  - Decodifica Speculativa  
  - Caching dei Prefissi  
  - Pre-riempimento a Blocchi  
  - La lunghezza massima del contesto (`max_model_len`) è limitata alla soglia di sicurezza del sistema (tipicamente ≤ 8192 token).  
- **Ottimizzazione delle prestazioni limitata**: Per garantire la stabilità del sistema e la concorrenza multitasking, i modelli in modalità automatica utilizzano una strategia di allocazione della memoria conservativa (`gpu_memory_utilization` ≤ 0.8).

> **Nota Importante**:  
> La modalità automatica è adatta per **verificare rapidamente la compatibilità del modello** o **scenari di distribuzione standardizzati**, ma **non è adatta per l'inferenza ad alte prestazioni in ambienti di produzione**. Per prestazioni complete, utilizza la **modalità manuale (dev)**.

---

### 2. Modalità Manuale (dev) — Modalità Sviluppatore

> **La modalità manuale ha una priorità maggiore rispetto alla modalità automatica**. Quando esiste un file di configurazione `.yaml` nelle directory `embedding`, `llm` o `reranker` sotto `dev/`, il sistema **ignora completamente tutti i modelli in `auto/`**.

#### Struttura dei File di Configurazione:  
Le directory `embedding`, `llm` e `reranker` sotto `dev/` devono contenere i seguenti tre file di configurazione YAML per avviare i servizi dei modelli corrispondenti:

| Nome del File         | Scopo                                |
|-----------------------|--------------------------------------|
| `embedding_run.yaml`  | Avviare il servizio di modello di incorporamento di testo |
| `llm_run.yaml`        | Avviare il servizio di modello di linguaggio su larga scala |
| `reranker_run.yaml`   | Avviare il servizio di modello di riclassificazione |

> Tutti i file devono essere posizionati nelle cartelle corrispondenti sotto `dev/`, e **i nomi dei file devono corrispondere esattamente**, sensibili a maiuscole e minuscole.

#### Esempio di File di Configurazione (`llm_run.yaml`):

```yaml
model: /home/rm01/models/dev/llm/Qwen3-30B-A3B-Instruct-2507-AWQ
port: 58000
gpu_memory_utilization: 0.85
max_model_len: 24576
served_model_name: RM-01 LLM
enable_prefix_caching: true
enable_chunked_prefill: true
max_num_batched_tokens: 512
block_size: 16
tensor_parallel_size: 1
dtype: auto
```

> **Nota**:  
> - Il percorso `model` deve essere un **percorso assoluto** che punti alla directory dei file del modello (non un file compresso).  
> - La `port` deve essere nell'intervallo `58000–58999` e non deve entrare in conflitto con altri servizi.  
> - Si consiglia di impostare `gpu_memory_utilization` tra 0.7–0.9 per massimizzare il throughput.  
> - Tutti i parametri seguono la **documentazione ufficiale di vLLM** `https://docs.vllm.ai/en/latest/index.html`. Assicurati della compatibilità con la versione di vLLM attualmente utilizzata sulla scheda di memoria **CFexpress Type-B**.

#### Processo di Avvio:  
1. Copia i file del modello (directory completa) in `dev/llm/` (o `embedding/`, `reranker/`).  
2. Crea e posiziona il file di configurazione `.yaml` corrispondente.  
3. Inserisci la scheda di memoria **CFexpress Type-B** e riavvia l'RM-01.  
4. Il sistema caricherà automaticamente la configurazione in `dev/` e avvierà il servizio corrispondente.  
5. Una volta avviato il servizio, può essere accessibile tramite interfacce come `http://10.10.99.98:58000/v1/chat/completions`.

> **Raccomandazione**: Per il primo dispiegamento, utilizza le opzioni `--load-format auto` e `--dtype auto` fornite da vLLM per adattarsi automaticamente al formato del modello.

---

### Note su Sicurezza e Manutenzione

- **Vietato accedere direttamente tramite SSH al Modulo di Inferenza**: Tutta la gestione dei modelli deve essere effettuata tramite la scheda di memoria **CFexpress Type-B**.  
- **I file dei modelli devono essere pesi grezzi**: Non utilizzare file compressi (.zip/.tar.gz), pacchetti crittografati o formati non standard.  
- **Permessi dei File**: Tutti i file dei modelli devono essere leggibili (`chmod 644`), e le directory devono essere eseguibili (`chmod 755`).  
- **Controllo delle Versioni**: Si consiglia di utilizzare Git o convenzioni di denominazione dei file (ad esempio, `Qwen3-30B-A3B-Instruct-v1.2-20250930`) per gestire le versioni dei modelli.  
- **Raccomandazione di Backup**: Esegui il backup delle directory `dev/` e `auto/` prima di aggiornare i modelli per evitare la perdita di configurazione.

---

### Raccomandazioni per la Selezione della Modalità

| Scenario                        | Modalità Raccomandata                |
|--------------------------------|--------------------------------------|
| Verifica rapida della compatibilità del modello | Modalità Automatica (auto)           |
| Inferenza ad alte prestazioni in produzione | Modalità Manuale (dev) + configurazione ottimizzata |
| Distribuzione parallela di più modelli | Modalità Manuale (dev) + più file `.yaml` |
| Debug di sviluppo, validazione di prototipi | Modalità Manuale (dev)              |