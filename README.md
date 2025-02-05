# Contenido del Notebook

# EJERCICIO 1 [4 pt]

El dataset 'gimnasio.xlsx' representa los entrenamientos de una persona.

* Cada fila en la hoja 'Series' representa una serie de un entrenamiento: un ejercicio con X kg o tiempo y N repeticiones, realizado un día determinado.
* Se han podido realizar varias series del mismo ejercicio el mismo día.
* Se han podido realizar series de diferentes ejercicios el mismo día.
* En la hoja 'Categorías' se identifica a qué tipología de ejercicio corresponde cada ejercicio.

[0.25 pt] <br>
(1) Lee en una línea *todas las hojas de fichero Excel* 'gimnasio.xlsx'.
<br>
Luego, identifica a través de los datos importados los nombres de las hojas y crea dos dfs: 
* df_series 
* df_categorias


```python
import pandas as pd
df = pd.concat(pd.read_excel("gimnasio.xlsx",sheet_name=None),ignore_index=True)
df_series = pd.read_excel("gimnasio.xlsx", sheet_name= "Series")
df_categorias = pd.read_excel("gimnasio.xlsx", sheet_name= "Categorías")
```

[0.25 pt] <br>
(2) Si es necesario, cambia los tipos de datos a los más apropiados. Si no es necesario, justifica por qué no lo es.

```python
df_categorias.info()
df_series.info()

#No han sido necesario ningun cambio en las variables ya que al leer los dataframes en el caso de categorias las variables son de categoryua object
#y se pueden tratar con strings sin ningun problema para demostrarlo he creado una nueva columna con el conteo de las a que tiene cada fila de la columna categoria
df_categorias["cuenra"] = df_categorias["Categoria"].str.count("a")

#en el caso de series podemos ver claramente como detecta de manera correcta la columna de fecha con un valor datetime64[ns] que es el correcto
#el nombre de ejercicio tambien como object que es correcto ya que se puede tratar como string igual que en el data frame anterior
#peso lo detecta como float64 y es correcto ya que el peso es un numero pero en este caso usa un . para incluir medios kilos
#y por ultimo el numero de repeticiones tambien es correcto como integer ya que son repeticiones completas

```

```python
#borro la culmna anteriormente creada 

df_categorias = df_categorias.drop(columns= ["cuenra"])
```

[0.25 pt] <br>
(3) ¿Cuántos ejercicios distintos tiene registrados por cada categoría?

```python
df_categorias.groupby("Categoria").size()
```

[0.25 pt] <br>
(4) ¿Cuál fue el día en que realizó menos series, y qué ejercicios diferentes realizó ese día?

```python
dias_ejercicios = df_series.groupby("Fecha")["Ejercicio"].value_counts()
dia_que_menos = df_series.groupby("Fecha")["Ejercicio"].count().idxmin()

print(f"El dia que menos ejercicios se han hecho es {dia_que_menos} con: \n{dias_ejercicios.loc[dia_que_menos]} ejercicios")
```

[0.25 pt] <br>
(5) El _volumen de entrenamiento_ son los kg totales levantados en un entrenamiento. 
Calcula el volumen de entrenamiento de cada ejercicio cada día. Para ello, multiplica el peso por el número de repeticiones y suma por las series realizadas de cada ejercicio.
<br>
¿Qué día realizó el máximo volumen de entrenamiento del ejercicio "Press Banca"?

```python
df_series["vulumen_ejercicio"] = df_series["Peso"] * df_series["Repeticiones"]
dia_Series = df_series.groupby(["Fecha"]).sum()
dia_Series
```

[0.5 pt] <br>
(6) Entre los días en los que entrenó, ¿cuántos días no entrenó ningún ejercicio de pierna?

```python
pierna = ["Cuádriceps", "Femoral", "Gemelo", "Prensa", "Sentadilla"]
dias_pierna = df_series[df_series["Ejercicio"].isin(pierna)]
dias_con_ejercicio_de_pierna =len(dias_pierna["Fecha"].unique())
dias_entrenados = df_series["Fecha"].unique()
dias_entrenados = len(dias_entrenados)
dias_entrenados - dias_con_ejercicio_de_pierna
```

[0.25 pt] <br>
(7) Entre los días que fue a entrenar, ¿*en promedio*, qué *día de la semana* realiza menos volumen de entrenamiento total (sumando todos los ejercicios)? Es decir, ¿qué día de la semana es el "más tranquilo" normalmente?

```python
# Ensure the 'Dia_de_la_semana' column is created
df_series["Dia_de_la_semana"] = df_series["Fecha"].dt.day_name()

# Group by 'Dia_de_la_semana' and calculate the mean of 'Peso' and 'Repeticiones'
df_series.groupby("Dia_de_la_semana")[["Peso", "Repeticiones"]].apply(lambda x: (x["Peso"] * x["Repeticiones"]).mean()).rename("Volumen_tot_entrenamiento").sort_values(ascending=True).idxmin()


```

