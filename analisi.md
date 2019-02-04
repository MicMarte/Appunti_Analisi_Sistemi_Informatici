# Spiegazione testuale analisi

## Collecting semantcs

Analizzare le tracce è un'operazione molto onerosa, che richiede il salvataggio in memoria di tutte gli stati raggiunti da una singola traccia per ogni punto di programma. Oltre agli stati vanno memorizzati anche i collegamenti tra essi, in modo che ogni traccia abbia la propria catena di stati. Per semplificare ciò è utile usare un'approssimazione: la Collecting Semantincs. In sostanza ad ogni punto di programma è associato un bucket, che contiene gli stati che sono stati raggiunti da una qualsiasi delle tracce in quel punto. Questa approssimazione però perde il collegamento tra stati, rendendo impossibile ricostruire gli stati di una traccia. Questo potrebbe quindi includere tracce che non possono verificarsi, ma che sono ammesse dagli stati raggiungibili. *(vedi figura)*.

Un'ulteriore approssimazione è costituita dagli intervalli: per ogni punto di programma anziché memorizzare i singoli stati si memorizza solo l'intervallo più piccolo nel quale sono tutti racchiusi.

## CFG

Un CFG, control flow graph, è un grafo che rappresenta il flusso di controllo di un programma o di una procedura. È grafo orientato con possibili cicli, un punto di ingresso e un punto di uscita, i cui nodi sono basic block mentre gli archi sono le possibili transizioni tra blocchi. Un basic block è una sequenza di istruzioni di programma con un solo punto di ingresso e un solo punto di uscita. Per costruirlo quindi si crea un blocco ogni volta che c'è un salto o un etichetta di ingresso *(Vedi figura di esempio)*.

Un basic block può essere costituito da singole istruzioni o da una sequenza massimale di istruzioni che non può essere espansa, detto extended basic block. Ogni cammino sul CFG rappresenta una traccia di esecuzione e come lo stato evolve nel tempo.

È tuttavia possibile costruire anche un CFG per i punti di programma. Ognuno di loro rappresenta un nodo, mentre gli archi sono etichettati con l'istruzione che rappresentano. Questo CFG è usato per le analisi semantiche.

Nel CFG possono essere calcolati eventuali loop naturali, inner loop (che non contengono altri cicli), e varie funzioni sui nodi quali predecessori e successori. Inoltre è molto importante la relazione di **Dominanza**: Un nodo`v` domina `u` se ogni cammino dall'entry fino a `u` passa per `v`. L'ultimo dominatore è detto immediato, ed è unico per ogni nodo.

## Available expression

Un espressione del tipo `x<-e` si dice disponibile in un certo punto `P` di programma se
- È definita in un punto precedente a `P`
- `x` non è ridefinita tra il blocco e `P`
- Le variabili di `e` non sono rifefinite tra il blocco e `P`

Per ottimizzare il codice si può sostituire il calcolo di un'espressione disponibile con la deferenziazione della variabile in cui è stata assegnata

#### Esempio
```python
x = 7 + z
y = 3 * 8 + x
w = 7 + z
```
Nella terza riga l'espressione `7 + z` è dipsonibile, quindi anziché ricalcolarla si puù utilizzare il suo valore salvato in `x`. Il codice ottimizzato diventa quindi

```python
x = 7 + z
y = 3 * 8 + x
w = x
```

## Liveness

Una variabile è viva in ogni punto di programma tra la sua definizione e il suo ultimo uso. Per calcolarle partiamo dal fondo e saliamo fino ad un uso della variabile... Questa variabile resta viva finchè non troviamo la sua definizione, sopra la quale risulta non viva. Se una avariabile è viva vuol dire che il suo valore può essere letto.

Questa analisi risulta utile per cercare codice morto, variabili non inizializzate o registri liberi.

## True liveness

Poiché l'analisi di Liveness potrebbe eliminare le variabili morte, questo potrebbe rendere delle variabili che in precedenza erano vive morte, costringendoci quindi il ricalcolo della proprietà. Per evitare questo si applica una restrizione sulla analisi, effettuata tramite l'ausilio di una tabella delle "True live variables", inizializzata a `T`. In questo modo evitiamo il caso tipico di un programma

```
x = x + 1
```

Per cui la variabile `x` risulta viva, perché usata nel ramo destro, ma non veramente viva.

**TODO: Non ne sono ancora sicuro**

## Very busy expression

Un'espressione si dice Very Busy se in un certo punto di programma è stata sicuramente calcolata. In ogni percorso dal nodo iniziale alla fine un espressione è Very Busy se la variabile a cui è assegnata non è utilizzata prima di quel punto ed è il primo assegnamento di quella variabile e di quelle presenti nell'espressione. In un'ottimizzazione si può utilizzare questa analisi per anticipare dei calcoli o, per esempio, estrarli dai cicli; si possono inoltre anticipare tutte le inizializzazioni ad inizio programma.

## Reaching definitions

Le reaching definitions ci dicono, per ogni punto di programma, le variabili che sono disponibili e in che punto di programma sono state definite. Per ogni punto quindi si ha un insieme di coppie. A seconda del cammino percorso le definizioni disponibili potrebbero essere diverse, per questo ad una variabile possono corrispondere più punti di programma, rendendo così l'analisi di tipo possible.