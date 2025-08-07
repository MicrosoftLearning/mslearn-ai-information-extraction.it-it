---
lab:
  title: Analizzare moduli da modelli personalizzati di Informazioni sui documenti di Azure AI
  description: Creare un modello di Informazioni sui documenti personalizzato per estrarre dati specifici dai documenti.
---

# Analizzare moduli da modelli personalizzati di Informazioni sui documenti di Azure AI

Si supponga che un'azienda richieda attualmente ai dipendenti di compilare manualmente i fogli degli ordini e di immettere i dati in un database. L'azienda ha richiesto di usare i servizi di intelligenza artificiale per migliorare il processo di immissione dei dati. Si decide di creare un modello di Machine Learning per leggere il modulo e produrre dati strutturati che possano essere usati per aggiornare automaticamente un database.

**Informazioni sui documenti di Azure AI** è un servizio di intelligenza artificiale di Azure che consente agli utenti di creare software di elaborazione dati automatizzato. Questo software può estrarre testo, coppie chiave/valore e tabelle dai documenti modulo usando il riconoscimento ottico dei caratteri (OCR). Informazioni sui documenti di Azure AI include modelli predefiniti per il riconoscimento di fatture, ricevute e biglietti da visita. Il servizio offre anche la possibilità di eseguire il training di modelli personalizzati. Questo esercizio si concentrerà sulla creazione di modelli personalizzati.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni simili usando più SDK specifici del linguaggio, tra cui:

