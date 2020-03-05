---
title: "Laravel 7 - Lo nuevo"
date:  2020-03-05T18:50:40-03:00
draft: false
tags:
    - framework
    - laravel
    - laravel 7
    - php
---

Laravel 7 es la ultima version de laravel que se lanzo el 3 de Marzo del 2020.
En este post vamos a listar las nuevas funcionalidades del framework basandome en la [documentacion oficial](https://laravel.com/docs/7.x/releases).

## Laravel Airlock
Este es un nuevo paquete creado por [Taylor Otwell](https://github.com/taylorotwell) como ya sabran es el creador de laravel, siendo un nuevo paquete oficial tambien forma parte de la documentacion [Airlock Docs](https://laravel.com/docs/7.x/airlock).
El problema que viene a solucionar Airlock es cuando necesitas un sistema de authenticacion para aplicaciones variadas, como SPA(single page application), aplicaciones mobiles, y simples APIs basadas en tokens. Este paquete no es un reemplazo al sistema de authenticacion del framework, sino una buena extension para estos casos donde el sistema actual no cumple con estos casos. Proximamente voy a hacer un post dedicado exclusivamente a este paquete.

## Custom Eloquent Casts
Laravel actualmente ya cuenta con varios casts types dentro de el, pero no es tan raro que uno en su aplicacion necesite realizar los propios. Anteriormente esto lo realizabamos directamente en el modelo utilizando "Accessors" y "Muttators", esta nueva funcionalidad nos brinda una manera de tener estos casts organizados y reutilizables a lo largo de nuestra codigo.

Para implementar un nuevo cast nuestra clase solamente deberia implementar la interfaz `CastsAttributes`. Esta interfaz require que se definan `get` y `set` metodos. El metodo `get` se encarga de transformar el valor de la base de datos al valor que uno desea. El metodo `set` se encarga de transformar del valor deseado a uno para ser guardado en la base de datos.

En la documentacion podemos ver un ejemplo de como esta implementado el cast json de laravel. En mi caso vamos a hacer una variacion siendo cast a un objeto especifico. Supongamos que tenemos un campo que guarda dinero, por conveniencia en la base de datos decidimos no guardar valor flotante decidimos que sea de un tipo entero almacenando centavos, pero en nuestro codigo queremos utilizarlo en una clase `Money` que tiene funciones varias tales como conversiones operaciones y demas (esas funciones no entran en el scope actual)


```php
<?php
namespace App\Casts;

use App\Money;
use Illuminate\Contracts\Database\Eloquent\CastAttributes;

class Money implements CastsAttributes
{
    /**
     * Cast the given value.
     *
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  string  $key
     * @param  mixed  $value
     * @param  array  $attributes
     * @return array
     */
    public function get($model, $key, $value, $attributes)
    {
        return new Money($value);
    }

    /**
     * Prepare the given value for storage.
     *
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  string  $key
     * @param  Money  $value
     * @param  array  $attributes
     * @return string
     */
    public function set($model, $key, $value, $attributes)
    {
        return $value->toCents();
    }
}
```

Una vez que tenemos nuestro cast podemos utilizarlo en los modelos.

```php
<?php

namespace App;

use App\Casts\Money;
use Illuminate\Database\Eloquent\Model;

class Account extends Model
{
    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'balance' => Money::class,
    ];
}
```

## Blade Component Tags & Improvements

Bueno este es un poco dificil pero voy a hacer lo mejor que pueda para explicarlo. Ahora con blade definiendo un componente podemos tener una clase dedicada la cual contendra la logica especifica a este componente. Esto permite construir componentes mas complejos con la logica fuera de la vista misma.
Utilizando el ejemplo de la pagina de release de laravel lo voy a analizar.

Asumiendo el componente `App\View\Components\Alert` es de la siguiente manera.

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The alert type.
     *
     * @var string
     */
    public $type;

    /**
     * Create the component instance.
     *
     * @param  string  $type
     * @return void
     */
    public function __construct($type)
    {
        $this->type = $type;
    }

    /**
     * Get the class for the given alert type.
     *
     * @return string
     */
    public function classForType()
    {
        return $this->type == 'danger' ? 'alert-danger' : 'alert-warning';
    }

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|string
     */
    public function render()
    {
        return view('components.alert');
    }
}
```
Y tambien teniendo que la vista para el componente es la siguiente

```php
<!-- /resources/views/components/alert.blade.php -->

<div class="alert {{ $classForType }}" {{ $attributes }}>
    {{ $heading }}

    {{ $slot }}
</div>
```

Como se puede ver en la vista podemos hacer uso de `$classForType` lo cual va a ejecutar la funcion que tenemos en nuestro componente al momento del render. Asi mismo el componente pasa atravez de la variable `$attributes` todos los atributos de html definidos al momento del uso, lo mismo para `$heading` y `$slot` que no son cosas definidas en la clase sino globales a todos los componentes.

Para utilizarlo en otra vista ahora podemos hacer uso de "Component's tag"

```php
<x-alert type="error" class="mb-4">
    <x-slot name="heading">
        Alert content...
    </x-slot>

    Default slot content...
</x-alert>
```

Aparentemente hay mas atras de los nuevos componentes de Blade pero con esta introduccion podemos ver que se volvieron mucho mas poderosos.

## HTTP Client

Este es muy interesante para mi, en muchisimas applicaciones trabaje con APIs y otros servicios que requerian manipular request y desde que conoci Guzzle nunca mas quise ni quiero hacer un curl a mano. Aca en esta version del framework nos trae una implementacion alrededor de Guzzle mas que interesante.

del ejemplo podemos ver una API dinamica que nos permite hacer un post con header (usualmente para authenticacion)

```php
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://test.com/users', [
    'name' => 'Taylor',
]);