[0.25 pt] <br>
(8) Un día, esta persona se lesionó y pasó mucho tiempo si ir a entrenar. ¿Qué día se lesionó?

```python
# Ordenar el dataframe por la columna 'Fecha'
df_series_sorted = df_series.sort_values(by='Fecha')

# Calcular la diferencia de días entre fechas consecutivas
df_series_sorted['Diferencia_dias'] = df_series_sorted['Fecha'].diff().dt.days

# Encontrar el día con la mayor diferencia
dia_lesion = df_series_sorted.loc[df_series_sorted['Diferencia_dias'].idxmax(), 'Fecha']

print(f"La persona se lesionó el día: {dia_lesion}")
```

[0.25 pt] <br>
(9) Crea un gráfico de barras que muestre, para cada mes (incluyendo el año), el número de entrenamientos que hizo.

```python
numero_entrenamientos=  df_series.groupby(pd.Grouper(key='Fecha',freq='M')).size().reset_index(name="conteo")

import matplotlib.pyplot as plt
fig, ax = plt.subplots()

ax.bar(numero_entrenamientos["Fecha"], numero_entrenamientos["conteo"], width=1, edgecolor="white", linewidth=0.7)
plt.show()
```

[0.5 pt] <br>
(10) Genera una tabla donde cada fila sea cada año-mes de entrenamiento, las columnas sean las diferentes categorías de ejercicios, y el contenido de cada celda sea el volumen total de entrenamiento de esa categoría en ese mes.
<br>
Algo así:
| Mes y Año | Brazo | Core | Espalda | ... | 
|-----------|-----------|-----------|-----------|-----------|
| 2023-09    | ...     | ...     | ...     | ...     |
| 2023-10   | ...     | ...     | ...     | ...     | 
| 2023-11   | ...     | ...     | ...     | ...     |
| ...    | ...     | ...     | ...     | ...     | 


```python
# Merge df_series with df_categorias to include 'Categoria' information
df_series = df_series.merge(df_categorias, on='Ejercicio', how='left', suffixes=('_series', '_categorias'))

# Convertir la columna 'Fecha' al formato año-mes
df_series['Año_Mes'] = df_series['Fecha'].dt.to_period('M')

# Agrupar por año-mes y categoría, y sumar el volumen de entrenamiento
tabla_volumen = df_series.groupby(['Año_Mes', 'Categoria'])['vulumen_ejercicio'].sum().unstack(fill_value=0)

# Mostrar la tabla resultante
print(tabla_volumen)
```

[0.25 pt]<br>
(11) Genera tantos ficheros csv como categorías haya, escribiendo en cada uno de ellos el contenido de df_series filtrado para esa categoría dada. Es decir, habrá un fichero con los datos de Brazo, otro con los datos de Core, otro con los de Espalda etc.

```python
brazo = df_series[df_series["Categoria"] == "Brazo"]
brazo.to_csv("Brazo.csv", index=False)

core = df_series[df_series["Categoria"] == "Core"]
core.to_csv("Core.csv", index=False)

espalda = df_series[df_series["Categoria"] == "Espalda"]
espalda.to_csv("Espalda.csv", index=False)

hombro = df_series[df_series["Categoria"] == "Hombro"]
hombro.to_csv("Hombro.csv", index=False)

pecho = df_series[df_series["Categoria"] == "Pecho"]
pecho.to_csv("Pecho.csv", index=False)

pierna = df_series[df_series["Categoria"] == "Pierna"]
pierna.to_csv("Pierna.csv", index=False)
```

[0.5 pt] <br>
(12) Muestra en pantalla un mensaje con las categorías, tal que así:

"Estas son las categorías que puedes consultar:
<br>
1 - Hombro <br>
2 - Pierna <br>
3 - Espalda <br>
4 - Pecho <br>
5 - Brazo <br>
6 - Core <br>
Elige un número y te mostraré el nombre de un ejercicio de esa categoría.
"


Espera un input del usuario, y devuélvele lo que prometes.
<br>
Por ejemplo, si el usuario pulsa "1", muéstrale un ejercicio de la categoría "Hombro", cualquiera.