- [Libreria client di Informazioni sui documenti di Azure AI per Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Libreria client di Informazioni sui documenti di Azure AI per Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Libreria client di Informazioni sui documenti di Azure AI per JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Questo esercizio richiede circa **30** minuti.

## Creare una risorsa di Informazioni sui documenti di Azure AI

Per usare il servizio Informazioni sui documenti di Azure AI, è necessaria una risorsa di Informazioni sui documenti di Azure AI o di Servizi Azure AI nella sottoscrizione di Azure. Si userà il portale di Azure per creare una risorsa.

1. In una scheda del browser aprire il portale di Azure in `https://portal.azure.com`, accedere con l’account Microsoft associato alla sottoscrizione di Azure.
1. Nella pagina iniziale del portale di Azure, passare alla casella di ricerca superiore e digitare **Document Intelligence**, quindi premere **Invio**.
1. Nella pagina **Informazioni sui documenti** selezionare **Crea**.
1. Nella pagina **Crea Informazioni sui documenti** creare una nuova risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: sottoscrizione di Azure.
    - **Gruppo di risorse**: creare o selezionare un gruppo di risorse
    - **Area**: Qualsiasi area disponibile
    - **Nome**: Un nome valido per la risorsa di Informazioni sui documenti
    - **Piano tariffario**: Gratuito F0 (*se non è disponibile un livello gratuito, selezionare* Standard S0).
1. Al termine della distribuzione, selezionare **Vai alla risorsa** per visualizzare la pagina **Informazioni generali** della risorsa.

## Preparare lo sviluppo di un'app in Cloud Shell

L'app di Traduzione testo verrà sviluppata usando Cloud Shell. I file di codice per l'app sono stati forniti in un repository GitHub.

1. Nel portale di Azure, usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova Cloud Shell nel portale di Azure, selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Ridimensionare il riquadro di Cloud Shell in modo da visualizzare sia la console della riga di comando che il portale di Azure. È necessario usare la barra di divisione per passare da un riquadro all'altro.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-ai-info/Labfiles/custom-doc-intelligence
    ```

## Raccogliere documenti per il training

Si useranno i moduli di esempio come questo per eseguire il training di un modello di test: 

![Immagine di una fattura usata in questo progetto.](./media/Form_1.jpg)

1. Nella riga di comando eseguire `ls ./sample-forms` per elencare il contenuto nella cartella **sample-forms**. Si noti che sono presenti file che terminano con **.json**e **.jpg** nella cartella.

    Si useranno i file con estensione **jpg** per eseguire il training del modello.  

    I **file .json** sono stati generati automaticamente e contengono informazioni sull'etichetta. I file verranno caricati nel contenitore di archiviazione BLOB insieme ai moduli.

1. Nel **portale di Azure** passare alla pagina **Panoramica** della risorsa, se non è già visualizzata. Nella sezione *Informazioni di base* prendere nota del **Gruppo di risorse**, dell'**ID sottoscrizione** e della **Posizione**. Questi valori saranno necessari nei passaggi successivi.
1. Eseguire il comando `code setup.sh` per aprire **setup.sh** in un editor di codice. Questo script verrà usato per eseguire i comandi dell’interfaccia della riga di comando di Azure necessari per creare le altre risorse di Azure necessarie.

1. Nello script **setup.sh** esaminare i comandi. Il programma:
    - Creerà un account di archiviazione nel gruppo di risorse di Azure
    - Caricherà file dalla cartella *sampleforms* locale in un contenitore denominato *sampleforms* nell'account di archiviazione
    - Stamperà un URI di firma di accesso condiviso

1. Modificare le dichiarazioni delle variabili **subscription_id**, **resource_group** e **posizione** con i valori appropriati per la sottoscrizione, il gruppo di risorse e il nome della posizione in cui è stata distribuita la risorsa di Informazioni sui documenti.

    > **Importante**: Per la stringa **location**, assicurarsi di usare la versione del codice della posizione. Ad esempio, se la posizione è "Stati Uniti orientali", la stringa nello script deve essere `eastus`. Si può notare che la versione è il pulsante **Visualizzazione JSON** sul lato destro della scheda **Informazioni di base** del gruppo di risorse nel portale di Azure.

    Se la variabile **expiry_date** si trova nel passato, aggiornarla a una data futura. Questa variabile viene usata durante la generazione dell'URI della firma di accesso condiviso. In pratica è necessario impostare una data di scadenza appropriata per la firma di accesso condiviso. Per altre informazioni sulla firma di accesso condiviso, fare clic [qui](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works).  

1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

1. Immettere i comandi seguenti per rendere eseguibile lo script e per eseguirlo:

    ```PowerShell
   chmod +x ./setup.sh
   ./setup.sh
    ```

1. Al termine dello script, riesaminare l’output visualizzato.

1. Nel portale di Azure, aggiornare il gruppo di risorse e verificare che contenga l’account di archiviazione di Azure appena creato. Aprire l’account di archiviazione e nel riquadro a sinistra selezionare **Browser di archiviazione**. In Browser archiviazione espandere quindi i **contenitori BLOB** e selezionare il contenitore **sampleforms** per verificare che i file siano stati caricati dalla cartella locale **custom-doc-intelligence/sample-forms**.

## Eseguire il training del modello usando Document Intelligence Studio

Ora si eseguirà il training del modello usando i file caricati nell’account di archiviazione.

1. Aprire una nuova scheda del browser e passare a Document Intelligence Studio in `https://documentintelligence.ai.azure.com/studio`. 
1. Scorrere verso il basso fino alla sezione **Modelli** personalizzati e selezionare il riquadro **Modello di estrazione personalizzato**.
1. Se richiesto, accedere con le credenziali di Azure.
1. Se viene chiesto quale risorsa di Informazioni sui documenti di Azure AI usare, selezionare la sottoscrizione e il nome della risorsa usati durante la creazione della risorsa Informazioni sui documenti di Azure AI.
1. In **Progetti personali** creare un nuovo progetto con la configurazione seguente:

    - **Immettere i dettagli del progetto**:
        - **Nome progetto**: Nome valido per il progetto
    - **Configura risorsa del servizio**:
        - **Sottoscrizione**: sottoscrizione di Azure.
        - **Gruppo di risorse**: Gruppo di risorse in cui è stata distribuita la risorsa di Informazioni sui documenti
        - **Risorsa di Informazioni sui documenti** Risorsa di Informazioni sui documenti (selezionare l'opzione *Imposta come predefinita* e usare la versione API predefinita)
    - **Connetti origine dati training**:
        - **Sottoscrizione**: sottoscrizione di Azure.
        - **Gruppo di risorse**: Gruppo di risorse in cui è stata distribuita la risorsa di Informazioni sui documenti
        - **Account di archiviazione**: L'account di archiviazione creato dallo script di installazione (selezionare l'opzione *Imposta come predefinito*, selezionare il contenitore BLOB `sampleforms` e lasciare vuoto il percorso della cartella)

1. Quando il progetto viene creato, nella parte superiore destra della pagina selezionare **Esegui il training** per eseguire il training del modello. Usare le configurazioni seguenti:
    - **ID modello**: Un nome valido per il modello (*il nome dell'ID modello sarà necessario nel passaggio successivo*)
    - **Modalità di compilazione**: Modello.
1. Selezionare **Vai a Modelli**.
1. Il training può richiedere del tempo. Attendere che lo stato sia **Operazione completata**.

## Testare il modello personalizzato di Informazioni sui documenti

1. Tornare alla scheda del browser contenente il portale di Azure e Cloud Shell. Nella riga di comando eseguire il comando seguente per passare alla cartella contenente i file di codice dell'applicazione:

    ```
    cd Python
    ```

1. Installare il pacchetto Informazioni sui documenti eseguendo il comando seguente:

    ```
    python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    ```
   code .env
    ```

1. Nella pagina **Panoramica** per la risorsa di Informazioni sui documenti nel riquadro contenente il portale di Azure selezionare **Fare clic qui per gestire le chiavi** per visualizzare l'endpoint e le chiavi per la risorsa. Modificare quindi il file di configurazione con i valori seguenti:
    - Endpoint di Informazioni sui documenti
    - Chiave di Informazioni sui documenti
    - ID modello specificato durante il training del modello

1. Dopo aver sostituito i segnaposto, nell'editor di codice, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

1. Aprire il file di codice per l'applicazione client (`code Program.cs` per C#, `code test-model.py` per Python) ed esaminare il codice incluso, verificando in particolare che l'immagine nell'URL faccia riferimento al file in questo repository GitHub sul Web. Chiudere il file senza apportare modifiche.

1. Nella riga di comando immettere il comando seguente per eseguire il programma:

    ```
   python test-model.py
    ```

1. Visualizzare l’output e osservare come fornisca nomi di campi come `Merchant` e `CompanyPhoneNumber` per il modello.

## Eseguire la pulizia

Dopo aver finito con la risorsa Azure, non dimenticare di eliminarla nel [portale di Azure](https://portal.azure.com/?azure-portal=true) per evitare ulteriori modifiche.

## Ulteriori informazioni

Per altre informazioni sul servizio Document Intelligence, vedere la [documentazione di Document Intelligence](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true).
