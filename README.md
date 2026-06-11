# Relazione Tecnica - Quoridor
## 1. Introduzione
Questo documento descrive le scelte progettuali, l'architettura e i requisiti del progetto Quoridor. Il software è un'implementazione in Python del celebre gioco da tavolo, che permette a due giocatori di sfidarsi su una scacchiera 9x9. L'obiettivo è raggiungere il lato opposto della plancia prima dell'avversario, gestendo strategicamente il movimento del proprio pedone e il posizionamento di muri per ostacolare l'altro giocatore.
## 2. Modello di dominio
Il modello di dominio definisce le entità core del gioco e le loro interazioni logiche.
Le entità fondamentali sono:
•	QuoridorGame: L'orchestratore della partita che gestisce i turni, le regole di movimento e la verifica della vittoria.
•	Board: La plancia di gioco che mantiene lo stato dei muri piazzati e si occupa della visualizzazione testuale.
•	Player: Rappresenta l'utente, memorizzando il nome, la posizione corrente sulla scacchiera e il numero di muri ancora disponibili (partendo da 10).

**Diagramma delle Classi**:

`classDiagram`
    `direction TB`

    class View {
        interfaccia visiva
        input utente
    }

    class QuoridorEngine {
        turno corrente
        stato della partita
    }

    class GameState {
        situazione riassuntiva
    }

    class Board {
        dimensione griglia
        muri orizzontali posizionati
        muri verticali posizionati
    }

    class Player {
        nome
        colore
        posizione corrente
        muri in riserva
        obiettivo di vittoria
    }

    class GameTimer {
        regola temporale
    }

    class PlayerTimer {
        tempo residuo
    }

    class Transcript {
        contatore mosse
    }

    class MoveRecord {
        numero sequenziale
        tipo di azione
        coordinata algebrica
        istante temporale
    }

    class PlayerSnapshot {
        fotografia posizione
        fotografia muri riserva
    }

    class GameRecord {
        data di inizio
        vincitore
    }

    %% Relazioni con Molteplicità e Verbi (Associazioni)
    View "1" --> "1" QuoridorEngine : interagisce con
    QuoridorEngine "1" --> "1" GameState : genera

    %% Composizioni (rombo pieno)
    QuoridorEngine "1" *-- "1" Board : governa
    QuoridorEngine "1" *-- "2..4" Player : coordina
    QuoridorEngine "1" *-- "1" GameTimer : controlla tempo tramite
    QuoridorEngine "1" *-- "1" Transcript : annota mosse su
    GameTimer "1" *-- "2..4" PlayerTimer : è composto da
    Transcript "1" *-- "*" MoveRecord : accumula
    MoveRecord "1" *-- "2..4" PlayerSnapshot : immortala

    %% Associazioni logiche semplici
    Player "1" --> "1" PlayerTimer : è limitato da
    Transcript "1" --> "1" GameRecord : salva archivio in

![quoeidorGame](./quoridorGame.png)

## 3. Requisiti specifici

## 3.1 Requisiti funzionali:

**- US 01 — Visualizzare lo stato della scacchiera**

Come giocatore voglio che il terminale stampi a schermo la scacchiera 9x9 aggiornata con tutti i pezzi, in modo da avere la situazione chiara prima di digitare la mia prossima mossa.

Criteri di accettazione:

Il programma stampa una griglia quadrata 9x9, provvista di un sistema di coordinate ai margini
Le caselle in cui non sono presenti pedoni sono riempite da un carattere neutro di riempimento
La posizione esatta dei giocatori è chiaramente indicata dalle etichette P1 (Giocatore 1) e P2 (Giocatore 2).
La rappresentazione visiva deve includere i muri posizionati sulla scacchiera, rendendo evidente la differenza grafica tra un muro orizzontale e uno verticale.
Accanto o sotto la griglia, un contatore testuale riporta l'inventario dei muri ancora a disposizione di entrambi i giocatori.
Il sistema stampa un messaggio esplicito per comunicare di chi è il turno corrente.
Al termine di ogni mossa andata a buon fine, il terminale ristampa l'intera visuale di gioco aggiornata.

