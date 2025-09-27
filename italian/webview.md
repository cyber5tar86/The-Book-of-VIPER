## UIWebView e VIPER

In alcuni progetti potresti aver bisogno di utilizzare `UIWebView` per mostrare contenuto web. Anche se il suo utilizzo nell'architettura VIPER non è molto diverso dal suo utilizzo in altri pattern architetturali, ci sono alcune specifiche da considerare.

### Posizionamento di UIWebView nell'Architettura

`UIWebView` dovrebbe essere posizionata nel livello **View** del modulo VIPER. È un componente di presentazione che mostra contenuto all'utente, proprio come qualsiasi altra vista UI.

### Gestione del Contenuto Web

La logica per determinare quale contenuto caricare nella WebView dovrebbe risiedere nell'**Interactor**. Questo include:

- Costruzione degli URL
- Preparazione dei parametri delle richieste
- Gestione dell'autenticazione se necessaria

Il **Presenter** media tra la View contenente la WebView e l'Interactor, passando gli URL preparati alla View e gestendo gli eventi dalla WebView.

### Esempio di Implementazione

```objc
// View
@protocol WebViewModuleViewInput <NSObject>
- (void)loadURL:(NSURL *)url;
- (void)showLoadingIndicator;
- (void)hideLoadingIndicator;
@end

// Presenter
- (void)viewDidLoad {
    [self.view showLoadingIndicator];
    [self.interactor prepareContentURL];
}

- (void)didPrepareURL:(NSURL *)url {
    [self.view hideLoadingIndicator];
    [self.view loadURL:url];
}

// Interactor
- (void)prepareContentURL {
    NSURL *contentURL = [self.webContentService buildURLForContent:self.contentId];
    [self.output didPrepareURL:contentURL];
}
```

### Gestione della Navigazione Web

Per la navigazione web (link cliccati dall'utente), il **Router** dovrebbe essere responsabile delle decisioni di navigazione:

- Se aprire il link nella stessa WebView
- Se aprire un nuovo modulo
- Se aprire Safari o un browser esterno

```objc
// WebView Delegate nel View Controller
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    if (navigationType == UIWebViewNavigationTypeLinkClicked) {
        [self.output didRequestNavigationToURL:request.URL];
        return NO; // Prevenire il caricamento automatico
    }
    return YES;
}
```

### Vantaggi dell'Approccio VIPER per WebView

1. **Separazione delle Responsabilità**: La logica per preparare il contenuto web è separata dalla sua visualizzazione
2. **Testabilità**: Puoi facilmente mockare l'Interactor per testare diversi scenari di caricamento
3. **Flessibilità**: Facile cambiare la fonte del contenuto web senza modificare la View
4. **Riutilizzabilità**: Il modulo WebView può essere riutilizzato in diverse parti dell'applicazione