```python
# Mostrar mensaje con las categorías
categorias = df_categorias['Categoria'].unique()
mensaje = "Estas son las categorías que puedes consultar:\n"
for i, categoria in enumerate(categorias, 1):
    mensaje += f"{i} - {categoria} \n"
mensaje += "Elige un número y te mostraré el nombre de un ejercicio de esa categoría.\n"
print(mensaje)

# Esperar input del usuario
numero = int(input("Introduce un número: "))

# Mostrar un ejercicio de la categoría seleccionada
categoria_seleccionada = categorias[numero - 1]
ejercicio = df_categorias[df_categorias['Categoria'] == categoria_seleccionada]['Ejercicio'].sample().values[0]
print(f"Un ejercicio de la categoría {categoria_seleccionada} es: {ejercicio}")
```

[0.25] <br>
(13) ¿Qué ejercicios utiliza para entrenar bíceps?

```python
for index, row in df_categorias.iterrows():
    if "Bíceps" in row["Ejercicio"]:
        print(row["Ejercicio"])
        
        
```

# EJERCICIO 2 [1 pt]

He aquí un mensaje en código secreto:<br>
_BOJNPWBTCJFO_
<br>

Para construir este mensaje, se ha escrito un texto original y luego se ha cambiado cada letra por la siguiente en el abecedario. <br>
Por ejemplo, la palabra HOLA pasaría a ser IPMB, y la palabra ZONA pasaría a ser APOB (la Z pasa a ser A, y la N pasa a ser O, ya que no consideraremos la Ñ).<br>
Vamos a asumir textos que están TODO EN MAYÚSCULAS, SIN SIGNOS DE PUNTUACIÓN Y SIN ESPACIOS.
<br>
**Programa dos funciones: una para codificar un mensaje, y otra para decodificarlo.**
<br>
Comprueba que, aplicando primero la codificación y luego la decodificación, recuperas un mensaje original que hayas escrito. 
<br>
Decodifica el mensaje secreto _BOJNPWBTCJFO_.
<br>
Como ayuda, aquí dispones de un string que contiene todas las letras mayúsculas del abecedario.

```python
import string
all_letters = string.ascii_uppercase
all_letters
```

```python
def codificacion(mensaje):
    nuevomensaje = ""
    for letra in mensaje.upper():  # Convertimos el mensaje a mayúsculas
        if letra in all_letters:
            indice = all_letters.index(letra)  # Encontramos la posición de la letra
            nueva_letra = all_letters[(indice + 1) % len(all_letters)]  # Avanzamos una posición (con rotación)
            nuevomensaje += nueva_letra
        else:
            nuevomensaje += letra  # Mantener caracteres que no son letras
    
    return nuevomensaje
```

```python
codificacion("hola")
```

```python
def decodificacion(mensaje):
    nuevomensaje = ""
    for letra in mensaje.upper():  # Convertimos el mensaje a mayúsculas
        if letra in all_letters:
            indice = all_letters.index(letra)  # Encontramos la posición de la letra
            nueva_letra = all_letters[(indice - 1) % len(all_letters)]  # Avanzamos una posición (con rotación)
            nuevomensaje += nueva_letra
        else:
            nuevomensaje += letra  # Mantener caracteres que no son letras
    
    return nuevomensaje
```

```python
decodificacion("BOJNPWBTCJFO")
```

# EJERCICIO 3 (2 pt)

Esto es un juego de dos jugadores: el Jugador 1 crea una lista con tres números del 1 al 10, los que él elija (se da un ejemplo).
<br>
El Jugador 2 desconoce los números, pero sabe que son tres, que son diferentes y que van del 1 al 10. Su objetivo es ir introduciendo números en el teclado, de uno en uno, hasta dar con los tres elegidos por su compañero. 
<br><br>
Escribe un programa que, dados los tres números elegidos por el primer jugador, vaya pidiéndole números al segundo jugador hasta que dé con todos. En cada intento, el programa le dirá al Jugador 2 si es uno de los números ocultos o no. Al finalizar, escribe en un csv todos sus intentos: el número introducido y si estaba entre los elegidos por el Jugador 1 o no.

```python
numeros_jugador_1 = [1,4,5]
print("Hola, Jugador 2. He pensado en tres números diferentes del 1 al 10. Tienes que acertarlos todos, pero de uno en uno.")
```

```python
seguimos = True
aciertos = 0
intentos = []
while seguimos:
    a = int(input("Meta un numero: "))
    intentos.append(a)
    
    if a in numeros_jugador_1:
        print("Ese numero estaba entre los tres y ha sido elimando de la lista")
        aciertos += 1
        numeros_jugador_1.remove(a)
    else:
        print("Ese numero no estaba entre los tres")
    
    if aciertos == 3:
        seguimos = False
        break

intentoscsv = pd.DataFrame()
intentoscsv["Intentos"] = intentos  
intentoscsv.to_csv(f"intentos{intentos}.csv", index=False)
```

# EJERCICIO 4 [3 pt]

Trabajaremos con el fichero 'pokemon.csv'.

