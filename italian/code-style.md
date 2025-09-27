Utilizzare lo stesso stile di codice da parte di tutti gli sviluppatori del team non è meno importante di un punto di vista unificato sull'architettura dell'applicazione e le responsabilità di oggetti diversi. Avere accordi per lavorare con lo stack VIPER non è un'eccezione.

### Regole Generali per il Modulo

- Il nome del modulo dovrebbe riflettere completamente il suo scopo. Il suffisso *Module* non dovrebbe essere incluso nel nome.
  
  **Esempio:** `MessageFolder`, `PostList`, `CacheSettings`.
  
- Tutti gli elementi del modulo sono divisi in sottocartelle all'interno di una cartella del modulo.

  **Esempio:**
  
  ```
  /NewPostUserStory
      /NewPostModule
          /Assembly
          /Interactor
          /Presenter
          /Router
          /View
      /ChooseAvatarModule
          /Assembly
          /Interactor
          /Presenter
          /Router
          /View
  ```
  
- Se dopo aver scritto il modulo alcuni dei suoi elementi rimangono inutilizzati, che siano classi, protocolli o metodi, vengono rimossi.
- Tutti gli helper sono situati nella sottocartella del loro livello.

- Tutti i metodi attraverso i quali i livelli comunicano tra loro dovrebbero essere sincroni.

  **Esempio:**
  
  ```objc
  @interface InteractorInput
  - (void)obtainDataFromNetwork;
  ...
  
  @interface InteractorOutput
  - (void)didObtainDataFromNetwork:(NSArray *)data;
  ...
  ```
  
- Tutti i metodi dei protocolli che coprono gli elementi del modulo dovrebbero iniziare con verbi - questo aiuta a indicare esplicitamente che ogni componente ha comportamento, non stato.
- I metodi che denotano l'inizio di un'azione dovrebbero iniziare con verbi che esprimono comando o richiesta (verbi imperativi).
- I metodi che denotano completamento di azione o processo, constatazione di fatto dovrebbero iniziare con verbo al passato.

### Livello Interactor
#### Classe `Interactor`

##### Denominazione
`<ModuleName>Interactor.h / <ModuleName>Interactor.m`

##### Regole Aggiuntive

- L'interactor non mantiene stato, solo dipendenze situate nella sua interfaccia pubblica.
- L'interactor mantiene riferimento weak al presenter. La variabile è chiamata `output`.

#### Protocollo `<InteractorInput>`
##### Denominazione
`<ModuleName>InteractorInput.h`

##### Descrizione
Contiene metodi per comunicare con l'interactor. L'interactor è coperto da questo protocollo dal punto di vista del presenter.

##### Esempi di Metodi

```objc
- (NSArray *)obtainNewsFromCache;
- (void)obtainMessageWithId:(NSString *)messageId;
- (void)performLoginWithUsername:(NSString *)username password:(NSString *)password;
```

#### Protocollo `<InteractorOutput>`
##### Denominazione
`<ModuleName>InteractorOutput.h`

##### Descrizione
Contiene metodi attraverso i quali l'interactor comunica con il livello superiore del modulo. Il presenter è solitamente coperto da questo protocollo.

##### Pattern Comuni dei Metodi:

- Nella maggior parte dei casi usiamo `did` come prefisso per ogni metodo - questo indica il ruolo passivo dell'interactor, che può eseguire una serie di azioni su richiesta e notificare sul loro completamento.

### Livello Presenter
#### Classe `Presenter`
##### Denominazione
`<ModuleName>Presenter.h / <ModuleName>Presenter.m`

##### Regole Aggiuntive

- A differenza di tutti gli altri elementi, il presenter ha stato. Si trova nell'extension privata.
- Il presenter mantiene riferimento weak alla view. La variabile è chiamata `view`.
- Il presenter mantiene riferimento strong al router. La variabile è chiamata `router`.
- Il presenter mantiene riferimento strong all'interactor. La variabile è chiamata `interactor`.

#### Protocollo `<ModuleInput>`
##### Denominazione
`<ModuleName>ModuleInput.h`

##### Descrizione
Contiene metodi attraverso i quali altri moduli o il suo contenitore possono comunicare con il modulo.

#### Protocollo `<ModuleOutput>`
##### Denominazione
`<ModuleName>ModuleOutput.h`

##### Descrizione
Contiene metodi attraverso i quali il modulo comunica con il suo contenitore o altri moduli.

### Livello Router
#### Classe `Router`
##### Denominazione
`<ModuleName>Router.h / <ModuleName>Router.m`

#### Protocollo `<RouterInput>`
##### Denominazione
`<ModuleName>RouterInput.h`

##### Descrizione
Contiene metodi di transizione ad altri moduli che possono essere chiamati dal presenter.

##### Pattern Comuni dei Metodi

- Per coerenza, tutti i metodi di questo protocollo iniziano con `open-` (aprire qualche modulo), `close-` (chiudere modulo) o `embed-` (incorporare modulo figlio nel contenitore).

### Livello View
#### Classi di Visualizzazione (ViewController, View, Cell)
##### Denominazione
`<ModuleName>View.h / <ModuleName>View.m`, `<ModuleName>ViewController.h / <ModuleName>ViewController.m`, `<ModuleName>Cell.h / <ModuleName>Cell.m`.

##### Regole Aggiuntive

- Tutti gli `IBOutlet` e `IBAction` (cioè tutte le dipendenze e interfacce) della View sono spostati nella sua interfaccia pubblica.
- La View mantiene riferimento strong al presenter. La variabile è chiamata `output`.

#### Protocollo <ViewInput>
##### Denominazione
`<ModuleName>ViewInput.h`

##### Descrizione
Contiene metodi attraverso i quali il presenter può controllare la visualizzazione o ottenere dati inseriti dall'utente.

#### Protocollo <ViewOutput>
##### Denominazione
`<ModuleName>ViewOutput.h`

##### Descrizione
Contiene metodi attraverso i quali la View notifica il presenter sui cambiamenti del suo stato.

##### Pattern Comuni dei Metodi
- Nella maggior parte dei casi usiamo `did` come prefisso per ogni metodo - questo indica il ruolo passivo della View.

### Livello Assembly
#### Classe Assembly
##### Denominazione
`<ModuleName>Assembly.h / <ModuleName>Assembly.m`

##### Regole Aggiuntive

- Solo il metodo che configura la View è spostato nell'interfaccia Assembly. Impostare l'intero stack VIPER sono dettagli di implementazione dell'assembly.

### Commenti

- Tutti i metodi dei protocolli e delle classi concrete sono necessariamente coperti da commenti javadoc dettagliati.
- I commenti sono scritti per le interfacce di tutti i componenti unici del modulo.
- Per le interfacce di tutti i componenti non unici (interactor, presenter, view), così come i loro protocolli, viene scritto lo stesso commento che descrive lo scopo dell'intero modulo.

### Test

- Per ogni componente del modulo viene creato un test case separato con nome come `<ModuleName>ViewControllerTests.m`.
- Cerchiamo di seguire la regola un test - una verifica.
- Per separare l'implementazione del test in blocchi logici, usiamo la notazione *given/when/then*.
- I test di metodi di protocolli diversi sono separati usando `#pragma mark -`.