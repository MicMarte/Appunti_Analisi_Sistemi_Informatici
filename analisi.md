# Spiegazione testuale analisi

## Collecting semantcs

Quando si analizza un programma è utile studiare le tracce; una traccia di esecuzione è una sequenza di stati raggiunti da una singola esecuzione in ogni punto di programma. Analizzare le tracce è un'operazione molto onerosa, che richiede il salvataggio in memoria di tutte gli stati raggiunti per ogni traccia. Oltre agli stati vanno memorizzati anche i collegamenti tra essi, in modo che ogni traccia abbia la propria catena di stati. Per semplificare ciò si può usare un'approssimazione: la Collecting Semantincs. In sostanza ad ogni punto di programma è associato un bucket che contiene tutti gli stati che sono stati raggiunti da una qualsiasi delle tracce in quel punto. Questa approssimazione però perde il collegamento tra stati, rendendo impossibile ricostruire la catena degli stati di una singola traccia. Questo potrebbe quindi includere tracce che non possono verificarsi, ma che sono ammesse dagli stati raggiungibili. *(vedi figura)*.

Un'ulteriore approssimazione è costituita dagli intervalli: per ogni punto di programma anziché memorizzare i singoli stati si memorizza solo l'intervallo più piccolo nel quale sono tutti racchiusi.

## CFG

Un CFG, control flow graph, è un grafo che rappresenta il flusso di controllo di un programma o di una procedura. È rappresentato mediante un grafo orientato con possibili cicli, avente un punto di ingresso e un punto di uscita, i cui nodi sono basic block mentre gli archi sono le possibili transizioni tra blocchi. Un basic block è una sequenza di istruzioni del programma con un solo punto di ingresso e un solo punto di uscita. Per costruirlo quindi si crea un blocco ogni volta che c'è un salto o un etichetta di ingresso *(Vedi figura di esempio)*.

Un basic block può essere costituito da singole istruzioni o da una sequenza massimale di istruzioni che non può essere espansa, detto extended basic block. Ogni cammino sul CFG rappresenta una traccia di esecuzione e ad ogni blocco si può associare una funzione, che esprime come lo stato evolve nel tempo una volta eseguito il blocco.

È tuttavia possibile costruire anche un CFG per i punti di programma. Ognuno di loro rappresenta un nodo, mentre gli archi sono etichettati con l'istruzione che rappresentano. Questo CFG è usato per le analisi semantiche.

Nel CFG possono essere calcolati eventuali loop naturali, inner loop (che non contengono altri cicli), e varie funzioni sui nodi quali predecessori e successori. Inoltre è molto importante la relazione di **Dominanza**: Un nodo`v` domina `u` se ogni cammino dall'entry fino a `u` passa per `v`. L'ultimo dominante è detto immediato, ed è unico per ogni nodo.

## Available expression

Un espressione del tipo `x<-e` si dice disponibile in un certo punto `P` di programma se
- È definita in un blocco `B` precedente a `P`
- `x` non è ridefinita tra `B` e `P`
- Le variabili di `e` non sono rifefinite tra `B` e `P`

Per ottimizzare il codice si può sostituire il calcolo di un'espressione disponibile con la deferenziazione della variabile in cui è stata assegnata

#### Esempio
```python
x = 7 + z
y = 3 * 8 + x
w = 7 + z
```
Nella terza riga l'espressione `7 + z` è dipsonibile, quindi anziché ricalcolarla si può utilizzare il suo valore salvato in `x`. Il codice ottimizzato diventa quindi

```python
x = 7 + z
y = 3 * 8 + x
w = x
```

## Liveness

Una variabile è viva in ogni punto di programma tra la sua definizione e il suo ultimo uso. Per calcolarle partiamo dal fondo e saliamo fino ad un uso della variabile; questa variabile resta viva finchè non troviamo la sua definizione, sopra la quale risulta non viva. Se una avariabile è viva vuol dire che il suo valore può essere letto. È un'analisi di tipo possible, poichè è sufficiente che sia viva in un ramo per renderla viva in quel punto di programma.

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

## Copy propagation

L'analisi di copy propagation ci dice, per ogni punto di programma, l'insieme di coppie di variabili che contengono lo stesso valore. È un'analisi di tipo forward e definite, che può essere utilizzata nelle ottimizzazione per evitare copie inutili. L'unica istruzione che genera una nuova coppia è `x <- y`, mentre ogni altra modifica di una di queste variabili uccide tutte le coppie in cui è presente.

## Intervalli

Questa analisi è solo semantica, in quanto ci dice cosa calcola un programma. Per ogni punto viene associato ad ogni variabile un intervallo che comprende i valori che questa può assumere. È perciò necessario definire nella semantica le operazioni tra intervalli per ogni possibile istruzione, incluse operazioni aritmetiche e di confronto.

In alcuni casi però i cicli possono essere ripetuti molte volte, per cui si utilizza il widening: se dopo due iterazioni un estremo di un intervallo continua a crescere lo si approssima con l'infinito. Una volta raggiunto il punto fisso è possibile applicare il narrowing per restringere l'intervallo.

## Segni

Quella dei segni è un'analisi simile agli intervalli, ma semplificata, in cui le variabili possono assumere, nel dominio astratto, solo i valori positivo o negativo, con lo 0 incluso.
