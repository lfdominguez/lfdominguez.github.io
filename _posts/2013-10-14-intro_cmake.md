---
layout: post
title: Mis días con Autotools están contados ...
tags: [cmake, autotools, c++]
summary: Breve introducción a CMake
---

## Introducción

Bueno, en realidad ya conté los días de **Autotools** desde hace mucho tiempo, pues esa cantidad de ficheros para construir una aplicación nunca han sido mi fuerte. Cuando utilizaba **mi viejo GNU-Amigo Autotools** siempre esperaba encontrarme con alguna herramienta que me facilitara el construir mis proyectos, cuando aquello (10mo grado) ni idea de que era **SCons** o **CMake**. Luego de entrar a nuestra Universidad conocí a **CMake** gracias al código fuente de KDE (el cual se pasó de utilizar **Autotools** a **CMake** a partir de su **major** versión 4 *por algo será ;)*).
En este post no me dedicaré para nada en criticar a tan **legendaria** herramienta que es **Autotools**, pues aparte de sus pros y contras, es la herramienta por defecto de los proyectos de GNU en la actualidad. Me reservo esa discusión, pues sería el inicio de una laaaarga discusión y ese no es mi objetivo.

## CMake

CMake no es más que una herramienta multiplataforma para generar los ficheros necesarios para construir proyectos. Su nombre proviene de **Cross Platform Make** (Make Multiplataforma), . Mantiene diferentes tipos de proyectos, actualmente en su versión **2.8.11.2** soportaba los siguientes generadores:

- **Unix Makefiles** -- Genera los estándares Makefiles tan ampliamente utilizados.
- **Ninja** -- Genera los build.ninja (Experimental por ahora, aunque lo he probado y no me ha dado ningun error y es **MUY** rápido).
- **CodeBlocks** -- Genera un proyecto para CodeBlocks.
- **Eclipse CDT4** -- Genera un proyecto para Eclipse.
- **KDevelop 3** -- Genera un proyecto para KDevelop en su versión 3 (No la recomiendo en un post posterior hablaré sobre su versión 4).
- **Microsoft VisualStudio** -- Genera un proyecto para ser utilizado en VisualStudio (disponible en plataformas Windows).
- **XCode** -- Genera proyecto para ser utilizado en XCode (disponible en platagormas de Apple).
- **Sublime Text 2** -- Genera un proyecto para Sublime Text en su versión 2 (si quisieran más datos sobre éste vean a **raven**, es su IDE por preferencia).

### Principales funcionalidades

- Ficheros de configuración escritos en un lenguaje de script específico para CMake.
- Análisis automático de dependencias para C, C++, Fortran y Java.
- Soporte para SWIG, Qt, FLTK, etc.. a través del lenguaje de script de CMake.
- Detección de cambios en ficheros usando **timestamps** tradicionales.
- Soporte para construcciones paralelas.
- Compilación cruzada.
- Soporte de Macros (buscar/configurar software).
- Sintaxis intuitiva.
- Admite jerarquía de directorios complejas y detecta bibliotecas.
- Vista global de todas las dependencias, usando CMake para generar un diagrama **graphviz**.
- Integrado con DART (Software), CDash, CTest y CPack, una colección de herramientas para prueba y liberación de software (nunca pensé que hacer un .deb con CMake fuera tan fácil).

### Aplicaciones que utlizan CMake

Y aquí una lista de algunas aplicaciones que utilizan CMake para construirse:

- **KDE** - A partir de la versión 4 de este conjunto de herramientas para el escritorio, no hay mucho que decir que no sepan; si quieres saber más sobre él, pues a instalarlo.
- **Avidemux** - Para tratamiento de Videos.
- **Compiz** - Gestor de composición de escritorio, quién no ha escuchado de él (Ventanas como gelatina, fuego en la pantalla, etc..).
- **Kicad** - Herramienta CAD.
- **OpenCV** - Biblioteca para tratamiento de imágenes utilizada nada más y nada menos que por la NASA y donde han contribuido disímiles instituciones como Intel, IBM, etc.. además de ser de código libre.
- **Poopler** - Biblioteca para el trabajo con PDF, en lo personal la he utilizado con su wrapper para Qt y es muy rápida, a la vez de fácil de utilizar (gracias Qt :D).
- **PvPGN** - Uhmmm, muchos sabrán que se trata de uno de los mejores servidores de Battle.net.

Estas son algunas herramientas de las que recuerde que haya visto que utilizan CMake, entre otras por supuesto, para más información saben a quien consultar :D.

### Comparación

Utilizo CMake frente a Makefiles **a mano** y Autotools por estas razones:

- Más cómodo y fácil.
- Menor curva de aprendizaje.
- No usa **M4 :D**.
- Muy portable.
- Más fácil para extender.
- Mucha menor cantidad de ficheros necesarios para construir mi aplicación.
- Documentación más cómoda.

### Estructura

- **CMakeLists.txt** -- Formato del fichero, es el que define el flujo de control de sintaxis de CMake.
- **Módulos CMake** -- Son los que extienden la funcionalidad de CMake, principalmente en la búsqueda de aplicaciones/herramientas/bibliotecas (Ejemplos: FindQt4.cmake, FindJava.cmake).

## Basta de teoría y manos a la obra

### Primero las instrucciones básicas para construir las aplicaciones

Los procedimientos que utilizaré son los aconsejados por los propios desarrolladores de CMake y la comunidad en general, pues como CMake genera una serie de ficheros internos para su posterior uso, además de los propios ficheros propios del tipo de generador (por defecto Makefiles) es bueno crear una carpeta donde se almacenarán todos esos ficheros creados en el proceso de construcción tanto de CMake como de los Makefiles, por lo tanto los pasos son:

