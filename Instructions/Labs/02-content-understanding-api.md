---
lab:
  title: Sviluppare un'applicazione client di comprensione dei contenuti
  description: Usare l'API REST Comprensione del Contenuto di Azure AI per sviluppare un'app client per un analizzatore.
---

# Sviluppare un'applicazione client di comprensione dei contenuti

In questo esercizio si usa Comprensione del Contenuto di Azure AI per creare un analizzatore che estrae informazioni dai biglietti da visita. Si svilupperà quindi un'applicazione client che usa l'analizzatore per estrarre i dettagli dei contatti dai biglietti da visita analizzati.

Questo esercizio richiede circa **30** minuti.

## Creare un hub e un progetto Fonderia Azure AI

Le funzionalità di Fonderia Azure AI che verranno usate in questo esercizio richiedono un progetto basato su una risorsa *hub* di Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure. Chiudere tutti i riquadri dei suggerimenti o di avvio rapido che vengono aperti al primo accesso e, se necessario, usare il logo **Fonderia Azure AI** in alto a sinistra per passare alla home page, simile all'immagine seguente (chiudere il riquadro **Aiuto** nel caso sia aperto):

    ![Screenshot del portale di Azure AI Foundry.](./media/ai-foundry-home.png)

1. Nel browser passare a `https://ai.azure.com/managementCenter/allResources` e selezionare **Crea nuovo**. Scegliere quindi l'opzione per creare una nuova **risorsa Hub IA**.
1. Nella procedura guidata **Crea un progetto**, immettere un nome valido per il progetto e selezionare l'opzione per crearne uno nuovo. Usare quindi il collegamento **Rinomina hub** per specificare un nome valido per il nuovo hub, espandere **Opzioni avanzate** e specificare le impostazioni seguenti per il progetto:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Nome hub**: un nome valido per l'hub
    - **Località**: Scegliere una delle località seguenti:\*
        - Australia orientale
        - Svezia centrale
        - Stati Uniti occidentali

    > \*Al momento della stesura di questo articolo, Comprensione del Contenuto di Azure AI è disponibile solo in queste aree.

    > **Suggerimento**: se il pulsante **Crea** è ancora disabilitato, assicurarsi di rinominare l'hub in un valore alfanumerico univoco.

1. Attendere che il progetto venga creato e quindi passare alla pagina di panoramica del progetto.

## Usare l'API REST per creare un analizzatore di Comprensione del Contenuto

Si userà l'API REST per creare un analizzatore in grado di estrarre informazioni dalle immagini dei biglietti da visita.

