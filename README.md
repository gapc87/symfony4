# symfony 4

### Instalación


Estructura de carpetas completa:
```
composer create-project symfony/website-skeleton my-project
```

Estructura de carpetas simple:
```
composer create-project symfony/skeleton my-project
```

Si utilizamos el skeleton simple no se instalará el servidor PHP de Symfony,
para instalarlo:

```
composer require server --dev
```

##### Verificando la Estructura del Proyecto


* `config/`: Configuración de rutas, servicios y paquetes.

* `src/`: Todo el código PHP.

* `templates/`: Todas las plantillas de Twig.

* `bin/`: El bin/console vive aquí (y otros archivos ejecutables menos importantes).

* `var/`: Aquí es donde se almacenan los archivos creados automáticamente, como los archivos de caché (var/cache/) y los registros (var/log/).

* `vendor/`: Las bibliotecas de terceros (es decir, "proveedores") viven aquí. Estos se descargan a través del administrador de paquetes Composer.

* `public/`: Esta es la raíz del documento para su proyecto: aquí coloca los archivos accesibles públicamente.

---

Para iniciar el servidor:
```
php bin/console server:run
```

---

### Controladores y Rutas

#### [Controladores](https://symfony.com/doc/current/controller.html)


Para crear un controlador desde la consola:
```
php bin/console make:controller TestController
```
Este comando creará:
* El controlador: (*/src/Controller/TestController.php*)
* Una plantilla: (*/templates/test/index.html.twig*)

Podemos generar un CRUD completo a partir de una [entidad Doctrine](https://symfony.com/doc/current/doctrine.html):
```
php bin/console make:crud Product
```

Un controlador es una función de PHP que lee información del objeto `Request` 
y crea y devuelve un objeto `Response`. La respuesta podría ser una página
HTML, JSON, XML, una descarga de archivos, una redirección, un error 404 o
cualquier otra cosa que quieras. El controlador ejecuta cualquier lógica
arbitraria que su aplicación necesite para representar el contenido de una página.

Symfony viene con dos bases opcionales `Controller` y `AbstractController`. Puede
extender cualquiera de los dos para obtener acceso a algunos [métodos de ayuda](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/ControllerTrait.php).


Agregue la instrucción `use` encima de su clase  controlador y luego modifíque *TestController* para ampliarla:

````php
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class TestController extends Controller { //... }
````

Ahora tiene acceso a métodos como `$this->render()` o `$this->generateUrl()`:

* El método `render()` representa una plantilla y coloca ese contenido en un objeto `Response`:
```php
// renders templates/lucky/number.html.twig
return $this->render('lucky/number.html.twig', array('name' => $name));
```

* El método `generateUrl()` es un método auxiliar que genera la URL para una ruta determinada:

```php
$url = $this->generateUrl('app_lucky_number', array('max' => 10));
```
* Si desea redirigir al usuario a otra página, use los métodos `redirectToRoute()` y `redirect()`:
```php
use Symfony\Component\HttpFoundation\RedirectResponse;

// ...
public function index()
{
    // redirects to the "homepage" route
    return $this->redirectToRoute('homepage');

    // redirectToRoute is a shortcut for:
    // return new RedirectResponse($this->generateUrl('homepage'));

    // does a permanent - 301 redirect
    return $this->redirectToRoute('homepage', array(), 301);

    // redirect to a route with parameters
    return $this->redirectToRoute('app_lucky_number', array('max' => 10));

    // redirects externally
    return $this->redirect('http://symfony.com/doc');
}
```

Mas sobre controladores: 
* [Obtención de servicios](https://symfony.com/doc/current/controller.html#fetching-services)
* [Gestión de errores y páginas 404](https://symfony.com/doc/current/controller.html#managing-errors-and-404-pages)
* [El objeto Request como argumento del controlador](https://symfony.com/doc/current/controller.html#the-request-object-as-a-controller-argument)
* [Administrar la sesión](https://symfony.com/doc/current/controller.html#managing-the-session)
* [Mensajes Flash](https://symfony.com/doc/current/controller.html#flash-messages)
* [Objetos Request y Response](https://symfony.com/doc/current/controller.html#the-request-and-response-object)
* [Devolver JSON](https://symfony.com/doc/current/controller.html#returning-json-response)
* [Respuestas de archivos en tiempo real](https://symfony.com/doc/current/controller.html#streaming-file-responses)
* [Aún mas...](https://symfony.com/doc/current/controller.html#learn-more-about-controllers)

#### Rutas

Podemos definir las rutas para este controlador utilizando
`Annotation` para cada método que definamos en el controlador:

```php
/**
 * @Route("/test", name="test")
 */
 public function index()
 {
     return $this->render('test/index.html.twig', [
         'controller_name' => 'TestController',
     ]);
 }
```
Si no tenemos `Annotation` como dependencia del proyecto, podemos instalarlo:
```
composer require annotations
```

Otra forma de definir las rutas es por medio del archivo *config/routes.yaml*
```yaml
test:
  path:     /test/{_locale}/{year}/{slug}.{_format}
  controller: App\Controller\TestController::index
  defaults:
      _format: html
  requirements:
      _locale:  en|fr
      _format:  html|rss
      year:     \d+
```
Esta ruta solo funcinará si en la URL, `{_locale}` es 
`en` o `fr` y si `{year}` es un número. También muestra cómo
puede usar un punto entre marcadores de posición en lugar de
una barra inclinada. Las URL que coinciden con esta
ruta pueden ser similares a:

* /test/en/2010/my-post
* /test/fr/2010/my-post.rss
* /test/en/2013/my-latest-post.html


Para ver las rutas que tenemos definadas en el proyecto:
```
php bin/console debug:router
```

---

### Crear y usar plantillas

Twig define tres tipos de sintaxis especial:

* `{{ ... }}` "Dice algo": imprime una variable o el resultado de una expresión en la plantilla.
* `{% ... %}`"Hace algo": una etiqueta que controla la lógica de la plantilla; se usa para ejecutar sentencias como for-loops por ejemplo.
* `{# ... #}` "Comentar algo": es el equivalente de /* comment */ en PHP. Se usa para agregar comentarios de una o varias líneas. El contenido de los comentarios no está incluido en las páginas representadas.

Twig también contiene filtros, que modifican el contenido antes de ser procesados. Lo siguiente hace que la variable title sea mayúscula antes de representarla:
```
{{ title|upper }}
```

Twig viene con una larga lista de [etiquetas](http://twig.sensiolabs.org/doc/tags/index.html), [filtros](http://twig.sensiolabs.org/doc/filters/index.html) y [funciones](http://twig.sensiolabs.org/doc/functions/index.html) que están disponibles por defecto. Incluso puede agregar sus propios filtros personalizados, funciones (y más) a través de una [extensión Twig](https://symfony.com/doc/current/templating/twig_extension.html).

El siguiente ejemplo utiliza un estándar `for` la etiqueta y la función `cycle()` para imprimir etiquetas div diez veces, con la alternancia odd, even clases:
```twig
{% for i in 1..10 %}
    <div class="{{ cycle(['even', 'odd'], i) }}">
      <!-- some HTML here -->
    </div>
{% endfor %}
```
[mas sobre Twig...](https://symfony.com/doc/current/templating.html#template-inheritance-and-layouts)