# Utilisation des outils de SIG et data analyse pour la planification du district

![image_planning.PNG](geospatial_microplanification_files%5Cmarkdown_1_attachment_0_0.png)

## Introduction
Selon l'UNICEF la microplanification est l�un des outils utilis�s par les agents de sant� pour s�assurer que les services de vaccination touchent chaque communaut�. La microplanification est utilis�e pour identifier les communaut�s prioritaires, pour surmonter les obstacles existants et pour dresser des plans de travail donnant lieu � des solutions.        
Elle est souvent utilis�e lors de la vaccination de routine ou des campagnes, lors de la distribution des MILDA, lors des campagnes de traitement de masse (onchocercose, filariose lymphatique , trachome...).    
Avant d��laborer les microplans, les responsables doivent disposer d�autant d�informations que possible sur les populations cibles. Les donn�es d�mographiques, les rapports des programmes de sant� et les cartes actualis�es des aires de sant� constituent des points de d�part dans le processus de microplanification.      

Une carte op�rationnelle doit comprendre:
- Une zone de desserte connue
- Les villages et les communaut�s couverts par les sites fixes, des sites pour services de proximit� (strat�gies avanc�es) ou lors de journ�es de sant� de la m�re et de l'enfant.
- Les points (POI) d'int�ret importants (points d'eau, �coles, lieux de culte, domiciles de CHW, points de transits de nomades...)
- Les obstacles aux prestations de service (inondation, routes impraticables)
- Les distances et temps de trajet entre sites importants
- Les zones urbaines peuvent utiliserles plans des rues, les cartes satelittaires...)

L'utilisation de ces outils permet de r�pondre � des questions telles que:
- Quel est le lieu de culte le plus �loign� d'une formation sanitaire sp�cifique ?
- Quels sont les domiciles de relais communautaires distants de moins de 10 km du centre de sant�?
- Quels sont les quartiers � plus de 5 km d'un site de vaccination en fixe ou en strat�gie avanc�?


Le pr�sent travaille vise � montrer l'importance des syst�mes d'information g�ographique, du mobile health (kobotoolbox) ainsi que les outils de data analyse (python et ses librairies) dans la microplanification.

## Datasets

Cinq (5) fichiers ont �t� utilis�s pour ce travail de microplanification des activit�s de sant�.
- Le shapefile des aires du district de sant� de YOKO qui comporte les donn�es des 8 aires de sant�.
- Le fichier shapefile de l'aire de sant� de NDJOLE
- Le fichier shapefile de toutes les formations sanitaires du district de sant� de YOKO
- Le fichier excel des points d'int�rets d'une partie de l'aire de NDJOLE. Ces points ont �t� collect�s � partir de l'outil kobocollect.
- Le fichier geojson avec les limites administratives des arrondissements du Cameroun.

La projection pour la zone �tudi�e est : WGS 84 / UTM zone 32N EPSG:32632

## Acquisition, nettoyage et exploration de donn�es.

- Acquerir les donn�es
- Verifier leurs types
- R�cup�rer leur crs.
- Nettoyer les donn�es
- Visualiser ces donn�es

### Importations des librairies


```python
import pandas as pd
import geopandas as gpd
from shapely.geometry import Point
import matplotlib.pyplot as plt
%matplotlib inline
```

#### Shapefile des aires de sant�


```python
district = gpd.read_file('District Yoko/aires_DS_Yoko.shp')
```


```python
type(district)
```




    geopandas.geodataframe.GeoDataFrame




```python
# V�rifier le crs du dataframe
district.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



Ce geodataframe utilise la projection g�ographique WGS 84


```python
district.head()
```




    
![png](geospatial_microplanification_files%5Coutput_13_0.png)
    




```python
district.shape
```




    (8, 12)




```python
district.columns
```




    Index(['OBJECTID_1', 'Id', 'Nom_AS', 'Nom_Dist', 'Source1', 'Source2',
           'Code_AS', 'Source3', 'Region', 'Area', 'Population', 'geometry'],
          dtype='object')




```python
district = district[['Nom_AS', 'Nom_Dist', 'Code_AS', 'Region', 'Area', 'Population', 'geometry']]
```


```python
district
```




    
![png](geospatial_microplanification_files%5Coutput_17_0.png)
    




```python
district.shape
```




    (8, 7)




```python
#district.plot(figsize=(15,15), edgecolor="white")
#plt.show()
```


```python
district.plot(figsize=(15,15), edgecolor="white", column='Nom_AS', legend=True)
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_20_0.png)
    


Nous nous interesserons plus sp�cifiquement � l'aire de sant� de NDJOLE en gris et plus au sud

#### Shapefile de l'aire de NDJOLE


```python
djole_health_area = gpd.read_file('District Yoko/AS_Centre_Nom_AS__Ndjole.shp')
```


```python
djole_health_area.shape
```




    (1, 11)




```python
type(djole_health_area)
```




    geopandas.geodataframe.GeoDataFrame




```python
djole_health_area.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich




```python
djole_health_area.plot(figsize=(15,15))
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_27_0.png)
    


#### Shapefile des formations sanitaires


```python
fosas = gpd.read_file('District Yoko/FOSA_Cameroon_DISTRICT__Yoko.shp')
```


```python
type(fosas)
```




    geopandas.geodataframe.GeoDataFrame




```python
fosas.crs
```

Ce geodataframe utilise aucune coordonn�e. Nous assignons la projection g�ographique WGS 84. La m�me que celle du g�odataframe district


```python
fosas = gpd.GeoDataFrame(fosas, geometry=fosas.geometry, crs = district.crs)
```


