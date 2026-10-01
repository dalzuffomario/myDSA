Documentazione Tecnica: Generatore DSA (Dichiarazione di Servizio)

Progetto: Portale web per la creazione, compilazione automatica e invio della Dichiarazione di Servizio (DSA).
Organizzazione: Croce Rossa Italiana - Corpo Militare Volontario (V° Centro di Mobilitazione Veneto).
Autore e Gestione IT: Mario Dal Zuffo.
1. Panoramica dell'Architettura

Il progetto è strutturato come un'applicazione web "serverless" (Single Page Application). Non necessita di un server web tradizionale a pagamento, ma sfrutta un'architettura ibrida divisa in due componenti principali:

    Frontend (Interfaccia Utente): Ospitato gratuitamente su GitHub Pages. Gestisce l'interfaccia, la validazione dei dati, il ritaglio della firma e la generazione materiale del file PDF in locale sul dispositivo dell'utente.

    Backend (Motore Email): Ospitato su Google Apps Script (Google Drive). Riceve il PDF generato dal frontend in formato testuale sicuro (Base64) e lo spedisce tramite l'infrastruttura di posta di Google.

Librerie Esterne Utilizzate (tramite CDN)

    pdf-lib ([unpkg.com/pdf-lib](https://unpkg.com/pdf-lib)): Il motore principale per leggere il PDF vuoto originale, scriverci sopra i testi e incollarvi la firma.

    SweetAlert2 (cdn.jsdelivr.net/npm/sweetalert2): Sostituisce i vecchi e brutti messaggi di sistema del browser con popup moderni, animati e personalizzati.

    Cropper.js ([cdnjs.cloudflare.com/ajax/libs/cropperjs](https://cdnjs.cloudflare.com/ajax/libs/cropperjs)): Libreria avanzata per la manipolazione delle immagini, utilizzata per ritagliare la firma in modo dinamico direttamente da smartphone o PC.

2. Struttura dei File su GitHub

La repository su GitHub funge da cuore dell'interfaccia utente. I file necessari al funzionamento sono rigorosamente:

    index.html: Il file principale che contiene tutto (HTML per la struttura, CSS per la grafica, JavaScript per la logica).

    Nuovo DSA_PDF.pdf: Il modulo DSA cartaceo digitalizzato, completamente vuoto. Funge da "foglio di base" su cui il sistema stampa le coordinate.

    banner_cri.png: L'immagine di intestazione mostrata in cima alla pagina web.

    griglia.html (Opzionale): Strumento di utilità creato per stampare il PDF con una griglia millimetrata, utile in caso di rivoluzione totale del modulo di base.

    README.md: Il file di testo in formato Markdown che conterrà questa esatta documentazione.

3. Logica di Funzionamento e Flusso Dati
A. Persistenza dei Dati (LocalStorage)

Per evitare che il militare debba compilare i propri dati anagrafici ogni volta, il sistema sfrutta il localStorage del browser.

    Auto-Lock: Al primo invio di una DSA, i dati anagrafici vengono salvati nella memoria del dispositivo e i campi vengono "congelati" (Read-Only).

    Sblocco: Per modificare un dato (es. cambio di residenza o di grado), l'utente deve spuntare la casella "Modifica dati anagrafici salvati". Togliendo la spunta, i nuovi dati vengono sovrascritti permanentemente in memoria.

B. Gestione Intelligente della Firma

Il trattamento della firma è stato ingegnerizzato per essere sicuro, leggero e persistente:

    L'utente scatta la foto alla propria firma autografa.

    Si apre la modale di Cropper.js, dove l'utente può zoomare e ritagliare via il tavolo o lo sfondo inutile.

    Il ritaglio viene trasformato in una stringa di testo crittografata (Base64) e salvata nel browser.

    Da quel momento, l'utente vedrà l'anteprima verde "Firma Salvata" e non dovrà mai più fotografarla per le DSA successive, a meno che non decida di rimuoverla usando l'apposito tasto.

C. Motore di Stampa (Generazione PDF)

Quando l'utente clicca su "Crea / invia DSA", il codice JavaScript:

    Scarica in memoria (ArrayBuffer) il file Nuovo DSA_PDF.pdf.

    Prende il dizionario delle Coordinate X, Y (const coordDef = { ... }).

    Usa la funzione drawText per scrivere le parole sulle righe corrette e drawCross per disegnare le 'X' dentro i quadratini delle opzioni.

    Calcolo Firma Sicura: La firma viene ridimensionata automaticamente. Il sistema impone un limite di altezza massima (MAX_HEIGHT = 55px). Indipendentemente da come la foto è stata scattata o ritagliata, il sistema la comprime proporzionalmente facendola rientrare in un "recinto invisibile", garantendo matematicamente che non copra mai le diciture sovrastanti del PDF (es. "Firma Responsabile / Datore di Lavoro").

D. Flusso di Invio

Il file PDF compilato viene esportato in un formato stringa e mandato tramite POST al server Google Apps Script.
Il backend è programmato per generare due email separate:

    Sempre: Invia una mail ufficiale all'Ufficio Operazioni (con destinatario, corpo e oggetto formattato secondo lo standard Grado Nome Cognome - Data Inizio / Data Fine - Servizio).

    Opzionale: Se l'utente clicca su "Sì, mandami una copia" dal popup finale di SweetAlert, il server manda una seconda email all'indirizzo personale dell'utente. Se l'utente clicca su "No, voglio solo scaricarla", il server invia solo all'Ufficio, mentre il browser scarica fisicamente il file PDF nella cartella "Download" del PC/Smartphone dell'utente.

4. Pannello Amministratore (Calibrazione Millimetrica)

Il codice include un ambiente di configurazione avanzato, nascosto agli utenti normali, che permette di spostare i testi sul PDF in caso di futuri aggiornamenti del modulo cartaceo.

    Accesso: Si clicca sul bottone grigio "Login amministratore" a fondo pagina.

    Credenziali fisse nel codice:

        Utente: admin

        Password: CDMPadova1!

    Funzionamento: Il pannello mostra tutte le voci scrivibili e permette di variare gli assi X (destra/sinistra), gli assi Y (alto/basso) e la larghezza massima in pixel della firma.

    Modalità Test: Cliccando su "Genera PDF di Prova con Griglia", il sistema sforna un PDF compilato con dati fittizi (Mario Rossi, ecc.) e sovrappone una griglia rossa e blu per verificare visivamente l'allineamento perfetto.

    Esportazione: Una volta trovata la calibrazione perfetta, il pulsante "Copia per GitHub" impacchetta i nuovi numeri in un blocco di codice JavaScript già formattato. L'amministratore IT dovrà solo fare copia-incolla di questo blocco all'interno del file index.html su GitHub per rendere le modifiche definitive per tutti gli utenti globali.

5. Guida al Backend (Google Apps Script)

Il codice residente su Google Drive gestisce l'elaborazione finale e la consegna. Se in futuro dovesse cambiare l'indirizzo dell'ufficio operazioni, la modifica andrà fatta esclusivamente su Google Drive, non su GitHub.

Struttura dello Script (Codice.gs):
JavaScript

function doPost(e) {
  // 1. Decodifica il JSON ricevuto da GitHub
  // 2. Trasforma il Base64 di nuovo in un file fisico (Blob PDF)
  // 3. Imposta l'indirizzo email dell'Ufficio Operazioni (modificare la variabile opsEmail)
  // 4. Invia la mail all'Ufficio Operazioni tramite MailApp.sendEmail
  // 5. Controlla il parametro "sendToUser": se è true, invia una seconda mail all'utente
  // 6. Restituisce uno status "success" per sbloccare il popup sul sito web
}

Note per l'aggiornamento Backend: Ogni singola volta che si modifica il testo dello script su Google, è obbligatorio ricaricare il progetto cliccando su Esegui deployment -> Gestisci deployment -> Modifica -> Nuova versione. Se non si crea una nuova versione, il sito web continuerà a puntare al codice vecchio.
