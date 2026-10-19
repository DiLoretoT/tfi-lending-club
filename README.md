# Predicción de default en préstamos personales de Lending Club

Trabajo Final Integrador de la Diplomatura en Ciencia de Datos y Análisis Avanzado de la UTN BA (2026). Tomás Di Loreto, Grupo 1.

El trabajo construye un modelo que estima la probabilidad de default de cada solicitud de préstamo personal de Lending Club con la información disponible el día de la solicitud. Un préstamo cae en default cuando el cliente deja de pagarlo, y en el dataset figura como Charged Off. El modelo se compara con la calificación de riesgo de Lending Club, que va de A (menor riesgo) a G (mayor riesgo), aprobando la misma cantidad de solicitudes. El resultado se mide en ganancia neta, que es el interés cobrado a los clientes aprobados que pagaron menos la pérdida de los aprobados que cayeron en default (monto prestado menos todo lo cobrado).

## Resultados principales

El modelo elegido es un gradient boosting, que combina muchos árboles de decisión chicos entrenados uno después de otro. Se entrenó con los préstamos a 36 meses otorgados hasta 2013 y se probó con los 283.019 préstamos otorgados en 2015.

| Solicitudes aprobadas | Ganancia neta con la regla A-G | Ganancia neta con el modelo |
|---|---|---|
| 70 % | 183,6 millones de USD | 201,6 millones de USD |
| 80 % | 205,8 millones de USD | 219,9 millones de USD |
| 90 % | 225,1 millones de USD | 233,6 millones de USD |

Un bootstrap de 1.000 remuestreos da para la diferencia al 80 % un intervalo del 95 % de 12,5 a 15,6 millones de dólares.

El Gini mide de 0 a 1 qué tan bien un criterio ordena a los clientes de menor a mayor riesgo. El modelo tiene un Gini de 0,389 en los préstamos de 2015, contra 0,357 de la calificación A-G, y supera a la regla A-G en cada año probado entre 2012 y 2015. El informe completo está en el PDF del repositorio.

## Contenido del repositorio

| Archivo o carpeta | Contenido |
|---|---|
| `TFI_LendingClub.ipynb` | Notebook con todo el análisis, desde la carga de datos hasta las figuras del informe |
| `figuras/` | Figuras que genera el notebook y que usa el informe |
| `TFI_Grupo01_Tomas Di Loreto.pdf` | Informe final |
| `requirements.txt` | Librerías y versiones usadas |
| `datos/` | Carpeta donde va el archivo de datos, que no se sube al repositorio por su tamaño |

## Cómo ejecutar el notebook

1. Descargar el dataset Lending Club Loan Data desde Kaggle (https://www.kaggle.com/datasets/wordsforthewise/lending-club). Se necesita una cuenta de Kaggle.
2. Descomprimir el archivo `accepted_2007_to_2018Q4.csv` (1,6 GB) dentro de la carpeta `datos/` del repositorio.
3. Crear un entorno con Python 3.12 e instalar las librerías con `pip install -r requirements.txt`.
4. Abrir `TFI_LendingClub.ipynb` y ejecutar todas las celdas en orden. En una computadora con 16 GB de RAM la ejecución completa tarda unos 6 minutos.

El notebook lee el archivo en tandas de 200.000 filas y conserva solo los préstamos terminados, para no superar la memoria disponible.

## Decisiones principales

- La población son los préstamos terminados (Fully Paid y Charged Off) otorgados entre 2007 y 2018. Los préstamos en curso se descartan porque todavía no tienen un resultado final. Los otorgados entre 2016 y 2018 no se usan para entrenar ni para probar, porque al cierre de los datos muchos seguían en curso y los terminados tienen una tasa de default sesgada.
- La validación es temporal. Los préstamos a 36 meses se entrenan con 2007 a 2013, se valida con 2014 y se prueba con 2015, que es el último año en el que todos los préstamos a 36 meses ya habían vencido al cierre de los datos en diciembre de 2018. Los préstamos a 60 meses se prueban con 2013 por el mismo motivo.
- Las columnas de pagos (`total_pymnt`, `total_rec_int` y `recoveries`) se usan solo para calcular la ganancia y la pérdida de cada préstamo, nunca como variables del modelo, porque el día de la solicitud no existían.
- Las variables del buró de crédito, que es la empresa que registra el historial financiero de las personas, están casi vacías hasta 2011 y parcialmente completas en 2012. El gradient boosting las usa con los valores faltantes, y la regresión logística y las redes neuronales los completan con la mediana.
- El estado de residencia se probó y se descartó, porque no mejora los resultados y puede discriminar de forma indirecta.
- La comparación entre modelos y grupos de variables se hace con la cohorte de validación (2014), y la cohorte de prueba (2015) se usa para medir el resultado final.
- El punto de corte y la comparación entre modelos se deciden por ganancia neta en dólares, porque aprobar a un cliente que cae en default cuesta en promedio 4.664 dólares y rechazar a uno que habría pagado cuesta 1.800 dólares de interés, así que el primer error cuesta 2,6 veces lo que cuesta el segundo.

## Fuentes

- Lending Club Loan Data, Kaggle. https://www.kaggle.com/datasets/wordsforthewise/lending-club
- Documentación de scikit-learn, TensorFlow y Keras, y SHAP.
