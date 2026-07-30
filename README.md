# 🚀 Cloud & IoT Edge Showcase: Virtualizzazione INSANE

Questo repository funge da vetrina per il mio progetto di integrazione e virtualizzazione di **INSANE** (Integrated aNd Selective Acceleration at the Network Edge), un middleware sviluppato per l'accelerazione di rete nelle applicazioni distribuite. 

*A causa di restrizioni di riservatezza, il codice sorgente completo non è pubblico. In questo repository sono disponibili la presentazione architetturale e la relazione tecnica completa del progetto.*

## 🎯 Obiettivo del Progetto
L'obiettivo principale è stata la containerizzazione di applicazioni basate su INSANE e la loro successiva integrazione all'interno di un orchestratore. Il progetto ha previsto il deployment di un'applicazione demo e di un demone di rete (`nsnd`), permettendo la comunicazione ad alte prestazioni tramite memoria condivisa.

## 🛠 Tech Stack
* **Container & Orchestration:** Docker, Kubernetes (Kubeadm, K3s).
* **Networking & CNI:** Calico, Flannel, Multus, Macvlan.
* **OS & Low-level:** Linux (Ubuntu), HugePages (memoria condivisa Zero-Copy).
* **Protocolli:** TCP/IP, UDP.

## 🏗 Architettura e Scelte Progettuali

* **Pattern DaemonSet vs Sidecar:** Per l'implementazione su Kubernetes, ho progettato l'architettura utilizzando il pattern DaemonSet al posto del Sidecar. Questa scelta architetturale garantisce che un'istanza specifica del demone INSANE venga eseguita su tutti i nodi del cluster, ottimizzando le risorse a livello di nodo e semplificando la configurazione degli IP.
* **Gestione Avanzata della Rete:** Per garantire il corretto isolamento dei container Docker, ho abbandonato la modalità `--network host` in favore di reti `macvlan`, assegnando indirizzi IP specifici sulla rete fisica. Su Kubernetes, ho testato diverse soluzioni CNI (Flannel, Multus) per poi adottare **Calico**, configurando IP statici dedicati per i pod dei demoni.
* **Zero-Copy & HugePages:** Ho abilitato la comunicazione zero-copy tra i container mappando i volumi `/tmp` (per le socket Unix di controllo) e `/dev/hugepages` (per il trasferimento dati). Questo ha richiesto la configurazione di flag di sicurezza (es. `--privileged` su Docker) e la corretta allocazione delle risorse (richiesta esplicita di `hugepages-2Mi`) nei manifest YAML di Kubernetes.
* **Disaccoppiamento delle Configurazioni:** Ho utilizzato le **ConfigMaps** di Kubernetes per gestire in modo dinamico i file di configurazione (`nsnd.cfg`) dei demoni, evitando di dover ricostruire le immagini Docker ad ogni cambio di IP.

## 📊 Analisi delle Performance (Overhead di Virtualizzazione)
Ho condotto una campagna di test approfondita per valutare il tempo di trasmissione dei messaggi (da 512B a 1024B, fino a 100.000 pacchetti) confrontando tre ambienti: **Bare Metal, Docker e Kubernetes**. 

**Risultati chiave:**
1. **Scalabilità:** Il sistema scala linearmente all'aumentare del carico, senza crolli di performance improvvisi.
2. **Impatto della Virtualizzazione:** L'overhead introdotto dai container Docker è minimo e offre prestazioni quasi identiche al Bare Metal.
3. **Overhead K8s:** Kubernetes introduce la latenza più significativa a causa del livello di virtualizzazione aggiuntivo e dell'utilizzo del Container Network Interface (CNI).
4. **Comportamento dei Protocolli:** Il protocollo TCP risente maggiormente dell'aumento di latenza (rispetto a UDP) a causa dei meccanismi nativi di controllo del flusso.

## 📂 Contenuto del Repository
* `Relazione_Virtualizzazione_INSANE.pdf`: Documentazione tecnica completa del setup, dei test e delle sfide risolte.
* `Presentazione_Architettura.pptx`: Slide di sintesi sull'architettura e sui risultati dei benchmark.
