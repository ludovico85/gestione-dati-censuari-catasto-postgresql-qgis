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
## 6. Elaborazione dei dati nello schema `catasto_ter`
Le elaborazioni possono essere fatte in PostgreSQL tramite command line, con PgAdmin oppure direttamente in QGIS utilizzando lo strumento `Esegui SQL PostgreSQL`

### Creazione delle tabelle ausiliari `partite_speciali_terreni`, `codici_diritto`, `qualita`
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
``` sql
SET search_path TO catasto_terreni;
CREATE TABLE qualita
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice TEXT,
	descrizione TEXT
);

INSERT INTO qualita (codice, descrizione)
VALUES
('1','seminativo'),
('2','semin irrig'),
('3','semin arbor'),
('4','sem arb irr'),
('5','sem irr arb'),
('6','sem pez fos'),
('7','sem arb p f'),
('8','prato'),
('9','prato irrig'),
('10','prato arbor'),
('11','prato ir ar'),
('12','prato marc'),
('13','prato mar ar'),
('14','marcita'),
('15','risaia'),
('16','risaia stab'),
('17','orto'),
('18','orto irrig'),
('19','orto arbor'),
('20','orto ar irr'),
('21','orto frutt'),
('22','orto pez fos'),
('23','orto fiori'),
('24','orto ir fi'),
('25','ort viv flo'),
('26','vivaio'),
('27','viv orn fl'),
('28','giardini'),
('29','vigneto'),
('30','vigneto arb'),
('31','vigneto irr'),
('32','vig uva tav'),
('33','vign frutt'),
('34','vign ulivet'),
('35','vign mandor'),
('36','uliveto'),
('37','uliv agrum'),
('38','uliv fichet'),
('39','ul fich man'),
('40','uliv frass'),
('41','uliv frutt'),
('42','uliv sommac'),
('43','uliv vignet'),
('44','uliv sugher'),
('45','uliv mandor'),
('46','ul man pist'),
('47','frutteto'),
('48','frutt irrig'),
('49','agrumeto'),
('50','agrum aranc'),
('51','agrum irrig'),
('52','agrum uliv'),
('53','alpe'),
('54','aranceto'),
('55','canneto'),
('56','cappereto'),
('57','carrubeto'),
('58','castagneto'),
('59','cast frutto'),
('60','cast frass'),
('61','chiusa'),
('62','eucalipteto'),
('63','ficheto'),
('64','fico india'),
('65','fico mandor'),
('66','frassineto'),
('67','gelseto'),
('68','limoneto'),
('69','mandorleto'),
('70','mandor fich'),
('71','mandor fico'),
('72','mandarineto'),
('73','noceto'),
('74','noccioleto'),
('75','nocc vignet'),
('76','palmeto'),
('77','pescheto'),
('78','pioppeto'),
('79','pistacch'),
('80','pometo'),
('81','querceto'),
('82','querc ghian'),
('83','roseto'),
('84','saliceto'),
('85','salceto'),
('86','sommaccheto'),
('87','sommac arb'),
('88','somm mandor'),
('89','sommac uliv'),
('90','sughereto'),
('91','pascolo'),
('92','pascolo arb'),
('93','pasc cespug'),
('94','pascolo bc'),
('95','pascolo bm'),
('96','pascolo ba'),
('97','bosco ceduo'),
('98','bosco misto'),
('99','bosco alto'),
('100','palud spart'),
('101','incolt prod'),
('102','orto irr ar'),
('103','nocciol irr'),
('104','sem car irr'),
('105','pereto'),
('106','sem irr prot'),
('107','bosco rap ac'),
('126','serra'),
('127','funghicoltur'),
('130','arativi'),
('131','seminativi'),
('132','prati'),
('133','orti'),
('134','vigneti'),
('135','pascoli'),
('136','alpi'),
('137','lag pal st'),
('138','boschi'),
('150','incolt ster'),
('151','lago pubbl'),
('152','laguna'),
('153','stagno'),
('180','cava'),
('181','lago pesca'),
('182','miniera'),
('183','salina'),
('184','stagn pesca'),
('185','tonnara'),
('186','torbiera'),
('187','valle pesca'),
('188','acque priv'),
('200','aeroporto d'),
('201','aer fort d'),
('202','autovia sp'),
('203','area dem pp'),
('204','banchina'),
('205','cimitero'),
('206','ferrovia sp'),
('207','fortificaz'),
('208','giard pub'),
('209','giard dem'),
('210','giard com'),
('211','giard prov'),
('212','molo'),
('213','parco pubb'),
('214','parco deman'),
('215','parco comun'),
('216','parco prov'),
('217','p v rimembr'),
('218','pza d armi'),
('219','porto'),
('220','somm arg 2'),
('221','somm arg 3'),
('222','tranvia sp'),
('223','somm arg 1'),
('250','canale bon'),
('251','canale irr'),
('252','gora'),
('270','antichita'),
('271','area fab dm'),
('272','area promis'),
('273','area rurale'),
('274','area urbana'),
('275','corte urban'),
('276','costr no ab'),
('277','fa div sub'),
('278','fabb promis'),
('279','fabb rurale'),
('280','fabb diruto'),
('281','fr div sub'),
('282','ente urbano'),
('283','fu d accert'),
('284','porz acc fr'),
('285','porz acc fu'),
('286','porz di fa'),
('287','porz di fr'),
('288','porz rur fp'),
('290','porz di fu'),
('300','acque esent'),
('301','piazza pubb'),
('302','strade pubb'),
('350','accesso'),
('351','accessorio'),
('352','aia'),
('353','andito'),
('354','androne'),
('355','area'),
('356','ascensore'),
('357','autorimessa'),
('358','ballatoio'),
('359','bindolo'),
('360','bottino'),
('361','bucataio'),
('362','canale priv'),
('363','cantina'),
('364','capanna'),
('365','cappella'),
('366','carbonile'),
('367','casello'),
('368','casotto'),
('369','cisterna'),
('370','concimaia'),
('371','conigliera'),
('372','corridoio'),
('373','corte'),
('374','fontana'),
('375','fontanile'),
('376','forno'),
('377','frantoio'),
('378','garage'),
('379','garitta'),
('380','grotta'),
('381','ingresso'),
('382','latrina'),
('383','lavanderia'),
('384','lavatoio'),
('385','legnaia'),
('386','locale dep'),
('387','loggia'),
('388','luogo dep'),
('389','macero'),
('390','montacarichi'),
('391','muro'),
('392','noria'),
('393','oratorio'),
('394','palmeto'),
('395','passaggio'),
('396','passo'),
('397','piazza'),
('398','piazzale'),
('399','pollaio'),
('400','pompa'),
('401','porcile'),
('402','portico'),
('403','portineria'),
('404','portone'),
('405','pozzo'),
('406','resede'),
('407','rifugio ant'),
('408','rimessa'),
('409','ripostiglio'),
('410','scala'),
('411','scolo acqua'),
('412','seccatoio'),
('413','sedime'),
('414','sentina'),
('415','sorgiva'),
('416','sottoscala'),
('417','sottopassag'),
('418','spazio'),
('419','stalla'),
('420','strada priv'),
('421','terrazzo'),
('422','tettoia'),
('423','tinaia'),
('424','vano'),
('425','vasca'),
('450','rel ente ur'),
('451','rel acc com'),
('452','rel f d sub'),
('453','rel acq es'),
('454','relit strad'),
('455','terr n form'),
('500','casa e corte'),
('501','fabb e corte'),
('504','incolto'),
('505','improdutt'),
('506','sentiero'),
('507','ponte'),
('508','cavalcavia'),
('509','casa stalla'),
('510','chiesa'),
('511','edificabile'),
('512','cortile'),
('513','magazzino'),
('514','fosso'),
('515','casa'),
('516','edificio'),
('517','fabbricato'),
('602','lag pal sta'),
('993','modello 26'),
('997','sop var ter'),
('998','soppresso'),
('999','disponibile');
```
### Creazione delle viste
Il file `ter` è costituito da 4 differenti tipi record (field_6). La particella è identificata attraverso il campo IDENTIFICATIVO IMMOBILE (field_3).