**- US 2 — Muovere il pedone**

Come giocatore voglio inserire le coordinate della casella di destinazione sul terminale, in modo da spostare il mio pedone sulla scacchiera.

Criteri di accettazione:

Il giocatore inserisce un comando testuale
Lo spostamento è consentito solo in verticale o in orizzontale di una singola casella alla volta
Il pedone non può oltrepassare i muri piazzati né uscire dai confini della griglia 9x9.
Se la mossa inserita non è valida, il terminale mostra un messaggio di errore specifico (es. "Mossa non valida: muro presente") e richiede nuovamente l'inserimento, senza far saltare il turno.

**- US 3 — Piazzare i muri orizzontali e verticali**

Come giocatore voglio inserire le coordinate per piazzare un muro orizzontale o un muro verticale sul terminale, in modo da sbarrare la strada al mio avversario.

Criteri di accettazione

Il giocatore inserisce un comando che specifica la posizione e l'orientamento (orizzontale o verticale).
Il muro posizionato deve coprire esattamente l'interstizio tra due caselle adiacenti.
Il giocatore deve avere ancora muri a disposizione (contatore > 0), altrimenti il terminale mostra un messaggio di errore.
Il muro non può sovrapporsi né incrociarsi con muri già presenti sulla scacchiera.
Il posizionamento non deve chiudere del tutto i percorsi dell'avversario verso la sua linea di meta
Se la mossa è valida, il contatore dei muri del giocatore diminuisce di 1 e il turno passa all'avversario.

**- US 4 — Setup finale**

In qualità di giocatore voglio poter avviare l'intero gioco da un unico punto di accesso principale (classe main) e avere un'interfaccia testuale rifinita

Criteri di accettazione:
• Il file/classe principale (main.py) avvia correttamente il ciclo di gioco collegando tutte le classi precedenti.
• All'avvio del programma, viene stampato a schermo il titolo "Quoridor" in modo chiaro.
• È implementata una funzione per la pulizia dello schermo del terminale che mantenga la console ordinata tra un turno e l'altro.
• Il codice generale è stato revisionato per rimuovere eventuali stampe di debug superflue e garantire un flusso senza interruzioni.

**- US 5 — Verificare la vittoria**

Come giocatore voglio che il sistema controlli le condizioni di vittoria dopo ogni mossa, in modo da terminare la partita quando l'obiettivo viene raggiunto.

Criteri di accettazione:

Il controllo avviene automaticamente senza bisogno di comandi da parte dei giocatori.
Se il pedone del Giocatore 1 raggiunge una qualsiasi casella della riga 9 (o viceversa la riga 1 per il Giocatore 2), la partita si interrompe.
Il terminale stampa un messaggio chiaro di vittoria
Il gioco smette di accettare comandi di movimento e chiede se si vuole tornare al menu o uscire.

**- US 6 — Abbandonare la partita**

Come giocatore voglio poter digitare un comando di resa sul terminale durante il mio turno, in modo da ritirarmi dalla partita.

Criteri di accettazione:

Il giocatore digita un comando testuale specifico (es. "abbandona")
Il gioco interrompe immediatamente il turno corrente.
Sul terminale viene stampato un messaggio che dichiara vincitore l'avversario a tavolino per ritiro.
La partita in corso viene terminata.

**- US 7 — Mostrare un messaggio di aiuto**

Come giocatore voglio poter digitare un comando di aiuto sul terminale in qualsiasi momento, in modo da consultare la lista dei comandi disponibili.

Criteri di accettazione:

Digitando un comando specifico (es. help), il terminale stampa un elenco formattato con tutti i comandi di gioco supportati e una breve spiegazione.
L'utilizzo di questo comando non viene considerato come una mossa e non fa passare il turno all'avversario.
Dopo aver stampato l'aiuto, il sistema rimane in attesa del comando di gioco del giocatore corrente.

**- US 8 — Uscire dal gioco**

Come giocatore voglio poter digitare un comando di uscita sul terminale, in modo da chiudere l'applicazione in modo sicuro.

Criteri di accettazione:

Digitando un comando specifico (es. esci o quit), il programma termina immediatamente l'esecuzione.
Il terminale del Mac ritorna al suo prompt dei comandi standard, senza mostrare stringhe di errore o eccezioni del codice.

**- US 10 — Aggiunta di un limite di tempo per la partita**
In qualità di giocatore, voglio poter impostare un limite di tempo per i turni o per l'intera partita, in modo tale da rendere le sfide più veloci e competitive.

Criteri di accettazione:
• All'avvio del gioco, il sistema permette di scegliere se attivare o meno la modalità a tempo.
• Se attivato, il timer scorre in tempo reale e il tempo rimanente è visibile nell'interfaccia (o aggiornato a ogni input).
• Se il tempo a disposizione di un giocatore scade durante il suo turno, il sistema interrompe la partita e assegna la vittoria all'avversario.
• L'implementazione del timer non blocca l'inserimento dei comandi da tastiera.

**- US 11 — Replay di una partita conclusa e visualizzazione della trascrizione delle mosse**
In qualità di giocatore, voglio visualizzare e salvare la trascrizione di tutte le mosse effettuate, in modo tale da avere uno storico della partita e poter analizzare le strategie usate, e voglio poter caricare la trascrizione di una partita precedente e farne partire il replay, in modo tale da rivedere l'evoluzione della scacchiera passo dopo passo.

Criteri di accettazione:
• Il sistema registra ogni mossa valida (movimento del pedone o piazzamento di un muro) in una notazione standard (es. coordinata di arrivo o tipo/posizione del muro).
• Durante la partita, i giocatori possono digitare un comando (es. log o history) per stampare a schermo l'elenco delle mosse giocate fino a quel momento.
• Al termine della partita, la trascrizione completa viene salvata in un file di testo dedicato (es. match_history.txt).
• Il sistema prevede una modalità "Replay" accessibile all'avvio del programma.
• L'utente può specificare il file della partita da riprodurre.
• Il sistema riproduce le mosse in ordine cronologico, aggiornando la scacchiera a schermo.
• L'utente può avanzare alla mossa successiva premendo un tasto (es. Invio) o usare comandi per andare avanti/indietro.

**- US 12 — Modalità multiplayer a 4 giocatori**
In qualità di giocatore voglio poter impostare una partita con 4 partecipanti

Criteri di accettazione:
• All'avvio del gioco è possibile selezionare la modalità a 2 o a 4 giocatori.
• Nella modalità a 4, la scacchiera inizializza 4 pedoni posizionati al centro dei 4 lati del tabellone.
• Il contatore iniziale dei muri per ogni giocatore è impostato a 5 (invece di 10).
• Il sistema di turni ruota ciclicamente tra i 4 giocatori.
• La condizione di vittoria è aggiornata: ogni giocatore vince se raggiunge un qualsiasi punto del lato opposto a quello di partenza.

**- US 13 — Avvio del gioco con un interfaccia grafica avanzata**
In qualità di giocatore, voglio poter avviare il gioco con un'interfaccia grafica avanzata al posto del terminale

Criteri di accettazione:
• Il sistema supporta un flag da riga di comando per la modalità grafica. Digitando sul terminale del Mac il comando python3 main.py --gui, il gioco si avvia in una finestra dedicata usando la libreria PyQt6
• Senza il flag --gui, il gioco continua a funzionare perfettamente nel terminale (CLI).
• La GUI renderizza correttamente la griglia 9x9, i pedoni e i muri in 2D.
• I giocatori possono cliccare con il mouse per selezionare la casella di destinazione del pedone o l'intersezione in cui piazzare il muro.
• La logica di base (vittoria, regole di movimento, esaurimento muri) rimane invariata e condivisa tra CLI e GUI.

