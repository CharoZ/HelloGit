# Archivos para empaquetar

## 1. Setup.py

En general dentro de este archivo se pone casi toda la info importante. Se suele usar sólo la función setup() de setuptools. Esta función tiene muchos parámetros, abajo voy a listar los que vaya probando.

#### 1.1. name

Nombre del paquete a instalar (string).

```
name="mypackage" 
```


#### 1.2 version 

Número de la versión de tu paquete. Toma un string también. 

#### 1.3 packages

Acá le decís qué cosas van en tu paquete. Podés pasarle una lista de ubicaciones manualmente, o podés usar funciones de setuptools para que encuentren los paquetes: 

`packages=find_packages()`: esta función encuentra automáticamente todo lo que está adentro de src o lib. Va a tomar todo lo que tenga un \_\_init\_\_.py, y los test pueden ser un problema. Una solución es ponerlos afuera de src, otra es pasarle `exclude=[]` con una lista de strings (formato glob) para que no los tome. Te devuelve una lista con los nombres de los paquetes y subpaquetes. En general se usa para proyectos muy grandes donde son muchos nombres. Si no, se listan a mano. 

`packages=find_namespace_packages()`: Es una variante de find_packages(). Si no se especifica nada también toma todo lo que haya en la carpeta. Lo bueno de esta función es que podés setearle un `include=[]`, que funciona exactamente al revés que `exclude`, es decir, se le pasa el patrón que sí queremos que agarre. 

En ambas funciones podemos decirle en qué carpeta queremos que busque cosas. Entonces, si los tests están separados, no los agarraría. Queda como: 
```
packages=find_packages(where="src")
``` 

Hasta acá tendríamos un setup.py mínimo. 

#### 1.4. metadata

setup() tiene varios parámetros para la metadata que aparecería en PyPI: 

```
author="tunombre",
author_email="tu@mail.com",
description= "Breve descripción de lo que es tu paquete.",
keywords="nlp rompiendo cosas",
url="http://example.com/HelloWorld/",
project_urls={
    "Bug Tracker": "https://bugs.example.com/HelloWorld/",
    "Documentation": "https://docs.example.com/HelloWorld/"
},
long_description= "Descripción larga del paquete",
license="MIT",
classifiers=[
        "License :: OSI Approved :: Python Software Foundation License",
        "Programming Language :: Python :: 3"
    ]

```
long_description podría coincidir con el archivo README. Si queremos hacer esto hay que incluir el readme en el archivo MANIFEST.in y leerlo en setup.py. 

Se pueden agregar más parámetros, pero estos me parece que son los que pueden llegar a ser más útiles. 

TO-DO: ver bien cómo es lo de classifier y la versión de python. 

#### 1.5. install_requires

Acá le indicamos qué paquetes vamos a necesitar instalar para que corra nuestro paquete, se instalan (hasta donde entiendo) con pip. Se le pasa una lista se strings con los paquetes. Podemos especificar qué versión o versiones se necesitan con operadores lógicos como los de python (<=, >=, ==, !=, <, >). Si no se le pasa nada, pip va a buscar la última versión disponible. 

```
install_requires=[    
        "numpy==0.2",
        "fasttext<=1.3",
        "sklearn"
    ],
```
Cuando una dependencia no es un paquete disponible en PyPI no se pasa en install_requires.

#### 1.6. dependency_links

Cuando tenemos una dependencia que es, por ejemplo, un repo, podemos indicárselo con esta función. Se pasa como una lista de strings con los URL a donde va a ir a descargar. El URL tiene que ser un link de descarga directa, un link a una web que tiene el enlace de descarga directa o directamente el URL de un repo. 

Todo se va a descargar automáticamente cuando se instale el paquete. 

```
 dependency_links=[
        "git+https://AR-CDO@dev.azure.com/AR-CDO/Maite/_git/maite-nlp",
        "https://www.webdementiritas.com"
    ],
```

Nota: setuptools soporta agregarle el prefijo git+\<URL\> pero esto pip 19 en adelante no lo toma bien. Se puede pasar sólo el URL, pero no sé si hay alguna desventaja en hacerlo así. 

#### 1.7.extras_require

Podemos tener ciertas funcionalidades optativas que tengan sus propias dependencias. En estos casos __no__ queremos que se instalen automáticamente. En estos casos, lo que hacemos es especificarlo con extras_require, que toma un diccionario con el nombre del feature como key y la dependencia como value. Esto sólo se va a instalar si algún proyecto requiere esa funcionalidad. 

```
 extras_require={
        "PDF":  ["ReportLab>=1.2", "RXP"],
        "reST": ["docutils>=0.3"],
    }
```
Entonces, supongamos que lo anterior forma parte del setup.py de un proyecto que se llama ProyectoA en el que PDF es optativo. Ahora supongamos que un segundo proyecto, ProyectoB, tiene a ProyectoA __con__ PDF. En ese caso, en el setup.py de ProyectoB vamos a tener: 

```
setup(
    name="Project-B",
    install_requires=["ProyectoA[PDF]"],
    ...
)
```

Al indicarle esto, la instalación de ProyectoB va a instalar automáticamente ProyectoA __con__ lo que requiera PDF. De este modo, lo que necesite PDF sólo se va a instalar automáticamente cuando instalemos ProyectoB. 

Ahora, si en ProyectoA llega a cambiar esa dependencia (por ejemplo, que sea absorbida por funcionalidades nuevas el paquete) su setup.py ya no va a tenerla. ¿Qué pasa entonces con ProyectoB? Para evitar tener que cambiar todos los setup.py por un cambio en un paquete solo, lo que se puede hacer es dejar el extra que necesita ProyectoB vacío:

```
 extras_require={
        "PDF":  [],
        "reST": ["docutils>=0.3"],
    }
```

De este modo, al instalar ProyectoB lo que va a suceder es que va a instalar todo ProyectoA, y nada va a romper. La ventaja de esto es que evitás tener que cambiar el setup.py dos paquetes por una actualización de sólo uno de ellos. 

#### 1.8. entry_points

En entry_points lo que hacemos es elegir módulos, funciones u objetos para que quien se instale nuestro paquete pueda correrlas por línea de comando, y les damos un nombre que puede (o no) ser más amigable que el que está en el código. Tenemos que pasarle un diccionario que tenga el nombre deseado y la ubicación del módulo, objeto o función.

Podría ser el caso que querramos que un extra se instale solamente para cierto script dentro del mismo proyecto. Para eso podemos usar entry_points. Lo que hacemos acá es decirle que cumpla una dependencia sólo si se corre cierto script. En este caso le podemos indicar las dependencias específicas que tenga. 

```
 entry_points = {
        'my_ep_group_id': [
            'nombre_lindo = myns.mypkg.mymodule:the_function [PDF]'
        ]
    },
```
El diccionario contiene keys que son grupos de entry points y el value es un string o lista de strings. Cada string tiene la estructura key-value también pero separado por `=`, las keys de estos strings son los nombres que le queremos dar (lo que va a ponerse en la consola) y el value es la ubicación del módulo, que puede o no tener un `:` y la función u objeto que queramos de adentro del módulo. 

TO-DO: investigar más cómo es el tema de los grupos de entry points y para poder correr cosas por línea de comando e incluir scripts en otros lenguajes (https://python-packaging.readthedocs.io/en/latest/command-line-scripts.html - https://chriswarrick.com/blog/2014/09/15/python-apps-the-right-way-entry_points-and-scripts/).



## 2. MANIFEST.in

## 3. Setup.cfg

## 4. README.md

## 5. LICENSE