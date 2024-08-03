# Sviluppo del test

Ho cercato di seguire tutte le indicazioni per lo sviluppo del test.

Non ho potuto usare docker perchè andava in conflitto con qualche software aziendale sul PC su cui sviluppavo (ho utilizzato il PC aziendale poichè sprovvisto di PC personale). Ci ho perso un sacco di tempo per capire con cosa andava in conflitto ma poi ho rinunciato perchè stavo perdendo tempo.

Al posto del docker ho utilizzato WSL (è possibile importare l'immagine con WSL --import).

L'immagine WSL contiene già tutto l'occorrente per testare le funzionalità.

Ho utilizzato Ubuntu 22 su cui ho installato e configurato MySql, Apache2, Php, Lumen, Redis.

Ho modificato il virtualhost di Apache in modo tale che utilizzasse Lumen.

Ho creato le tabelle su MySql, comprese di indici e relazioni.

Ho installato i moduli necessari per l'utilizzo di PHP.

Ho configurato Lumen per l'accesso al DB.

Ho configurato Ubuntu,MySql,Apache2,Php e Lumen perchè utilizzino tutti la stessa timezone.

## Sviluppo

Nel mio caso, l'indirizzo IP della WSL è 172.26.254.11 ma nel vostro caso potrebbe essere differente, quindi potrebbe essere necessario modificare le URL degli endpoint.

Gli endpoint di create e update possono essere chiamati sia da CURL che da pagina PHP che ho realizzato appositamente per lo scopo.

Tutti i files risiedono in _/var/www/lumen_api/_ (scusate i permessi 777 ma non volevo perdere tempo con l'accesso via FTP).

Ad ogni errore nelle chiamate endpoint (token errato, formato dati errato, stringhe vuote, etc) viene restituito un json contenente l'errore, come ad esempio: _{"error":"Numeric Id expected"}_.

Ad ogni azione andata a buon fine viene restituito un json con un messaggio, tipo _{"message":"Created", "newid"=>5}_.
  
Ho utilizzato un Token come autorizzazione alle operazione di creazione, modifica e cancellazione. Il token è molto semplice in realtà, è un banale MD5 della data odierna (in php: _md5(date("Ymd")_).

Utilizzare il file _token_generate.php_ per visualizzare il token da chiamare con i CURL.

La creazione del profile ha la rimozione del prefisso internazionale, sia esso scritto con il + che con lo 00 iniziali (per farlo ho realizzato una funzione con RegEx).
 
# Chiamate agli endpoint

La chiamata all'endpoint http://172.26.254.11/ visualizza le due tabelle (profiles e profile_attributes) per agevolare la visualizzazione di ciò che è contenuto nel database.
 
**Visualizzazione di tutti i profiles** (compresi i relativi attributes):  
http://172.26.254.11/profile/
 
**Visualizzazione di uno specifico profile** (compresi i suoi relativi attributes):  
http://172.26.254.11/profile/1/
 
**Creazione di un profile**  
Si può anche usare il file create_profile.php (con le dovute modifiche):  
```
curl -X PUT http://172.26.254.11/profile/ 
     -H 'Content-Type: application/json' 
     -H 'Accept: application/json' 
     -H 'X-Token: TOKEN' 
     -d '{"name": "Paolo", "lastname": "Rossi", "phone": "+413311002167"}'
```

**Modifica di un profile** (indicare l'ID)  
Si può anche usare il file update_profile.php (con le dovute modifiche): 
```
curl -X PATCH http://172.26.254.11/profile/1/
     -H 'Content-Type: application/json'
     -H 'Accept: application/json'
     -H 'X-Token: TOKEN'
     -d '{"name": "Paolo", "lastname": "Rossi", "phone": "+413311002167"}'
```

**Cancellazione di un profile** (indicare l'ID)  
Si può anche usare il file delete_profile.php (con le dovute modifiche): 
```
curl -X DELETE http://172.26.254.11/profile/1/
     -H 'Content-Type: application/json'
     -H 'Accept: application/json'
     -H 'X-Token: TOKEN'
```

**Creazione di un profile attribute**  
Si può anche usare il file create_attribute.php (con le dovute modifiche):  
```
curl -X PUT http://172.26.254.11/attribute/ 
     -H 'Content-Type: application/json' 
     -H 'Accept: application/json' 
     -H 'X-Token: TOKEN' 
     -d '{"attribute": "Ingegnere", "profile_id": 1}'
```

**Modifica di un profile attribute** (indicare l'ID)  
Si può anche usare il file update_attribute.php (con le dovute modifiche): 
```
curl -X PATCH http://172.26.254.11/attribute/1/
     -H 'Content-Type: application/json'
     -H 'Accept: application/json'
     -H 'X-Token: TOKEN'
     -d '{"attribute": "Ingegnere", "profile_id": 1}'
```

**Cancellazione di un profile attribute** (indicare l'ID)  
Si può anche usare il file delete_attribute.php (con le dovute modifiche): 
```
curl -X DELETE http://172.26.254.11/attribute/1/
     -H 'Content-Type: application/json'
     -H 'Accept: application/json'
     -H 'X-Token: TOKEN'
```

# Middleware

Ho creato un middleware di base che registra tutte le operazioni nel file _/var/www/lumen_api/app/Storage/Logs/access.log_ che viene anche visualizzato nella pagina con endpoint http://172.26.254.11/.

I dati che vengono registrati sono dati "base" (non sono entrato nel dettaglio).

Qui sotto un esempio di riga del file log.

```
2024-08-04 01:13:29	GET	/profile/2/
2024-08-04 01:13:33	GET	/profile/1/
2024-08-04 01:13:36	GET	/profile/
```