```python
fosas.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



Ce geodataframe utilise desormais la projection g�ographique WGS 84


```python
fosas.head()
```




    
![png](geospatial_microplanification_files%5Coutput_36_0.png)
    




```python
fosas.columns
```




    Index(['REGION', 'DISTRICT', 'HEALTH_ARE', 'NAME', 'PEV', 'Lat', 'Lon',
           'Altitude', 'Type', 'Accuracy', 'comments', 'Form', 'geometry'],
          dtype='object')




```python
fosas = fosas[['REGION', 'DISTRICT', 'HEALTH_ARE', 'NAME', 'PEV', 'Lat', 'Lon', 'Altitude', 'Type', 'Form', 'geometry']]
```


```python
fosas.head()
```




    
![png](geospatial_microplanification_files%5Coutput_39_0.png)
    




```python
fosas.shape
```




    (31, 11)




```python
# Visualisation des fosas
#fosas.plot(figsize=(15,15), color='red')
#plt.show()
```

#### Fichier excel des points d'int�ret (POI)


```python
pois = pd.read_excel('Point_d_interets.xlsx')
```


```python
pois.head()
```




    
![png](geospatial_microplanification_files%5Coutput_44_0.png)
    




```python
pois.shape
```




    (281, 22)




```python
pois.columns
```




    Index(['start', 'end',
           'Point d'int�r�t utile pour votre micro-planification. Veuillez g�olocaliser ce point et collecter les infos � propos.',
           'Quelle type de point voulez-vous renseignez ?',
           'Si autre, pr�cisez le type de site',
           'Nom du site (nom du chef de quartier, du relais communautaire... bref du site)',
           'Nom du quartier', 'Enregistrez les coordonn�es du site',
           '_Enregistrez les coordonn�es du site_latitude',
           '_Enregistrez les coordonn�es du site_longitude',
           '_Enregistrez les coordonn�es du site_altitude',
           '_Enregistrez les coordonn�es du site_precision',
           'Quelle type de point voulez-vous renseignez ?.1', '_id', '_uuid',
           '_submission_time', '_validation_status', '_notes', '_status',
           '_submitted_by', '_tags', '_index'],
          dtype='object')




```python
# Changer (R�duire) le nom des colonnes
col_dict = {
    'Quelle type de point voulez-vous renseignez ?':'Type de poi',
    'Si autre, pr�cisez le type de site':'autre type de poi',
    'Nom du site (nom du chef de quartier, du relais communautaire... bref du site)':'Nom du poi',
    'Nom du quartier':'Quartier',
    '_Enregistrez les coordonn�es du site_latitude':'Latitude',
    '_Enregistrez les coordonn�es du site_longitude':'Longitude',
    '_Enregistrez les coordonn�es du site_altitude':'Altitude'
}

pois.rename(columns=col_dict, inplace=True)
```


```python
pois.head()
```




    
![png](geospatial_microplanification_files%5Coutput_48_0.png)
    




```python
cols = list(col_dict.values())
cols
```




    ['Type de poi',
     'autre type de poi',
     'Nom du poi',
     'Quartier',
     'Latitude',
     'Longitude',
     'Altitude']




```python
pois = pois[cols]
```


```python
pois.head()
```




    
![png](geospatial_microplanification_files%5Coutput_51_0.png)
    




```python
pois.shape
```




    (281, 7)




```python
type(pois)
```




    pandas.core.frame.DataFrame



##### Pour faciliter les analyse spatiales, il faut transformer ce dataframe en geodataframe


```python
pois['geometry'] = pois.apply(lambda x : Point(x.Longitude, x.Latitude), axis=1)
```

    C:\Users\hp\anaconda3\lib\site-packages\pandas\core\dtypes\cast.py:122: ShapelyDeprecationWarning: The array interface is deprecated and will no longer work in Shapely 2.0. Convert the '.coords' to a numpy array instead.
      arr = construct_1d_object_array_from_listlike(values)
    


```python
pois.head()
```




    
![png](geospatial_microplanification_files%5Coutput_56_0.png)
    



Une colonne nouvelle geometry avec les valeur de type Point


```python
type(pois)
```




    pandas.core.frame.DataFrame




```python
pois_geo = gpd.GeoDataFrame(pois, geometry=pois.geometry, crs = district.crs)
```


```python
type(pois_geo)
```




    geopandas.geodataframe.GeoDataFrame




```python
pois_geo.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich




```python
pois_geo['Type de poi'].value_counts(dropna=False)
```




    Carrefour                                                    46
    Site de strat�gie avanc�e (vaccination, CPN recentr�e...)    38
    Source d'eau                                                 37
    Domicile de relais de communautaire                          30
    Lieu de culte                                                24
    Domicile de chef de quartier                                 24
    Autre                                                        23
    Ecole                                                        20
    Formation sanitaire                                          16
    Pont                                                          9
    March�                                                        7
    Domicile de tradi-praticien                                   5
    NaN                                                           1
    Site de distribution de MILDA                                 1
    Name: Type de poi, dtype: int64




```python
pois_geo['Type de poi'].value_counts(dropna=False).sum()
```




    281



On d�nombre 281 POI dont 23 autres pois sans cat�gories et qui se pr�sentent comme suit:


```python
pois_geo['autre type de poi'].unique()
```




    array([nan, 'Auberge', 'Poste agricole', 'Gare routi�re', 'Agence voyage',
           'Restaurant', 'Gendarmerie ', 'St�le centenaire �glise catholique',
           "Soci�t� d'exploitation foresti�re", 'Talla', 'Foyer ',
           'Gare routi�re ', 'Limite entre zone yoko-ntui(issandja-nguila)',
           'Une Vall� avec rivi�re', 'Village ',
           'Piste cacaoy�re- vers cosovo', 'Un quartier', 'Bord ', 'Quartier',
           'Acoucheuse traditionnelle', 'Bac de Bioko (fleuve Mbam)',
           'Bac debioko', 'CS family haerth care', 'Parc national maoendjim'],
          dtype=object)



Il faut enlever les espaces vides � la fin de certaines valeurs comme "Gendarmerie "


