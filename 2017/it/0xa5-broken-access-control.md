# A5:2017 Broken Access Control

| Threat agents/Attack vectors | Problematica di sicurezza  | Impatti |
| -- | -- | -- |
| Access Lvl : Sfruttabilità 2 | Diffusione 2 : Individuazione 2 | Tecnici 3 : Business |
| Sfruttare le vulnerabilità sul controllo degli accessi è una delle abilità principali degli attaccanti. Strumenti di [SAST](https://www.owasp.org/index.php/Source_Code_Analysis_Tools) e [DAST](https://www.owasp.org/index.php/Category:Vulnerability_Scanning_Tools) possono rilevare l'assenza di meccanismi di controllo degli accessi ma non possono verificarne l'efficacia se presenti. Tali meccanismi si possono identificare manualmente, o in modo automatizzato per ricercarne l'assenza in certi framework. | Le problematiche sul controllo degli accessi sono prevalenti in quanto sono difficili da identificare in modo automatizzato, e manca un testing funzionale efficace da parte degli sviluppatori. Il testing automatizzato, sia statico che dinamico, non è adatto per l'dentificazione di meccanismi di controllo degli accessi efficaci. Una procedura manuale è il modo migliore per rilevare meccanismi di controllo degli accessi mancanti o inefficaci, includendo metodi HTTP (GET vs PUT, ecc), controller, direct object references, ecc. | L'impatto tecnico è un attaccante che impersona utenti o amministratori, o utenti in grado di utilizzare funzionalità amministrative, o la creazione, la lettura, la modifica o la cancellazione di tutti i dati. L'impatto sul business dipende dai requisiti di protezione dell'applicazione e dei dati. |

## Sono vulnerabile?

Il controllo degli accessi fa rispettare una politica tale che gli utenti non possano agire al di fuori delle autorizzazioni previste. Fallimenti di questi controlli portano a divulgazione non autorizzata di informazioni, modifica o cancellazione di dati o svolgimento di funzionalità di business fuori dal dominio dell'utente. Vulnerabilità di questo tipo includono:

* Bypassare il controllo degli accessi modificando l'URL, lo stato interno dell'applicazione, la pagina HTML o più semplicemente utilizzando uno strumento di attacco alle API.
* Permettere che la chiave primaria venga modificata in quella del record di un altro utente, permettendo di leggere o modificare l'account di terzi.
* Elevazione di privilegi. Agire come un utente senza aver svolto il login, o come un admin quando si è svolto il login come un utente base.
* Manipolazione dei metadati, come svolgere il replay o modificare un JSON Web Token (JWT), un cookie o dei campi nascosti per elevare i privilegi, o abusare la funzionalità di invalidazione dei JWT.
* Configurazioni errate degli header CORS che permettono accessi non autorizzati alle API.
* Browsing forzato verso pagine protette da autenticazione come utente non autenticato o verso pagine privilegiate come utente standard. Accesso alle API senza controlli degli accessi per POST, PUT e DELETE.

## Come prevenire?

Il controllo degli accessi è efficace solo se applicato anche lato server o su API server-less, dove l'attaccante non può modificarne i controlli o i metadati.

* Ad eccezione delle risorse pubbliche, applicare la politica "deny by default".
* Centralizzare l'implementazione del controllo degli accessi e minimizzare l'utilizzo degli header CORS.
* Modellare il controllo degli accessi sul concetto di "ownership", anzichè permettere agli utenti di creare, leggere, modificare o cancellare qualsiasi record.
* Il modello dei dati deve rappresentare i requisiti di business in modo stringente.
* Disabilitare il directory listing dal web server e assicurarsi che non siano presenti metadati (es. .git) e file di backup.
* Memorizzare sul file di log gli errori dei meccanismi di controllo degli accessi, avvertire gli admin quando necessario (es. errori ripetuti)
* Implementare meccanismi di rate limiting su API e controller per minimizzare il danno da strumenti di attacco automatizzati.
* I token JWT devono essere invalidati dal server al momento del logout.
* Gli sviluppatori e lo staff di QA dovrebbero includere test funzionali unitari e di integrazione sui meccanismi di controllo degli accessi.

## Esempi di Scenari di Attacco

**Scenario #1**: L'applicazione utilizza dati non verificati in una chiamata SQL che accede alle informazioni sugli account:

```
  pstmt.setString(1, request.getParameter("acct"));
  ResultSet results = pstmt.executeQuery();
```

Un attaccante modifica semplicemente il parametro 'acct' nel browser per inviare un numero di account a suo piacimento. Se non verificato opportunamente, l'attaccante può accedere all'account di qualsiasi utente.

`http://example.com/app/accountInfo?acct=notmyacct`

**Scenario #2**: Un attaccante forza la navigazione verso degli URL. Sono richiesti dei privilegi amministrativi per accedere alla pagina di amministrazione.

```
  http://example.com/app/getappInfo
  http://example.com/app/admin_getappInfo
```

È presente una vulnerabilità sia che un utente non autenticato possa accedere a una delle pagine, sia che ci possa accedere un utente non amministratore.

## Riferimenti

### OWASP

* [OWASP Proactive Controls: Access Controls](https://www.owasp.org/index.php/OWASP_Proactive_Controls#6:_Implement_Access_Controls)
* [OWASP Application Security Verification Standard: V4 Access Control](https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project#tab=Home)
* [OWASP Testing Guide: Authorization Testing](https://www.owasp.org/index.php/Testing_for_Authorization)
* [OWASP Cheat Sheet: Access Control](https://www.owasp.org/index.php/Access_Control_Cheat_Sheet)

### Esterni

* [CWE-22: Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')](https://cwe.mitre.org/data/definitions/22.html)
* [CWE-284: Improper Access Control (Authorization)](https://cwe.mitre.org/data/definitions/284.html)
* [CWE-285: Improper Authorization](https://cwe.mitre.org/data/definitions/285.html)
* [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)
* [PortSwigger: Exploiting CORS misconfiguration](https://portswigger.net/blog/exploiting-cors-misconfigurations-for-bitcoins-and-bounties)