[0.5 pt] <br>
(1) Carga el fichero e identifica sus missings:
* Muestra en pantalla una tabla con el número de missings de cada columna, pero __mostrando solamente aquellas columnas que tengan algún missing.__
<br>
* ¿Cuál es la columna con más missings?
<br>
* ¿Cuántas *filas* hay que tengan *más de una variable* no informada?

```python
df_pokemon = pd.read_csv("pokemon.csv")
missing_columns = df_pokemon.columns[df_pokemon.isnull().any()]
print(df_pokemon[missing_columns].isnull().sum())
print(f"la que mas missings tiene: {df_pokemon.isnull().sum().idxmax()}")
```

```python
# Filtrar filas que tienen al menos un valor missing
filas_con_missing = df_pokemon[df_pokemon.isnull().any(axis=1)]

# Contar cuántas filas cumplen la condición
cantidad_filas_con_missing = len(filas_con_missing)

# Imprimir el resultado
print(f"Hay {cantidad_filas_con_missing} filas con al menos un valor missing.")
```

[0.25 pt]<br>
(2) Comprueba mediante código que el 100% de los registros para los que se desconoce la altura se desconoce también el peso.

```python
# Filtrar los registros donde la altura es nula
altura_nula = df_pokemon[df_pokemon['height_m'].isnull()]

# Comprobar si todos los registros con altura nula también tienen el peso nulo
peso_nulo = altura_nula['weight_kg'].isnull().all()

print(f"¿El 100% de los registros para los que se desconoce la altura se desconoce también el peso? {peso_nulo}")
```

[0.75 pt]<br>
(3) <br>
Verás que hay algunas columnas que empiezan por la palabra "against". Lo que sigue a "against" es el nombre de un tipo de ataque, que coincide con los tipos existentes de Pokémon. El valor de esas columnas indica cuánto daño le hacen al Pokémon de la instancia analizada los ataques del tipo en cuestión.
<br>
Por ejemplo, Bulbasaur sufre más contra fuego (against_fire = 2), que contra eléctrico (against_electric = 0.5).
<br>
<br>
Extrae, por medio de código, **una lista con los nombres de los tipos de ataques que aparecen tras la palabra "against" en las columnas mencionadas**.
<br>
La respuesta debe tener esta pinta: `[bug, dark, ..., water]`
<br>
<br>
Comprueba que la longitud de la lista es igual al número de 'type1' diferentes que hay.

```python
variable = "against_"
lista = []
columnas = df_pokemon.columns
for col in columnas:
    if variable in col:
        lista.append(col)

for i in range(len(lista)):
    lista[i] = lista[i].replace("against_", "")
lista
```

[0.25 pt]<br>
(4) ¿Cuál es el tipo primario de Pokémon que, en promedio, más sufre contra agua (against_water)?

```python
df_pokemon.groupby("type1")["against_water"].mean().idxmax()
```

[0.25 pt]<br>
(5) ¿Cuál es el Pokémon con el nombre más largo?

```python

df_pokemon["Largura_nombre"] = df_pokemon["name"].str.lower().str.len()
df_pokemon["Largura_nombre"] = pd.to_numeric(df_pokemon["Largura_nombre"])
idlargo = df_pokemon["Largura_nombre"].idxmax()
print(df_pokemon.loc[idlargo, 'name'])

```

[1 pt] <br>
(6)<br>
Juego "Adivina el tipo". <br>
Dado el dataset de Pokémon, preséntale al jugador todos los tipos primarios disponibles con un "print" y elige aleatoriamente el nombre de un Pokémon. El jugador debe adivinar el tipo primario al que pertenece en un máximo de cuatro intentos, escribiendo por pantalla el tipo en minúsculas.
<br><br>
Para elegir aleatoriamente el nombre de un Pokémon puedes hacer uso del módulo `random`.

```python
import random
print(f"los tipos primarios disponibles son: {df_pokemon['type1'].unique()}")
id_aleatorio = random.randint(0, len(df_pokemon) - 1)
nombre = df_pokemon.iloc[id_aleatorio]["name"]
tipo = df_pokemon.iloc[id_aleatorio]["type1"]

seguimos = True
aciertos = 0
intentos = 0
while seguimos:
    a = input("Meta un numero: ").lower()
    intentos += 1   
    if a ==  tipo:
        print(f"Ese es el tipo primario del pokemon")
        print(f"El nombre del pokemon es: {nombre}")
        aciertos += 1
        numeros_jugador_1.remove(a)
    else:
        print("Ese no es el tipo que buscamos")
    
    if aciertos == 1:
        seguimos = False
        break
    if intentos == 4:
        print("intentalo de nuevo")
        print(f"El tipo del pokemon es: {tipo}")
        print(f"El nombre del pokemon es: {nombre}")
        seguimos = False
        break
```

