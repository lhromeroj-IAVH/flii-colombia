# flii-colombia

**Descripción:** La integridad ecológica es un atributo de los sistemas biológicos ampliamente aplicado en contexto de toma de decisiones y gestión de ecosistemas, y presentada frecuentemente en documentos de política pública en Colombia. La integridad en ecosistemas forestales ha experimentado un avance metodológico en los últimos años, gracias al uso de información proveniente de sensores remotos que han permitido estimar la intensidad de la transformación de coberturas naturales y uno de los indicadores más usados para la estimación de presiones humanas sobre los ecosistemas es la huella espacial humana (HEH). En este repositorio se presenta una explicación de los flujos de trabajo aplicados para la estimación de presiones humanas directas a partir de componentes utilizados para el cálculo de la huella espacial humana para Colombia, así como los procedimientos para inferir las presiones humanas que no se pueden detectar directamente con sensores remotos, y la estimación de pérdida de conectividad forestal a partir de la información de deforestación.  Cada uno de estos tres componentes (presiones observadas, presiones inferidas y pérdida de conectividad) serán preparados y ajustados para calcular el índice de integridad forestal a escala de paisaje siguiendo la metodología de Grantham et al., (2020).

*El flujo de datos y procesos implementados para la obtención del índice se encuentran sintetizado en la siguiente figura:*

![Flowchart drawio](https://user-images.githubusercontent.com/84154963/228911298-90ecf956-e0ac-4ed6-99b6-a1b0ec5547c4.png)

## Rutinas informáticas en la plataforma Google Earth Engine (GEE)

La rutina construida para el cálculo de integridad forestal se compone principalmente de fases de preparación e integración de insumos asociados a actividades humanas y su impacto sobre los ecosistemas. En esta ocasión se utilizaron tres variables originalmente asociadas a huella espacial humana (Correa et al., 2020; Figura anterior): 1) Uso de la tierra, 2) Densidad poblacional y 3) Tiempo de intervención, y dos variables también asociadas a huella espacial humana pero modificadas para esta metodología: 4) Densidad de carreteras y 5) Índice de condición estructural de bosque, construido por Hansen et al. (2019).


### Presiones Directas

**Intensidad de uso del suelo:** a continuación se muestra el código usado para cargar los insumos necesarios para el cálculo de la intensidad de  uso de la tierra. Estos insumos corresponden principalmente al mapa de ecosistemas potenciales de Colombia (Etter, 2020), al mapa binario de transformación de Colombia para el año 2014 (Etter et al., 2011), al mapa de coberturas de la tierra elaborado por la European Space Agency (ESA, 2021) y al mapa de embalses de Colombia construido por el Instituto Geográfico Agustín Codazzi. Adicionalmente, con fines de filtrar la información de las capas globales utilizadas en esta rutina al territorio de Colombia, se utilizó la base de datos simplificada de polígonos limitantes de los países del mundo (LSIB 2017), disponible en Google Earth Engine. Estos nuevos códigos no corresponden a una magnitud que refleja el comportamiento del uso de la tierra, sino que muestran las combinaciones totales de los insumos al momento de evaluar este atributo sobre el territorio. Por esta razón, estos códigos son transformados a valores de intensidad de uso, siguiendo la metodología de Correa et al. (2020) y luego ajustadas a valores de presiones directas según la metodología de Grantham et al (2020).

