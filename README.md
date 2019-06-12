# Atelopus



Teniamos tres librerias de ddRAD para un total de 32 individuos de Atelopus. Por tanto, necesitabamos hacer primero el demultiplexing por cada pool por separado de la siguiente manera: 

	ipyrad -p params-pool1.txt -s 12

Luego de hacer esto para los tres pools de secuencias, debiamos hacerle un "merge" a los tres params files (no se puede hacer manualmente porque son necesarios los archivos `.json` que son creados en los primeros dos pasos). 

Para hacer el merge de los tres pools, se hace lo siguiente: 

	ipyrad -m merged-params pool1_params_c.txt pool2_params_c.txt pool3_params_c.txt

Finalmente, para poder correr varios analisis de ipyrad y determinar el optimo para el clustering threshold, hicimos un "branching" del analisis a partir del 3er paso, con el siguiente codigo (y para cada clustering threshold):

	ipyrad -p params-merged-params.txt -b clust86

Luego de eso, se montaron el resto de los pasos y analisis de ipyrad para cada uno de los "branches" con su distinto clustering threshold, utilizando un script de `bash` y con el siguiente codigo: 

	#!/bin/bash

	ipyrad -p params-clust91.txt -s 34567
	ipyrad -p params-clust92.txt -s 34567
	ipyrad -p params-clust93.txt -s 34567
	#etc....

Luego de finalizados todos los analisis, se recopilaron resultados a nivel global (es decir, por genotipificacion/matriz de datos), utilizando los outputs de [clust86_stats.txt](https://github.com/pesalerno/Atelopus/blob/master/clust86_stats.txt) y de [s6_cluster_stats_c86.txt](https://github.com/pesalerno/Atelopus/blob/master/s6_cluster_stats_c86.txt), lo que nos dieron los siguientes graficos:

1. En este grafico se puede ver que tanto el numero de loci/clusters como el numero de "singletons" (clusters unicos para un individuo) aumenta relativamente gradual con cada incremento del threshold, con un aumento un poco mayor luego de clust_thresh=94. 

![](https://github.com/pesalerno/Atelopus/blob/master/ipyrad-results-2.png)

2. En este grafico se ve que a pesar de que el numero total de loci antes de filtros y de calculo de error incrementa bastante a medida que incrementa el clustering threshold, el numero de loci despues de todos los filtros incrementa muy poco. 

![](https://github.com/pesalerno/Atelopus/blob/master/ipyrad-results-1.png)