1. Aprire una nuova scheda del browser (mantenendo aperto il Portale Fonderia Azure AI nella scheda esistente). In una nuova scheda del browser, passare al [portale di Azure](https://portal.azure.com) su `https://portal.azure.com`, accedendo con le credenziali di Azure se richiesto.

    Chiudere eventuali notifiche di benvenuto per visualizzare la pagina iniziale del portale di Azure.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nell'abbonamento.

    Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. È possibile ridimensionare o ingrandire questo riquadro per ottimizzare l'esperienza d'uso.

    > **Suggerimento**: Ridimensionare il riquadro in modo da poter lavorare principalmente in Cloud Shell, ma visualizzare comunque le chiavi e l'endpoint nella pagina portale di Azure. Sarà necessario copiarli nel codice.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro Cloud Shell immettere i comandi seguenti per clonare il repository GitHub contenente i file di codice per questo esercizio (digitare il comando o copiarlo negli Appunti e quindi fare clic con il pulsante destro del mouse nella riga di comando e incollarlo come testo normale):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Suggerimento**: quando vengono incollati i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    La cartella contiene due immagini di biglietti da visita digitalizzate e i file di codice Python necessari per compilare l'app.

1. Nel riquadro della riga di comando di Cloud Shell, immettere il comando seguente per installare le librerie che verranno utilizzate:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice sostituire i segnaposto **YOUR_ENDPOINT** e **YOUR_KEY** con l'endpoint dei Servizi di Azure AI e una delle relative chiavi (copiati dal portale di Azure), quindi assicurarsi che **ANALYZER_NAME** sia impostato su `business-card-analyzer`.
1. Dopo aver sostituito i segnaposto, nell'editor di codice, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

    > **Suggerimento**: È ora possibile ingrandire il riquadro di Cloud Shell.

1. Nella riga di comando di Cloud Shell immettere il comando seguente per visualizzare il file JSON **biz-card.json** fornito:

    ```
   cat biz-card.json
    ```

    Scorrere il riquadro di Cloud Shell per visualizzare il codice JSON nel file, che definisce uno schema dell'analizzatore per un biglietto da visita.

1. Dopo aver visualizzato il file JSON per l'analizzatore, immettere il comando seguente per modificare il file di codice Python **create-analyzer.py** fornito:

    ```
   code create-analyzer.py
    ```

    Il file di codice Python viene aperto in un editor di codice.

1. Esaminare il codice che:
    - Carica lo schema dell'analizzatore dal file **biz-card.json**.
    - Recupera l'endpoint, la chiave e il nome dell'analizzatore dal file di configurazione dell'ambiente.
    - Chiama una funzione denominata **create_analyzer**, che non è attualmente implementata

1. Nella funzione **create_analyzer** trovare il commento **Create a Content Understanding analyzer** e aggiungere il codice seguente (prestando attenzione a mantenere il rientro corretto):

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. Esaminare il codice aggiunto, che:
    - Crea intestazioni appropriate per le richieste REST
    - Invia una richiesta HTTP *DELETE* per eliminare l'analizzatore, se esiste già.
    - Invia una richiesta HTTP *PUT* per avviare la creazione dell'analizzatore.
    - Controlla la risposta per recuperare l'URL di callback *Operation-Location*.
    - Invia ripetutamente una richiesta HTTP *GET* all'URL di callback per controllare lo stato dell'operazione fino a quando non è più in esecuzione.
    - Conferma l'esito positivo (o negativo) dell'operazione all'utente.

    > **Nota**: Il codice include alcuni ritardi intenzionali per evitare di superare il limite di frequenza delle richieste per il servizio.

1. Usare il comando **CTRL+S** per salvare le modifiche al codice, ma mantenere aperto il riquadro dell'editor di codice nel caso in cui sia necessario correggere eventuali errori nel codice. Ridimensionare i riquadri in modo da visualizzare chiaramente il riquadro della riga di comando.
1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per eseguire il codice Python:

    ```
   python create-analyzer.py
    ```

1. Esaminare l'output del programma, che dovrebbe indicare che l'analizzatore è stato creato.

## Usare l'API REST per analizzare il contenuto

Dopo aver creato un analizzatore, è possibile usarlo da un'applicazione client tramite l'API REST di comprensione dei contenuti.

1. Nella riga di comando di Cloud Shell immettere il comando seguente per modificare il file di codice Python **read-card.py** fornito:

    ```
   code read-card.py
    ```

    Il file di codice Python viene aperto in un editor di codice:

1. Esaminare il codice che:
    - Identifica il file di immagine da analizzare, con il valore predefinito **biz-card-1.png**.
    - Recupera l'endpoint e la chiave per la risorsa di Servizi di Azure AI dal progetto, usando le credenziali di Azure dalla sessione di Cloud Shell corrente per l'autenticazione.
    - Chiama una funzione denominata **analyze_card**, che non è attualmente implementata

1. Nella funzione **analyze_card** trovare il commento **Use Content Understanding to analyze the image** e aggiungere il codice seguente (prestando attenzione a mantenere il rientro corretto):

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. Esaminare il codice aggiunto, che:
    - Legge il contenuto del file di immagine
    - Imposta la versione dell'API REST Comprensione del Contenuto da usare
    - Invia una richiesta HTTP *POST* all'endpoint di Comprensione del Contenuto, indicando di analizzare l'immagine.
    - Controlla la risposta dell'operazione per recuperare un ID per l'operazione di analisi.
    - Invia ripetutamente una richiesta HTTP *GET* all'endpoint di Comprensione del Contenuto per controllare lo stato dell'operazione fino a quando non è più in esecuzione.
    - Se l'operazione ha esito positivo, salva la risposta JSON, quindi analizza il codice JSON e visualizza i valori recuperati per ogni campo specifico del tipo.

    > **Nota**: In questo semplice schema di biglietti da visita tutti i campi sono stringhe. Il codice seguente illustra la necessità di controllare il tipo di ogni campo in modo da poter estrarre valori di tipi diversi da uno schema più complesso.

1. Usare il comando **CTRL+S** per salvare le modifiche al codice, ma mantenere aperto il riquadro dell'editor di codice nel caso in cui sia necessario correggere eventuali errori nel codice. Ridimensionare i riquadri in modo da visualizzare chiaramente il riquadro della riga di comando.
1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per eseguire il codice Python:

    ```
   python read-card.py biz-card-1.png
    ```

1. Esaminare l'output del programma, che dovrebbe mostrare i valori per i campi nel biglietto da visita seguente:

    ![Biglietto da visita per Roberto Tamburello, dipendente di Adventure Works Cycles.](./media/biz-card-1.png)

1. Usare il comando seguente per eseguire il programma con un biglietto da visita diverso:

    ```
   python read-card.py biz-card-2.png
    ```

1. Esaminare i risultati, che dovrebbero riflettere i valori in questo biglietto da visita:

    ![Biglietto da visita per Mary Duartes, dipendente di Contoso.](./media/biz-card-2.png)

1. Nel riquadro della riga di comando di Cloud Shell usare il comando seguente per visualizzare la risposta JSON completa restituita:

    ```
   cat results.json
    ```

    Scorrere per visualizzare il codice JSON.

## Eseguire la pulizia

Al termine del lavoro con il servizio di comprensione dei contenuti, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Nel portale di Azure eliminare le risorse create in questo esercizio.
