

# Microfronted con single-spa-angular

By rafa@rafapenya.com

## Crear la aplicacion host o aplicacion base

1. instalar **create-single-spa**

```
npm install --global create-single-spa
```

2. crear la aplicación base

```
create-single-spa
```

#### Pasos de la instalación:

 - Directory for new project: **nombre de la aplicación base**
 - Select type to generate: **single-spa root config** //IMPORTANTISIMO!!!
 - which package manager to use: **npm/yarn**
 - Will this project use typescript? **Yes** 
 - Use Layout Engine? **Yes** 
 - Organization name: escribir el nombre común que usarán todas las aplicaciones para conectarse, por ejemplo **YourOrganization**.
 
 Pulsar **ENTER**

La instalación se llevará a cabo y finalmente dará el mensaje de éxito y creará nuestra aplicacion host. 

Los archivos importantes y sobre los que nos centraremos más adelante están situados en el directorio **/src**

  - declarations.s.ts
  - index.ejs
  - microfrontend-layout.html
  - YourOrganization-root-config.ts //el nombre de este archivo empieza por el mismo nombre que escribiste como organization en el poroceso de instalacion.

 **IMPORTANTE:** para trabajar con aplicaciones hijas hechas co nAngular es importante modificar una línea dentro del archivo index.ejs de lo contrario dará fallo con el archivo **zone.js**

```
    <!--
    If you need to support Angular applications, uncomment the script tag below to ensure only one instance of ZoneJS is loaded
    Learn more about why at https://single-spa.js.org/docs/ecosystem-angular/#zonejs
  -->
    <script src="https://cdn.jsdelivr.net/npm/zone.js@0.11.3/dist/zone.min.js"></script> <!--descomentar esta linea-->
```

Seguidamente, cambiamos al directorio de nuestra app base/host y levantamos la aplicacion

```
npm start
```

para comprobar el funcionamiento y si todo va bien, se levantará en **localhost:9000** y mostrará una página de bienvenida con varios pasos, podemos ignorar el contenido, pero si lo vemos sabremos que nuestra aplicación base ha sido creada con éxito.


## Crear un proyecto Angular ara añadirlo como hijo en la aplicación base

Para ello crearemos un proyecto angular usando la CLI de manera convencional, es decir:

```
ng new nombre-aplicacion-hija
```

Responderemos a las instrucicones de instalacion:
 - Angular routing: **Yes**
 - stylesheet: **CSS/ SCSS etc**

Pulsamos intro y se procede a la instalación.
Cambiamos al directorio creado.

Seguidamente usaremos el comando *add de ng* para añadir *single-spa-angular*:

```
ng add single-spa-angular
```

pulsamos intro y esperamos a que nos haga la pregunta:

> The package single-spa-angular@version will be installed and executed.
> Would you like to procedd? **YES**

esto instalará el paquete.

A continuación volverá a preguntar si quieres usar el *routing Angular*:

> Does your application use Angular routing? **YES**

Despues pedirá el puerto desde el que se iniciará esta aplicacion hija:

> What port should your project run on? 

Dado que el puerto de la aplicación base que hemos creado anteriormente es el puerto **9000**, por coherencia, este puerto debería ser el **9001/9002/9003** dependiendo de si queremos reservar un puerto previo para navBars o Landings, etc, pero puede ser cualquier puerto disponible y en desuso.

Finalmente creará y modificará archivos en la app hija.

Se crearán dos nuevos scripts en el archivo *package.json* de la app hija:

```
"build:single-spa:nombredelaapp":"ng build nombredelaapp --configuration production"
"serve:single-spa:nombredelaapp":"ng s --project nombredelaapp --disable-host-check --port 9002 --live-reload false"
```

Este último nos servirá para levantar esta aplicación como single-spa.

**NOTA:** si quisieramos modificar el puerto, deberíamos hacerlo en esa línea y además en el archivo **angular.json**, 
en la línea **"deployUrl"**, donde ponga **localhost:9002** o el numero de puerto a cambiar.

#### Directorio empty-route

Al añadir *single-spa*, se crea un directorio llamado **empty-route**, el arcihvo que contiene sirve como apoyo. Nosotros utilizaremos este archivo para una **404**, y enrutaremos los *not found* hacia este componente.