- TIPO RECORD 1: contiene le caratteristiche della particella. E' il record di interesse che verrà utilizzato per ricostruire il dato spaziale;

- TIPO RECORD 2: deduzioni della particella;

- TIPO RECORD 3: riserva della particella;

- TIPO RECORD 4: porzioni della particella.

```sql
CREATE OR REPLACE VIEW ter1 AS SELECT * from ter WHERE field_6 = '1';
CREATE OR REPLACE VIEW ter2 AS SELECT * from ter WHERE field_6 = '2';
CREATE OR REPLACE VIEW ter3 AS SELECT * from ter WHERE field_6 = '3';
CREATE OR REPLACE VIEW ter4 AS SELECT * from ter WHERE field_6 = '4';
```
#### Tipo di record 1
Dalla vista `ter1` si selezionano alcuni campi d'interesse: i dati generali (da field_1 a field_6), gli elementi identificativi della particella (da field_7 a field_11), i dati caratteristici della particella (da field_12 a field_23) e il numero della partita (field_36).
```sql
CREATE OR REPLACE VIEW ter1_colnames AS
SELECT
  field_1 AS codice_amministrativo,
  field_2 AS sezione,
  field_3 AS identificativo_immobile,
  field_4 AS tipo_immobile,
  field_5 AS progressivo,
  field_6 AS tipo_record,
  field_7 AS foglio,
  field_8 AS numero,
  field_9 AS denominatore,
  field_10 AS subalterno,
  field_11 AS edificialita,
  field_12 AS qualita,
  field_13 AS classe,
  field_14 AS ettari,
  field_15 AS are,
  field_16 AS centiare,
  field_17 AS flag_reddito,
  field_18 AS flag_porzione,
  field_19 AS flag_deduzioni,
  field_20 AS reddito_dominicale_lire,
  field_21 AS reddito_agrario_lire,
  field_22 AS reddito_dominicale_euro,
  field_23 AS reddito_agrario_euro,
  field_36 AS partita
FROM ter1
```
#### Tipo di record 2 (da completare)
#### Tipo di record 3
Oltre ai dati generali (da field_1 a field_6), è presente la tabella delle riserve che può contenere più campi. E' impossibile conoscere a priori in numero massimo di partite.

CREATE OR REPLACE VIEW ter1_colnames AS
SELECT
  field_1 AS codice_amministrativo,
  field_2 AS sezione,
  field_3 AS identificativo_immobile,
  field_4 AS tipo_immobile,
  field_5 AS progressivo,
  field_6 AS tipo_record,
  field_7 AS codice_riserva,
  field_8 AS partita_iscrizione_riserva,
  field_9 AS codice_riserva,
  field_10 AS partita_iscrizione_riserva,
  field_11 AS codice_riserva,
  field_12 AS partita_iscrizione_riserva,
  field_13 AS codice_riserva,
  field_14 AS partita_iscrizione_riserva,
FROM ter1
