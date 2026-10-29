# Architettura di Grammatica Latina

## Scopo del documento

Questo documento descrive la struttura attuale dell'app: interfaccia, navigazione, contenuti, stato locale, acquisti, pubblicità e distribuzione. È una descrizione tecnica destinata al portfolio, non una copia del progetto privato.

Per ragioni di riservatezza non riporta credenziali, identificativi degli account e dei servizi, indirizzi degli ambienti, configurazioni di firma o dettagli operativi privati. Il codice e il catalogo dei testi non sono inclusi nel repository.

## 1. Vista d'insieme

Grammatica Latina è principalmente un'app mobile che esegue sul dispositivo la navigazione, la consultazione dei contenuti e la lettura. Il materiale didattico e letterario è incluso nell'app come dati strutturati. Non viene scaricato da un catalogo remoto a ogni apertura.

I servizi esterni intervengono soprattutto negli acquisti e nella pubblicità. Il progetto di sviluppo comprende anche un piccolo servizio API e strumenti di anteprima, ma questi non sono il motore della Biblioteca.


```text
testo e dati inclusi nell'app
             |
             v
schermate React Native <-- stato locale e progressi sul dispositivo
             |
             +--> RevenueCat <--> acquisti dello store
             |
             +--> Google AdMob
```

## 2. Tecnologie principali

| Area | Tecnologia | Ruolo |
| --- | --- | --- |
| Interfaccia mobile | React Native | Schermate, componenti e interazioni native |
| Ambiente applicativo | Expo | Sviluppo, risorse e integrazione con le piattaforme |
| Linguaggio | TypeScript | Tipi dei dati e controlli durante lo sviluppo |
| Navigazione | Expo Router | Schede principali e percorsi di dettaglio |
| Stato condiviso | React Context | Progressi di studio e stato Premium |
| Stato asincrono | TanStack Query | Caricamento e aggiornamento dei dati RevenueCat |
| Persistenza locale | AsyncStorage | Salvataggio dei progressi sul dispositivo |
| Acquisti | RevenueCat e store della piattaforma | Acquisto una tantum, verifica dei diritti e ripristino |
| Pubblicità | Google Mobile Ads / AdMob | Banner e annunci interstitial |
| Icone | Lucide e SVG | Icone dell'interfaccia |
| Servizio API di supporto | Node.js ed Express | Servizio separato, attualmente limitato a una verifica di disponibilità |
| Organizzazione del progetto | pnpm workspace | App, servizi e pacchetti condivisi |
| Build Android | Expo Application Services | Preparazione del pacchetto per Google Play |

## 3. Organizzazione del progetto

Il codice privato è organizzato in un workspace con componenti separati:

- **App mobile:** schermate, componenti condivisi, contenuti, lettore e integrazioni native.
- **Servizio API:** processo Express indipendente. Non gestisce attualmente il catalogo dei libri o i progressi di studio.
- **Pacchetti condivisi:** specifica OpenAPI, client generato e strumenti di validazione. Il workspace include anche un pacchetto per l'accesso al database, che non va confuso con un archivio attivo dei contenuti dell'app.
- **Ambiente di prototipazione:** spazio separato per le anteprime di interfaccia, non una funzionalità distribuita agli utenti.

La presenza di un pacchetto o di un servizio nel workspace non significa che ogni schermata ne dipenda. Le funzionalità di consultazione lavorano soprattutto su dati locali.

## 4. Navigazione e schermate

Expo Router organizza la navigazione a partire dalle schermate. Le cinque sezioni principali sono:

1. **Grammatica:** categorie, argomenti, spiegazioni ed esempi.
2. **Versioni:** autori, brani selezionati e analisi grammaticali.
3. **Strumenti:** paradigmi, declinazioni, coniugazioni e vocabolario.
4. **Biblioteca:** autori, opere e libri.
5. **Ricerca:** ricerca locale negli argomenti grammaticali.

La Biblioteca segue il percorso **autore → opera → libro → lettura**. Una schermata separata presenta Premium e consente acquisto e ripristino. Le schermate di dettaglio sono gestite da una navigazione a stack.

Alla radice dell'app, i provider condivisi rendono disponibili il client delle richieste asincrone, lo stato degli acquisti, i progressi di studio e la gestione delle aree sicure del dispositivo.

## 5. Contenuti e modello dei dati

Grammatica, brani analizzati, strumenti e Biblioteca sono definiti in moduli di dati TypeScript inclusi nel pacchetto dell'app.

Il modello della Biblioteca distingue tre livelli:

- **Autore:** nome, periodo, descrizione ed elenco delle opere.
- **Opera:** titolo, lingua, sottotitolo, forma del testo ed elenco dei libri.
- **Libro:** titolo, collegamenti all'autore e all'opera, lingua, descrizione, unità di contenuto e testo.

Latino e italiano sono rappresentati come edizioni distinte. La forma del testo può essere poesia, prosa o teatro. Le etichette dei conteggi possono essere specifiche dell'opera: il numero di sezioni del catalogo non deve essere confuso con la numerazione dei versi nel lettore.

Le spiegazioni grammaticali supportano testo, paragrafi, interruzioni di riga ed enfasi. Componenti condivisi interpretano questa formattazione e presentano le tabelle, compreso lo scorrimento orizzontale quando necessario.

Non è presente un sistema editoriale remoto per aggiornare il catalogo in tempo reale: i nuovi contenuti richiedono una nuova versione dell'app.

## 6. Lettore