> El directorio empty-route en una aplicación single-spa-angular generalmente contiene un componente Angular que no tiene contenido visible. Este componente "vacío" se utiliza para manejar rutas que no corresponden a ninguna funcionalidad específica dentro de tu microfrontend. En el contexto de single-spa, es especialmente útil para asegurar que cuando la aplicación hija no necesita mostrarse (por ejemplo, cuando se navega a otra ruta que no involucra esa aplicación), no haya contenido visible ni errores de enrutamiento. Es común usarlo para manejar casos de rutas no encontradas o "404" en la configuración de enrutamiento de Angular.


### Configuración del App Routing

En el caso de rutas *not found* redirigiremos al componente **EmptyRoute**

Añadiremos en el arreglo de rutas lo siguiente

```
{path: '**', component: EmptyRouteComponent }
```

A continuación añadiremos un *provider* a **@NgModule** de esta forma, que definirá el valor del base href como '/'

```
@NgModule({
   //... imports
   //...exports
   providers: [{provide:APP_BASE_HREF , useValue: '/'}]
})
```

Levantamos la aplicacoin:

```
npm run serve:single-spa:nombredelaapp
```

**NOTA:** Es posible que dé un error sobre **@angular-builders/custom-webpack**, si es el caso, debemos instalarlo

```
npm i @angular-builders/custom-webpack
```

Una vez instalado volvemos a levantar la aplicacion de la misma forma.

### Prueba en navegador

En el navegador iremos a **http://localhost:9002** (o el puerto elegido) y veremos que en la pantalla no se ve nada, está en blanco, pero en la consola del navegador no habrá ningun error.

**Esto es normal y debe ser así**, sin embargo podremos ver en la pestaña el nombre de la aplicación.

**NOTA:** para ver el contenido de la aplicación en modo *standalone (independientemente)* debemos levantarla como aplicacion nomral **(revisar esto si es cierto)**

Abrimos el inspector de Chrome desde **http://localhost:9002** y vamos a la pestaña **"Red"**,
allí veremos los archivos cargamos, debemos buscar **main.js**, alli comprobaremos que está cargando

> http://localhost:9002/main.js

esto es importante porque será la ruta con la que cargaremos nuestra app hija dentro de la app padre.


## Configuración de la aplicacion Base/host para albergar la app hija

Abrimos una nueva terminal y **cambiamos de nuevo a nuestra app base**.

**IMPORTANTE:** la app hija debe permanecer levantada.

Abriremos el archivo **index.ejs** para editarlo y buscaremos esta sección y añadiremos la linea indicada a continuación:

```
    <% if (isLocal) { %>
    <script type="systemjs-importmap">
      {
        "imports": {
          "@single-spa/welcome": "https://unpkg.com/single-spa-welcome/dist/single-spa-welcome.js",
          "@rpenya/root-config": "//localhost:9000/rpenya-root-config.js", //nuestra aplicación base
          "@rpenya/nombredelaapphija": "http://localhost:9002/main.js" // AÑADIR ESTA lÍNEA
        } 
      }
    </script>
    <% } %>
```

- **@rpenya/nombredelaapphija** debe ser un **identificador único**, no debe repetirse en otras aplicaciones hija
- La url de acceso es la resultante al levantar la app hija, en la pestaña **"Red"** del navegador: **http://localhost:9002/main.js**


Una vez hecho, abrimos para editar el archivo **microfrontend-layout.html**

Por defecto este archivo aparece así:

```
<single-spa-router>
  <main>
    <route default>
      <application name="@single-spa/welcome"></application>
    </route>
  </main>
</single-spa-router>
```

Debemos modificarlo para que aparezca nuestra app hija por defecto (en caso de ser una landing page tenerlo en cuenta)

```
<single-spa-router>
  <main>
    <route default>
       <application name="@rpenya/nombredelaapphija"></application>
    </route>
  </main>
</single-spa-router>
```
donde el identificador único **@rpenya/nombredelaapphija** pertenece a nuestra aplicación previamente configurada en **index.ejs**

Ahora levantaremos la app base/host:

```
npm run start
```

Es importante recordar que la app hija debe estar levantada.

Vamos al navegador y abrimos:

> http://localhost:9000/

Y ahí podremos ver el contenido, esta vez sí,  de la aplicacion hija que está cargando por defecto, tal y como hemos establecido.



### Levantar la app hija como Standalone

Para ello, solamente debemos levantar el proyecto usando el comando:

```
npm run start:standalone
```