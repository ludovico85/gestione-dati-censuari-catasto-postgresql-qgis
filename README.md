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
5.2. [catasto fabbricati](#fabbricati)
6. [Elaborazione dei dati nello schema `catasto_ter`](#catasto_ter)


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
  - Delimitatori personalizzati 🠊 Altri 🠊 |
  - Virgolette 🠊 vuoto
  - Caratteri di controllo 🠊 vuoto
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
## 6. Elaborazione dei dati nello schema `catasto_ter` <a name="catasto_ter"></a>
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
  t.field_1 AS codice_amministrativo,
  t.field_2 AS sezione,
  t.field_3 AS identificativo_immobile,
  t.field_4 AS tipo_immobile,
  t.field_5 AS progressivo,
  t.field_6 AS tipo_record,
  t.field_7 AS foglio,
  t.field_8 AS numero,
  t.field_9 AS denominatore,
  t.field_10 AS subalterno,
  t.field_11 AS edificialita,
  t.field_12 AS qualita,
  q.descrizione AS descrizione_qualita,
  t.field_13 AS classe,
  t.field_14 AS ettari,
  t.field_15 AS are,
  t.field_16 AS centiare,
  t.field_17 AS flag_reddito,
  t.field_18 AS flag_porzione,
  t.field_19 AS flag_deduzioni,
  t.field_20 AS reddito_dominicale_lire,
  t.field_21 AS reddito_agrario_lire,
  t.field_22 AS reddito_dominicale_euro,
  t.field_23 AS reddito_agrario_euro,
  regexp_replace(t.field_36, '^0+', '') AS partita,
  p.descrizione AS descrizione_partita,
  CASE WHEN t.field_3 = ter4.field_3 THEN 'si' ELSE 'no' END AS porzioni
FROM ter1 t
LEFT JOIN qualita q ON t.field_12 = q.codice
LEFT JOIN partite_speciali_terreni p ON regexp_replace(t.field_36, '^0+', '') = p.codice
LEFT JOIN ter4 ON ter4.field_3 = t.field_3
```
#### Tipo di record 2 (da completare)
#### Tipo di record 3
Oltre ai dati generali, contiene il codice della riserva e la partita di iscrizione riserva, quest'ultima già presente nel campo partita di ter1_colnames. Nessuna operazione necessaria.
#### Tipo di record 4
Non è possibile conoscere il numero di porzioni della particella a priori. Per tale motivo è stato scelto un massimo di 4 porzioni (che può essere modificato, modificando la query).
```sql
CREATE OR REPLACE VIEW ter4_colnames AS
SELECT
  field_1 AS codice_amministrativo,
  field_2 AS sezione,
  field_3 AS identificativo_immobile,
  field_4 AS tipo_immobile,
  field_5 AS progressivo,
  field_6 AS tipo_record,
  field_7 AS identificativo_porzione_1,
  field_8 AS qualita_1,
  a.descrizione AS descrizione_qualita_1,
  field_9 AS classe_1,
  field_10 AS ettari_1,
  field_11 AS are,
  field_12 AS centiare,
  field_13 AS identificativo_porzione_2,
  field_14 AS qualita_2,
  b.descrizione AS descrizione_qualita_2,
  field_15 AS classe_2,
  field_16 AS ettari_2,
  field_17 AS are_2,
  field_18 AS centiare_2,
  field_19 AS identificativo_porzione_3,
  field_20 AS qualita_3,
  c.descrizione AS descrizione_qualita_3,
  field_21 AS classe_3,
  field_22 AS ettari_3,
  field_23 AS are_3,
  field_24 AS centiare_3,
  field_25 AS identificativo_porzione_4,
  field_26 AS qualita_4,
  d.descrizione AS descrizione_qualita_4,
  field_27 AS classe_4,
  field_28 AS ettari_4,
  field_29 AS are_4,
  field_30 AS centiare_4
FROM ter4 t
LEFT JOIN qualita a ON t.field_12 = a.codice
LEFT JOIN qualita b ON t.field_12 = b.codice
LEFT JOIN qualita c ON t.field_12 = c.codice
LEFT JOIN qualita d ON t.field_12 = d.codice
```
#### Titolarità
Si ricostruisce dapprima la vista con il nome dei campi e qualche informazione aggiuntiva (come la descrizione del diritto), successivamente si suddivide il dataset per le titolairtà delle persone fisiche e dei soggetti giuridici.

```sql
CREATE OR REPLACE VIEW tit_colnames AS
SELECT DISTINCT ON (field_28)
field_1 AS codice_amministrativo,
field_2 AS sezione,
field_3 AS identificativo_soggetto,
field_4 AS tipo_soggetto,
field_5 AS identificativo_immobile,
field_6 AS tipo_immobile,
regexp_replace(field_7, '\s', '', 'g') AS codice_diritto,
field_8 AS titolo_non_codificato,
field_9 AS quota_numeratore_possesso,
field_10 AS quota_denominatore_possesso,
field_11 AS regime,
field_12 AS soggetto_riferimento,
field_13 AS data_validita_atto_generato,
field_14 AS tipo_nota_generato,
field_15 AS numero_nota_generato,
field_16 AS progressivo_nota_generato,
field_17 AS anno_nota_generato,
field_18 AS data_registrazione_atti_generato,
field_19 AS partita,
field_20 AS data_validita_atto_concluso,
field_21 AS tipo_nota_concluso,
field_22 AS numero_nota_concluso,
field_23 AS progessivo_nota_concluso,
field_24 AS anno_nota_concluso,
field_25 AS data_registrazione_atti_concluso,
field_26 AS identificativo_mutazione_iniziale,
field_27 AS identificativo_mutazione_finale,
field_28 AS identificativo_titolarita,
field_29 AS codice_causale_atto_generante,
field_30 AS descrizione_atto_generante,
d.descrizione AS descrizione_diritto
FROM tit
LEFT JOIN catasto_terreni.codici_diritto d ON regexp_replace(tit.field_7, '\s', '', 'g') = d.codice
```
```sql
CREATE OR REPLACE VIEW titp AS
	SELECT * FROM tit_colnames
	WHERE tipo_soggetto = 'P';
```
```sql
  CREATE OR REPLACE VIEW titg AS
	SELECT * FROM tit_colnames
	WHERE tipo_soggetto = 'G';
```
#### Soggetti
Si ricostiuiscono dapprima i nomi dei campi per le perosne fisiche e i soggetti giuridici.
```sql
CREATE OR REPLACE VIEW sogp AS
SELECT
field_1 AS codice_amministrativo,
field_2 AS sezione,
field_3 AS identificativo_soggetto,
field_4 AS tipo_soggetto,
field_5 AS cognome,
field_6 AS nome,
field_7 AS sesso,
field_8 AS data_nascita,
field_9 AS codice_amministratvio_comune_nascita,
field_10 AS codice_fiscale,
field_11 AS indicazioni_supplementari
FROM sog
WHERE field_4 = 'P';
```
```sql
CREATE OR REPLACE VIEW sogg AS
SELECT
field_1 AS codice_amministrativo,
field_2 AS sezione,
field_3 AS identificativo_soggetto,
field_4 AS tipo_soggetto,
field_5 AS denominazione,
field_6 AS codice_amministrativo_sede,
field_7 AS codice_fiscale
FROM sog
WHERE field_4 = 'G';
```
#### Join titolarità soggetti
Si ricosituiscono le relazioni tra le titolairtà e i soggetti (giuridici e persone fisiche) e successivamente si esegue l'union tra le due viste in modo da avere un un'unica vista con tutte le relazioni.
```sql
CREATE OR REPLACE VIEW titp_sogp AS
SELECT
	t.identificativo_immobile,
	t.tipo_immobile,
	t.identificativo_soggetto identificativo_soggetto_tit,
	t.descrizione_diritto as diritto,
	concat(t.quota_numeratore_possesso, '/', t.quota_denominatore_possesso) AS quota,
    p.identificativo_soggetto as identificativo_soggetto_sogp,
    p.cognome,
    p.nome,
    p.data_nascita,
    p.codice_fiscale
	FROM titp t
	LEFT JOIN sogp p ON t.identificativo_soggetto = p.identificativo_soggetto;
```
```sql
CREATE OR REPLACE VIEW titg_sogg AS
SELECT
	t.identificativo_immobile,
	t.tipo_immobile,
	t.identificativo_soggetto identificativo_soggetto_tit,
	t.descrizione_diritto as diritto,
	concat(t.quota_numeratore_possesso, '/', t.quota_denominatore_possesso) AS quota,
    g.identificativo_soggetto as identificativo_soggetto_sogg,
    g.denominazione,
    g.codice_amministrativo_sede,
    g.codice_fiscale
	FROM titg t
	LEFT JOIN sogg g ON t.identificativo_soggetto = g.identificativo_soggetto;
```
```sql
CREATE OR REPLACE VIEW tit_sogp_sogg_partite_speciali AS
SELECT
g.identificativo_immobile as identificativo_immobile,
g.tipo_immobile as tipo_immobile,
'soggetto giuridico' as tipo_soggetto,
g.diritto as diritto,
g.quota as quota,
g.identificativo_soggetto_tit as identificativo_soggetto_tit,
g.identificativo_soggetto_sogg as identificativo_soggetto,
g.denominazione as denominazione,
g.codice_amministrativo_sede as codice_amministrativo_sede,
NULL as data_nascita,
g.codice_fiscale as codice_fiscale
FROM catasto_terreni.titg_sogg g
UNION ALL
SELECT
p.identificativo_immobile as identificativo_immobile,
p.tipo_immobile as tipo_immobile,
'persona fisica' as tipo_soggetto,
p.diritto as diritto,
p.quota as quota,
p.identificativo_soggetto_tit as identificativo_soggetto_tit,
p.identificativo_soggetto_sogp as identificativo_soggetto,
concat(p.cognome, ' ', p.nome) as denominazione,
NULL as codice_amministrativo_sede,
p.data_nascita as data_nascita,
p.codice_fiscale as codice_fiscale
FROM catasto_terreni.titp_sogp p
UNION ALL
SELECT
t.identificativo_immobile as identificativo_immobile,
t.tipo_immobile as tipo_immobile,
'partita speciale' as tipo_soggetto,
NULL as diritto,
NULL as quota,
NULL as identificativo_soggetto,
t.descrizione_partita as denominazione,
NULL as codice_amministrativo_sede,
NULL as data_nascita,
NULL as codice_fiscale,
NULL as idimm_idsog_diritto
FROM catasto_terreni.ter1_colnames as t
WHERE partita IN ('0','1','2','3','4','5')
```
#### Join finale dei dati censuari del catasto terreni
Ultimo passaggio è quello di unire le viste finali:
- `ter1_colnames`: contiene le informazioni sulla particella
- `tit_sogp_sogg_partite_speciali`: contiene le informazioni sulle titolarità e sui soggetti

```sql
CREATE OR REPLACE VIEW dati_censuari_ter AS
SELECT row_number() OVER ()::integer AS gid,
ter.codice_amministrativo,
ter.sezione,
ter.identificativo_immobile AS identificativo_immobile_ter,
ter.tipo_immobile,
ter.progressivo,
ter.tipo_record,
ter.foglio,
ter.numero,
ter.denominatore,
ter.subalterno,
ter.edificialita,
ter.qualita,
ter.descrizione_qualita,
ter.classe,
ter.ettari,
ter.are,
ter.centiare,
ter.flag_reddito,
ter.flag_porzione,
ter.flag_deduzioni,
ter.reddito_dominicale_lire,
ter.reddito_agrario_lire,
ter.reddito_dominicale_euro,
ter.reddito_agrario_euro,
ter.partita,
ter.descrizione_partita,
ter.porzioni,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per la relazione con le geometrie del catasto
	WHEN length(ter.foglio) = 1 THEN concat(ter.codice_amministrativo, '_000', ter.foglio, '_', regexp_replace(ter.numero, '^0+', ''))
    	ELSE
		(
			CASE
		 	WHEN length(ter.foglio) = 2 THEN concat(ter.codice_amministrativo, '_00', ter.foglio, '_', regexp_replace(ter.numero, '^0+', ''))
		 	ELSE
				(
					CASE
			 		WHEN length(ter.foglio) = 3 THEN concat(ter.codice_amministrativo, '_0', ter.foglio, '_', regexp_replace(ter.numero, '^0+', ''))
			 		ELSE
			 			(
							CASE
							WHEN length(ter.foglio) = 4 THEN concat(ter.codice_amministrativo, '_', ter.foglio, '_', regexp_replace(ter.numero, '^0+', ''))
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
t.identificativo_immobile as identificativo_immobile_tit,
t.tipo_soggetto,
t.diritto,
t.quota,
t.identificativo_soggetto_tit,
t.identificativo_soggetto,
t.denominazione,
t.codice_amministrativo_sede,
t.data_nascita,
t.codice_fiscale
FROM ter1_colnames as ter
RIGHT JOIN tit_sogp_sogg_partite_speciali t ON ter.identificativo_immobile = t.identificativo_immobile;
```
