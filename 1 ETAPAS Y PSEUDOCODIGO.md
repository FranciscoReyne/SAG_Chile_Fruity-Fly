
# ETAPAS Y PSEUDOCODIGO

A continuacion presentamos una serie de etapas generales para una visualizacion y UX amigable e intuituva de datos de plaga de la mosca.

---

1. Obtención de datos: Desde link SAG, extraer datos manual o automaticamente. descargar fichas tecnicas que son archivos pdf, con datos en tablas.

        https://www.sag.gob.cl/ambitos-de-accion/mosca-de-la-fruta/publicaciones?field_tema_otros_documentos_target_id=2749&field_tipo_de_publicacion_target_id=244&field_fecha_otros_value=&title=&order=field_fecha_otros&sort=desc


2. pasar de pdf a tabla

        camelot: fx read_pdf
   

3. pasar de tabla a pandas dataframe.
        
        df = tables[0].df
        df = pd.DataFrame(df)





4. procesar datos: identificar variables de interés. trabajo de datos satelitales, puntos de muestreo, puntos positivos. Mapas: KeplerGL.

        https://www.youtube.com/watch?v=HpqWSjuUl_A

5. Crear y transformar variables: de datosen tablas a gridmap (rectangular, hexagonal, octagonal).

        https://mappinggis.com/2024/07/uso-de-qgis-para-crear-mapas-hexagonales/


6. (opcional) Resumen de datos, agrupación de datos: Total, total por region, comuna.


7. Propagacion. Estimacion reglas simples,  reg. lineal, machine learning,
8. mapeo heatmap, modelo de propagación (infectabilidad, etc modelos de epidemias).
9. Visualización y Ux de información: kepler gl e installador (.exe)


10. Comentarios finales.



---
---
---



# 2. Código en Google-Colab.


### Instalacion de libreria camelot

        !pip install camelot-py


## pasar de pdf a dataframe: Código base

        import pandas as pd
        from IPython.display import display
        
        # Extraer tablas (desde la primera página)
        tables = camelot.read_pdf("FICHA_A1_Malloa_19032025.pdf", pages="1")
        # Convertir la primera tabla a DataFrame
        df = tables[0].df
        df = pd.DataFrame(df)
        
        #mostrar
        display(df)


### Para mostrar en el mapa el total por comuna: seleccionar dato de ultima fila / columna

    #seleccionar ultima fila
    # Si quieres obtenerlo como DataFrame:
    ultima_fila_df = df.tail(1)
    #mostrar
    display(ultima_fila_df)

ultima columna

    # ultimo valor (total _ total)
    ultima_columna_df_sin_nans = ultima_fila_df.iloc[:, [5]].dropna()
    display(ultima_columna_df_sin_nans)

ultimo valor

    numero = int(ultima_fila_df.iloc[0, 0])
    print(numero)  # Ahora es un número, no un DataFrame


---

### COORDENADAS - UTM WGS 84 A Lat/Lon (EPSG:4326).

Tienes tus coordenadas en **UTM WGS 84**, necesitas convertirlas a **Lat/Lon (EPSG:4326)** para que Kepler.gl pueda interpretarlas correctamente. 🚀  

Aquí tienes cómo hacerlo en Python usando **PyProj**:  

```python
from pyproj import Transformer

# Define la zona UTM y las coordenadas originales
utm_x = TU_X_COORD
utm_y = TU_Y_COORD
zona = TU_ZONA  # Zona UTM (ejemplo: 19)
hemisferio = "north"  # Cambia a "south" si estás en el hemisferio sur

# Convertir de UTM WGS 84 a Lat/Lon
transformer = Transformer.from_crs(f"EPSG:326{zona}" if hemisferio == "north" else f"EPSG:327{zona}", "EPSG:4326")
lon, lat = transformer.transform(utm_x, utm_y)

print(f"Coordenadas en Lat/Lon: {lat}, {lon}")
```

✅ **Convierte tus coordenadas UTM a formato compatible con Kepler.gl**  
✅ **Permite visualizar correctamente los puntos en el mapa**  

Ajusta y haz la prueba con los datos del SAG. 



---


# TRASPASO DE DATOS A MAPAS

##  GEOPANDAS contextily: 
##  Convertir de UTM a lat/lon:Transformer pyproj

!pip install geopandas
!pip install matplotlib
!pip install pyproj
!pip install contextily


    import geopandas as gpd 
    import matplotlib.pyplot as plt
    from pyproj import Transformer
    import contextily as ctx
    
    # Coordenadas UTM (ejemplo)
    utm_x = 345000
    utm_y = 6295000
    zona = 19  # Zona UTM
    hemisferio = "south"  # "north" si estás en el hemisferio norte
    
    # Convertir de UTM a lat/lon
    transformer = Transformer.from_crs(f"EPSG:327{zona}" if hemisferio == "south" else f"EPSG:326{zona}", "EPSG:4326")
    lon, lat = transformer.transform(utm_x, utm_y)
    
    # Crear el punto correctamente
    point = gpd.GeoSeries(gpd.points_from_xy([lon], [lat]), crs="EPSG:4326")  # GeoSeries en lugar de lista
    
    # Crear GeoDataFrame con el punto
    gdf = gpd.GeoDataFrame(geometry=point)
    
    # Graficar con fondo cartográfico
    fig, ax = plt.subplots(figsize=(8, 8))
    gdf.plot(ax=ax, color="red", markersize=100, label="Ubicación UTM")
    ctx.add_basemap(ax, crs=gdf.crs)
    ax.legend()
    plt.show()



      import folium
    
    # Crear mapa centrado en la ubicación
    mapa = folium.Map(location=[lat, lon], zoom_start=15)
    
    # Agregar marcador
    folium.Marker([lat, lon], popup="Ubicación UTM").add_to(mapa)
    
    # Mostrar mapa interactivo
    mapa



!pip install keplergl


    from keplergl import KeplerGl
    import pandas as pd
    
    # Crear DataFrame con coordenadas
    df = pd.DataFrame({'lat': [lat], 'lon': [lon]})
    
    # Crear mapa interactivo
    mapa = KeplerGl(height=600)
    mapa.add_data(data=df, name="Ubicación UTM")
    
    mapa



---

🚀🚀 😃  🚀🚀

🚀🚀🚀 😃 🚀 🚀🚀