```python
pois_geo['autre type de poi'] = pois_geo['autre type de poi'].apply(lambda x:str(x).strip())
```


```python
pois_geo['autre type de poi'].unique()
```




    array(['nan', 'Auberge', 'Poste agricole', 'Gare routi�re',
           'Agence voyage', 'Restaurant', 'Gendarmerie',
           'St�le centenaire �glise catholique',
           "Soci�t� d'exploitation foresti�re", 'Talla', 'Foyer',
           'Limite entre zone yoko-ntui(issandja-nguila)',
           'Une Vall� avec rivi�re', 'Village',
           'Piste cacaoy�re- vers cosovo', 'Un quartier', 'Bord', 'Quartier',
           'Acoucheuse traditionnelle', 'Bac de Bioko (fleuve Mbam)',
           'Bac debioko', 'CS family haerth care', 'Parc national maoendjim'],
          dtype=object)



Les 23 valeurs 'autre' se repatissent comme suit


```python
pois_geo['autre type de poi'].value_counts()
```




    nan                                             258
    Gare routi�re                                     2
    Une Vall� avec rivi�re                            1
    CS family haerth care                             1
    Bac debioko                                       1
    Bac de Bioko (fleuve Mbam)                        1
    Acoucheuse traditionnelle                         1
    Quartier                                          1
    Bord                                              1
    Un quartier                                       1
    Piste cacaoy�re- vers cosovo                      1
    Village                                           1
    Limite entre zone yoko-ntui(issandja-nguila)      1
    Auberge                                           1
    Foyer                                             1
    Talla                                             1
    Soci�t� d'exploitation foresti�re                 1
    St�le centenaire �glise catholique                1
    Gendarmerie                                       1
    Restaurant                                        1
    Agence voyage                                     1
    Poste agricole                                    1
    Parc national maoendjim                           1
    Name: autre type de poi, dtype: int64




```python
pois_geo['autre type de poi'].value_counts().sum()
```




    281



