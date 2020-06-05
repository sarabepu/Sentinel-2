# Informe final monitoria de investigación 2020-1
## Sara María Bejarano
En este informe me centraré en el avance hecho en el análisis de imágenes satelitales Sentinel-2 durante la segunda parte del semestre. Considero que el avance más importante es la documentación rigurosa del proceso, ya que en internet no hay muchos recursos sobre Sentinel-2 (a diferencia de LandSat).

## Objetivo: 
Recrear con imágenes Sentinel-2 análisis previos hechos con imágenes LandSat 8 en Tolima. 

¿Por qué?

- LandSat tiene imágenes de resolución 30mx30m (por cada 30 metros se ve un píxel) mientras que Sentinel tiene hasta 10mx10m en algunas bandas.
- No se pueden utilizar imágenes de mejores resoluciones (cómo las de Google Earth) porque se necesitan imágenes tomadas con periodicidad fija (10 días en el caso de Sentinel 2)  para poder hacer análisis sobre cultivos.
## Desafios: 
## - Instalación del cubo: 
Para el análisis se utiliza el OPEN DATA CUBE (ODC). Es una estructura de cubo de datos que optimiza los análisis a las imágenes. Ya que estas son muy pesadas y una vez indexadas en un cubo, se pueden hacer consultas fácilmente. 
 
