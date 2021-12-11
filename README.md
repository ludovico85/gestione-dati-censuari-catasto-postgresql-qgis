# gestione-dati-censuari-catasto-postgresql-qgis
Contiene le query di postgresql per l'importazione dei dati censuari catastali e la loro gestione in QGIS

![GitHub last commit](https://img.shields.io/github/last-commit/ludovico85/gestione-dati-censuari-catasto-postgresql-qgis?color=green&style=plastic)

# Contenuti
1. [Prerequisiti](#prerequisiti)
2. [Breve descrizione dei dati catastali censuari](#descrizione)
3. [Importazione delle cartografie catastali in CXF](#cxf)
4. [Creazione del database e degli schemi in PostgreSQL](#db)
5. [Importazione dei dati in QGIS](#qgis)
5.1. [catasto terreni](#terreni)


## 1. Prerequisiti <a name="prerequisiti"></a>
Software necessari:
- [QGIS](https://www.qgis.org/it/site/)
- [PostgreSQL](https://www.postgresql.org/)
- plugin per QGIS [cxf_in](https://github.com/saccon/CXF_in)

Dati:
- Cartografie catastali in formato CXF
- Dati censuari catasto fabbricati
- Dati censuari catasto terreni
## 2. Breve descrizione dei dati catastali censuari <a name="descrizione"></a>
Le informazioni descritte in questa sezione derivano dal documento a cura dell'Agenzia dell'Entrate (DOC. ES-23-IS-05) liberamente consultabile all'indirizzo: https://wwwt.agenziaentrate.gov.it/mt/ServiziComuniIstituzioni/ES-23-IS-05_100909.pdf.

Per maggiori dettagli sui servizi riservati ai comuni di può consultare: https://www.agenziaentrate.gov.it/portale/web/guest/schede/fabbricatiterreni/portale-per-i-comuni/servizi-portale-dei-comuni/estrazione-dati-catastali.

I dati censuari sono costituiti da 4 tipi di file:

- file fabbricati (.FAB);

- file terreni (.TER);

- file soggetti (.SOG);

- file titolarità (.TIT);

- file parametri della richiesta (.PRM).

Ogni tipo di file è costituito da una tabella che può contenere diversi tipi di record. Il collegamento tra i tipi di file è assicurato dalla presenta di chiavi specifiche:

- .FAB/.TER contengono la chiave identificativo immobile;

- .SOG contiene la chiave identificativo soggetto;

- .TIT contiene sia la chiave identificativo immobile che la chiave identificativo soggetto (oltre che la chiave identificativo titolarietà);

## Importazione delle cartografie catastali in CXF <a name="cxf"></a>
Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## 4. Creazione del database e degli schemi in PostgreSQL <a name="db"></a>
- Creare un nuovo database in PostgreSQL
- Creare uno schema denominato catasto_terreni e uno schemda denominato catasto_fabbricati
```sql
CREATE SCHEMA catasto_terreni;
CREATE SCHEMA catasto_fabbricati;
```
## 5. Importazione dei dati in QGIS <a name="qgis"></a>
### 5.1. Catasto terreni <a name="terreni"></a>
Caricare in QGIS un nuovo layer testo delimitato e selezionare il file con estensione .TER. Importare con i seguenti parametri:
- Formato file
  - delimitatori personalizzati 🠊 Altri 🠊 |
- Opzioni Record e campi
  - non spuntare "Il primo record ha i nomi dei campi"
  - spuntare "La virgola è il separatore decimale"
- Definizione della geometria
  - Nessuna geometria

Nel nome del layer utilizzare 'ter'

![ter](/img/import_ter.png)

Assicurarsi di avere la connessione adl database creato nel punto [4](#db)

Dagli `Strumenti di Processing` di QGIS cercare `Esporta in PostgreSQL`. Inserire i seguenti parametri:
- Layer da importare 🠊 ter
- Database 🠊 il nome del database creato al punto [4](#db)
- Schema 🠊 catasto_terreni
- Nome della tabella 🠊 ter

Ripetere i passaggi precedenti per importare le tabelle .TIT e .SOG

Alla fine della procedura il database dovrebbe essere così organizzato:

```
   -- database
     |-- catasto_terreni
       |-- ter
       |-- tit
       |-- sog
     |-- catasto_fabbricati
```
### 5.2. Catasto fabbricati <a name="fabbricati"></a>
Caricare in QGIS un nuovo layer testo delimitato e selezionare il file con estensione .FAB. Importare con i seguenti parametri:
- Formato file
  - delimitatori personalizzati 🠊 Altri 🠊 |
- Opzioni Record e campi
  - non spuntare "Il primo record ha i nomi dei campi"
  - spuntare "La virgola è il separatore decimale"
- Definizione della geometria
  - Nessuna geometria

Nel nome del layer utilizzare 'fab'

Assicurarsi di avere la connessione adl database creato nel punto [4](#db)

Dagli `Strumenti di Processing` di QGIS cercare `Esporta in PostgreSQL`. Inserire i seguenti parametri:
- Layer da importare 🠊 fab
- Database 🠊 il nome del database creato al punto [4](#db)
- Schema 🠊 catasto_fabbricati
- Nome della tabella 🠊 fab

Ripetere i passaggi precedenti per importare le tabelle .TIT e .SOG

Alla fine della procedura il database dovrebbe essere così organizzato:

```
   -- database
     |-- catasto_terreni
       |-- ter
       |-- tit
       |-- sog
     |-- catasto_fabbricati
       |-- ter
       |-- tit
       |-- sog
```
## 6. Elaborazione dei dati nello schema catasto_ter
Le elaborazioni possono essere fatte in PostgreSQL tramite command line, con PgAdmin oppure direttamente in QGIS utilizzando lo strumento `Esegui SQL PostgreSQL`

```sql
SET search_path TO catasto_terreni;
CREATE TABLE partite_speciali_terreni
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_partita TEXT,
	descrizione TEXT
);

INSERT INTO partite_speciali_terreni (codice_partita, descrizione)
VALUES
('1', 'aree di enti urbani e promiscui'),
('2', 'accessori comuni ad enti rurali e ad enti rurali e urbani'),
('3', 'aree di fabbricati rurali, o urbani da accertare, divisi in subalterni'),
('4', 'acque esenti da estimo'),
('5', 'strade pubbliche'),
('0', 'particelle soppresse')
```