##### Les POI  "autres" peuvent �tre encore re-categoris�s en:     
    - "Hotel et Restaurant"      
    - "Gare routi�re et Agence"        
    - "Domicile Accoucheuse traditionnelle"      
    - "Edifice public" (bac, gendarmerie, poste agricole)      
    - "Monument" (st�le)       
    - "B�timent priv�" (foyer, Soci�t� d'exploitation foresti�re )  
    - "Parc"


```python
dict_poi = {
    "Auberge":"Hotel et Restaurant",
    "Restaurant":"Hotel et Restaurant",
    "Gare routi�re":"Gare routi�re et Agence",
    "Agence voyage":"Gare routi�re et Agence",
    "Gendarmerie":"Edifice public",
    "Bac de Bioko (fleuve Mbam)":"Edifice public",
    "Bac debioko":"Edifice public",
    "Poste agricole":"Edifice public",
    "St�le centenaire �glise catholique":"Monument",
    "Foyer":"B�timent priv�",
    "Soci�t� d'exploitation foresti�re":"B�timent priv�",
    "Parc national maoendjim":"Parc",
    "Acoucheuse traditionnelle":"Domicile Accoucheuse traditionnelle"
}
```


```python
pois_geo["autre type de poi"].replace(dict_poi, inplace=True)
```


```python
pois_geo['autre type de poi'].value_counts()
```




    nan                                             258
    Edifice public                                    4
    Gare routi�re et Agence                           3
    B�timent priv�                                    2
    Hotel et Restaurant                               2
    Un quartier                                       1
    CS family haerth care                             1
    Domicile Accoucheuse traditionnelle               1
    Quartier                                          1
    Bord                                              1
    Une Vall� avec rivi�re                            1
    Piste cacaoy�re- vers cosovo                      1
    Village                                           1
    Limite entre zone yoko-ntui(issandja-nguila)      1
    Talla                                             1
    Monument                                          1
    Parc                                              1
    Name: autre type de poi, dtype: int64




```python
list(pois_geo['autre type de poi'].value_counts().keys())
```




    ['nan',
     'Edifice public',
     'Gare routi�re et Agence',
     'B�timent priv�',
     'Hotel et Restaurant',
     'Un quartier',
     'CS family haerth care',
     'Domicile Accoucheuse traditionnelle',
     'Quartier',
     'Bord',
     'Une Vall� avec rivi�re',
     'Piste cacaoy�re- vers cosovo',
     'Village',
     'Limite entre zone yoko-ntui(issandja-nguila)',
     'Talla',
     'Monument',
     'Parc']




```python
# Les valeurs de POI � exclure de l'analyse
autres_pois = [
 'Un quartier',
 'CS family haerth care',
 'Quartier',
 'Une Vall� avec rivi�re',
 'Piste cacaoy�re- vers cosovo',
 'Village',
 'Limite entre zone yoko-ntui(issandja-nguila)',
 'Talla',
 'Bord']
```


```python
pois_geo = pois_geo[~pois_geo['autre type de poi'].isin(autres_pois)]
```


```python
pois_geo.shape
```




    (272, 8)




```python
pois_geo.isnull().sum()
```




    Type de poi           1
    autre type de poi     0
    Nom du poi            4
    Quartier             36
    Latitude              3
    Longitude             3
    Altitude              3
    geometry              0
    dtype: int64



##### On note une valeur manquante pour la variable 'Type de poi' qui est visiblement une �cole


```python
pois_geo.loc[pois_geo['Type de poi'].isna()]
```




    
![png](geospatial_microplanification_files%5Coutput_82_0.png)
    




```python
pois_geo.loc[pois_geo['Type de poi'].isna(), 'Type de poi'] = 'Ecole'
```


```python
pois_geo['autre type de poi'].value_counts(dropna=False)
```




    nan                                    258
    Edifice public                           4
    Gare routi�re et Agence                  3
    Hotel et Restaurant                      2
    B�timent priv�                           2
    Monument                                 1
    Domicile Accoucheuse traditionnelle      1
    Parc                                     1
    Name: autre type de poi, dtype: int64




```python
pois_geo['Type de poi'].value_counts(dropna=False)
```




    Carrefour                                                    46
    Site de strat�gie avanc�e (vaccination, CPN recentr�e...)    38
    Source d'eau                                                 37
    Domicile de relais de communautaire                          30
    Lieu de culte                                                24
    Domicile de chef de quartier                                 24
    Ecole                                                        21
    Formation sanitaire                                          16
    Autre                                                        14
    Pont                                                          9
    March�                                                        7
    Domicile de tradi-praticien                                   5
    Site de distribution de MILDA                                 1
    Name: Type de poi, dtype: int64




```python
pois_geo['Type de poi'].value_counts(dropna=False).sum()
```




    272



##### On va affecter aux valeurs manquantes de la colonne 'Type de pois', la valeur correspondant de la colonne 'autre type de poi'.Pour cela on cr�e une fonction 


```python
def change_autre(row):
    if row['Type de poi']=="Autre":
        return row['autre type de poi']
    else:
        return row['Type de poi']
```


```python
pois_geo['Type de poi'] = pois_geo.apply(change_autre, axis=1)
```


```python
pois_geo['Type de poi'].value_counts()
```




    Carrefour                                                    46
    Site de strat�gie avanc�e (vaccination, CPN recentr�e...)    38
    Source d'eau                                                 37
    Domicile de relais de communautaire                          30
    Lieu de culte                                                24
    Domicile de chef de quartier                                 24
    Ecole                                                        21
    Formation sanitaire                                          16
    Pont                                                          9
    March�                                                        7
    Domicile de tradi-praticien                                   5
    Edifice public                                                4
    Gare routi�re et Agence                                       3
    B�timent priv�                                                2
    Hotel et Restaurant                                           2
    Monument                                                      1
    Domicile Accoucheuse traditionnelle                           1
    Site de distribution de MILDA                                 1
    Parc                                                          1
    Name: Type de poi, dtype: int64



##### On note un grand nombre de POI de type "carrefour". A la place on aurait pu collecter des g�otraces en lieu et place des points


```python
type(pois_geo)
```




    geopandas.geodataframe.GeoDataFrame




```python
pois_geo.shape
```




    (272, 8)




```python
pois_geo.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich




```python

```

#### Fichier geojson des limites administratives au niveau arrondissement


```python
cam_3 = gpd.read_file('Cameroon/geoBoundaries-CMR-ADM3_simplified.geojson')
```


```python
cam_3.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich




```python
type(cam_3)
```




    geopandas.geodataframe.GeoDataFrame




```python
cam_3.head()
```




    
![png](geospatial_microplanification_files%5Coutput_100_0.png)
    




```python
cam_3.shapeName.unique()
```




    array(['Mbonge', 'Magba', 'Mélong', 'Fokoué', 'Bibemi', 'Fonfuka',
           'Nwa', 'Ambam', 'Baschéo', 'Ngoura', 'Dir', 'Fundong',
           'Mayo-Oulo', 'Bandja', 'Bangangté', 'Bikok', 'Mbankomo',
           'Malentouen', 'Benakuma', 'Touboro', 'Santchou', 'Zhoa',
           'Nitoukou', 'Bengbis', 'Baré', 'Nguti', 'Béka', 'Bamenda II',
           'Ngoro', 'Nkolafamba', 'Douala VI', 'Galim', 'Nanga-Eboko',
           'Kétté', 'Kar-Hay', 'Yokadouma', 'Belo', 'Dziguilao',
           'Logone-Birni', 'Mamfe', 'Mouloundou', 'Mokolo', 'Dikome-Balue',
           'Kaélé', 'Akonolinga', 'Ekondo-Titi', 'Ngambé-Tikar', "Ngan'ha",
           'Yingui', 'Mbiame', 'Banyo', 'Martap', 'Tombel', 'Yaoundé III',
           'Yaoundé IV', 'Tabati', 'Pitoa', 'Rey-Bouba', 'Ayos',
           'Bafoussam I', 'Wum', 'Bokito', 'Akwaya', 'Campo', 'Mboma',
           'Nkondjock', 'Njimom', 'Bafoussam II', 'Deuk', 'Bétaré-Oya',
           'Ndikiniméki', 'Ndélélé', 'Furu-Awa', 'Douala IV', 'Djohong',
           'Efoulan', 'Goulfey', 'Tchatibali', 'Bankim', 'Garoua-Boulaï',
           'Kribi II', 'Toko', 'Yabassi', 'Yoko', 'Lokoundjé', 'Waza',
           'Pete-Bandjoun', 'Wina', 'Messock', 'Mouanko', 'Bondjock', "Ma'an",
           'Batouri', 'Edzendouan', 'Lembe Yezoum', 'Mbouda', 'Ako', 'Mozogo',
           'Mudemba', 'Bamusso', 'Batchenga', 'Hina', 'Konye', 'Bamenda III',
           'Meyomessala', 'Ebolowa I', 'Maroua II', 'Mindif', 'Ndu', 'Bélel',
           'Galim-Tignère', 'Akoéman', 'Bassamba', 'Eyumodjock', 'Guéré',
           'Makary', 'Meiganga', 'Ntui', 'Mintom', 'Guémé', 'Bafia',
           'Baham', 'Ebolowa II', 'Widikum-Boffé', 'Kentzou', 'Nyambaka',
           'Yaoundé V', 'Ouli', 'Touloum', 'Sangmélima', 'Bangem',
           'Bamenda I', 'Kolofata', 'Bafang', 'Mayo-Darlé', 'Mbengwi',
           'Tokombéré', 'Nkongsamba II', 'Tcholliré', 'Mbang', 'Kai-Kai',
           'Guidiguis', 'Garoua III', 'Foumbot', 'Elig-Mfomo', 'Mayo-Baléo',
           'Makenene', 'Mbé', 'Misaje', 'Moutourwa', 'Maroua I', 'Lomié',
           'Yaoundé VI', 'Bafut', 'Mbanga', 'Niété', 'Lolodorf', 'Nkambe',
           'Nkoteng', 'Soa', 'Dargala', 'Kobdombo', 'Kouoptamo',
           'Moulvoudaye', 'Djoum', 'Ngaoundal', 'Kontcha', 'Pétté',
           'Njinikom', 'Soulèdé-Roua', 'Mindourou', 'Kousséri',
           'Massangam', 'Esse', 'Meyomessi', 'Poli', 'Afanloum', 'Angossas',
           'Babadjou', 'Biwong-Bulu', 'Douala V', 'Fongo-Tongo', 'Ngaoui',
           'Nsem', 'Zoétélé', 'Zina', 'Yaoundé VII', 'Yaoundé II',
           'Yaoundé I', 'Yagoua', 'Wabane', 'Tubah', 'Touroua', 'Tonga',
           'Tinto', 'Tiko', 'Somalomo', 'Santa', 'Salapoumbé', "Sa'a",
           'Pouma', 'Penka-Michel', 'Penja', 'Oveng', 'Ombessa', 'Olanguina',
           'Olamzé', 'Okala', 'Obala', 'Nyanon', 'Nkum', 'Nkor', 'Nkong-Zem',
           'Nkongsamba III', 'Nkongsamba I', 'Nkol-Métet', 'Nijkwa', 'Ngwei',
           'Nguibassal', 'Nguelemendouka', 'Nguélébok', 'Ngoyla', 'Ngoumou',
           'Ngoulémakong', 'Ngong', 'Ngomedzap', 'Ngog-Mapubi',
           'Ngaoundéré III', 'Ngaoundéré II', 'Ngaoundéré I', 'Ngambé',
           'Ndoukoula', 'Ndop', 'Ndom', 'Ndobian', 'Mvengue', 'Mvangan',
           'Muyuka', 'Mora', 'Monatélé', 'Mombo', 'Mogodé', 'Minta',
           'Mfou', 'Messondo', 'Messaména', 'Meri', 'Menji', 'Mengueme',
           'Mengong', 'Mengang', 'Mbangassina', 'Mbandjock', 'Mbalmayo',
           'Mayo-Hourna', 'Matomb', 'Massok', 'Maroua III', 'Manjo',
           'Mandjou', 'Makak', 'Maga', 'Madingring', 'Loum', 'Lobo',
           'Limbé III', 'Limbé II', 'Limbé I', 'Lagdo', 'Kyé-Ossi',
           'Kumbo', 'Kumba III', 'Kumba II', 'Kumba I', 'Koza', 'Koutaba',
           'Kon-Yambetta', 'Kombo-Itindi', 'Kombo-Abedimo', 'Kiiki', 'Banwa',
           'Banka', 'Bangourain', 'Bangou', 'Bana', 'Bamendjou', 'Balikumbat',
           'Bali', 'Bakou', 'Bafoussam III', 'Babessi', 'Awaé', 'Atok',
           'Andek', 'Alou', 'Akono', 'Akom II', 'Abong-Mbang', 'Datchéka',
           'Kékem', 'Kalfou', 'Jakiri', 'Isanguele', 'Idenau', 'Idabato',
           'Hilé - Alifa', 'Guider', 'Gobo', 'Gazawa', 'Garoua II',
           'Garoua I', 'Gari-Gombo', 'Gachiga', 'Foumban', 'Fotokol',
           'Figuil', 'Evodoula', '�\x89séka', 'Endom', 'Elak',
           '�\x89déa II', '�\x89déa I', 'Ebone', 'Ebebda', 'Dzeng',
           'Dschang', 'Doumé', 'Doumaintang', 'Douala III', 'Douala II',
           'Douala I', 'Dizangué', 'Dimako', 'Dibombari', 'Dibang',
           'Dibamba', 'Diang', 'Demding', 'Dembo', 'Darak', 'Buea', 'Bourha',
           'Bot-Makak', 'Bonaléa', 'Bogo', 'Blangoua', 'Biyouha',
           'Biwong-Bané', 'Bipindi', 'Bibey', 'Bertoua II', 'Bertoua I',
           'Bélabo', 'Bazou', 'Bayangam', 'Batié', 'Batibo', 'Batcham',
           'Kribi I', 'Tignère'], dtype=object)




```python
len(cam_3.shapeName.unique())
```




    360



On d�nombre 360 arrondissements


```python
#Visualisation des arrondissements
cam_3.plot()
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_104_0.png)
    


##### On extrait les 2 arrondissements qui constituent le district (YOKO, NGAMBE TIKAR)


```python
cam_adm_district_yoko = cam_3[cam_3.shapeName.isin(['Yoko', 'Ngambé-Tikar'])]
```


```python
cam_adm_district_yoko.plot(column='shapeName', legend=True, figsize=(10,10), edgecolor='white')
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_107_0.png)
    