return $response['id'];
```

Dado que la mayoria de los casos de usos se sentran hoy en dia enviando y recibiendo JSON del equipo de laravel decidieron que este comportamiento sea el default, y si deseamos enviar como un formulario se define de una manera muy simple e intuitiva

```php
$response = Http::asForm()->post('http://test.com/users', [
    'name' => 'Sara',
    'role' => 'Privacy Consultant',
]);
```

Otro de los agregados mas interesantes que se pueden ver para esta nueva funcionalidad es la de testing, la cual a primera vista trae grandes beneficios. Para los que esten acostumbrados a los Fake de laravel como queue, mail, events va a ser muy intutivo.

```php
Http::fake([
    // Stub a JSON response for GitHub endpoints...
    'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

    // Stub a string response for Google endpoints...
    'google.com/*' => Http::response('Hello World', 200, ['Headers']),

    // Stub a series of responses for Facebook endpoints...
    'facebook.com/*' => Http::sequence()
                            ->push('Hello World', 200)
                            ->push(['foo' => 'bar'], 200)
                            ->pushStatus(404),
]);
```

Del ejemplo podemos ver como para nuestro test podemos definir un array donde la clave es una expresion regular asociada al dominio que se realizaria el pedido http y definir una respuesta definida, tanto con data, status code y headers.
Tambien podemos ver que se puede definir una secuencia que se repetira por cada llamada que hagamos, que puede ser mas que interesante para no dejar caso sin testear

### Fluent String Operations

Laravel ya contaba con una clase dedicada a los strings `Illuminate\Support\Str`, esta misma cuenta con muchos metodos utiles para la manipulacion de strings, pero ahora se tomo un aproach para encadenar los metodos para lograr un codigo mas declarativo

```php
return (string) Str::of('  Laravel Framework 6.x ')
                    ->trim()
                    ->replace('6.x', '7.x')
                    ->slug();
```
No hay mucho mas para decir que de esta manera se puede interpretar a simple vista lo que realiza el codigo y como mencione anteriormente logrando un codigo mas declarativo que siempre es deseado.

### Route Model Binding Improvements

Asumiendo que ya conocen Route Model binding que sino se estan perdiendo de algo demasiado bueno busquenlo, ahora tenemos unas mejoras.

#### Key Customization
Antes para poder definir que no sea el id teniamos que ir al modelo y declarar que campo se iba a utilizar esto traia como desventaja que a lo largo de nuestra applicacion el modelo se podia utilizar solo de esta forma, ahora se puede definir directamente en la ruta

```php
Route::get('api/posts/{post:slug}', function (App\Post $post) {
    return $post;
});
```
En este caso utilizara el campo slug en lugar del id que es el default

### Automatic Scoping
Aca laravel nos trae una mejora haciendo que al momento de tener multiples route binding va a generar que el segundo va a estar dentro del scope del primero (esto es para cuando se define una key diferente en el cual se utilizaria un where, dado que sino es mas rapido para la base utilizar el id).

```php
use App\Post;
use App\User;

Route::get('api/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

## Multiple Mail Drivers

En este caso lo que nos trae Laravel es la posibilidad de que al momento de enviar mail definir que Driver queremos usar, se explica que esto nos da versatilidad para en caso de necesitarlo algunas funciones de nuestra aplicacion como mails en cantidad utilizen un servicio, y otros como mail simples otros. y en caso de no definirlo se utilizara el default.

## Route Caching Speed

No hay mucho que decir mas que gracias a las mejoras de Symfony e implementadas en laravel ahroa contamos con nuestra aplicacion mas rapida sin tener que cambiar nada.

## CORS Support
Quizas se hayan cruzado con el error de CORS en el browser sin saber que es, en pocas palabras es cuando intetamos desde una pagina traer contenido o postear contenido de ahi el nombre Cross-Origin Resource Sharing. Ahora laravel nos permite por configuration definir que paths permitimos, methodos, a que sitios muy util en caso de que nuestra api sirva a otros sitios.

## Query Time Casts
El ejemplo que se plantea es muy interesante, nunca antes realize una query asi pero me interesa probarla, pero lo que ahora vamos a poder hacer es al momento de realizar query y obtener campos calculados castearlos como lo hariamos de la definicion en el modelo


```php
$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
            ->whereColumn('user_id', 'users.id')
])->withCasts([
    'last_posted_at' => 'date'
])->get();
```

## MySql 8+ Database Queue Improvements
Con esta mejora basicamente ahora es factible utilizar nuestra base de datos como motor de queue en un ambiente productivo, siempre y cuando contemos con las versiones adecuadas.

## Artisan `test` Command

Este comando es un wraper a ejecutable de php pero brindandonos una UX en la consola mucho mas agradable, yo muy probablemte hare el cambio para empezar a utilizarla


## Markdown Mail Template Improvements
Se actualizo el template con un nuevo diseño basado en la paleta de colors de Tailwind

## Stub Customization
Desde que tengo recuerdos utilizo los comandos de make de laravel, en algunos projectos siempre me encontre modificando algunas cosas de los archivos creados, si estuviste en la misma situacion, ahora laravel nos permite customizar para que se adapten a tus necesidades, es extremadamente util ser capaz de customizarlo por diferentes projectos y necesidades.


## Queue `maxExceptions` Configuration

Anteriormente laravel nos permitia definir cuantas veces queriamos que un Job intente procesarse, ahora tenemos una opcion de definir en caso de que falle con una exception no controlada lo intente una cantidad diferente.
Si nuestro Job tiene aluna logica para buscar algun recurso o algo que lo tenga que hacer reintentar y uno desea que lo haga una cantidad mayor, de que si el mismo fallo por un error no controlado.
