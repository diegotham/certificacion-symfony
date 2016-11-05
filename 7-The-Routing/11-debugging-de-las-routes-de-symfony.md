Cuando se añaden y customizan routes es bastante útil ir viendo y obteniendo información de las routes que se van creando. Una buena forma de ver todas las routes de la aplicación es a través del comando de consola _debug:router_.
```
php bin/console debug:router
```

Este comando mostrará una lista de todas las **routes** configuradas en tu **aplicación**:
```
homepage              ANY       /
contact               GET       /contact
contact_process       POST      /contact
article_show          ANY       /articles/{_locale}/{year}/{title}.{_format}
blog                  ANY       /blog/{page}
blog_show             ANY       /blog/{slug}
```

Puedes también obtener información específica de una route incluyendo el nombre de la route después del comando.
```
php bin/console debug:router article_show
```

De la misma forma, si quieres testear si una URL coincide con una route, puedes emplear el comando **router:match**:
```
php bin/console router:match /blog/my-latest-post
```

Este comando imprimirá la route con la que coincide la URL:
```
Route "blog_show" matches
```