## Comment visualiser plusieurs couches sur un m�me graphique


```python
# Visualiser la couche des aires et celles des formations sanitaires
f, ax = plt.subplots(figsize=(15,15))
district.plot(ax=ax, edgecolor='white')
fosas.plot(ax=ax, color='red')
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_109_0.png)
    



```python
# Visualiser la couche des aires et celles des points d'int�ret
f, ax = plt.subplots(figsize=(15,15))
district.plot(ax=ax, edgecolor='white')
pois_geo.plot(ax=ax,column='Type de poi', legend=True, legend_kwds={'bbox_to_anchor': (1.5, 1)})
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_110_0.png)
    


##### Certains points d'int�ret se retrouvent hors des limites du districts surtout au niveau de l'aire de NDJOLE au sud du district


```python
# Visualiser la couche des aires et celles des limites administratives des arrondissements de YOKO et NGAMBE TIKAR
f, ax = plt.subplots(figsize=(15,15))
cam_adm_district_yoko.plot(ax=ax, color='green', alpha=0.5)
district.plot(ax=ax, alpha=0.5, edgecolor='white')
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_112_0.png)
    


![legende.PNG](geospatial_microplanification_files%5Cmarkdown_113_attachment_0_0.png)

On constate que la carte du district de sant� ne "match" pas avec celle des limites administratives. Ce qui peut expliquer le fait que certains POI au niveau de NDJOLE dans le sud se retrouve hors des limites du district


```python

