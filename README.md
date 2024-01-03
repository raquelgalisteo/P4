PAV - P4: reconocimiento y verificación del locutor
===================================================
Raquel Galisteo y Xavier Moreno
-------------------------------

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/mod/resource/view.php?id=3654387?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.

  - ___SoX_ (Sound eXchange, the Swiss Army knife of audio manipulation) sirve para realizar múltiples tareas con ficheros de audio. Algunas de sus funciones son:__
    1. __Pasar de un formato de señal o fichero a otro.__
    1. __Realizar algunas operaciones de procesado de señal como transformadas.__
    1. __Reducción de ruido.__

    __En nuestro caso, utilizamos esta herramienta para extraer las características, es decir, los coeficientes.__

  - ___$X2X_ permite la conversión entre distintos formatos de datos. En nuestro caso pasamos de un signed integer de 16 bits a short float y de float a ascii para guardar la fmatrix en un fichero.__

  - ___$FRAME_ extrae frame a frame toda una secuencia. En nuestro caso se cogen tramas de longitud de 240 (`-l 240`) con un periodo de 80 (`-p 80`).__

  - ___$WINDOW_ enventana los datos. Como no especificamos el tipo de ventana, se utiliza una Blackman. En nuestro caso, los datos de entrada tienen una longitud de 240 (`-l 240`) y los datos de salida una longitud de 240 (`-L 240`).__

  - ___$LPC_ calcula los coeficientes LPC utilizando el método de Levinson-Durbin. Se utilizan los parámetros `-l` que indica la longitud del frame (`-l 240`) y `-m` que indica el orden del LPC.__

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 51 del script `wav2lp.sh`).
  ```bash
  ncol=$((lpc_order+1)) # lpc p =>  (gain a1 a2 ... ap) 
  nrow=`$X2X +fa < $base.lp | wc -l | perl -ne 'print $_/'$ncol', "\n";'`
  ```
  __El fmatrix crea una matriz que incluye el número de filas (`nrow`), correspondientes a las tramas de la señal, así como el número de columnas (`ncol`), correspondientes a los coeficientes de cada trama.__ 
  
  __El número de columnas se calcula como el número de coeficientes del orden del predictor lineal más uno, puesto que en el primer elemento se almacena la ganancia de predicción.__
  
  __En primer lugar, para obtener el número de filas se convierte la señal parametrizada a texto, mediante el comando `$X2X`. Seguidamente se cuentan las filas utilizando `wc -l`. Finalmente, utilizando un comando `perl` introduce una línea (`-e`) de manera repetida (`-n`).__

  * ¿Por qué es más conveniente el formato *fmatrix* que el SPTK?

  __Porque permite tener todos los datos del fichero de forma ordenada (tanto las tramas de cada fila como los coeficientes de las señales en cada columna), haciendo más fácil el acceso a ellos.__

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:
  ```bash
  sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 | $LPC -l 240 -m $lpc_order | $LPCC -m $lpc_order -M $lpcc_order > $base.lp
  ```

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
  ```bash
  sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
	$MFCC -l 240 -s 8 -m $mfcc_order -n $mfcc_nfilter > $base.mfcc
  ```

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  
  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    
    - __LP:__
        <img src="imag/Region_Coverage_LP.png" align="center">
        <img src="imag/Comando_Reg_LP.png" align="center">

    - __LPCC:__
        <img src="imag/Region_Coverage_LPCC.png" align="center">
        <img src="imag/Comando_Reg_LPCC.png" align="center">

    - __MFCC:__
        <img src="imag/Region_Coverage_MFCC.png" align="center">
        <img src="imag/Comando_Reg_MFCC.png" align="center">

  + ¿Cuál de ellas le parece que contiene más información?

    __La propiedad de correlación indica el parecido entre dos señales. Cuanto más correladas estén, menos información nueva aportarán. Como podemos apreciar en las gráficas anteriores, las gráficas de MFCC y LPCC son las que tienen los puntos más separados, es decir, las más incorreladas. Por este motivo, serán las que aporten más información, mientras que la que aporta menos información es la gràfica de LP.__


- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.

  |                        | LP | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] | -0.630121 | 0.380297 | 0.360059 |

  - __LP:__
    <img src="imag/Comando_Parametros2y3_LP.png" align="center">
    <img src="imag/Parametros2y3_LP.png" align="center">

  - __LPCC:__
    <img src="imag/Comando_Parametros2y3_LPCC.png" align="center">
    <img src="imag/Parametros2y3_LPCC.png" align="center">

  - __MFCC:__
    <img src="imag/Comando_Parametros2y3_MFCC.png" align="center">
    <img src="imag/Parametros2y3_MFCC.png" align="center">
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.

    __Los coeficientes obtenidos son coherentes con las gráficas calculadas anteriormente. Por un lado, el mayor coeficiente (en valor absoluto) se obtiene para el caso de los LP, siendo este el más cercano a 1. Por otra lado, los coeficientes LPCC y MFCC tienen un valor menor al de LP, lo que significa que estarán menos correlados y, por tanto, aportarán más información. En definitiva, podemos concluir que los MFCC son los más incorrelados, es decir, los que más información nos aportan.__

- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

  __De acuerdo con la teoría, para el cálculo de LPCC debería ser suficiente con 13 coeficientes, mientras que, para MFCC, se suelen coger 13 coeficientes y entre 20 y 40 filtros.__


### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.

- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.

  |               | LP | LPCC | MFCC |
  |---------------|:----:|:----:|:----:|
  | Tasa de error | 7.26% | 0.51% | 0.89% |

  __Tasa Error LP:__
  <img src="imag/TasaError_LP.png" align="center">

  __Tasa Error LPCC:__
  <img src="imag/TasaError_LPCC.png" align="center">

  __Tasa Error MFCC:__
  <img src="imag/TasaError_MFCC.png" align="center">


### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.

  |                 | LP | LPCC | MFCC |
  |-----------------|:----:|:----:|:----:|
  | Umbral óptimo   | 0.373737057570103 | 0.276933713207707 | 0.383636495059995 |
  | Pérdidas        | 73/250 = 0.2920 | 7/250 = 0.0280 | 15/250 = 0.0600 |
  | Falsas Alarmas  | 17/1000 = 0.0170 | 1/1000 = 0.0010 | 9/1000 = 0.0090|
  | Cost Detection  | 44.5 | 3.7 | 14.1 |

  __LP:__
  <img src="imag/Score_LP.png" align="center">

  __LPCC:__
  <img src="imag/Score_LPCC.png" align="center">

  __MFCC:__
  <img src="imag/Score_MFCC.png" align="center">
 
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

  __Para generar los ficheros `class_test.log` y `verif_test.log`, hemos ejecutado en el terminal los dos siguientes comandos respectivamente:__

  `FEAT=lpcc run_spkid finalclass`
  `FEAT=lpcc run_spkid finalverif` 

  __Ambos ficheros se encuentran en la carpeta _work_ del repositorio de la práctica.__

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
  