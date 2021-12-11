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

### Creazione delle tabelle ausiliari 'partite_speciali_terreni', 'qualita',
```sql
SET search_path TO catasto_terreni;
CREATE TABLE partite_speciali_terreni
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice TEXT,
	descrizione TEXT
);

INSERT INTO partite_speciali_terreni (codice, descrizione)
VALUES
('1', 'aree di enti urbani e promiscui'),
('2', 'accessori comuni ad enti rurali e ad enti rurali e urbani'),
('3', 'aree di fabbricati rurali, o urbani da accertare, divisi in subalterni'),
('4', 'acque esenti da estimo'),
('5', 'strade pubbliche'),
('0', 'particelle soppresse')
```
```sql
SET search_path TO catasto_terreni;
CREATE TABLE codici_diritto
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice TEXT,
	descrizione TEXT
);

INSERT INTO codici_diritto (codice, descrizione)
VALUES
('10','Proprietà'),
('1s','Proprietà  superficiaria'),
('1t','Proprietà  per l''area'),
('20','Nuda proprietà'),
('2s','Nuda proprietà  superficiaria'),
('30','Abitazione'),
('3','Comproprietario'),
('3s','Abitazione su proprietà  superficiaria'),
('40','Diritto del concedente'),
('4','Comproprietario per'),
('50','Enfiteusi'),
('60','Superficie'),
('70','Uso'),
('7','Comproprietario del fabbricato'),
('7s','Uso proprietà  superficiaria'),
('7s','Uso proprietà  superficiaria'),
('80','Usufrutto'),
('8a','Usufrutto con diritto di accrescimento'),
('8e','Usufrutto su enfiteusi'),
('8s','Usufrutto su proprietà  superficiaria'),
('8','Comproprietario per l''area'),
('90','Servità'),
('100','Oneri'),
('12','Concedente in parte'),
('14','Livellario parziale per'),
('15','Usufruttuario parziale per'),
('20','Livellario'),
('21','Livellario per'),
('22','Livellario in parte'),
('25','Enfiteuta in parte'),
('26','Colono perpetuo'),
('27','Colono perpetuo per'),
('28','Colono perpetuo in parte'),
('30','Usufruttuario parziale'),
('33','Cousufruttuario generale'),
('36','Usufruttuario generale di livello'),
('37','Usufruttuario parziale di livello'),
('39','Usufruttuario parziale di enfiteusi'),
('40','Usufruttuario generale di colonia'),
('41','Usufruttuario parziale di colonia'),
('42','Usufruttuario generale di dominio diretto'),
('43','Usufruttuario parziale di dominio diretto'),
('50','Cousufruttuario per'),
('52','Usuario perpetuo'),
('53','Usuario a tempo determinato'),
('60','Cousufruttuario di livello'),
('61','Cousufruttuario generale di livello'),
('62','Usufruttuario di livello di'),
('64','Comproprietario per parte di'),
('70','Usufruttuario di colonia per'),
('71','Usufruttuario di dominio diretto per'),
('72','Cousufruttuario generale con diritto di'),
('16','Utilista della superficie'),
('17','Utilista della superficie per'),
('35','Beneficiario'),
('65','Beneficiario per'),
('54','Beneficiario di dominio diretto'),
('46','Possessore'),
('47','Possessore per'),
('48','Compossessore'),
('49','Compossessore per'),
('55','Contestatario'),
('56','Contestatario per'),
('57','Contestatario per usufrutto'),
('99','Presenza di titolo non codificato'),
('990','Presenza di titolo non codificato'),
('0','Assenza di titolo');
```