Il lettore prepara gli elementi da visualizzare a partire dai dati del libro e usa una lista virtualizzata per evitare di montare tutto il testo contemporaneamente.

- **Poesia:** blocchi di cinque versi con intervallo numerico laterale. I versi mantengono gli a capo originali e il testo non è racchiuso in card.
- **Prosa:** sezioni numerate, con paragrafi e separazioni adatte alla lettura continua.
- **Teatro:** battute con nome del personaggio e testo associato.

Nel primo libro dell'Eneide latina, le righe introduttive precedenti ad “Arma virumque cano” restano visibili senza entrare nella numerazione dei versi.

La trasformazione per la visualizzazione non riscrive i testi sorgente. Gli accessi ai contenuti Premium vengono controllati prima di mostrare il libro.

## 7. Stato locale e progressi

Lo stato temporaneo delle schermate, come il testo di ricerca, è gestito con gli strumenti standard di React. I progressi di studio sono condivisi attraverso un contesto e salvati con AsyncStorage.

Il modello locale comprende argomenti visitati, tempo di studio e informazioni sulle giornate di studio. Non esiste attualmente una sincronizzazione di questi progressi tra dispositivi né un account applicativo dedicato per conservarli nel cloud.

La ricerca degli argomenti grammaticali filtra i dati locali. Non dipende da un motore di ricerca remoto e non effettua una ricerca globale in tutti i testi letterari.

## 8. Acquisto Premium

Premium è un acquisto una tantum. RevenueCat fornisce all'app le offerte disponibili e lo stato del diritto di accesso dell'utente, collegandosi allo store della piattaforma.

Il flusso principale è:

1. L'app carica le informazioni cliente e le offerte disponibili.
2. La schermata Premium presenta i vantaggi e il prezzo.
3. L'utente conferma l'acquisto tramite lo store.
4. L'app aggiorna le informazioni RevenueCat.
5. Le schermate leggono il nuovo stato e aggiornano l'accesso ai contenuti e la pubblicità.

Il ripristino degli acquisti utilizza lo stesso servizio e aggiorna nuovamente lo stato condiviso. La gestione dell'acquisto non passa dal piccolo servizio Express del workspace.

I primi quattro libri dell'Eneide, in entrambe le lingue, sono gratuiti. Il resto della Biblioteca richiede Premium. Il controllo di accesso nell'interfaccia non va presentato come cifratura o protezione DRM dei testi inclusi nell'app.

## 9. Pubblicità

L'app integra banner e interstitial tramite Google AdMob. Gli interstitial sono collegati alla chiusura dei contenuti, con condizioni di frequenza e distanza temporale per limitarne la comparsa.

Lo stato Premium viene condiviso anche con il sistema pubblicitario. Gli annunci vengono evitati per gli utenti Premium e l'inizializzazione viene rimandata finché il relativo stato non è noto.

L'integrazione pubblicitaria distingue gli ambienti nativi supportati dalle anteprime web ed Expo Go, dove i moduli nativi possono non essere disponibili. Un'anteprima non è quindi equivalente al comportamento della build Android distribuita.

## 10. Interfaccia e componenti condivisi

Colori e tipografia sono centralizzati. Le schermate usano il tema chiaro o scuro e componenti condivisi per elementi ricorrenti, come intestazioni, testo formattato, tabelle e pubblicità.

Le aree sicure, i margini e lo scorrimento tengono conto delle dimensioni del dispositivo. I contenuti lunghi vengono presentati con liste o schermate scorrevoli. L'obiettivo è separare il contenuto dalla sua impaginazione, senza duplicare i dati nelle schermate.

## 11. Servizi di supporto e rete

Il servizio Express del workspace espone attualmente una verifica di disponibilità. Non è un backend completo per utenti, acquisti o catalogo.

Un server Node.js separato può servire la pagina di presentazione, le risorse statiche e i manifest generati dal processo di build Expo. È un supporto alla distribuzione delle anteprime e delle risorse, distinto dall'installazione tramite Google Play.

I testi e i materiali inclusi non richiedono di essere scaricati a ogni consultazione. Acquisti, ripristino, aggiornamento dello stato Premium e pubblicità dipendono invece dai rispettivi servizi di rete: l'app non va descritta come interamente indipendente dalla connessione.

## 12. Build e distribuzione

Lo sviluppo utilizza Expo e Metro, con anteprime native e web. Le funzioni che dipendono da moduli nativi o dagli store devono essere verificate anche in una build appropriata, non soltanto nell'anteprima.

Per Android, la configurazione di produzione genera un **Android App Bundle (AAB)** destinato a Google Play. Google Play gestisce poi la distribuzione dei pacchetti installabili ai dispositivi. Non è previsto un APK scaricabile da questo repository.

Il repository portfolio non contiene workflow di rilascio, chiavi di firma, configurazioni dei servizi o pacchetti binari. La documentazione della disponibilità su Google Play indica una distribuzione prevista, non certifica che la pubblicazione sia già avvenuta.

## 13. Limiti e confini

L'architettura attuale non comprende un catalogo remoto aggiornabile in tempo reale, una sincronizzazione cloud dei progressi o un sistema di account applicativi. La struttura multipiattaforma di Expo non implica che ogni piattaforma sia già pronta per la pubblicazione.

Questa documentazione spiega come collaborano i componenti, ma non sostituisce una verifica di sicurezza, una dichiarazione sulla privacy o una certificazione della conformità degli store. Configurazioni e credenziali restano fuori dal repository pubblico.
