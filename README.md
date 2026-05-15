# Relazione Tecnica - Quoridor
## 1. Introduzione
Questo documento descrive le scelte progettuali, l'architettura e i requisiti del progetto Quoridor. Il software è un'implementazione in Python del celebre gioco da tavolo, che permette a due giocatori di sfidarsi su una scacchiera 9x9. L'obiettivo è raggiungere il lato opposto della plancia prima dell'avversario, gestendo strategicamente il movimento del proprio pedone e il posizionamento di muri per ostacolare l'altro giocatore.
## 2. Modello di dominio
Il modello di dominio definisce le entità core del gioco e le loro interazioni logiche.
Le entità fondamentali sono:
•	QuoridorGame: L'orchestratore della partita che gestisce i turni, le regole di movimento e la verifica della vittoria.
•	Board: La plancia di gioco che mantiene lo stato dei muri piazzati e si occupa della visualizzazione testuale.
•	Player: Rappresenta l'utente, memorizzando il nome, la posizione corrente sulla scacchiera e il numero di muri ancora disponibili (partendo da 10).
•	Col: Un modulo di utilità per gestire i colori ANSI nel terminale, migliorando l'usabilità dell'interfaccia.

**Diagramma delle Classi**:

`classDiagram`

    class QuoridorGame {
        +Board tabella
        +Player giocatore1
        +Player giocatore2
        +Player turno_corrente
        +play()
        +reset_game(mantieni_nomi)
        +puoi_muovere(p_partenza, p_arrivo)
        +muro_valido(tipo, r, c)
        +esiste_percorso(pos, r_target)
    }
    
    class Board {
        +int size
        +set h_muri
        +set v_muri
        +draw(p1, p2)
    }
    
    class Player {
        +String name
        +String symbol
        +String color
        +list pos
        +int muri
        +int target_row
        +place_wall() bool
        +has_won() bool
    }
    
    class Col {
        +String RED
        +String BLUE
        +String CYAN
        +String YELLOW
        +String GREEN
        +String MAGENTA
        +String RESET
    }

    QuoridorGame "1" *-- "1" Board : contiene
    QuoridorGame "1" *-- "2" Player : gestisce
    QuoridorGame ..> Col : usa
    Board ..> Col : usa

![quoeidorGame](./quoeidorGame.png)

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

## 3.2 Requisiti non funzionali
•	RNF1 (Usabilità): Il sistema deve garantire un'interfaccia a riga di comando (CLI) facilmente leggibile, sfruttando i colori ANSI e ripulendo lo schermo terminale a ogni turno per mantenere l'interfaccia ordinata.
•	RNF2 (Portabilità): Il sistema deve poter essere eseguito senza problemi di dipendenze su diversi sistemi operativi, prevedendo un ambiente isolato tramite container Docker.
•	RNF3 (Manutenibilità):Il codice è stato sottoposto a un processo di revisione continua (US 9). L'utilizzo del linter Ruff ha permesso di risolvere bug latenti e garantire che ogni file segua rigorosamente gli standard PEP 8, facilitando future estensioni del codice.
•	RNF4 (Efficienza): Il sistema deve convalidare le mosse e, in particolare, verificare i percorsi validi durante il piazzamento dei muri in tempi non percettibili dall'utente, garantendo un'esperienza di gioco fluida.
•	RNF5 (Affidabilità/Robustezza): Il sistema deve gestire correttamente eventuali input errati o non validi da parte dell'utente (es. coordinate scritte male, comandi inesistenti) mostrando un messaggio di errore chiaro e permettendo di reinserire il comando, senza mai causare il crash dell'applicazione.

## 4. System Design

**4.1 Stile Architetturale**

Abbiamo adottato il pattern architetturale Model-View-Controller (MVC). Questo stile è particolarmente indicato per i sistemi interattivi,perche separa nettamente la logica di presentazione dei dati dalla logica di business:
•	Model : Include le classi che rappresentano il dominio dell'applicazione: la classe Player (che mantiene posizione, muri residui e target) e le strutture dati interne della classe Board (set di muri posizionati) costituiscono il Modello.
•	View : È responsabile della rappresentazione visuale dei dati in particolare nel file board.py (tramite il metodo draw() che disegna la griglia) e supportata dal modulo colors.py per l'interfaccia a riga di comando (CLI).
•	Controller : Gestisce la sequenza di interazioni con l'utente, accetta l'input testuale e lo trasforma in comandi per il Modello o la Vista. La classe QuoridorGame funge da Controller: gestisce il ciclo dei turni (game loop), esegue la validazione complessa delle mosse (come l'algoritmo Breadth-First Search per verificare che i muri non blocchino il percorso) e aggiorna di conseguenza il Modello.

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


8. Analisi retrospettiva
8.1 Sprint 0
Durante lo Sprint 0 il team si è focalizzato sulla configurazione dell'ambiente di sviluppo e sulla definizione dell'architettura di base.
•	Azioni correttive: Nella fase iniziale abbiamo riscontrato alcune difficoltà nella sincronizzazione dei branch su GitHub e nella gestione dei conflitti sui file principali.
•	Soluzione intrapresa: Per risolvere il problema, abbiamo stabilito una politica rigorosa di naming dei branch (es. feature/nome-funzionalità) e l'obbligo di effettuare una Pull Request con revisione obbligatoria da parte di un altro membro del team prima del merge nel main. Questo ha drasticamente ridotto gli errori di integrazione.