```

## Analyse spatiale

### Analyse spatiale de type jointure pour r�cup�rer seulement les POI qui se trouvent dans l'aire de NDJOLE


```python
pois_in_djole = gpd.sjoin(pois_geo, djole_health_area, op='within')
```

    C:\Users\hp\anaconda3\lib\site-packages\IPython\core\interactiveshell.py:3472: FutureWarning: The `op` parameter is deprecated and will be removed in a future release. Please use the `predicate` parameter instead.
      if (await self.run_code(code, result,  async_=asy)):
    


```python
pois_in_djole.head()
```




    
![png](geospatial_microplanification_files%5Coutput_119_0.png)
    




```python
#visualisation uniquement des POI de l'aire de NDJOLE dans tout le district.
f, ax = plt.subplots(figsize=(15,15))
district.plot(ax=ax, edgecolor='white')
pois_in_djole.plot(ax=ax,column='Type de poi', legend=True, legend_kwds={'bbox_to_anchor': (1.4, 1)})
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_120_0.png)
    


##### Apr�s analyse spatiale, on a r�cup�r� uniquement les POI se retrouvant dans l'aire de NDJOLE


```python
#visualisation uniquement de l'aire de NDJOLE avec ces POI.
f, ax = plt.subplots(figsize=(25,25))
djole_health_area.plot(ax=ax, edgecolor='white')
pois_in_djole.plot(ax=ax,column='Type de poi', legend=True)
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_122_0.png)
    



```python

```

### Analyse spatiale de type jointure pour r�cup�rer seulement les FOSA qui se trouvent dans l'aire de NDJOLE


```python
fosas_in_djole = gpd.sjoin(fosas, djole_health_area, op='within')
```

    C:\Users\hp\anaconda3\lib\site-packages\IPython\core\interactiveshell.py:3472: FutureWarning: The `op` parameter is deprecated and will be removed in a future release. Please use the `predicate` parameter instead.
      if (await self.run_code(code, result,  async_=asy)):
    


```python
# Visualisation des FOSA dans l'aire de NDJOLE
f, ax = plt.subplots(figsize=(25,25))
djole_health_area.plot(ax=ax)
fosas_in_djole.plot(ax=ax,color='red', markersize=50, legend=True)
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_126_0.png)
    


### Visualisation avec annotations


```python
#ax = district.plot(figsize=(25,25), edgecolor='white')
#fosas.plot(ax=ax, markersize=100, color='red')

#for x, y, label in zip(fosas.geometry.x, fosas.geometry.y, fosas.NAME):
#    ax.annotate(label, xy=(x, y), xytext=(3, 3), textcoords="offset points")

#plt.show()
```


```python

```

## Changement des projections des geodataframes et leur sauvegarde sous forme de fichier shapefile et gpkg

### Conversion des g�odataframe en projection utm
##### La projection pour la zone �tudi�e est : WGS 84 / UTM zone 32N EPSG:32632


```python
district_utm_32632 = district.to_crs(epsg=32632)
fosas_utm_32632 = fosas.to_crs(epsg=32632)
pois_utm_32632 = pois_geo.to_crs(epsg=32632)
pois_in_djole_utm_32632 = pois_in_djole.to_crs(epsg=32632)
djole_health_area_utm_32632 = djole_health_area.to_crs(epsg=32632)
```


```python
#district_utm_32632.crs
```


```python
#pois_in_djole_utm_32632.crs
```


```python
#fosas_utm_32632.crs
```


```python

```

### Sauvegarder les g�odataframes avec la nouvelle projection sous forme de fichiers (shapefile ou gpkg)


```python
# Write converted data to a file
#district_utm_32632.to_file("data_output/gpkg/district_utm_32632.gpkg",driver="GPKG")
#district_utm_32632.to_file("data_output/shapefile/district_utm_32632.shp",driver="ESRI Shapefile")
```


```python
# Write converted data to a file
#fosas_utm_32632.to_file("data_output/gpkg/fosa_utm_32632.gpkg",driver="GPKG")
#fosas_utm_32632.to_file("data_output/shapefile/fosa_utm_32632.shp",driver="ESRI Shapefile")
```


```python
# Write converted data to a file
#pois_utm_32632.to_file("data_output/gpkg/pois_utm_32632.gpkg",driver="GPKG")
#pois_utm_32632.to_file("data_output/shapefile/pois_utm_32632.shp",driver="ESRI Shapefile")
```


```python
# Write converted data to a file
#pois_in_djole_utm_32632.to_file("data_output/gpkg/pois_in_djole_utm_32632.gpkg",driver="GPKG")
#pois_in_djole_utm_32632.to_file("data_output/shapefile/pois_in_djole_utm_32632.shp",driver="ESRI Shapefile")
```


```python
# Write converted data to a file
#djole_health_area_utm_32632.to_file("data_output/gpkg/djole_health_area_utm_32632.gpkg",driver="GPKG")
#djole_health_area_utm_32632.to_file("data_output/shapefile/djole_health_area_utm_32632.shp",driver="ESRI Shapefile")
```


```python

```

## Visualisation avec folium

#### Fusionner toutes les aires en une entit� g�om�trique


```python
district_geometry = district.unary_union
```


```python
type(district_geometry)
```




    shapely.geometry.polygon.Polygon




