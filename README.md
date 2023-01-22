# BowBin
Una pipeline per la rilevazione di virus in dati metagenomici

## Tabella dei contenuti:
- [Descrizione](#descrizione)
- [Installazione](#installazione)
- [Requisiti](#requisiti)
- [Esecuzione di BowBin](#esecuzione-di-bowbin)
  - [Dati in input](dati-in-input)
  - [Rilevazione dei virus](rilevazione-dei-virus)
  - [Binning](binning)
  - [Dati in output](dati-in-output)
- [File e cartelle](#file-e-cartelle)
- [Autori](#autori)

## Descrizione
BowBin è una pipeline progettata per la rilevazione di genomi virali in dati metagenomici.

BowBin riceve in input il file contenente i dati metagenomici e un insieme di genomi virali, divide i genomi virali in scaffolds, esegue Bowtie2 per allineare le reads metagenomiche con gli scaffolds e, grazie a Metabat2 e a vari scripts, genera la coverage table da cui è possibile osservare le specie di virus rilevati nei dati metagenomici, con il relativo numero di scaffolds. 

Inoltre la pipeline prevede un ulteriore step di binning. In questa fase, particolarmente utile quando il numero di genomi virali dati in input è molto elevato, vengono raggruppati gli scaffolds in bin. Il processo di binning prevede l'utilizzo di Binsanity o di vRhyme, entrambi i tool utilizzano la coverage table. 



## Installazione
Per usare la pipeline è possibile clonare il repository

`git clone https://github.com/strumenti-formali-per-la-bioinformatica/bowbin-pipeline-rilevazione-virus-dati-metagenomici`



Inoltre bisogna installare i tool elencati nella sezione successiva.
## Requisiti
La pipeline è stata realizzata e testata utilizzando i tool elencati di seguito. Si consiglia di utilizzare le versioni indicate per evitare problemi di compatibilità.
- [Python3](https://www.python.org/) (versione 3.9.13)
- [Bowtie2](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml) (versione 2.2.5)
  - Compilatore: gcc version 9.3.0
- [Samtools](http://www.htslib.org/) (versione 1.6)
  - Versione htslib: 1.6
- [MetaBat2](https://bitbucket.org/berkeleylab/metabat/src/master/) (versione 2.15)
- [vRhyme](https://github.com/AnantharamanLab/vRhyme)
- [BinSanity](https://github.com/edgraham/BinSanity)

Per facilitare l'installazione dei tools si consiglia l'utilizzo di [anaconda](https://anaconda.org/) e [conda](https://docs.conda.io/en/latest/). 

Di seguito i passi per l'installazione dei tools: 
1. Creare un nuovo ambiente con python `conda create -n BowBin python=3.9`
2. Installare Bowtie2 `conda install -c bioconda bowtie2=2.2.5`
3. Installare Samtools `conda install -c bioconda samtools=1.6`
4. Installare MetaBat2 `conda install -c bioconda metabat2=2.15`
5. Per installare Binsanity: [https://github.com/edgraham/BinSanity/wiki/Installation](https://github.com/edgraham/BinSanity/wiki/Installation)
6. Per installare vRhyme: [https://github.com/AnantharamanLab/vRhyme#install](https://github.com/AnantharamanLab/vRhyme#install)

NOTA: il file `coverage_table_convert.py` presente in `vRhyme` è stato modificato appositamente per questa pipeline, si consiglia di usare il file `scripts/coverage_table_convert` settando i flag aggiunti: `-multiplyAvg` e `-multiplyStdev`, in base ai dati in input.

## Esecuzione di BowBin
Di seguito si riportano i passaggi per l'esecuzione di BowBin sui dati contenuti nella cartella `example`
### Dati in input
La pipeline prende in input i dati metagenomici `example/metagenomics.fasta` e un insieme di genomi virali `example/genomes.fasta`
### Rilevazione dei virus
Dividere i genomi virali in scaffolds: 
- `python3 scripts/split.py genomes.fasta scaffolds.fasta 218`. Il primo parametro di `split.py` è il file in input che contiene i genomi virali, il seconda parametro indica il nome del file di output, l'ultimo parametro indica la lunghezza dello scaffold espresso in numero di righe.

Creazione dell'indice Bowtie2 usando il file che contiene gli scaffolds:
- `bowtie2 -build scaffolds.fasta indiceScaffolds`

Allineamento degli scaffolds con i dati metagenomici `example/metagenomics.fasta`:
- `bowtie2 --no-unal --no-discordant -f -x indiceScaffolds -U metagenomics.fasta -S risultatiAllineamento.sam`

Trasformazione file `.sam` in `.bam`:
- `samtools view -S -b risultatiAllineamento.sam > risultatiAllineamento.bam`

Ordinamento del file `.bam`:
- `samtools sort risultatiAllineamento.bam > risultatiAllineamento.sort.bam`

Creazione della coverage table:
- `jgi_summarize_bam_contig_depths --outputDepth depth.txt risultatiAllineamento.sort.bam`

Trasformazione della coverage table:
- `python3 scripts/coverage_table_convert.py -i depth.txt -o coverage_table.tsv -multiplyAvg 900 -multiplyStdev 150`
   

### Binning
### Dati in output


## File e cartelle

## Autori
[Alessando Aquino](https://github.com/AlessandroUnisa), [Nicolapio Gagliarde](https://github.com/GagliardeNicolapio/)
