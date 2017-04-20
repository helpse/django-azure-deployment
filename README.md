# Desplegar un proyecto Django en Azure
Desplegaremos un proyecto Django usando Web App Service en Azure desde un repositorio local.

## Creación de la app
Dentro del Panel de Azure ir a `New > Web + Mobile > Web App` y llenar los datos (app name, resource group ).

Desplegaremos el código desde nuestro repositorio local Git, para configurar ello vamos a 

`Deployment Options > Choose Source > Local Git Repository`

Para obtener la url de Git correspondiente vamos a

`Properties > GIT Url`

Copiamos la url y la agregamos a nuestro proyecto con el siguiente comando

`git remote add azure {GIT_URL}`

## Configuracion del proyecto local
En la raiz de nustro proyecto copiamos los 4 archivos que hay en este repositorio:
* ptvs_virtualenv_proxy.py
* runtime.txt
* web.2.7.config
* web.3.4.config

**runtime.txt**

Aqui especificamos que versión de python usaremos podemos escoger entre 2.7 y 3.4. Para ello en el archivo escribimos `python-2.7` o `python-3.4`

**web.{VERSION}.config**

Estos son archivos de configuración para el servidor, según la versión escogida en **runtime.txt** se usará uno u otro. Aquí modificaremos la linea 4 `{PROJECT_NAME}.wsgi.application` y especificamos la ruta de la aplicación wsgi por ejemplo `myproject.wsgi.application`.

**ptvs_virtualenv_proxy.py**

Este archivo será ejecutado por el servidor IIS, este script cargará el entorno virtual e importará la app wsgi buscando la ruta que se especifícó en **web.config**. Para más información sobre el proceso puede verse [wfastcgi](https://pypi.python.org/pypi/wfastcgi "wfastcgi")

Para que las dependencias sean instaladas asegurese de tenerlas especificadas en el archivo **requirements.txt**.

Para configurar los archivos estáticos agregue esta linea en el archivo **settings.py** 

`STATIC_ROOT = os.path.join(BASE_DIR, 'static')`

Agregar la url de tu proyecto en los hosts permitidos en el archivo settings.py, por ejemplo.
```
ALLOWED_HOSTS = [
    'myproject.azurewebsites.net',
]
```

Es importante tener un archivo **.gitignore** en donde ignoremos el entorno virtual, ya que este lo creará Azure.

## Despliegue del proyecto
Comitear todos los cambios con git y hacer un push a la url que se obtuvo al inicio 

`git push azure master`

Azure recibirá los cambios y ejecutará lo siguiente:
* Si aún no hay un entorno virtual lo creará e instalará las dependencias.
* Ejecutará el comando collectstatic de Django.

## Resolución de problemas
Para obtener mas detalles asegurese de configurar el log en `Diagnostics Logs`. Azure por defecto no muestra mucho detalles sobre los errores de tipo 500, para poder verlos puede agregar la siguiente linea en **web.config**
```
  <system.web>
    ...
    <customErrors="off" />
    ...
  </system.web>
  <system.webServer>
    ...
    <httpErrors errorMode="Detailed" />
    ...
  </system.webServer>
```