```python
# On r�cup�re le centre du polygone district
district_centroide = district_geometry.centroid
```


```python
district.shape
```




    (8, 7)




```python
display(district_geometry)
```


    
![svg](geospatial_microplanification_files%5Coutput_150_0.svg)
    


Le resultat est objet unique de type geometry.Polygon


```python
print(district_centroide)
```

    POINT (12.084625074625666 5.558017685238768)
    

##### L'argument location est une liste de 2 �l�ments dont le premier est la latitude et le second est la longitude)


```python
#Latitude avant la longitude
district_location = [district_centroide.y, district_centroide.x]
```


```python
district_location
```




    [5.558017685238768, 12.084625074625666]



##### L'argument location de la fonction folium.map doit �tre de coordonn�es g�ographique WGS 84


```python
type(district_location)
```




    list




```python
import folium
district_map = folium.Map(location=district_location, zoom_start=15)
```


```python
#display(district_map)
```


```python
#district_map = folium.Map(location=district_location, zoom_start=9)
#folium.GeoJson(district_utm_32632.geometry).add_to(district_map)
#folium.GeoJson(fosas_utm_32632.geometry).add_to(district_map)
#display(district_map)
```


```python
djole_center = djole_health_area.geometry.centroid
```

    C:\Users\hp\AppData\Local\Temp\ipykernel_13116\136739247.py:1: UserWarning: Geometry is in a geographic CRS. Results from 'centroid' are likely incorrect. Use 'GeoSeries.to_crs()' to re-project geometries to a projected CRS before this operation.
    
      djole_center = djole_health_area.geometry.centroid
    


```python
djole_center
```




    0    POINT (11.83688 4.84787)
    dtype: geometry




```python
djole_location = [djole_center.y, djole_center.x]
```


```python
djole_location
```




    [0    4.847866
     dtype: float64,
     0    11.836876
     dtype: float64]




```python
pois_in_djole.head()
```




    
![png](geospatial_microplanification_files%5Coutput_165_0.png)
    




```python
#djole_map = folium.Map(location=djole_location, zoom_start=11)
#folium.GeoJson(djole_health_area.geometry).add_to(djole_map)
#folium.GeoJson(pois_in_djole.geometry).add_to(djole_map)
#display(djole_map)
```


```python
#djole_map = folium.Map(location=djole_location, zoom_start=11)
#folium.GeoJson(djole_health_area.geometry).add_to(djole_map)
# Create a location and marker with each iteration for the djole_map
#for row in pois_in_djole.iterrows():
#    row_values = row[1] 
#    location = [row_values['Latitude'], row_values['Longitude']]
#    popup =  str(row_values['Type de poi']) + ' ' + str(row_values['Nom du poi'])
#    marker = folium.Marker(location = location, popup = popup)
#    marker.add_to(djole_map)

# Display the map
#display(djole_map)
```


```python
# Folium avec image satelittaire et popup
#folium.TileLayer(
#        tiles = 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
#        attr = 'Esri',
#        name = 'Esri Satellite',
#        overlay = False,
#        control = True
#).add_to(djole_map)
#display(djole_map)
```


```python

```

## Analyse spatiale

## Analyse spatiale: les zones tampons

Cr�er une zone tampon de 5 km autour des formations sanitaires


```python
fosas_utm_32632_buffered = fosas_utm_32632.copy()
```


```python
fosas_utm_32632_buffered['geometry'] = fosas_utm_32632_buffered.apply(lambda row:row.geometry.buffer(5000),axis=1)
```


```python
fosas_utm_32632_buffered.head()
```




    
![png](geospatial_microplanification_files%5Coutput_175_0.png)
    




```python
type(fosas_utm_32632_buffered)
```




    geopandas.geodataframe.GeoDataFrame



##### Ce geodataframe tampon a sa variable "g�ometry" de type polygone. On peut donc visualiser


```python
fosas_utm_32632_buffered.plot(figsize=(15,15))
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_178_0.png)
    


##### Visualisation des formations sanitaires (points rouges) et de leurs zones tampons (en blanc)


```python
ax = district_utm_32632.plot(figsize=(15,15), edgecolor='white')
fosas_utm_32632_buffered.plot(ax=ax, alpha=0.5, color='white')
fosas_utm_32632.plot(ax=ax, markersize=25, color='red')

plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_180_0.png)
    



```python

```

## Analyse spatiale:distance

### Quels sont les quartiers/villages � plus de 5 km d'un service de vaccination dans l'aire de NDJOLE ?

##### On r�cup�re les POI des quartiers repr�sent�s par le domicile de leurs chefs 


```python
pois_quartier_djole_utm_32632 = pois_in_djole_utm_32632[pois_in_djole_utm_32632['Type de poi'] == 'Domicile de chef de quartier']
```

##### On r�cup�re les POI des sites de trat�gies ainsi que leur zone tampon de 5 km 


```python
# POI de strat�gie avanc�
pois_SA_djole_utm_32632 = pois_in_djole_utm_32632[pois_in_djole_utm_32632['Type de poi'] == 'Site de strat�gie avanc�e (vaccination, CPN recentr�e...)']
```


```python
pois_SA_djole_utm_32632_buffered = pois_SA_djole_utm_32632.copy()
```


```python
pois_SA_djole_utm_32632_buffered['geometry'] = pois_SA_djole_utm_32632_buffered.apply(lambda row:row.geometry.buffer(5000),axis=1)
```

##### On r�cup�re les POI type formation sanitaire ainsi que leur zone tampon de 5 km 


```python
fosas_in_djole_utm_32632 = fosas_in_djole.to_crs(epsg=32632)
```


```python
fosas_utm_32632_buffered.head()
```




    
![png](geospatial_microplanification_files%5Coutput_192_0.png)
    




```python
fosas_utm_32632_buffered_in_djole = fosas_utm_32632_buffered[fosas_utm_32632_buffered['HEALTH_ARE'] == 'Ndjole']
```

