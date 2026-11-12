# Entrenador de gimnasio: errores de técnica en el remo con barra

Proyecto de visión por computador que detecta dos errores de técnica en el remo con barra a partir de una foto: la curvatura excesiva de la zona lumbar y un ángulo incorrecto del torso (muy erguido, con menos de 45° de inclinación, o por debajo de la paralela al piso). Autor: Matías Martínez Moreno. Fecha: 12 de noviembre de 2026.

## Qué hace

Para cada foto, un modelo da dos probabilidades independientes, una por error. Una repetición se considera correcta cuando ninguna de las dos supera su umbral. Primero se estima la pose con RTMPose y se recorta el tronco de la persona. Ese recorte, de 224 píxeles, entra a una red convolucional preentrenada en ImageNet con una cabeza nueva de dos salidas.

Se compararon ResNet-18, ResNet-50 y ConvNeXt-S con tres estrategias de transfer learning (backbone congelado, ajuste parcial y ajuste completo) y tres semillas. El modelo final es un ensamble de cinco semillas de ConvNeXt-S con ajuste completo. Los datos son las fotos del remo de Fitness-AQA (Parmar et al., ECCV 2022), con sus particiones oficiales: 18 201 fotos de 1835 repeticiones.

## Resultados

La elección del modelo, de los hiperparámetros y de los umbrales se hizo solo con validación. El conjunto de prueba se evaluó una vez al final. Los intervalos son del 95 %, con remuestreo por repetición.

| Prueba | Lumbar | Torso |
|---|---|---|
| AP | 0,93 (0,89 a 0,96) | 0,82 (0,72 a 0,89) |
| F1 | 0,87 (0,82 a 0,91) | 0,77 (0,69 a 0,82) |
| Precisión y recall | 0,85 y 0,88 | 0,77 y 0,76 |
| AP de un modelo que adivina | 0,18 | 0,09 |

Además se probó el modelo, sin reentrenarlo, con siete videos propios grabados de perfil en dos lugares (47 repeticiones). En los videos con error lumbar detectó las 13 repeticiones y no marcó ninguna de las 5 repeticiones correctas. Para el torso detectó 17 de 29 repeticiones (recall de 0,59, intervalo de 0,39 a 0,76), por debajo del 0,81 que había en la prueba por repetición. Las 12 repeticiones con el torso muy bajo se marcaron como error lumbar. Una explicación posible es que al bajar tanto el torso la espalda baja también se curva, pero no se comprobó.

Con el giro de la cámara el error de torso se degrada rápido: la AP baja a 0,73 con 5 grados, a 0,53 con 10 y a 0,28 con 20, mientras que la del error lumbar no baja de 0,85. Con Grad-CAM el mapa se concentra en la espalda baja para el error lumbar (4,0 veces su área) y en la espalda alta para el torso (2,8 veces), aunque muestra dónde mira el modelo y no la causa de la decisión.

Los resultados del artículo original (F1 de 0,63 y 0,53) se citan solo como referencia, porque las condiciones no son comparables.

## Contenido

- `informe_remo.pdf`: informe de tres páginas con el método, los resultados y las limitaciones.
- `remo_entrenamiento.ipynb`: preparación de datos, pose y recortes, comparación de redes y estrategias, ajuste de hiperparámetros, ensamble, umbrales, evaluación en prueba y Grad-CAM.
- `remo_validaciones.ipynb`: variaciones de luz, nitidez y giro, y prueba con videos propios.

Los notebooks están ejecutados y con sus salidas. Las figuras que mostraban fotos de Fitness-AQA se sustituyeron por dibujos esquemáticos, y los fotogramas de los videos propios tienen la cara difuminada.

## Cómo ejecutarlo

Se ejecutó en Kaggle con dos GPU T4 e Internet activado. El primer notebook tarda cerca de siete horas y el segundo unos 14 minutos. El segundo usa la salida del primero y un conjunto de datos con los videos propios, que no se publica.

Fitness-AQA no se redistribuye aquí. Las fotos y las anotaciones completas se piden a los autores en https://github.com/ParitoshParmar/Fitness-AQA.

## Limitaciones

Los pesos de partida son distintos en cada red, así que la comparación mide arquitectura y pesos a la vez. Las probabilidades no están calibradas y la validación se usó muchas veces. Las particiones separan repeticiones, no personas. Los videos son de una sola persona en dos lugares, con la etiqueta puesta por intención al grabar. Con tres o cinco semillas, las diferencias pequeñas entre modelos no se pueden dar por reales.

## Referencias

- Parmar et al., Domain Knowledge-Informed Self-Supervised Representations for Workout Form Assessment, ECCV 2022 (Fitness-AQA).
- He et al., Deep Residual Learning for Image Recognition, CVPR 2016.
- Liu et al., A ConvNet for the 2020s, CVPR 2022.
