# Grammatica Latina — Architecture / Architettura

## English

### Purpose of this document

This document describes the app's current structure: interface, navigation, content, local state, purchases, advertising, and distribution. It is a technical description for the portfolio, not a copy of the private project.

It deliberately excludes credentials, account and service identifiers, environment addresses, signing configuration, and private operational details. The source code and text collection are not included in this repository.

### File map and internal logic

Paths below refer to the private codebase; the files themselves are not published here. The main app lives in **artifacts/grammatica-latina/**. This map groups the main files rather than listing every literary text or asset.

~~~text
artifacts/grammatica-latina/
├── app/
│   ├── _layout.tsx                 App startup, shared providers and navigation stack
│   ├── (tabs)/
│   │   ├── _layout.tsx             Bottom-tab navigation
│   │   ├── index.tsx               Grammar categories
│   │   ├── testi.tsx               Authors for guided text analysis
│   │   ├── strumenti.tsx           Reference-tool menu
│   │   ├── biblioteca.tsx          Library author catalogue and Premium entry point
│   │   └── ricerca.tsx             Local grammar search
│   ├── categoria/[id].tsx          Topics within a grammar category
│   ├── argomento/[id].tsx          Grammar lesson, examples and tables
│   ├── testi/[autore].tsx          Analysed passages by author
│   ├── testo/[id].tsx              Passage and its analysis
│   ├── autore/[id].tsx             Library author and works
│   ├── opera/[id].tsx              Books belonging to a work
│   ├── libro/[id].tsx              Access check and poetry/prose/theatre reader
│   ├── strumenti/                 Paradigms, declensions, conjugations and dictionary screens
│   └── paywall.tsx                 Premium presentation, purchase and restoration
├── data/
│   ├── grammar.ts                 Grammar topics and explanations
│   ├── texts.ts                   Selected passages and analyses
│   ├── library.ts                 Author/work/book catalogue and lookup helpers
│   ├── library/                   Text modules for individual books and editions
│   ├── paradigms.ts               Verb paradigms
│   ├── declensions.ts             Declension data
│   ├── conjugations.ts            Conjugation data
│   └── vocabulary.ts              Dictionary entries
├── components/                    Reusable visual and behaviour components
├── context/ProgressContext.tsx     Shared, locally persisted study progress
├── hooks/useColors.ts             Palette selected from device appearance
├── hooks/useAds.ts                Advertising initialisation based on Premium status
├── lib/revenuecat.tsx             Purchase SDK, queries, mutations and subscription context
├── lib/premiumGating.ts           Free-book access rules
├── lib/ads.native.ts              Native advertising adapter
├── lib/ads.ts                     Non-native advertising adapter
├── constants/colors.ts            Light/dark colour tokens
├── constants/typography.ts        Shared text styles
├── assets/                        Fonts, images and store artwork
├── scripts/                       Content and build utilities
├── server/serve.js                Static-resource, manifest and landing-page server
├── package.json                   Dependencies and development commands
├── app.json                       Expo and native app configuration
└── eas.json                       Store-build configuration
~~~

**Key shared components:** SectionHero.tsx renders section headers; FormattedText.tsx handles formatted explanations; HorizontalTableScroller.tsx handles wide tables; SearchBar.tsx provides the search input. TopicCard.tsx, AuthorCard.tsx and TextCard.tsx support grammar and guided-text browsing. Library author, work and book cards are currently defined directly in their respective screens. AdBanner.tsx renders banners, while AdInterstitial.ts manages interstitial loading and display. ErrorBoundary.tsx and ErrorFallback.tsx provide an application error boundary and fallback view.

**How the pieces work together:**

1. **Startup:** app/_layout.tsx initialises RevenueCat, loads fonts, handles the splash screen and mounts providers for queries, purchases, progress, safe areas and navigation.
2. **Navigation:** Expo Router maps files in app/ to screens. A card passes an author, work, book or topic ID; the destination uses that ID to look up the corresponding local data.
3. **Content:** screens read data modules and hand the results to shared components. Grammar and guided passages use their own datasets; the Library uses data/library.ts and its imported book modules.
4. **Reading:** libro/[id].tsx checks access, transforms the book's text into display items and renders a virtualised list. Poetry becomes five-line blocks; prose and theatre follow separate rendering branches. The original data is not modified.
5. **Progress and search:** ProgressContext.tsx loads and saves study progress using AsyncStorage. Search filters grammar data locally; there is no server request for each query.
6. **Premium:** revenuecat.tsx exposes purchase state and operations through useSubscription(). paywall.tsx starts purchase or restore operations; successful operations refresh customer information. premiumGating.ts supplies free-access rules, which the book screens combine with Premium status.
7. **Advertising:** useAds.ts waits for a confirmed non-Premium status before initialisation. The platform adapters isolate native-only code; banner and interstitial components manage their own presentation. Content screens notify the interstitial controller when eligible content is closed.
8. **Theme:** useColors.ts selects the palette from constants/colors.ts. Screens combine it with shared typography and component styles.

**Outside the mobile app:** artifacts/api-server/src/app.ts and routes/ define the small Express service; lib/api-spec/, lib/api-client-react/ and lib/api-zod/ contain the API contract, generated client and validation package. These support the workspace but are not the source of the locally bundled reading content.

### 1. Overview

Grammatica Latina is primarily a mobile app that handles navigation, content browsing, and reading on the device. Educational and literary material is bundled with the app as structured data. It is not downloaded from a remote catalogue each time the app opens.

External services mainly support purchases and advertising. The development workspace also includes a small API service and preview tools, but these do not power the Library.

~~~text
Content and data bundled with the app
             |
             v
React Native screens <-- local state and on-device study progress
             |
             +--> RevenueCat <--> store purchases
             |
             +--> Google AdMob
~~~

### 2. Main technologies

| Area | Technology | Role |
| --- | --- | --- |
| Mobile interface | React Native | Screens, components, and native interactions |
| Application environment | Expo | Development, assets, and platform integration |
| Language | TypeScript | Data types and development-time checks |
| Navigation | Expo Router | Main tabs and detail routes |
| Shared state | React Context | Study progress and Premium status |
| Asynchronous state | TanStack Query | Loading and refreshing RevenueCat data |
| Local persistence | AsyncStorage | Saving progress on the device |
| Purchases | RevenueCat and platform stores | One-time purchase, entitlement checks, and restoration |
| Advertising | Google Mobile Ads / AdMob | Banners and interstitial ads |
| Icons | Lucide and SVG | Interface icons |
| Supporting API service | Node.js and Express | Separate service, currently limited to a health check |
| Project organisation | pnpm workspace | App, services, and shared packages |
| Android builds | Expo Application Services | Preparing the package for Google Play |

### 3. Project organisation

The private codebase is organised as a workspace with separate components:

- **Mobile app:** screens, shared components, content, reader, and native integrations.
- **API service:** an independent Express process. It does not currently manage the book catalogue or study progress.
- **Shared packages:** an OpenAPI specification, a generated client, and validation tools. The workspace also includes a database-access package, which should not be confused with an active content database for the app.
- **Prototyping environment:** a separate space for interface previews, not a feature shipped to users.

A package or service being present in the workspace does not mean that every screen depends on it. Content browsing mainly uses local data.

### 4. Navigation and screens

Expo Router organises navigation around the app's screens. There are five main sections:

1. **Grammar:** categories, topics, explanations, and examples.
2. **Text analysis:** authors, selected passages, and grammatical analysis.
3. **Tools:** paradigms, declensions, conjugations, and vocabulary.
4. **Library:** authors, works, and books.
5. **Search:** local search through grammar topics.

The Library follows the path **author → work → book → reading**. A separate screen presents Premium and supports purchases and restoration. Detail screens use stack navigation.

At the root of the app, shared providers supply the asynchronous query client, purchase state, study progress, and device safe-area handling.

### 5. Content and data model

Grammar, analysed passages, tools, and the Library are defined in TypeScript data modules bundled with the app.

The Library model has three levels:

- **Author:** name, historical period, description, and list of works.
- **Work:** title, language, subtitle, text form, and list of books.
- **Book:** title, links to its author and work, language, description, content units, and text.

Latin and Italian are represented as separate editions. Text can be poetry, prose, or theatre. Count labels may be specific to a work: catalogue section counts should not be confused with verse numbering in the reader.

Grammar explanations support text, paragraphs, line breaks, and emphasis. Shared components interpret this formatting and display tables, including horizontal scrolling when needed.

There is no remote editorial system for updating the catalogue in real time. New content requires an updated version of the app.

### 6. Reader

The reader prepares display items from each book's data and uses a virtualised list to avoid mounting the entire text at once.

- **Poetry:** five-line blocks with verse ranges beside the text. Original line breaks are preserved, and the text is not enclosed in cards.
- **Prose:** numbered sections, with paragraphs and spacing suited to continuous reading.
- **Theatre:** speeches with the character's name and associated text.

In the first book of the Latin Aeneid, the introductory lines before “Arma virumque cano” remain visible but are excluded from verse numbering.

Display transformations do not rewrite the source texts. Premium access is checked before a book is shown.

### 7. Local state and progress

Temporary screen state, such as the search query, uses standard React state management. Study progress is shared through a context and saved with AsyncStorage.

The local model includes visited topics, study time, and information about study days. There is currently no cross-device progress synchronisation or dedicated app account for storing progress in the cloud.

Grammar search filters local data. It does not depend on a remote search engine and does not search across all literary texts.

### 8. Premium purchase

Premium is a one-time purchase. RevenueCat provides available offerings and the user's entitlement status through its integration with the platform store.

The main flow is:

1. The app loads customer information and available offerings.
2. The Premium screen displays the benefits and price.
3. The user confirms the purchase through the store.
4. The app refreshes RevenueCat customer information.
5. Screens read the updated state and adjust content access and advertising.

Restoring purchases uses the same service and refreshes the shared state. Purchases are not handled by the workspace's small Express service.

The first four books of the Aeneid are free in both languages. The rest of the Library requires Premium. Interface access checks should not be described as encryption or DRM protection for bundled texts.

### 9. Advertising

The app integrates banners and interstitial ads through Google AdMob. Interstitials are tied to closing content, with frequency and timing conditions that limit how often they appear.

Premium status is also shared with the advertising system. Ads are avoided for Premium users, and initialisation is deferred until the relevant status is known.

The advertising integration distinguishes supported native environments from web and Expo Go previews, where native modules may be unavailable. A preview is therefore not equivalent to the behaviour of the distributed Android build.

### 10. Interface and shared components

Colours and typography are centralised. Screens use light or dark themes and shared components for recurring elements such as headers, formatted text, tables, and advertising.

Safe areas, margins, and scrolling account for device dimensions. Long content is presented in lists or scrollable screens. The aim is to separate content from layout without duplicating data across screens.

### 11. Supporting services and networking

The workspace's Express service currently exposes a health check. It is not a complete backend for users, purchases, or the catalogue.

A separate Node.js server can serve the landing page, static assets, and manifests generated by the Expo build process. It supports preview and asset distribution and is separate from installation through Google Play.

Bundled texts and materials do not need to be downloaded for each reading session. Purchases, restoration, Premium status updates, and advertising depend on their respective network services. The app should therefore not be described as fully independent of an internet connection.

### 12. Build and distribution

Development uses Expo and Metro, with native and web previews. Features that depend on native modules or stores must also be checked in an appropriate build, not only in a preview.

For Android, the production configuration generates an **Android App Bundle (AAB)** for Google Play. Google Play then handles distribution of installable packages to devices. This repository does not provide a downloadable APK.

The portfolio repository contains no release workflows, signing keys, service configuration, or binary packages. The Google Play availability statement describes planned distribution; it does not certify that publication has already taken place.

### 13. Limits and boundaries

The current architecture does not include a real-time remote catalogue, cloud progress synchronisation, or an app-account system. Expo's cross-platform structure does not imply that every platform is ready for publication.

This documentation explains how the components work together. It is not a substitute for a security review, privacy statement, or store-compliance assessment. Configuration and credentials remain outside the public repository.

---

## Italiano

### Scopo del documento

Questo documento descrive la struttura attuale dell'app: interfaccia, navigazione, contenuti, stato locale, acquisti, pubblicità e distribuzione. È una descrizione tecnica destinata al portfolio, non una copia del progetto privato.

Per ragioni di riservatezza non riporta credenziali, identificativi degli account e dei servizi, indirizzi degli ambienti, configurazioni di firma o dettagli operativi privati. Il codice e il catalogo dei testi non sono inclusi nel repository.

### Mappa dei file e logica interna

I percorsi seguenti si riferiscono al codice privato: i file non vengono pubblicati in questo repository. L'app principale si trova in **artifacts/grammatica-latina/**. La mappa raggruppa i file principali, senza elencare ogni testo letterario o risorsa.

~~~text
artifacts/grammatica-latina/
├── app/
│   ├── _layout.tsx                 Avvio, provider condivisi e navigazione a stack
│   ├── (tabs)/
│   │   ├── _layout.tsx             Navigazione delle schede inferiori
│   │   ├── index.tsx               Categorie grammaticali
│   │   ├── testi.tsx               Autori delle versioni analizzate
│   │   ├── strumenti.tsx           Menu degli strumenti di consultazione
│   │   ├── biblioteca.tsx          Catalogo autori e accesso alla schermata Premium
│   │   └── ricerca.tsx             Ricerca locale nella grammatica
│   ├── categoria/[id].tsx          Argomenti di una categoria grammaticale
│   ├── argomento/[id].tsx          Lezione, esempi e tabelle
│   ├── testi/[autore].tsx          Brani analizzati di un autore
│   ├── testo/[id].tsx              Brano e relativa analisi
│   ├── autore/[id].tsx             Autore della Biblioteca e sue opere
│   ├── opera/[id].tsx              Libri appartenenti a un'opera
│   ├── libro/[id].tsx              Verifica accesso e lettore poesia/prosa/teatro
│   ├── strumenti/                 Schermate paradigmi, declinazioni, coniugazioni e vocabolario
│   └── paywall.tsx                 Presentazione Premium, acquisto e ripristino
├── data/
│   ├── grammar.ts                 Argomenti e spiegazioni grammaticali
│   ├── texts.ts                   Brani selezionati e analisi
│   ├── library.ts                 Catalogo autori/opere/libri e funzioni di ricerca per ID
│   ├── library/                   Testi dei singoli libri e delle diverse edizioni
│   ├── paradigms.ts               Paradigmi verbali
│   ├── declensions.ts             Dati delle declinazioni
│   ├── conjugations.ts            Dati delle coniugazioni
│   └── vocabulary.ts              Voci del vocabolario
├── components/                    Componenti riutilizzabili di interfaccia e comportamento
├── context/ProgressContext.tsx     Progressi condivisi e salvati sul dispositivo
├── hooks/useColors.ts             Scelta della palette in base al tema del dispositivo
├── hooks/useAds.ts                Avvio della pubblicità in base allo stato Premium
├── lib/revenuecat.tsx             SDK acquisti, query, operazioni e contesto condiviso
├── lib/premiumGating.ts           Regole dei libri accessibili gratuitamente
├── lib/ads.native.ts              Adattatore pubblicitario nativo
├── lib/ads.ts                     Adattatore pubblicitario per ambienti non nativi
├── constants/colors.ts            Colori del tema chiaro e scuro
├── constants/typography.ts        Stili tipografici condivisi
├── assets/                        Font, immagini e materiali per lo store
├── scripts/                       Utilità per contenuti e build
├── server/serve.js                Server risorse statiche, manifest e pagina di presentazione
├── package.json                   Dipendenze e comandi di sviluppo
├── app.json                       Configurazione Expo e dell'app nativa
└── eas.json                       Configurazione delle build per lo store
~~~

**Componenti condivisi principali:** SectionHero.tsx presenta gli header; FormattedText.tsx gestisce le spiegazioni formattate; HorizontalTableScroller.tsx gestisce le tabelle larghe; SearchBar.tsx fornisce il campo di ricerca. TopicCard.tsx, AuthorCard.tsx e TextCard.tsx supportano grammatica e versioni. Le card di autori, opere e libri della Biblioteca sono invece definite direttamente nelle rispettive schermate. AdBanner.tsx presenta i banner, mentre AdInterstitial.ts gestisce caricamento e visualizzazione degli interstitial. ErrorBoundary.tsx ed ErrorFallback.tsx forniscono la gestione degli errori di rendering e la schermata di recupero.

**Come collaborano i file:**

1. **Avvio:** app/_layout.tsx inizializza RevenueCat, carica i font, gestisce la schermata iniziale e monta i provider per query, acquisti, progressi, aree sicure e navigazione.
2. **Navigazione:** Expo Router associa i file di app/ alle schermate. Una card passa l'ID di autore, opera, libro o argomento; la destinazione usa quell'ID per recuperare i dati locali corrispondenti.
3. **Contenuti:** le schermate leggono i moduli di dati e li presentano tramite componenti condivisi. Grammatica e versioni hanno dataset distinti; la Biblioteca usa data/library.ts e i moduli dei libri che questo importa.
4. **Lettura:** libro/[id].tsx verifica l'accesso, trasforma il testo in elementi da visualizzare e usa una lista virtualizzata. La poesia diventa blocchi di cinque versi; prosa e teatro seguono rami di rendering distinti. I dati originali non vengono modificati.
5. **Progressi e ricerca:** ProgressContext.tsx carica e salva i progressi con AsyncStorage. La ricerca filtra localmente i dati grammaticali, senza richieste al server per ogni parola cercata.
6. **Premium:** revenuecat.tsx espone stato e operazioni tramite useSubscription(). paywall.tsx avvia acquisto o ripristino; dopo un'operazione riuscita vengono aggiornate le informazioni cliente. premiumGating.ts definisce gli accessi gratuiti, che le schermate dei libri combinano con lo stato Premium.
7. **Pubblicità:** useAds.ts attende che l'utente risulti non Premium prima di inizializzare gli annunci. Gli adattatori separano il codice esclusivamente nativo; banner e interstitial gestiscono la propria visualizzazione. Le schermate notificano al controller interstitial la chiusura dei contenuti idonei.
8. **Tema:** useColors.ts seleziona la palette da constants/colors.ts. Le schermate la combinano con la tipografia condivisa e gli stili dei componenti.

**Fuori dall'app mobile:** artifacts/api-server/src/app.ts e routes/ definiscono il piccolo servizio Express; lib/api-spec/, lib/api-client-react/ e lib/api-zod/ contengono contratto API, client generato e pacchetto di validazione. Sono parti di supporto del workspace, non la sorgente dei contenuti di lettura inclusi nell'app.

### 1. Vista d'insieme

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

### 2. Tecnologie principali

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

### 3. Organizzazione del progetto

Il codice privato è organizzato in un workspace con componenti separati:

- **App mobile:** schermate, componenti condivisi, contenuti, lettore e integrazioni native.
- **Servizio API:** processo Express indipendente. Non gestisce attualmente il catalogo dei libri o i progressi di studio.
- **Pacchetti condivisi:** specifica OpenAPI, client generato e strumenti di validazione. Il workspace include anche un pacchetto per l'accesso al database, che non va confuso con un archivio attivo dei contenuti dell'app.
- **Ambiente di prototipazione:** spazio separato per le anteprime di interfaccia, non una funzionalità distribuita agli utenti.

La presenza di un pacchetto o di un servizio nel workspace non significa che ogni schermata ne dipenda. Le funzionalità di consultazione lavorano soprattutto su dati locali.

### 4. Navigazione e schermate

Expo Router organizza la navigazione a partire dalle schermate. Le cinque sezioni principali sono:

1. **Grammatica:** categorie, argomenti, spiegazioni ed esempi.
2. **Versioni:** autori, brani selezionati e analisi grammaticali.
3. **Strumenti:** paradigmi, declinazioni, coniugazioni e vocabolario.
4. **Biblioteca:** autori, opere e libri.
5. **Ricerca:** ricerca locale negli argomenti grammaticali.

La Biblioteca segue il percorso **autore → opera → libro → lettura**. Una schermata separata presenta Premium e consente acquisto e ripristino. Le schermate di dettaglio sono gestite da una navigazione a stack.

Alla radice dell'app, i provider condivisi rendono disponibili il client delle richieste asincrone, lo stato degli acquisti, i progressi di studio e la gestione delle aree sicure del dispositivo.

### 5. Contenuti e modello dei dati

Grammatica, brani analizzati, strumenti e Biblioteca sono definiti in moduli di dati TypeScript inclusi nel pacchetto dell'app.

Il modello della Biblioteca distingue tre livelli:

- **Autore:** nome, periodo, descrizione ed elenco delle opere.
- **Opera:** titolo, lingua, sottotitolo, forma del testo ed elenco dei libri.
- **Libro:** titolo, collegamenti all'autore e all'opera, lingua, descrizione, unità di contenuto e testo.

Latino e italiano sono rappresentati come edizioni distinte. La forma del testo può essere poesia, prosa o teatro. Le etichette dei conteggi possono essere specifiche dell'opera: il numero di sezioni del catalogo non deve essere confuso con la numerazione dei versi nel lettore.

Le spiegazioni grammaticali supportano testo, paragrafi, interruzioni di riga ed enfasi. Componenti condivisi interpretano questa formattazione e presentano le tabelle, compreso lo scorrimento orizzontale quando necessario.

Non è presente un sistema editoriale remoto per aggiornare il catalogo in tempo reale: i nuovi contenuti richiedono una nuova versione dell'app.

### 6. Lettore

Il lettore prepara gli elementi da visualizzare a partire dai dati del libro e usa una lista virtualizzata per evitare di montare tutto il testo contemporaneamente.

- **Poesia:** blocchi di cinque versi con intervallo numerico laterale. I versi mantengono gli a capo originali e il testo non è racchiuso in card.
- **Prosa:** sezioni numerate, con paragrafi e separazioni adatte alla lettura continua.
- **Teatro:** battute con nome del personaggio e testo associato.

Nel primo libro dell'Eneide latina, le righe introduttive precedenti ad “Arma virumque cano” restano visibili senza entrare nella numerazione dei versi.

La trasformazione per la visualizzazione non riscrive i testi sorgente. Gli accessi ai contenuti Premium vengono controllati prima di mostrare il libro.

### 7. Stato locale e progressi

Lo stato temporaneo delle schermate, come il testo di ricerca, è gestito con gli strumenti standard di React. I progressi di studio sono condivisi attraverso un contesto e salvati con AsyncStorage.

Il modello locale comprende argomenti visitati, tempo di studio e informazioni sulle giornate di studio. Non esiste attualmente una sincronizzazione di questi progressi tra dispositivi né un account applicativo dedicato per conservarli nel cloud.

La ricerca degli argomenti grammaticali filtra i dati locali. Non dipende da un motore di ricerca remoto e non effettua una ricerca globale in tutti i testi letterari.

### 8. Acquisto Premium

Premium è un acquisto una tantum. RevenueCat fornisce all'app le offerte disponibili e lo stato del diritto di accesso dell'utente, collegandosi allo store della piattaforma.

Il flusso principale è:

1. L'app carica le informazioni cliente e le offerte disponibili.
2. La schermata Premium presenta i vantaggi e il prezzo.
3. L'utente conferma l'acquisto tramite lo store.
4. L'app aggiorna le informazioni RevenueCat.
5. Le schermate leggono il nuovo stato e aggiornano l'accesso ai contenuti e la pubblicità.

Il ripristino degli acquisti utilizza lo stesso servizio e aggiorna nuovamente lo stato condiviso. La gestione dell'acquisto non passa dal piccolo servizio Express del workspace.

I primi quattro libri dell'Eneide, in entrambe le lingue, sono gratuiti. Il resto della Biblioteca richiede Premium. Il controllo di accesso nell'interfaccia non va presentato come cifratura o protezione DRM dei testi inclusi nell'app.

### 9. Pubblicità

L'app integra banner e interstitial tramite Google AdMob. Gli interstitial sono collegati alla chiusura dei contenuti, con condizioni di frequenza e distanza temporale per limitarne la comparsa.

Lo stato Premium viene condiviso anche con il sistema pubblicitario. Gli annunci vengono evitati per gli utenti Premium e l'inizializzazione viene rimandata finché il relativo stato non è noto.

L'integrazione pubblicitaria distingue gli ambienti nativi supportati dalle anteprime web ed Expo Go, dove i moduli nativi possono non essere disponibili. Un'anteprima non è quindi equivalente al comportamento della build Android distribuita.

### 10. Interfaccia e componenti condivisi

Colori e tipografia sono centralizzati. Le schermate usano il tema chiaro o scuro e componenti condivisi per elementi ricorrenti, come intestazioni, testo formattato, tabelle e pubblicità.

Le aree sicure, i margini e lo scorrimento tengono conto delle dimensioni del dispositivo. I contenuti lunghi vengono presentati con liste o schermate scorrevoli. L'obiettivo è separare il contenuto dalla sua impaginazione, senza duplicare i dati nelle schermate.

### 11. Servizi di supporto e rete

Il servizio Express del workspace espone attualmente una verifica di disponibilità. Non è un backend completo per utenti, acquisti o catalogo.

Un server Node.js separato può servire la pagina di presentazione, le risorse statiche e i manifest generati dal processo di build Expo. È un supporto alla distribuzione delle anteprime e delle risorse, distinto dall'installazione tramite Google Play.

I testi e i materiali inclusi non richiedono di essere scaricati a ogni consultazione. Acquisti, ripristino, aggiornamento dello stato Premium e pubblicità dipendono invece dai rispettivi servizi di rete: l'app non va descritta come interamente indipendente dalla connessione.

### 12. Build e distribuzione

Lo sviluppo utilizza Expo e Metro, con anteprime native e web. Le funzioni che dipendono da moduli nativi o dagli store devono essere verificate anche in una build appropriata, non soltanto nell'anteprima.

Per Android, la configurazione di produzione genera un **Android App Bundle (AAB)** destinato a Google Play. Google Play gestisce poi la distribuzione dei pacchetti installabili ai dispositivi. Non è previsto un APK scaricabile da questo repository.

Il repository portfolio non contiene workflow di rilascio, chiavi di firma, configurazioni dei servizi o pacchetti binari. La documentazione della disponibilità su Google Play indica una distribuzione prevista, non certifica che la pubblicazione sia già avvenuta.

### 13. Limiti e confini

L'architettura attuale non comprende un catalogo remoto aggiornabile in tempo reale, una sincronizzazione cloud dei progressi o un sistema di account applicativi. La struttura multipiattaforma di Expo non implica che ogni piattaforma sia già pronta per la pubblicazione.

Questa documentazione spiega come collaborano i componenti, ma non sostituisce una verifica di sicurezza, una dichiarazione sulla privacy o una certificazione della conformità degli store. Configurazioni e credenziali restano fuori dal repository pubblico.