![Script2](https://user-images.githubusercontent.com/84154963/228964966-16d6717c-fcb4-4f60-bf0b-262b999e4ace.PNG)

Donde x corresponde a cada una de las variables crudas utilizadas para calcular las presiones directas sobre el ecosistema de bosque y 𝛽 el exponente de forma que la mediana de los valores diferentes de cero de la parte exponencial de la función fuera de 0.25, para que al realizar la resta con 1, la mediana total fuera de 0.75. 

**Índice de condición estructural del bosque (SCI):** este es un producto que obedece a la necesidad de incluir a los mapas cotidianos de extensión de bosque producidos por sensores remotos, elementos de calidad de bosque para la evaluación de servicios ecosistémicos y de la biodiversidad (Hansen et al., 2019). Estos insumos para la construcción del SCI corresponden a la distribución de bosque para el año 2000, la altura de bosque, el porcentaje de cobertura de dosel y el año de pérdida de bosque, los cuales se encuentran disponibles en Google Earth Engine, permitiendo que cualquier usuario pueda estimar este índice entre los años 2000 y 2019.

Los componentes de uso de la tierra e índice de condición estructural son los únicos insumos para la construcción de las presiones directas que se desarrollan de manera automatizada. Los demás componentes (tiempo de intervención, densidad poblacional y densidad de carreteras) son insumos previamente desarrollados que deben ser únicamente subidos al ambiente de Google Earth Engine y transformados mediante las funciones exponenciales. 

Presiones directas: Teniendo los insumos reclasificados en un rango entre 0 y 1 y con una resolución espacial de 300 m, se encuentran preparados y estandarizados para sumarse y generar el mapa de presiones directas para Colombia. Este mapa tendría un rango entre 0 y 5 y representaría el impacto directo que generan las actividades humanas asociadas a cambios de uso de la tierra, densificación de las poblaciones, modernización y expansión de las ciudades, y sus efectos sobre los ecosistemas de bosque a través del SCI (Figura 11). Además, el uso de la capa de SCI ayuda a complementar otras actividades de impacto directo que no fueron utilizadas explícitamente en este análisis, pero que fueron detectadas por los insumos de año de pérdida de Hansen et al. (2019). 

![Script6](https://user-images.githubusercontent.com/84154963/228973455-7f762692-4641-467b-8615-f673579b1f18.PNG)


### Presiones Inferidas

Las presiones inferidas, se refieren a actividades humanas que no cuentan con información espacial explícita, pero cuya relación con actividades humanas observadas permite predecir su ubicación e intensidad. Esta metodología distingue dos tipos de presiones inferidas. Por un lado, presiones que se manifiestan a una escala pequeña (5 Km) y presiones en respuesta a actividades esporádicas y que se manifiestan a una escala más amplia (12 Km). En general, las presiones inferidas responden a actividades como cacería, entresaca de madera, explotación de recursos no maderables y maderables a bajas intensidades. 

![Script7](https://user-images.githubusercontent.com/84154963/228976677-d7997eb0-68b2-4659-96b9-ebf237436eb7.PNG)

### Pérdida de conectividad de bosque

La conectividad del bosque permite estimar la integridad del bosque desde una escala de paisaje, considerando explícitamente la conectividad potencial. Grantham et al. (2020) estima esta conectividad potencial considerando la extensión de bosque a nivel mundial propuesta por Laestadius et al (2011) y que considera únicamente variables climáticas y edáficas. En el caso de Colombia, una aproximación similar fue realizada por Etter et al (2020) que estima la cobertura potencial de los ecosistemas colombianos, incluyendo por supuesto la información de bosques pertenecientes a los diferentes biomas de Colombia, y que es el insumo empleado para el cálculo de pérdida de conectividad. Con los insumos seleccionados se construyó la función de conectividad total de bosque para el año 2018 y la conectividad potencial a partir de una función Gaussiana normalizada del insumo ajustado de ecosistemas forestales potenciales de Etter (2020), representando los valores idóneos de conectividad que deberían presentar una zona con una configuración específica de bosque en caso de que no existiera ninguna clase de presión sobre estos. Esto permitirá comparar los valores potenciales de los observados e identificar las zonas en donde se ha perdido mayor cobertura de bosque, a partir del índice de pérdida de conectividad (LFCi) definido en la Ecuación 2, donde 0 indica las zonas con menor pérdida de conectividad y 1 las de mayor (Figura 17).

![Script10](https://user-images.githubusercontent.com/84154963/228978899-8b34a819-9730-4a68-bb17-346b8c5ea1dd.PNG)

![Script11](https://user-images.githubusercontent.com/84154963/228979804-816ff06d-a4cd-4b60-9390-73b0b2264a26.PNG)


### índice de integridad de bosque a escla de paisaje (FLII)

Teniendo los tres componentes ya calculados y estandarizados (presiones directas, presiones indirectas y pérdida de conectividad), se procede a calcular el índice de integridad de bosque a escala de paisaje, de acuerdo con la ecuación presentada en la Figura 1. El resultado final es un índice que varía entre 0 y 10, donde las áreas de bosque sin modificaciones detectables tienen valores de 10, mientras que aquellas a las que se les han detectado profundas transformaciones tienen valores de 0.

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