- Nos ubicamos en la carpeta donde se encuentra nuestro proyecto y que contiene el **CMakeLists.txt**.
  {% highlight bash linenos %}
  cd /tmp/ejemplo_src
  {% endhighlight %}

- Creamos y nos adentramos en el directorio donde se almacenarán todos los ficheros de la construcción, por defecto siempre se nombra **build**.
  {% highlight bash linenos %}
  mkdir build
  cd build
  {% endhighlight %}

- Ejecutamos CMake y le decimos que tome el **CMakeLists.txt** de la carpeta anterior.
{% highlight bash linenos %}
cmake ..
{% endhighlight %}

- Ahora como por defecto se han generado los ficheros necesarios para utilizar Makefiles, compilamos nuestra aplicación de ejemplo.
{% highlight bash linenos %}
make
{% endhighlight %}

- Por último ejecutamos nuestro programa, que se generará con el nombre que le habíamos asignado en el **ADD_EXECUTABLE** del **CMakeLists.txt**.
{% highlight bash linenos %}
./ejemplo
{% endhighlight %}

> **NOTA**: La importancia de utilizar el método de crear la carpeta **build** consiste en que todo los ficheros generados residirán en esa carpeta, de esa manera se evita la "contaminación" de la carpeta de fuentes, además que cuando tenemos nuestro proyecto bajo un Sistema de Control de Versiones (dígase **GIT**, **Mercurial**, **Bazaar**, **SVN**, etc) sería más fácil evitar que todos esos ficheros que no queremos tener "versionados" estén en un solo lugar, así se le podría poner que ignorase solamente la carpeta **build** y de esta manera no tendríamos que estar poniendo todas las extensiones que se generan en un proceso de construcción.

### Ejemplos

#### Programa sencillo

Imaginemos que tenemos una estructura como esta en nuestro proyecto:

{% highlight text %}
ejemplo_src
  ├── CMakeLists.txt
  └── main.cpp
{% endhighlight %}

Fichero **main.cpp**:

{% highlight c++ linenos %}
#include <iostream>

using namespace std;

int main (int argc, char *argv[])
{
  cout << "Hola Mundo!!!" << endl;

  return 0;
}
{% endhighlight %}

Fichero **CMakeLists.txt**:

{% highlight cmake linenos %}
PROJECT(ejemplo CPP)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET(ejemplo_SRC main.cpp)

ADD_EXECUTABLE(ejemplo ${ejemplo_SRC})
{% endhighlight %}

Explicando alguna de las funciones utilizadas en el **CMakeLists.txt**:

- `PROJECT`: Define el nombre del proyecto, así como los lenguajes implicados en él, en el caso de C/C++ es opcional.
- `CMAKE_MINIMUM_REQUIRED`: Le dice a CMake cual es la versión mínima que se requiere para poder generar el proyecto.
- `SET`: Declara una variable (en este caso `ejemplo_SRC`, cuando querramos obtener el valor de una variable, lo hacemos con `${VARIABLE}`) y todos los parámetros a continuación serían los valores de la variable.
- `ADD_EXECUTABLE`: Agrega un **target**(son la unidad de construcción básica de los **CMakeLists.txt**), en este caso es del tipo ejecutable, de esta manera CMake sabe que tiene que crear un fichero ejecutable compilando los ficheros pasados a continuación en la función.

Éstas funciones contienen muchos parámetros que aquí no recojo pues no es mi intención convertirme en un traductor de la ayuda de CMake (los invito a que la revisen que está muy clara).

#### Biblioteca sencilla

Ahora veremos como crear una biblioteca de una manera muy sencilla en CMake, teniendo como estructura de directorios:

{% highlight text %}
ejemplo_lib_src
  ├── CMakeLists.txt
  └── ejemplo.cpp
  └── ejemplo.hxx
{% endhighlight %}

Fichero **ejemplo.cpp**:

{% highlight c++ linenos %}
#include "ejemplo.hxx"

Ejemplo::Ejemplo () : m_numero (10)
{
}

int Ejemplo::numeroA ()
{
  return m_numeroA;
}

void Ejemplo::setNumeroA (const int valor)
{
  m_numeroA = valor;
}

int Ejemplo::numeroB ()
{
  return m_numeroB;
}

void Ejemplo::setNumeroB (const int valor)
{
  m_numeroB = valor;
}

int Ejemplo::suma ()
{
  return m_numeroA + m_numeroB;
}

int Ejemplo::resta ()
{
  return m_numeroA - m_numeroB;
}
{% endhighlight %}

Fichero **ejemplo.hxx**:

{% highlight c++ linenos %}
class Ejemplo {
public:
  explicit Ejemplo ();
  virtual ~Ejemplo () {}

  int numeroA () const;
  void setNumeroA (const int valor);

  int numeroB () const;
  void setNumeroB (const int valor);

  int suma () const;
  int resta () const;

private:
  int m_numeroA;
  int m_numeroB;
}
{% endhighlight %}

Fichero **CMakeLists.txt**:

{% highlight cmake linenos %}
PROJECT(ejemplo_lib CPP)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET(ejemplo_lib_SRC ejemplo.cpp)

ADD_LIBRARY(ejemplo_lib SHARED ${ejemplo_lib_SRC})
{% endhighlight %}

Explicando alguna de las funciones utilizadas en el **CMakeLists.txt**:

- **ADD_LIBRARY**: Esta es la función utilizada por CMake para definir que queremos crear una biblioteca, se especifica que tipo de biblioteca se requiere, ya sea estática (**STATIC**) o dinámica (compartida **SHARED**).
