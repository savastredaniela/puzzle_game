# puzzle_game
15Puzzle

Funzionalità implementate:

    • L'app 15Puzzle è una tab bar application con 5 tab: Play-the-Game (per giocare al completamento del puzzle), Selection (per la selezione della foto del puzzle), Settings (per settare la difficoltà e le impostazioni di gioco), Profile (per inserire/visualizzare le informazioni del giocatore con il punteggio associato), Rules (per visualizzare le regole del gioco).
    • È ottimizzata per iPhone (non per iPad). Può girare su tutti gli iPhone con firmware 5.1 o superiore (quindi iPhone 4, 4s, 5). Per l'iPhone 5 si adatta in modalità "letter box" alle dimensioni del nuovo schermo Retina da 4 pollici.
    • L'app ruota nelle 3 posizioni di rotazione richieste (tranne UIInterfaceOrientationPortraitUpsideDown)
    • Si gestiscono le immagini dinamicamente, infatti, si possono caricare dalla scheda Selection (4 foto sono di esempio, configurabili da un file di Constants, e altre si possono caricare dalla Galleria Fotografica dell'iPhone).
    • La difficoltà del puzzle si setta dinamicamente dalla scheda Settings, scegliendo il numero di righe e di colonne della griglia di gioco. Inoltre, si può decidere se visualizzare i numeri e/o l'immagine di sfondo del puzzle.
    • Nella scheda Profile, viene settato il nome utente, azzerando i punteggi ogni volta che se ne inserisce uno nuovo. Quando si completa il puzzle, vengono accreditati 100 punti (configurabili nel file di Constants – chiave kScoresIncrement) all'utente registrato nel Profile.
    • Nella scheda Rules, vengono presentate le regole del gioco, con la pagina Web (configurabile) di Wikipedia sulle regole del 15Puzzle.
    • Per poter iniziare un nuovo gioco, dalla scheda Play-the-Game, basta riselezionare una nuova immagine dalla scheda Selection oppure scuotere l'iPhone per re-iniziare una nuova partita. Quando si esce dall'app, il gioco viene messo in pausa e, al rientro, viene chiesto all'utente se vuole continuare la partita oppure iniziarne una nuova.


Framework utilizzati: 

    • CoreGraphics, UIKit e Foundation: framework nativi iOS per la gestione della grafica e degli effetti di transizione e supporto alle gesture
    • AudioToolbox: framework per la gestione dell’audio (al momento non utilizzato)


Organizzazione del progetto
    • Tutte le immagini utilizzate nel progetto si trovano nella cartella Images
    • In Resources sono presenti tutti i file XIB per le interfacce grafiche e i file .plist per la localizzazione e properties
    • In Localizable.strings sono mappate tutte le etichette testuali per titoli dei pannelli, messaggi di errore/conferma, ecc.
    • Tutte le classi si trovano nella cartella Classes, organizzata nelle sottodirectory dei tab implementati (Profile, Settings, Selection, Board) e altre sottodirectory di utilità (Util, Navigation)
    • La configurazione del progetto (chiavi e costanti) si trovano nel file di intestazione Constants.h


Pattern di programmazione

Il pattern di programmazione utilizzato è quello Model-View-Controller (MVC): ogni interfaccia grafica utente è implementata in un file XIB (tranne quella della Board – griglia di gioco, che viene generata programmaticamente attraverso i Tiles). Ad ogni UI è associato un Controller, che ne gestisce la logica e lo stile (ove necessario). Per ciascuna sezione (tab) sono previsti dei modelli (come Board, Profile, Configuration, ecc.) per memorizzarne le relative informazioni.
Per la memorizzazione delle informazioni, non viene utilizzato un database (come SQLite), ma si utilizza la classe NSUserDefaults:
https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/Reference/Reference.html 


Dettagli tecnici

    • Navigation (Classes>>Navigation):
La navigazione tra le view dell’app avviene grazie al CustomTabBarController, che permette di entrare nei contesti (sezioni) dell’app. Il CustomNavigationBarController permette la navigazione (push/pop) tra le view di una sezione. Entrambi i controller hanno il supporto all’autorotate abilitato (shouldAutorotate), in modo da permettere all’app di ruotare nelle posizioni desiderate. Ogni controller implementa il metodo shouldAutorotateToInterfaceOrientation, per permettere la rotazione nelle 3 posizioni richieste.
Il caricamento delle view dell’app nella tab bar viene effettuato nell’AppDelegate (didFinishLaunchingWithOptions), quando viene inizializzata l’app.
Quando si esce dall’app (applicationDidEnterBackground oppure applicationWillTerminate) viene salvato lo stato del gioco (board saveBoard), in modo da poter riprendere la partita quando si rientra nell’app.

    • Configurazione (Classes>>Util):
La configurazione del progetto è mappata nel file di intestazione Constrants.h: qui è possibile settare anche i filename e le info delle immagini di default da utilizzare per il puzzle (kDefaultPhoto1, kDefaultPhoto2, kDefaultPhoto3, kDefaultPhoto4).
Nella directory Util, si trovano anche le classi Configuration (per il salvataggio delle impostazioni di gioco scelte dall’utente nel tab Settings, utilizzando la classe NSUserDefaults) e altre classi di utilità, come Util.m e WaitImageView (per il messaggio di attesa di caricamento).

    • Play-The-Game (Classes>>Board): il core del gioco è costituito dalla classe Board e dal controller BoardController. Il BoardController si occupa di inizializzare la vista con la griglia di gioco e tutti i pulsanti per l’interazione utente sul tab Play-The-Game.  Inoltre, gestisce la logica di interazione con l’accelerometro (per iniziare una nuova partita, basta agitare l’iPhone – da provare sul device reale e non sul Simulatore) e di settare il punteggio del giocatore nel caso di vittoria (Profile setScores:score).
La griglia di gioco viene realizzata dalla classe Board: in fase di inizializzazione viene caricata la configurazione scelta dall’utente (o quella di default) richiamando la classe Configuration. Qui viene caricata la foto scelta (in Selection) per realizzare il puzzle e si disabilita il multi-touch (in modo da intercettare il singolo tocco dell’utente sul tassello di puzzle selezionato). 
L’immagine selezionata viene divisa in tiles (tasselli), la cui logica è gestita dalla classe Tile (descritta in seguito).
La creazione di una nuova partita viene fatta con il metodo createNewBoard: in un thread vengono creati i tiles (tasselli) del puzzle.
Quando si inizia una partita (start) vengono mischiati i tasselli (metodo scambleBoard) e viene aggiornata la griglia di gioco (updateGrid). Nella classe Board vengono anche gestite le logiche di resume e pause della partita. I metodi updateGrid , moveTileFromCoordinate toCoordinate, createTiles, si occupano del disegno dei tasselli sulla griglia di gioco, di aggiornarla quando l’utente sposta i tasselli e di aggiornare l’immagine del puzzle a video.
Infine, nella classe Board viene definita anche la logica per il salvataggio e il restoring della partita, quando si entra/esce dall’app.

La classe Tile si occupa di definire il tassello del puzzle: a partire dalle coordinate definite dalla classe Board (in base allo spazio di gioco e alla posizione del tassello a video) non fa altro che riferirsi ad un’area precisa dell’immagine del puzzle selezionata (createPhotoImage). Inoltre, definisce la logica di interazione con il tassello, intercettando lo spostamento/movimento applicato dall’utente (con conseguente aggiornamento delle coordinate), da passare alla classe Board per l’aggiornamento della griglia di gioco.

NOTA. La generazione della griglia di gioco nel tab “Play-The-Game” viene effettuata programmaticamente. Non vi è un file XIB di UI, ma i tiles sono generati tutti dinamicamente a partire dall’immagine selezionata.

    • Selection (Classes>>Selection): 
La classe che si occupa di definire il layout del tab “Selection” (in cui si seleziona l’immagine del puzzle) è PhotoController.m. Qui è possibile selezionare la foto scegliendo tra quelle predefinite (e mappate in Constants.h)  metodo photoDefaultButtonAction, (che carica la modale PhotoDefaultController per la scelta delle foto precaricate nell’app)oppure tra quelle della Galleria Fotografica dell’utente (photoLibraryButtonAction). Dopo aver selezionato la foto, viene passata alla Board per essere utilizzata quando si inizia una nuova partita (selectPhoto).
La GUI associata al tab Selection è PhotoDefaultView.xib associata alla modale di caricamento delle foto precaricate nell’app (PhotoDefaultController). 

    • Settings (Classes>>Settings):
SettingsController si occupa di caricare la vista delle impostazioni di gioco (SettingsView.xib), in cui sono presenti i picker per la selezione delle righe/colonne della griglia del puzzle e gli switch per settare se visualizzare o meno immagini e numeri sul puzzle. Quando si interagisce con i controlli (switch/picker) appare il bottone per il salvataggio dell’impostazione che setta le impostazioni sulla Board (saveButtonAction). 

    • Profile (Classes>>Profile):
ProfileController è il controller che si occupa di caricare le informazioni del giocatore (nome e punteggio) nella vista relativa (ProfileView.xib). Quando viene visualizzata la vista (viewWillAppear), vengono caricate le info utente (se presenti) memorizzate nel model Profile e salvate nell’NSUserDefaults. Il salvataggio del nome giocatore avviene grazie al metodo savePlayerUsername, che si occupa di salvare lo username nel Profile e resettare il punteggio del nuovo giocatore. 

    • Rules (Classes>>Rules):
Il RulesViewController contiene la logica di caricamento della WebView che renderizza la pagina web con le regole di gioco (la URL è parametrica e mappata nel file di Constants – chiave kRulesWebURL). La GUI associata è RulesView.xib





















