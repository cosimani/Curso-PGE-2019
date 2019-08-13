.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Clase 22 - PGE 2018
===================
(Fecha: 8 de noviembre)



Ejercicio 37:
============

- Descargar la escena `Habitación <https://github.com/cosimani/Curso-PGE-2018/blob/master/sources/clase19/Habitacion.zip?raw=true>`_

- Replicar lo que se visualiza en el siguiente video: https://www.youtube.com/watch?v=Jr_luYdSfRE



**Podemos ahora llevar las imágenes de la cámara como textura a OpenGL**

.. code-block:: c++

	class Visual : public Ogl  {
		Q_OBJECT
	public:
		Visual();
		void iniciarCamara();

	protected:
		void initializeGL();
		void resizeGL( int ancho, int alto );
		void paintGL();

	private:
		Capturador * capturador;
		QCamera * camera;

		void cargarTexturas();
		void cargarTexturaCamara();

		unsigned char *texturaCielo;
		unsigned char *texturaMuro;
		GLuint idTextura[ 2 ];

		unsigned char *texturaCamara;
		GLuint idTexturaCamara[ 1 ];
	};

	void Visual::iniciarCamara()  {
		capturador = new Capturador;

		QList<QCameraInfo> cameras = QCameraInfo::availableCameras();

		for (int i=0 ; i<cameras.size() ; i++)  {
			qDebug() << cameras.at(i).description();

			if (cameras.at(i).description().contains( "Truevision", Qt::CaseInsensitive ) )  {
				camera = new QCamera( cameras.at( i ) );
				camera->setViewfinder( capturador );
				camera->start(); // to start the viewfinder
			}
		}

		glGenTextures(1, idTexturaCamara);
	}

	void Visual::cargarTexturaCamara()  {

		QVideoFrame frameActual = capturador->getFrameActual();
		texturaCamara = frameActual.bits();

		glBindTexture( GL_TEXTURE_2D, idTexturaCamara[ 0 ] );  // Activamos idTextura.
		glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR ); 
		glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR ); 

		glTexImage2D( GL_TEXTURE_2D, 
		              0, 
		              3, 
		              frameActual.width(), 
		              frameActual.height(), 
		              0, 
		              GL_BGRA, 
		              GL_UNSIGNED_BYTE, 
		              texturaCamara );
	}

**Ejercicio:**

- Crear una escena con OpenGL con glOrtho para mostrar como textura las imágenes de la cámara en un QUADS.
- Luego probar con gluPerspective

**Resolución**

- Descargar `Código fuente <https://github.com/cosimani/Curso-PGE-2018/blob/master/sources/clase19/camaraOgl.zip?raw=true>`_

Ejercicio 38:
============

- Dentro de la habitación elegir una pared para colocar un monitor LCD con las imágenes de la cámara.

Ejercicio 39:
============

- En el ejercicio de la Habitación, mejorar los movimientos que se realizan con el mouse.

Ejercicio 40:
============

- Con la barra espaciadora se deberá saltar dentro de la escena.


**Resolución del Ejercicio con punteros a métodos de la clase Ordenador** 

.. figure:: images/clase17/ordenador.png

.. code-block:: c++

	#ifndef ORDENADOR
	#define ORDENADOR

	class Ordenador  {
	public:
	    void burbuja(int * v, int n)  {
	        int i, j, aux;
	        for(i=0 ; i<=n ; i++)  {
	            for(j=0 ; j<n-1 ; j++)  {
	                if(v[j] > v[j+1])  {
	                    aux = v[j];
	                    v[j] = v[j+1];
	                    v[j+1] = aux;
	                }
	            }
	        }
	    }

	    void insercion(int * v, int n)  {
	        int i, j, aux;
	        for (i=1 ; i<n; i++)  {
	            aux = v[i];
	            j = i - 1;
	            while ( (v[j] > aux) && (j >= 0) )  {
	                v[j+1] = v[j];
	                j--;
	            }
	            v[j+1] = aux;
	        }
	    }
	};

	#endif // ORDENADOR
	
.. code-block:: c++

	#ifndef LISTADOENTEROS_H
	#define LISTADOENTEROS_H

	#include <QVector>
	#include "ordenador.h"

	class ListadoEnteros : public QVector<int>  {
	public:

	    void ordenar(void (Ordenador::*pFuncionOrdenamiento)(int *, int))  {
	        (ordenador.*pFuncionOrdenamiento)(this->data(), this->size());
	    }

	private:
	    Ordenador ordenador;
	};

	#endif // LISTADOENTEROS_H
	
.. code-block:: c++

	#ifndef PRINCIPAL_H
	#define PRINCIPAL_H

	#include <QWidget>
	#include "listadoEnteros.h"

	namespace Ui {
	    class Principal;
	}

	class Principal : public QWidget  {
	    Q_OBJECT

	public:
	    explicit Principal(QWidget *parent = 0);
	    ~Principal();

	private:
	    Ui::Principal *ui;
	    ListadoEnteros listado;

	private slots:
	    void slot_ordenar();
	    void slot_valorNuevo();
	};

	#endif // PRINCIPAL_H

.. code-block:: c++

	#include "principal.h"
	#include "ui_principal.h"

	Principal::Principal(QWidget *parent) : QWidget(parent), ui(new Ui::Principal)  {
	    ui->setupUi(this);

	    connect(ui->pbOrdenar, SIGNAL(clicked()), this, SLOT(slot_ordenar()));
	    connect(ui->leValorNuevo, SIGNAL(returnPressed()), this, SLOT(slot_valorNuevo()));
	}

	Principal::~Principal()  {  delete ui;  }

	void Principal::slot_ordenar()  {

	    if (ui->cbMetodo->currentText() == "Burbuja")  {
	        void (Ordenador::*burbuja)(int *, int) = &Ordenador::burbuja;
	        listado.ordenar(burbuja);
	    }
	    else  {
	        void (Ordenador::*insersion)(int *, int) = &Ordenador::insercion;
	        listado.ordenar(insersion);
	    }

	    for (int i=0 ; i<listado.size() ; i++)  {
	        ui->teOrdenados->append(QString::number(listado.at(i)));
	    }
	}

	void Principal::slot_valorNuevo()  {
	    listado.push_back(ui->leValorNuevo->text().toInt());

	    ui->teValores->append(ui->leValorNuevo->text());

	    ui->leValorNuevo->clear();
	}


Cálculo de la tercer nota
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c++

	Promedio entre los porcentajes obtenidos en:
	    Nota final = ( Cuestionario 1 ( % ) + Cuestionario 2 ( % ) + Cuestionario 3 ( % ) + 
	                   Cuestionario 4 ( % ) + Ejercicios 1 al 6 ( % ) + Ejercicios 7 al 12 ( % ) + 
	                   Ejercicios 13 al 17 ( % ) + Ejercicios 18 al 22 ( % ) ) / 8