## 3.2 Requisiti non funzionali
• RNF1 (Usabilità): Il sistema deve garantire un'interfaccia chiara, ordinata e facilmente leggibile per l'utente durante tutte le fasi di gioco, fornendo un'esperienza coerente sia nella classica modalità a riga di comando (CLI) sia nella nuova visualizzazione grafica avanzata (GUI).
• RNF2 (Portabilità): Il sistema deve essere indipendente dalla piattaforma e poter essere eseguito senza problemi sui principali sistemi operativi, evitando conflitti di dipendenze anche per quanto concerne le librerie grafiche aggiuntive.
• RNF3 (Manutenibilità): Il codice sorgente deve rispettare gli standard di formattazione della community Python, risultando facilmente leggibile e predisposto a future estensioni 
• RNF4 (Efficienza): Il sistema deve validare le mosse in tempi non percettibili dall'utente, garantendo fluidità.
• RNF5 (Affidabilità/Robustezza): Il sistema deve gestire input errati o non validi mostrando messaggi di errore chiari senza causare crash. Inoltre, deve garantire la persistenza sicura e strutturata dei dati.

## 4. System Design

**4.1 Stile Architetturale**

Abbiamo adottato il pattern architetturale Model-View-Controller (MVC). Questo stile è particolarmente indicato per i sistemi interattivi,perche separa nettamente la logica di presentazione dei dati dalla logica di business:

•	Model : Include le classi che rappresentano il dominio dell'applicazione:

    •	Board (da board.py): Memorizza la struttura del campo da gioco (dimensioni) e i set (set()) delle coordinate dove sono fisicamente piazzati i muri orizzontali e verticali.
    •	Player (da player.py): Contiene lo stato del singolo partecipante: nome, colore, posizione corrente sulla griglia, numero di muri rimasti in riserva e riga/colonna obiettivo per la vittoria.
    •	GameTimer e PlayerTimer (da timer.py): Si occupano esclusivamente del calcolo numerico dei secondi a disposizione e della gestione dei preset (es. Bullet, Blitz).
    •	Transcript, GameRecord, MoveRecord, PlayerSnapshot (da transcript.py): Sono le strutture dati (principalmente dataclass) adibite alla memoria storica. Registrano le coordinate delle mosse, gli istanti temporali e creano un fascicolo salvabile della partita.
        
•	View : È responsabile della rappresentazione visuale dei dati:

    Interfaccia CLI (src/view/)
        •	View (da view.py): Si occupa interamente della stampa testuale. Formatta la scacchiera in ASCII art, gestisce i codici colore ANSI, mostra i menu e cattura l'input da tastiera (es. digitazione di "e4" o "PO e4").
    Interfaccia GUI (src/gui/)
        •	BoardWidget (da board_widget.py): Disegna fisicamente la griglia di Quoridor, le pedine arrotondate, i muri al neon e rileva i click del mouse tramite i metodi di PyQt/PySide.
        •	MainWindow (da main_window.py): Il contenitore principale che aggrega l'intero layout grafico della partita in corso.
        •	TimerWidget e TranscriptWidget (e MoveRow): Disegnano rispettivamente i display digitali del conto alla rovescia e la lista scorrevole delle mosse effettuate.
        •	ReplayDialog (da replay_dialog.py): La finestra secondaria adibita alla navigazione visuale delle partite salvate nello storico.
        
•	Controller : Gestisce la sequenza di interazioni con l'utente, accetta l'input testuale e lo trasforma in comandi per il Modello o la Vista:

    •	QuoridorEngine (da game_engine.py): È il nucleo delle regole (Business Logic). È qui che viene verificato se un muro è valido, se uno spostamento è legale (puoi_muovere) ed è qui che risiede l'algoritmo di ricerca (BFS in esiste_percorso) per impedire che un giocatore venga chiuso completamente.
    •	GameState (da game_engine.py): Una classe di "trasporto". Impacchetta la situazione calcolata dall'Engine per passarla in sicurezza alla View senza che quest'ultima debba spulciare nei calcoli complessi.
    •	QuoridorGame (da game.py): Gestisce il loop procedurale della modalità terminale. Coordina i passaggi: chiede l'input alla View, lo passa al QuoridorEngine per la convalida, elabora le eccezioni o i timeout, e comanda nuovamente alla View di aggiornare lo schermo.

**4.2 Principi di progettazione per il cambiamento**