Yo creé mi propia implementación del cubo ya que para desplegar la de Docker se necesita Windows pro o Linux. Seguí las instrucciones de la documentación oficial [acá](https://datacube-core.readthedocs.io/en/latest/ops/install.html) desde “Installation” hasta “Database Setup” 


## -Indexación de imágenes en el cubo:

Una vez se tiene el cubo y la base de datos lista se deben hacer los siguientes pasos: 

- Buscar y descargar imágenes de [acá]( https://scihub.copernicus.eu/dhus/#/home). En particular yo utilicé imágenes del producto S2MSI2A pero creo que el proceso no debe variar demasiado para otros productos.

- Activar el ambiente con el comando 

    ```activate cubeenv```
- Prender la base de datos 
- Agregar los productos con el comando 

    ```datacube product add <path-to-product-definition-yml>```

    Yo utilicé la definición encontrada [acá](https://github.com/opendatacube/datacube-core/blob/develop/docs/config_samples/dataset_types/s2_granules.yaml).
- Creé un script de preparación porque el ODC indexa las imágenes en un formato yaml. Más información sobre este formato puede ser encontrada [acá](https://datacube-core.readthedocs.io/en/latest/ops/dataset_documents.html#dataset-metadata-doc). Este script puede ser encontrado en este [link]() y toma los datos encontrados en el archivo metadata dentro del zip de las imágenes y lo coloca en el formato esperado. 
- Correr el script con el comando:

    ```python s2prepare_cophub_zip.py <ruta a la imagen comprimida>.zip --output ./<carpeta de salida>/ --no-checksum```

    - Si sale el error "OSError: Could not find lib geos_c.dll or load any of its variants." puede ser que el GDAL necesario para que funcione el cubo de 2.4.3 pero para el script es el 3. Entonces desactiva el ambiente cubeenv e instala gdal con 
    ```conda install -c conda-forge gdal```
    - Si no encuentra alguno de los archivos, verifique que el zip se llame igual que la imagen adentro (algo asi como *S2B_MSIL2A_20200104T152639_N0213_R025_T18NVL_20200104T192813*)


- Cambiar el code del platform en el archivo generado a “Sentinel_2B”. Este paso puede mejorarse en un futuro.
- Indexar la imágen con el comando:

    ```datacube dataset add <path-to-dataset-document-yaml>``` 


## Prueba de algoritmos sobre imágenes indexadas:
Antes de hacer algoritmos sobre las imágenes recomiendo leer esta [guia para principiantes](https://github.com/GeoscienceAustralia/dea-notebooks/tree/develop/Beginners_guide) dónde explican muy bien términos como productos, plataformas, bandas espectrales, etc..

**Siempre ANTES activar cubeenv, abrir el jupyter notebook en esa misma consola y prender base de datos postgreSQL**  

- Crear un jupyter notebook con el código para ver las imagenes indexadas:


 ```python
import datacube
dc = datacube.Datacube(app = "first")
xarr = dc.load(
    # Satellite 
    product="s2b_sen2cor_granule", output_crs='EPSG:4326',  resolution=(-0.00010,0.00010)
    # Area to be requested 
    # The query return the images that were obtained 
    # in the time range specified
    # Time format (YYYY-MM-DD)
)
xarr
 ```
 El valor de la resolución (-0.00010,0.00010) se debe al 10x10 de Sentinel-2. Probé colocando 0.00005 y no se ve de mejor calidad. Además con 0.0001 ya lanza errores como 

    Unable to allocate 1.97 GiB for an array with shape (3, 8000, 11000) and data type float64

Un error muy interesante es que al cargar las imagenes ingestadas **la latitud y la longitud aparecían invertidas**. Se arregló cambiando el GDAL a 2.4.3. Puedes leer más sobre este issue [aqui]( https://github.com/OSGeo/gdal/issues/1546)

- Graficar con por bandas de la siguiente manera: 

        dataset.<nombrebanda>.plot()

latitude = (4.76, 4.84)
longitude = (-75.91, -75.8)

<img src="https://i.imgur.com/3QPVCJ1.png" width="450" height="250" /> 

latitude = (4.76, 4.84)
longitude = (-76.1, -76)

<img src="https://i.imgur.com/S4DqMgc.png" width="450" height="250" /> 

latitude = (4.76, 4.84)
longitude = (-76.01, -75.9)

<img src="https://i.imgur.com/WMk8csl.png" width="450" height="250" /> 


Se recomienda que al graficar el área no tenga más de 0.1 de diferencia entre latitud y longitud. 

- Graficar imagenes TrueColor. 
Importe la funcion RGB de [aca](https://github.com/GeoscienceAustralia/dea-notebooks/blob/develop/Scripts/dea_plotting.py) y grafique asi:



<img src="https://i.imgur.com/r01CaZ8.png" width="450" height="250" /> 

<img src="https://i.imgur.com/4LpDYZi.png" width="450" height="250" /> 

<img src="https://i.imgur.com/CecON8b.png" width="450" height="250" /> 
)

Y se pueden mejorar la definición en algunos casos asi:
<img src="https://i.imgur.com/r01CaZ8.png" width="350" height="250" /> <img src="https://i.imgur.com/IHWQBcC.png" width="350" height="250"/>

<img src="https://i.imgur.com/CecON8b.png" width="350" height="250" /> <img src="https://i.imgur.com/EnarvP8.png" width="350" height="250"/>

Tambien se pueden hacer analisis más especificos conociendo las mezclas de las bandas:
"The false colour band combination (swir1, nir, green) emphasises growing vegetation in green, and water in deep blue:"
<img src="https://i.imgur.com/Gxez2qX.png" width="450" height="250" />

- Graficar indices de vegetación:

[Este cuaderno](https://github.com/GeoscienceAustralia/dea-notebooks/blob/2d44a50b7213ef82d27c942af73fa7cd2802ac2a/Beginners_guide/06_Basic_analysis.ipynb) muestra como realizar un analisis de la salud de la vegetación. Sin embargo no utiliza s2a_sen2cor_granule como producto. Para saber la correspondencia de las bandas hay que tener el cuenta las longitudes de onda de esta tabla:
<img src="https://www.researchgate.net/publication/327654383/figure/tbl1/AS:670813892132864@1536945903015/Spectral-bands-range-and-spatial-resolution-of-Sentinel-2A-MSI-and-Landsat-8-OLI.png" width="450" height="250" />



Sin embargo, los resultados no fueron los esperados:
<img src="https://i.imgur.com/KX0wNc7.png" width="450" height="250" />


### ***Algunos errores y cómo los solucioné***
    No module named 'odc'

**Solucion:**  Al importar RGB sale este error y necesité importar [odc](https://github.com/opendatacube/odc-tools) con este comando:

 ``pip install --extra-index-url="https://packages.dea.ga.gov.au" odc-ui``

     Cannot open <path>/datacube-script.py
**Solucion:**  Había cambiado el archivo datacube.conf de ubicación. Esta ubicación debe ser actualizada en la variable de entorno *DATACUBE_CONFIG_PATH*. 

```
"source" no se reconoce como un comando interno o externo
```
-  `` activate cubeenv`` en vez de `` source activate cubeenv`` 

    

```
OSError: Could not find lib geos_c.dll or load any of its variants. 
```
**solucion:** Puede ser que el GDAL necesario para que funcione el cubo de 2.4.3 pero para el script de preparacion es GDAL>=3. Entonces desactiva el ambiente cubeenv e instala gdal con y
    ```conda install -c conda-forge gdal```

 **Solucion:** Si no encuentra alguno de los archivos, verifique que el zip se llame igual que la imagen adentro (algo asi como *S2B_MSIL2A_20200104T152639_N0213_R025_T18NVL_20200104T192813*)


        No module named '<paquete>'
**solucion:** correr ```conda install <paquete>```



## Proximos pasos:
- Automatizar la indexación de imagenes ya que cada una se demora unas 6 horas descargando y son muy pesadas.
- Indexar imagenes en multiples fechas de la misma zona para hacer análisis más interesantes que muestren como cambian los cultivos en el tiempo
- Implementar con Sentinel los análisis hechos en [estos cuadernos de LandSat](https://github.com/ceos-seo/data_cube_notebooks/tree/master/DCAL). La principal dificultad es que hay algunas cosas como "fmask" que no trae el producto usado actualmente. Explorar nuevos productos como s2a_nrt_granule o aprender a calcularla.
- Arreglar la gráfica de NDVI, ya que se debería ver algo asi:

<img src="https://i.imgur.com/9nKAAg3.png" width="450" height="250" />

## Puedes encontrar el código en el el jupyter notebook analysis y un proceso más detallado (y desorganizado) en documentacion.MD