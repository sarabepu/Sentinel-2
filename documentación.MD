# Indexación de imágenes sentinel-2B S2MSI2A al open data cube:

- Descargar imágen de https://scihub.copernicus.eu/dhus/#/home 
- Seguir la documentación de https://datacube-core.readthedocs.io/en/latest/ops/install.html para instalar el cubo y enlazar la base de datos

- Activar el ambiente  ```activate cubeenv```
- Prender la base de datos 
- Agregar los productos ``` datacube product add <path-to-product-definition-yml>```

"Platform will tell you how/where the data is produced. Product is a key used to chose what representation of that platform's data you wish to index."

- Abrir una consola en la carpeta dónde está el documento s2prepare_cophub_zip.py
- Correr para crear el archivo de indexación:

```
python s2prepare_cophub_zip.py <ruta a la imagen comprimida>.zip --output ./<carpeta de salida>/ --no-checksum
```

- Cambiar el code del platform a Sentinel_2B -> por que?

- Indexar la imágen con el comando
```
datacube dataset add <path-to-dataset-document-yaml>
```


# Prueba de algoritmos sobre imagenes indexadas:

**Siempre ANTES activar cubeenv, abrir el jupyter notebook en esa misma consola y prender base de datos postgreSQL**  
- Crear un jupyter notebook con el código para ver las imagenes indexadas:
(El valor de -0.0027,0.0027 fue pensado para landsat según la documentación en https://github.com/ceos-seo/data_cube_ui/blob/master/docs/ingestion.md en la resolución de 30m no estoy segura como es para sentinel) 

 ```python
import datacube
dc = datacube.Datacube(app = "first")
xarr = dc.load(
    # Satellite 
    product="s2b_sen2cor_granule", output_crs='EPSG:4326', resolution=(-0.00027,0.00027)
    # Area to be requested 
    # The query return the images that were obtained 
    # in the time range specified
    # Time format (YYYY-MM-DD)
)
xarr
 ```

**UPDATE** 
    
Según https://app.sandbox.dea.ga.govau/ 
Sección begginers guide-  04-Loading_data

La resolución para 10mx10 debe ser  resolution=(-0.00010,0.00010)
Probé colocanto 0.00005 y no se ve de mejor calidad. Además con 0.0001 ya lanza errores como 

    Unable to allocate 1.97 GiB for an array with shape (3, 8000, 11000) and data type float64

"The example cell below reprojectsdata to a new CRS (UTM Zone 56S,`EPSG:32756`) and resolution (250 x250 m). Note that for most CRSs, thefirst resolution value is negative(e.g. `(-250, 250)`):

    
    ds_reprojected = dc.load(product="ls8_nbart_geomedian_annual",
                         x=(153.3, 153.4),
                         y=(-27.5, -27.6),
                         time=("2015-01-01", "2015-12-31"),
                         output_crs="EPSG:32756",
                         resolution=(-250, 250))
                         





 ## 28/04/2020-> desinstalé GDA->instalé GDA2.4.3

 Hay un problema, al hacer load() se cambian la latitud y la longitud de las imagenes cargadas
 Estaba probando si se arregló el switching de lat y lon

## 30/04/2020 

 - Funcionó cambiar GDAL a 2.4.3 para solucionar el problema de latitud y longitud
 https://github.com/OSGeo/gdal/issues/1546

 - Crear mascaras para ver agua y tierra. Explicar archivo clean_mask
 - revisar cuadernos en https://app.sandbox.dea.ga.gov.au/

 ## 02/05/2020

 - Entendí el parametro crs y resolution
 - Traté de encontrar la banda fmask para las máscaras
 - Averiguar como calcular fmask o si realmente es necesario

 ## 04/05/2020

 - When asking analysis questions about vegetation, it's useful to work with optical imagery, such as Sentinel-2 or Landsat. Sentinel-2 has 10 metre resolution; the DEA collection covers Australia back to 2017. The Landsat satellites (5, 7, and 8) have 30 metre resolution; the DEA collection covers Australia back to 1987. 
 
 For this analysis, we'll work with Sentinel-2, which produces higher-resolution images than Landsat. **For analyses that aim to measure change over very long periods of time (e.g. multiple decades), Landsat may be more suitable.**

 ## 06/05/2020

 Se puede graficar con por bandas de la siguiente manera: 

    dataset.<nombrebanda>.plot()

![ejemplo](https://i.imgur.com/3QPVCJ1.png)

Se recomienda que al graficar el area no tenga más de 0.1 de diferencia entre latitud y longitud. Aunque esta foto está cortada. Para este caso se uso el area de:

latitude = (4.76, 4.84)
longitude = (-75.91, -75.8)

 ## 09/05/2020

 Estoy haciendo pruebas para plottear diferentes bandas con la funcion rgb
 " The rgb() function maps three data variables/measurements from the loaded dataset to the red, green and blue channels that are used to make a three-colour image.

Providing the red, green and blue measurements from a dataset will produce a true colour image (akin to how humans view the landscape). Providing nir, red and green measurements or any other set of three satellite bands from a dataset will produce a false colour image."

 Hay un problema al importar la función RGB del repositorio https://github.com/GeoscienceAustralia/dea-notebooks/blob/develop/Scripts/dea_plotting.py

 
Dice que no puede importar  geopandas. Lo que hice fue eliminar *geopandas y skimage* de los imports del archivo utils/dea_plotting porque no se usan en la función rgb.
Necesité importar odc (https://github.com/opendatacube/odc-tools) con este comando:

    pip install --extra-index-url="https://packages.dea.ga.gov.au" odc-ui

tener en cuenta para las bandas: 
![bandas sentinel](https://www.researchgate.net/publication/327654383/figure/tbl1/AS:670813892132864@1536945903015/Spectral-bands-range-and-spatial-resolution-of-Sentinel-2A-MSI-and-Landsat-8-OLI.png)

![imagen](https://i.imgur.com/MNCKTCq.png)


Luego traté de calcular el Normalised Difference Vegetation Index (NDVI) Pero la resta de las bandas no da lo que espero

- Volviendo a lo de fmask. quizá este producto ayude s2a_nrt_granule. Investigar