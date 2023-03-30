# flii-colombia

**Objetivo:** Determinar la integridad de los bosques a escala de paisaje teniendo en cuenta las presiones directas e indirectas sobre los bosques y su pérdida de conectividad.

**Descripción:** La integridad ecológica es un atributo de los sistemas biológicos ampliamente aplicado en contexto de toma de decisiones y gestión de ecosistemas, y presentada frecuentemente en documentos de política pública en Colombia. El concepto de integridad ha tenido diferentes definiciones, pero muchas coinciden en considerar la integridad ecológica como un atributo agregado que implica la capacidad de los sistemas para mantener su funcionalidad, naturalidad, e identidad (Bridgewater et al. 2014). La integridad en ecosistemas forestales ha experimentado un avance metodológico en los últimos años, gracias al uso de información proveniente de sensores remotos que han permitido estimar la intensidad de la transformación de coberturas naturales y uno de los indicadores más usados para la estimación de presiones humanas sobre los ecosistemas es la huella espacial humana (HEH). Esta evalúa de forma cuantitativa y espacialmente explícita los impactos producidos por las presiones humanas, indicando un grado o magnitud de la influencia acumulada de las actividades antrópicas sobre los paisajes y ecosistemas (Correa et al., 2020).

Los insumos utilizados para la estimación de HEH se constituyen en insumos para estimar los impactos directos de las actividades antrópicas sobre los ecosistemas boscosos por lo que pueden contribuir a la estimación de la integridad forestal, y pueden complementarse con la incorporación de presiones indirectas como el deterioro producto de la explotación de recursos boscosos o de la fragmentación por pérdida de coberturas. En este repositorio se presenta una explicación de los flujos de trabajo aplicados para la estimación de presiones humanas directas a partir de componentes seleccionados utilizados para el cálculo de la huella espacial humana para Colombia, así como los procedimientos para inferir las presiones humanas que no se pueden detectar directamente con sensores remotos, y la estimación de pérdida de conectividad forestal a partir de la información de deforestación.  Cada uno de estos tres componentes (presiones observadas, presiones inferidas y pérdida de conectividad) serán preparados y ajustados para calcular el índice de integridad forestal a escala de paisaje siguiendo la metodología de Grantham et al., (2020).

*El flujo de datos y procesos implementados para la obtención del índice se encuentran sintetizado en la siguiente figura:*

![Flowchart drawio](https://user-images.githubusercontent.com/84154963/228911298-90ecf956-e0ac-4ed6-99b6-a1b0ec5547c4.png)

## Rutinas informáticas en la plataforma Google Earth Engine (GEE)

La rutina construida para el cálculo de integridad forestal se compone principalmente de fases de preparación e integración de insumos asociados a actividades humanas y su impacto sobre los ecosistemas. En esta ocasión se utilizaron tres variables originalmente asociadas a huella espacial humana (Correa et al., 2020; Figura anterior): 1) Uso de la tierra, 2) Densidad poblacional y 3) Tiempo de intervención, y dos variables también asociadas a huella espacial humana pero modificadas para esta metodología: 4) Densidad de carreteras y 5) Índice de condición estructural de bosque, construido por Hansen et al. (2019).


### Presiones Directas

**Intensidad de uso del suelo:** a continuación se muestra el código usado para cargar los insumos necesarios para el cálculo de la intensidad de  uso de la tierra. Estos insumos corresponden principalmente al mapa de ecosistemas potenciales de Colombia (Etter, 2020), al mapa binario de transformación de Colombia para el año 2014 (Etter et al., 2011), al mapa de coberturas de la tierra elaborado por la European Space Agency (ESA, 2021) y al mapa de embalses de Colombia construido por el Instituto Geográfico Agustín Codazzi. Adicionalmente, con fines de filtrar la información de las capas globales utilizadas en esta rutina al territorio de Colombia, se utilizó la base de datos simplificada de polígonos limitantes de los países del mundo (LSIB 2017), disponible en Google Earth Engine (Figura 2, líneas 1-2). 

