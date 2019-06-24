# Atelopus



Teniamos tres pools de librerias de ddRAD para un total de 31 individuos de *Atelopus*. Por tanto, necesitabamos hacer primero el demultiplexing por cada pool por separado, utilizando un [barcodes file por cada pool](https://github.com/pesalerno/Atelopus/tree/master/barcodes-files), de la siguiente manera: 

	ipyrad -p params-pool1.txt -s 12

Luego de hacer esto para los tres pools de secuencias, debiamos hacerle un "merge" a los tres params files (no se puede hacer manualmente porque son necesarios los archivos `.json` que son creados en los primeros dos pasos). 

Para hacer el *merge* de los tres pools, se hace lo siguiente: 

	ipyrad -m merged-params pool1_params_c.txt pool2_params_c.txt pool3_params_c.txt

Esto resulta que la linea 3 de los *"demultiplexed files"* salga asi: 

	                               ## [2] [raw_fastq_path]: Location of raw non-demultiplexed fastq files
	Merged: pool1c, pool2c, pool3c ## [3] [barcodes_path]: Location of barcodes file
                                   ## [4] [sorted_fastq_path]: Location of demultiplexed/sorted fastq files


Finalmente, para poder correr varios analisis de ipyrad y determinar el optimo para el clustering threshold, utilizando el argumento `-b` hicimos un *"branching"* del analisis a partir del 3er paso, con el siguiente codigo (y para cada clustering threshold):

	ipyrad -p params-merged-params.txt -b clust86

Los archivos de los parametros eran todos identicos a [este](https://github.com/pesalerno/Atelopus/blob/master/params-clust96.txt), solo variando el clustering threshold. Luego de eso, se editaron manualmente los params files en la linea 14:

	0.96                           ## [14] [clust_threshold]: Clustering threshold for de novo assembly

para poner el clustering threshold correcto, y se montaron el resto de los pasos y analisis de ipyrad para cada uno de los "branches" utilizando un script de `bash` y con el siguiente codigo: 

	#!/bin/bash

	ipyrad -p params-clust91.txt -s 34567
	ipyrad -p params-clust92.txt -s 34567
	ipyrad -p params-clust93.txt -s 34567
	#etc....

Luego de finalizados todos los analisis, se recopilaron resultados a nivel global (es decir, por genotipificacion/matriz de datos), utilizando los outputs de [clust86_stats.txt](https://github.com/pesalerno/Atelopus/blob/master/clust86_stats.txt) y de [s6\_cluster\_stats\_c86.txt](https://github.com/pesalerno/Atelopus/blob/master/s6_cluster_stats_c86.txt), lo que nos dieron los siguientes graficos:

1. En este grafico se puede ver que tanto el numero de loci/clusters como el numero de "singletons" (clusters unicos para un individuo) aumenta relativamente gradual con cada incremento del threshold, con un aumento un poco mayor luego de clust_thresh=94. 

![](https://github.com/pesalerno/Atelopus/blob/master/ipyrad-2.png)

2. En este grafico se ve que a pesar de que el numero total de loci antes de filtros y de calculo de error incrementa bastante a medida que incrementa el clustering threshold, el numero de loci despues de todos los filtros incrementa muy poco. 

![](https://github.com/pesalerno/Atelopus/blob/master/ipyrad-1.png)

Finalmente, se observa que a partir del clustering threshold de 94%, la gran mayoria de los loci nuevos que se construyen son filtrados como basura, por lo que lo mejor para ser conservadores es escoger los resultados basados en la genotipificacion usando **clust\_thresh\_93**.

Luego de probar los efectos de los distintos clustering thresholds, quisimos evaluar los efectos de tres parametros adicionales utilizando como base el **clust\_thresh\_93**: 

	5, 5                           ## [19] [max_Ns_consens]: Max N's (uncalled bases) in consensus (R1, R2)
	8, 8                           ## [20] [max_Hs_consens]: Max Hs (heterozygotes) in consensus (R1, R2)
	20, 20                         ## [22] [max_SNPs_locus]: Max # SNPs per locus (R1, R2)

En estos analisis, encontramos el siguiente resultado en cuanto a numeros de SNPs finales pre y post-filtering: 

![imagen-aca](https://github.com/pesalerno/Atelopus/blob/master/figures/max_SNPs_per_locus.png)

Los datos con los cuales se graficaron las anteriores figuras pueden encontrarse [aqui](https://github.com/pesalerno/Atelopus/blob/master/files/Ateolpus-clust-tests.xlsx).

**plink | filtrando datos**

Primero, transformamos el archivo `.vcf` utilizando **vcftools** de esta manera: 

	vcftools --vcf path/to/file.vcf --plink --out filename


Ahora que tenemos nuestros archivos `.ped` y `.map`, podemos correr los filtros en [plink](http://zzz.bwh.harvard.edu/plink/download.shtml#download), uno por uno, de la siguiente manera. 


Primero filtramos loci con demasiados datos que faltan:

    ./plink --file input-name --geno 0.25 --recode --out output-filename_a --noweb #filtrar loci que se genotipificaron en menos del 75% de los individuos


Este filtro hace que la matriz de datos de reduzca de 830228 SNPs a 3977 SNPS, con un ´genotyping rate´de 0.974593. Segundo, vemos los individuos que tienen muchos datos que faltan: 

    ./plink --file input-filename_a --mind 0.5 --recode --out output-filename_b --noweb

Este filtro nos indica que hay siete individuos en nuestros datos que tienen mas del 50% de los datos que faltan: 

	A4_R351A_varius_C_Richards
	C1_JCSRha
	D2_Q55103_A_elegans_Durango
	G1_Q32664_A_spumarius_Yasuni_km38
	G2_R93Amin
	H2_R94Amin
	H4_T6877_Bufo_nebulifer

Finalmente, veamos los loci que tienen un minor allele frequency menos al 1%:

    ./plink --file input-filename_b --maf 0.01 --recode --out output-filename_c --noweb
    
Con este ultimo filtro, solo se pierden un total de 42 SNPs, por lo que quedamos despues de este filtro con un total de 3935 SNPs. Luego de todos estos filtros tenemos nuestros archivos finales de [`.ped`](https://github.com/pesalerno/Atelopus/blob/master/files/Atelopus-06_19_d.ped)y [`.map`](https://github.com/pesalerno/Atelopus/blob/master/files/Atelopus-06_19_d.map).

	