L'architettura è stata pensata per minimizzare l'impatto di future modifiche, seguendo i principi di progettazione per il cambiamento:
•	Presentazione separata: La parte di codice relativa alla presentazione visiva è stata tenuta separata dal resto dell'applicazione per esempio è stata isolata la logica di calcolo dalla stampa a schermo.
•	Obiettivo di alta coesione: Abbiamo assegnato le responsabilità in modo tale da ottenere componenti con compiti ben definiti. Ad esempio, la classe Player ha l'unica responsabilità di gestire le proprie statistiche e verificare il raggiungimento della riga obiettivo (has_won()), delegando il resto al Controller.
•	Principio di Information Hiding: Ogni componente custodisce dei segreti al proprio interno (incapsulamento dei dati). I dettagli implementativi di basso livello, come i set contenenti le coordinate dei muri (h_muri, v_muri), sono nascosti all'interno di Board ed esposti al Controller solo per aggiornamenti controllati.

**4.3 Diagramma dei Package**

Nel nostro sistema, il package "Main" funge da client assoluto, mentre il cuore del gioco gestisce le entità base.

    flowchart TD
    
    subgraph src [" Sottosistema src/ (Core Application) "]

        
        Controller["game.py<br>(QuoridorGame)"]

        subgraph root [" Punto di Ingresso "]
            Main["main.py"]
        end
        
        subgraph Domain [" Modello e Dominio "]
            Board["board.py<br>(Board)"]
            Player["player.py<br>(Player)"]
        end
        
        subgraph Utils [" Utility "]
            Colors["colors.py<br>(Col)"]
        end
    end

    Main -.->|avvia| Controller
    Controller -.->|coordina| Board
    Controller -.->|coordina| Player
    Controller -.->|usa| Colors
    Board -.->|usa| Colors

![DMpackage](./DMpackage.png)

(Nota: le frecce tratteggiate indicano le dipendenze <<import>> o l'utilizzo delle API pubbliche tra i moduli).
4.4 Diagramma dei Componenti
Il diagramma dei componenti modella il sistema mostrando come le diverse parti (componenti) offrono o richiedono interfacce (API) per comunicare.
Nel nostro gioco CLI, QuoridorGame espone l'interfaccia principale play() al sistema operativo, e al suo interno richiede i servizi di gestione stato da Player e Board.



    componentDiagram

    component "main.py" as Main
    component "game.py" as Game
    component "board.py" as Board
    component "player.py" as Player

    Main ..> Game : <<requires>> \n play()
    Game ..> Board : <<requires>> \n draw()
    Game ..> Player : <<requires>> \n has_won() / place_wall()

### 4.5 Scelte Implementative e Tecnologiche
Per soddisfare i requisiti non funzionali sopra citati, il team ha preso le seguenti decisioni di progetto:

* **Interfaccia (View):** Per garantire l'usabilità (RNF1), si è scelto di utilizzare le sequenze di escape (colori ANSI) per differenziare visivamente i giocatori e i muri, e di effettuare il clear del terminale a ogni turno per simulare un'interfaccia statica.
* **Containerizzazione:** Per soddisfare il requisito di portabilità (RNF2), abbiamo optato per la containerizzazione tramite **Docker**, creando un ambiente di esecuzione isolato e identico su qualsiasi macchina.
* **Qualità del Codice:** Per assicurare la manutenibilità (RNF3), abbiamo integrato nel nostro flusso di lavoro il linter **Ruff**, che ha imposto e verificato automaticamente il rispetto delle convenzioni architetturali PEP 8.

8. Analisi retrospettiva
8.1 Sprint 0
Durante lo Sprint 0 il team si è focalizzato sulla configurazione dell'ambiente di sviluppo e sulla definizione dell'architettura di base.
•	Azioni correttive: Nella fase iniziale abbiamo riscontrato alcune difficoltà nella sincronizzazione dei branch su GitHub e nella gestione dei conflitti sui file principali.
•	Soluzione intrapresa: Per risolvere il problema, abbiamo stabilito una politica rigorosa di naming dei branch (es. feature/nome-funzionalità) e l'obbligo di effettuare una Pull Request con revisione obbligatoria da parte di un altro membro del team prima del merge nel main. Questo ha drasticamente ridotto gli errori di integrazione.
