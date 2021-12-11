# gestione-dati-censuari-catasto-postgresql-qgis
Contiene le query di postgresql per l'importazione dei dati censuari catastali e la loro gestione in QGIS

![GitHub last commit](https://img.shields.io/github/last-commit/ludovico85/gestione-dati-censuari-catasto-postgresql-qgis?color=green&style=plastic)

# Contenuti
1. [Prerequisiti](#prerequisiti)
2. [Breve descrizione dei dati catastali censuari](#descrizione)
3. [Importazione delle cartografie catastali in CXF](#cxf)
    1. [Sub paragraph](#subparagraph1)
3. [Another paragraph](#paragraph2)

## 1. Prerequisiti <a name="prerequisiti"></a>
Software necessari:
- [QGIS](https://www.qgis.org/it/site/)
- [Postgresql](https://www.postgresql.org/)
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

## Importazione dei dati in QGIS


### Sub paragraph <a name="subparagraph1"></a>
This is a sub paragraph, formatted in heading 3 style

## Another paragraph <a name="paragraph2"></a>
The second paragraph text