##### On visualise toutes les formations sanitaires et les sites de strat�gies avanc�es (en rouge), leur zone tampons (en blanc) et les quartiers (en noir).


```python
ax = djole_health_area_utm_32632.plot(figsize=(15,15))
fosas_utm_32632_buffered_in_djole.plot(ax=ax, alpha=0.5, color='white')
pois_SA_djole_utm_32632_buffered.plot(ax=ax, alpha=0.5, color='white')
fosas_in_djole_utm_32632.plot(ax=ax, color='red')
pois_SA_djole_utm_32632.plot(ax=ax, color="red")
pois_quartier_djole_utm_32632.plot(ax=ax, color="black")

plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_195_0.png)
    


Il ya deux quartiers qui sont � plus de 5 km


```python

```

### Quel est le lieu de culte le plus �loign� � vol d'oiseau du CSI de Minfoumbe ainsi que sa distance?

##### On r�cup�re tous les points d'int�ret de type lieu de culte


```python
pois_culte_djole_utm_32632 = pois_in_djole_utm_32632[pois_in_djole_utm_32632['Type de poi'] == 'Lieu de culte']
```


```python
pois_culte_djole_utm_32632.head()
```




    
![png](geospatial_microplanification_files%5Coutput_201_0.png)
    




```python
ax = djole_health_area_utm_32632.plot(figsize=(15,15))
pois_culte_djole_utm_32632.plot(ax=ax, color="black")
fosas_in_djole_utm_32632.plot(ax=ax, color='red')

plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_202_0.png)
    



```python
fosas_in_djole_utm_32632.geometry
```




    12    POINT (810353.429 527399.298)
    13    POINT (810658.857 536043.824)
    20    POINT (824860.880 532091.116)
    22    POINT (825411.900 532460.618)
    23    POINT (807167.250 527618.720)
    Name: geometry, dtype: geometry




```python
CSI_Minfoumbe = fosas_in_djole_utm_32632.loc[13, 'geometry']
```


```python
print(CSI_Minfoumbe)
```

    POINT (810658.8572248712 536043.8235710819)
    


```python
distance_to_minfoumbe = pois_culte_djole_utm_32632.distance(CSI_Minfoumbe)
```


```python
distance_to_minfoumbe
```




    25        38.173072
    32     15678.436805
    37     14775.156141
    38     15876.124670
    57      9135.154550
    62      9150.868824
    99      1000.034739
    112     7723.356073
    117     9269.001786
    140     8722.794481
    153    13774.313488
    156    15072.592494
    160    15045.400911
    205     8116.987472
    dtype: float64




```python
#Distance du lieu de culte le plus �loign� du CSI de Minfoumbe
distance_max = distance_to_minfoumbe.max()
```


```python
#Filtrer les lieux de culte � moins de 10 km du CSI
lieu_de_culte_le_plus_eloign� = pois_culte_djole_utm_32632[distance_to_minfoumbe == distance_max]
```


```python
lieu_de_culte_le_plus_eloign�
```




    
![png](geospatial_microplanification_files%5Coutput_210_0.png)
    




```python
lieu_de_culte_le_plus_eloign� = pois_culte_djole_utm_32632.loc[38, 'geometry']
```


```python
print(lieu_de_culte_le_plus_eloign�)
```

    POINT (826159.0738669313 532609.4941463514)
    


```python
ax = djole_health_area_utm_32632.plot(figsize=(15,15))
#fosas_in_djole_utm_32632.plot(ax=ax, color='red')
pois_culte_djole_utm_32632.plot(ax=ax, color="black")
gpd.GeoSeries([CSI_Minfoumbe]).plot(ax=ax, color='red', markersize=200)
gpd.GeoSeries([lieu_de_culte_le_plus_eloign�]).plot(ax=ax, color='white', markersize=200)

plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_213_0.png)
    



```python

```

### Quels sont les lieux de culte situ�s � moins de 10 km � vol d'oiseau du CSI de Minfoumbe


```python
pois_culte_under_10_km_utm_32632 = pois_culte_djole_utm_32632[distance_to_minfoumbe < 10000]
```


```python
pois_culte_under_10_km_utm_32632
```




    
![png](geospatial_microplanification_files%5Coutput_217_0.png)
    




```python
ax = djole_health_area_utm_32632.plot(figsize=(20,20))
pois_culte_djole_utm_32632.plot(ax=ax, color="black")
pois_culte_under_10_km_utm_32632.plot(ax=ax, color='white', markersize=400)
gpd.GeoSeries([CSI_Minfoumbe]).plot(ax=ax, color='red', markersize=200)
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_218_0.png)
    


##### Les lieux de culte sont en blanc et le CSI en rouge


```python

```

## Analyse spatiale: superficie

##### Superficie du district


```python
district_utm_32632.geometry.area
```




    0    4.783623e+09
    1    3.277116e+09
    2    2.608273e+09
    3    2.313289e+09
    4    1.666343e+09
    5    1.142068e+09
    6    4.536774e+09
    7    3.108962e+09
    dtype: float64




```python
district_utm_32632.geometry.area.sum()/1e6
```




    23436.448328891158



##### Pourcentage de la superficie du cameroun


```python
#Pourcentage de la superficie du cameroun
round(((district_utm_32632.geometry.area.sum()/1e6)/475000)*100,2)
```




    4.93




```python
#Visualisation des arrondissements
f, ax = plt.subplots(figsize=(15,15))
cam_3.plot(ax=ax)
district.plot(ax=ax, edgecolor='green', color='yellow')
plt.show()
```


    
![png](geospatial_microplanification_files%5Coutput_227_0.png)
    


Le district repr�sente 4,93 % de la superficie du cameroun

Le lien pour ce notebook [ici](https://github.com/Djatche/Microplannig/blob/master/geospatial_microplanification.ipynb)


```python

```
