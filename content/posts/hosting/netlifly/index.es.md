---
title: "Web Hosting con Netlify"
date:  2020-03-06T16:55:10-03:00
draft: false
tags:
    - hosting
    - web
    - netlify
---

Por distintas razones hoy nos podemos encontrar con la idea de tener nuestro sitio web, y llega ese momento de tener que elegir donde lo pondremos, si estas en este blog es problable que tengas algun conocimiento en desarrollo o tengas interes y un producto del que me siento encantado de usar como tal es [Netlify](https://netlify.com).

## Que es Netlify
Netlify se define como una plataforma unificada para projectos web automaticos, que reemplaza la estructura de hosting tradicional, ofreciendo integracion continua y pipelines de deployment.

Lo que mas me ecanta de todo esto es la integracion continua, esta me permite cuando trabajo en mis projectos web con netlify simplemente tengo que hacer un push a mi repositorio de git (yo uso github, netlify tambien soporta bitbucket y gitlab) y netlify esta vinculado con el mismo y dispara el proceso de leer el nuevo codigo y subirlo a mi sitio, con eso ya no tengo el problema de viejos sitios como conectarme al ftp y subir los archivos manualmente. Con esta funcionalidad ya me simplifica muchisimo el proceso.

## Que no es Netlify

Netlify no es un reemplazo a servidores o hosting asociado con un backend como puede ser Node, Php, python, DotNet o Java. Lo que hacen ellos es reemplazar el hosting para projectos web de la parte de HTML y Javascript. Este blog lo hosteo utilizando Netlify.

## Como empezar

Empezar con netlify es tan simple como tener subir tus archivos a un resposiorio de git, que incluso si no usan netlify es mas que recomendado para backup y versionado de codigo. Y entrar a [Netlify](https://netlify.com) crear una cuenta, pueden hacerlo usando su login de github por ejemplo, y vincular su cuenta de github para que pueda acceder al resposiorio donde tienen el codigo. De ahi indicar donde esta el codigo, si por ejemplo es el directorio root, o algun sub directorio que usen. Y ya esta tam simple como eso tiene su sitio.
Netlify tambien cuenta con la posibilidad de que vinculen su dominio o incluso para que puedan comprarlo atravez de ellos, pero si no necesitan pueden usar unos de los subdominios que ellos generaran para tu sitio automaticamente.
Desde luego la documentacion cuenta con muchisima informacion y es super intuitivo el sitio para utilizarlo.

## Builds

Si deciden utilizarlo van a encontrarse con el concepto de "Builds", otra funcionalidad que ofrece netlify es contruir o procesar los archivos para generar el sitio. Esto es comun para spa hechas con javascript, Next.js que es para armar sitios con vuejs, Hugo que es para armar sitios staticos (utilizo este para este sitio) y muchos mas incluyendo una funcionalidad para correr un container de docker y poder contruir nuestro sitio como necesitemos.

## Costos
Otra gran ventaja es el precio, que para una sola persona es gratis, las limitaciones que tienen son muy altas y con projectos chicos son claramente inalcanzables y si las alcanzan esta mas que bien pagar por el servicio que brindan.