Luego de realizar una primera visualización de las capas (Figura 2, líneas 4 - 10), con ayuda de la función .select() se toma cada uno de estos insumos y se construye un Raster Stack con 3 bandas mediante la función .addBands() (Figura 2, líneas 12 - 24), cada una de estas bandas correspondiendo a un insumo. Se aseguró que cada banda tuviera sus respectivos códigos como valores enteros para facilitar sus operaciones, mediante el atributo .int(). Al tener los insumos cargados y ajustados a bandas se construyó una expresión matemática con ayuda de la función .expression() (Líneas 28-30) que opera cada una de las bandas multiplicadas por un factor asociado, de tal manera que los códigos resultantes reflejan de manera directa los valores de los insumos empleados. Por ejemplo, un píxel con valor de ecosistema potencial de 27, donde se detecta transformación (1), con código de cobertura 190 ubicado en un píxel donde no hay embalses, arrojará un valor final de 1271190.  
 
![Script1](https://user-images.githubusercontent.com/84154963/228963750-c9026826-b6db-413a-a718-0d00e93e0200.PNG)

Estos nuevos códigos no corresponden a una magnitud que refleja el comportamiento del uso de la tierra, sino que muestran las combinaciones totales de los insumos al momento de evaluar este atributo sobre el territorio. Por esta razón, estos códigos son transformados a valores de intensidad de uso, siguiendo la metodología de Correa et al. (2020). Primero, se carga la matriz de equivalencia entre los códigos y los valores de intensidad de uso (Figura 3, línea 34), luego se construye una función en la que se aplica un reductor a cada elemento de la matriz de equivalencia, usando los selectores dados from y to que corresponden a los nombres de las columnas de la matriz (Figura 3, líneas 36 - 48). Esto resulta en una lista en donde cada valor de código del mapa se empareja con su valor de intensidad, facilitando así su reclasificación (Figura 3, línea 48). Finalmente, el mapa resultante es ajustado mediante una función exponencial que clasifica nuevamente los valores de intensidad (Ecuación 1; Figura 3, líneas 52 - 54), acotándolos entre valores continuos entre 0 y 1 (Figura 3, línea 56).

![Script2](https://user-images.githubusercontent.com/84154963/228964966-16d6717c-fcb4-4f60-bf0b-262b999e4ace.PNG)

Donde x corresponde a cada una de las variables crudas utilizadas para calcular las presiones directas sobre el ecosistema de bosque y 𝛽 el exponente de forma que la mediana de los valores diferentes de cero de la parte exponencial de la función fuera de 0.25, para que al realizar la resta con 1, la mediana total fuera de 0.75, siguiendo la metodología de Grantham et al. (2020). 

![Script3](https://user-images.githubusercontent.com/84154963/228965430-6e5e4b25-6bd8-4699-90b8-b290968ce20c.PNG)

**Índice de condición estructural del bosque (SCI):** este es un producto que obedece a la necesidad de incluir a los mapas cotidianos de extensión de bosque producidos por sensores remotos, elementos de calidad de bosque para la evaluación de servicios ecosistémicos y de la biodiversidad (Hansen et al., 2019). Estos insumos para la construcción del SCI corresponden a la distribución de bosque para el año 2000, la altura de bosque, el porcentaje de cobertura de dosel y el año de pérdida de bosque, los cuales se encuentran disponibles en Google Earth Engine, permitiendo que cualquier usuario pueda estimar este índice entre los años 2000 y 2019 (Figura 5).

![Script4](https://user-images.githubusercontent.com/84154963/228966966-2985c7a0-d764-41b3-aa1d-d4abf56fdf5d.PNG)

Hansen et al. (2019) estableció el SCI en un rango entre 1 y 18, donde 1 es asignado a bosques con altura menor a 5 metros, disturbados desde 2012 o con cobertura de dosel menor a 25% (muy baja condición estructural), y 18 es asignado a los bosques que no han mostrado disturbios desde el 2000, con alturas mayores a 20 metros y porcentaje de cobertura de dosel mayor a 95% (Muy alta condición estructural). No obstante, para este trabajo se invirtieron los valores de SCI para que, entre mayor sea el valor de SCI, menor es la condición estructural de bosque y por ende es mayor el impacto que genera sobre este ecosistema (Figura 6). Esta decisión fue tomada ya que todas las demás variables mostraron este comportamiento, demostrando mayor impacto sobre el ecosistema entre mayor sea su valor. Otro cambio adicional que se realizó a este índice, es que los píxeles definidos en el mapa de ecosistemas potenciales de Etter que no corresponden a bosques naturales, se les asignó el valor de 0 para que al aplicar la ecuación 1 sobre estos píxeles arroje un valor nulo de impacto (Figura 7, línea 126). El script para el cálculo del SCI se pueden consultar en [nature.com/SCI](https://www.nature.com/articles/s41597-019-0214-3#code-availability/). Finalmente se realizó el ajuste por función exponencial como proceso de escalamiento y comparación con los resultados de intensidad de uso de la tierra (Figura 7, línea 129). 

![Script5](https://user-images.githubusercontent.com/84154963/228970640-22e51c8c-d058-4b8f-8c21-9052564f11c9.PNG)

Los componentes de uso de la tierra e índice de condición estructural son los únicos insumos para la construcción de las presiones directas que se desarrollan de manera automatizada. Los demás componentes (tiempo de intervención, densidad poblacional y densidad de carreteras) son insumos previamente desarrollados que deben ser únicamente subidos al ambiente de Google Earth Engine y transformados mediante las funciones exponenciales. 

Densidad poblacional: La capa de densidad poblacional utilizada en este trabajo fue extraída de la base de datos JRC Earth Observation Data and Processing Platform (JEODPP) de la Unión Europea para el año 2015. Los valores de la capa de densidad están medidos en Número de personas por píxel y ajustada a una resolución espacial de 300 metros. 

Densidad de carreteras: La capa de densidad de carreteras es el mismo insumo utilizado en el componente de infraestructura utilizado por Grantham et al. (2020). Esta capa consta de 29 variables asociadas a carreteras que incluyen vías primarias, secundarias y terciarias, pasos peatonales, rutas de transporte público , etc. Las unidades de esta capa corresponden a la presencia de cada una de estas variables en cada píxel, multiplicada por un factor de peso que refleja la intensidad del impacto generado de cada tipo de carretera sobre el bosque.  

Tiempo de Intervención: El tiempo de intervención fue elaborado por Etter et al. (2008) y actualizada hasta el año 2014 a partir de las capas de transformación generadas por Etter et al. (2011). Esta capa identifica las áreas que han estado sometidas a un uso histórico para los años 1500, 1600, 1900, 1970, 1990, 2000 y 2014. Las unidades de este mapa son el número de años de intervención, y como todas las capas utilizadas para el cálculo de presiones directas, la resolución espacial es de 300 metros.

Presiones directas: Teniendo los cinco insumos reclasificados en un rango entre 0 y 1 y con una resolución espacial de 300 m, a pesar de evidenciar características tan diferentes del territorio colombiano, se encuentran preparados y estandarizados para sumarse y generar el mapa de presiones directas para Colombia por medio de la función .add(). Este mapa tendría un rango entre 0 y 5 y representaría el impacto directo que generan las actividades humanas asociadas a cambios de uso de la tierra, densificación de las poblaciones, modernización y expansión de las ciudades, y sus efectos sobre los ecosistemas de bosque a través del SCI (Figura 11). Además, el uso de la capa de SCI ayuda a complementar otras actividades de impacto directo que no fueron utilizadas explícitamente en este análisis, pero que fueron detectadas por los insumos de año de pérdida de Hansen et al. (2019). 

![Script6](https://user-images.githubusercontent.com/84154963/228973455-7f762692-4641-467b-8615-f673579b1f18.PNG)


### Presiones Inferidas

Las presiones inferidas, se refieren a actividades humanas que no cuentan con información espacial explícita, pero cuya relación con actividades humanas observadas permite predecir su ubicación e intensidad. Esta metodología distingue dos tipos de presiones inferidas. Por un lado, presiones que se manifiestan a una escala pequeña y presiones en respuesta a actividades esporádicas y que se manifiestan a una escala más amplia. En general, las presiones inferidas responden a actividades como cacería, entresaca de madera, explotación de recursos no maderables y maderables a bajas intensidades. Estas actividades de baja intensidad no son fácilmente detectables desde sensores remotos dado que no implican pérdidas de cobertura. Esta relación con actividades humanas observadas se calcula a partir de dos matrices de pesos construidas a partir de la modificación que sufre un píxel de una presión, por un lado de corto alcance y por el otro de largo alcance, generada por los píxeles vecinos. 

Las matrices de pesos se construyeron de tal forma que los efectos a corto alcance tuvieran una influencia hasta los 5 km, mientras que las de largo alcance presentan una influencia hasta los 12 km. Los valores de **𝛂**, **𝛌** y **𝛄** mostrados en la figura 1 se mantuvieron igual que los proporcionados por Grantham et al. (2020), y fueron empleados en un Kernel de 39 píxeles de largo (19 píxeles de radio con respecto al píxel central). 

De manera similar se realizó el cálculo de presiones inferidas de largo alcance con una matriz de pesos diferentes, variando la distancia de alcance hasta 12 Km (Figura 14). La única diferencia con respecto a la metodología empleada por Grantham et al. (2020), es que al momento de crear el mapa de presiones indirectas de largo alcance, los valores superiores al 5% del valor máximo de las presiones  fueron reclasificados y acotados al valor del 5%. En nuestro caso, este valor es de 0.4.

![Script7](https://user-images.githubusercontent.com/84154963/228976677-d7997eb0-68b2-4659-96b9-ebf237436eb7.PNG)


### Pérdida de conectividad de bosque

La conectividad del bosque permite estimar la integridad del bosque desde una escala de paisaje, considerando explícitamente la conectividad potencial. Grantham et al. (2020) estima esta conectividad potencial considerando la extensión de bosque a nivel mundial propuesta por Laestadius et al (2011) y que considera únicamente variables climáticas y edáficas. En el caso de Colombia, una aproximación similar fue realizada por Etter et al (2020) que estima la cobertura potencial de los ecosistemas colombianos, incluyendo por supuesto la información de bosques pertenecientes a los diferentes biomas de Colombia, y que es el insumo empleado para el cálculo de pérdida de conectividad.

Para este desarrollo se utilizaron los insumos globales de Hansen et al (2013), para determinar la conectividad forestal potencial y la configuración de pérdida de bosque como último componente del índice de integridad forestal (Figura 15). Se tomó esta decisión ya que, dentro de los insumos de presiones directas se encuentra el cálculo de SCI para Colombia, el cual se construyó a partir de insumos generados por Hansen. Para evitar discordancias entre la capa de SCI y la de pérdida de conectividad forestal, se optó por manejar nuevamente la información generada por Hansen. 

![Script8](https://user-images.githubusercontent.com/84154963/228978182-2d46761f-e333-4f91-8d27-be931f4223df.PNG)

Con los insumos seleccionados se construyó la función de conectividad total de bosque para el año 2018 y la conectividad potencial a partir de una función Gaussiana normalizada y del insumo ajustado de ecosistemas forestales potenciales de Etter (2020), representando los valores idóneos de conectividad que deberían presentar una zona con una configuración específica de bosque en caso de que no existiera ninguna clase de presión sobre estos (Figura 16). 

![Script9](https://user-images.githubusercontent.com/84154963/228978719-9cbe2388-0df6-4f90-bad0-dd0de20fa166.PNG)

Esto permitirá comparar los valores potenciales de los observados e identificar las zonas en donde se ha perdido mayor cobertura de bosque, a partir del índice de pérdida de conectividad (LFCi) definido en la Ecuación 2, donde 0 indica las zonas con menor pérdida de conectividad y 1 las de mayor (Figura 17).

![Script10](https://user-images.githubusercontent.com/84154963/228978899-8b34a819-9730-4a68-bb17-346b8c5ea1dd.PNG)

![Script11](https://user-images.githubusercontent.com/84154963/228979804-816ff06d-a4cd-4b60-9390-73b0b2264a26.PNG)


### índice de integridad de bosque a escla de paisaje (FLII)

Teniendo los tres componentes ya calculados y estandarizados (presiones directas, presiones indirectas y pérdida de conectividad), se procede a calcular el índice de integridad de bosque a escala de paisaje, de acuerdo con la ecuación presentada en la Figura 1 (Figura 18, líneas 375 - 381). El resultado final es un índice que varía entre 0 y 10, donde las áreas de bosque sin modificaciones detectables tienen valores de 10, mientras que aquellas a las que se les han detectado profundas transformaciones tienen valores de 0.

![Script12](https://user-images.githubusercontent.com/84154963/228981725-7a2caaf7-2de7-4cec-bdfd-95c6a62b0e46.PNG)

Este índice brinda información importante sobre la calidad de los bosques, atributo que resulta relevante para plantear estrategias de restauración basadas en el conocimiento del impacto generado por disturbios naturales y antrópicos. Estos disturbios afectan la capacidad de los bosques de proveer servicios ecosistémicos como soporte de la biodiversidad, almacenamiento de carbono, regulación de recursos hídricos y otros beneficios que provee a la sociedad humana (Hansen et al., 2019). Al considerarse un insumo tan relevante para la toma de decisiones, se espera que este índice se convierta en un referente para los países de las franjas tropicales y subtropicales para el estudio y el entendimiento de las dinámicas del bosque, en la cuantificación espacialmente explícita de su integridad y para el establecimiento de nuevas estrategias encaminadas a cumplir los acuerdos internacionales orientados hacia los objetivos del desarrollo sostenible.

![Script13](https://user-images.githubusercontent.com/84154963/228982110-8c477d9f-881b-4dc7-b7a3-b58300104227.PNG)


## Bibliografía

Bridgewater, P., Kim, R. E., & Bosselmann, K. (2014). Ecological integrity: A relevant concept for international environmental law in the Anthropocene? Yearbook of International Environmental Law, 25(1), 61–78.

Correa A., C. A., Etter, A., Díaz T., J., Rodríguez B., S., Ramírez, W., Corzo, G. 2020. Spatiotemporal evaluation of the human footprint in Colombia: Four decades of anthropic impact in highly biodiverse ecosystems. Ecological Indicators, 117, 1 - 10 pp. https://doi.org/10.1016/j.ecolind.2020.106630

Etter, A., Andrade, A., Saavedra, K., Amaya, P., Cortés, J. y Arévalo, P. (2020). Ecosistemas colombianos: amenazas y riesgos. Una aplicación de la Lista Roja de Ecosistemas a los ecosistemas terrestres continentales. Bogotá: Pontificia Universidad Javeriana y Conservación Internacional-Colombia.

Etter, A., McAlpine, C. A., Possingham, H. 2008. Historical Patterns and Drivers of Landscape Change in Colombia since 1500: A Regionalized Spatial Approach. Annals of the Association of American Geographers. Ann. Assoc. Am. Geogr. 98, 2 - 23.  https://doi.org/10.1080/00045600701733911

Grantham, H.S., Duncan, A., Evans, T.D. et al. Anthropogenic modification of forests means only 40% of remaining forests have high ecosystem integrity. Nat Commun 11, 5978 (2020). https://doi.org/10.1038/s41467-020-19493-3

Hansen, A.J., Burns, P., Ervin, J. et al. A policy-driven framework for conserving the best of Earth’s remaining moist tropical forests. Nat Ecol Evol 4, 1377–1384 (2020). https://doi.org/10.1038/s41559-020-1274-7      

Hansen, A.J., Barnett, K., Jantz, P., Phillips, L., Goetz, S.J., Hansen, M., Venter, O., Watson, J.E.M, Burns, P., Atkinson, S., Rodríguez-Buriticá, S., Ervin, J., Virnig, A., Supples, C., De Camargo, R. (2019). Global humid tropics forest structural condition and forest structural integrity maps. Nature scientific data. 6:232, 12 pp.

Hansen, M. C., P. V. Potapov, R. Moore, M. Hancher, S. A. Turubanova, A. Tyukavina, D. Thau, S. V. Stehman, S. J. Goetz, T. R. Loveland, A. Kommareddy, A. Egorov, L. Chini, C. O. Justice, and J. R. G. Townshend. 2013. “High-Resolution Global Maps of 21st-Century Forest Cover Change.” Science 342 (15 November): 850–53. Data available on-line from: http://earthenginepartners.appspot.com/science-2013-global-forest.

Laestadius, L., Maginnis, S., Minnemeyer, S., Potapov, P., Saint-Laurent, C., & Sizer, N. (2011). Mapping opportunities for forest landscape restoration. 62, 47–48.


