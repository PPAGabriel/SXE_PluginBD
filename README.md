# Plugin "Conquer With Words"

El siguiente plugin es un plugin de WordPress que permite a los usuarios de WordPress crear un post con un título y un contenido, y que el contenido sea analizado por el plugin para determinar si el contenido es positivo o negativo.

## ¿Cómo funciona?

El plugin utiliza dos arrays, uno corresponde a las palabras malsonantes y el otro a las palabras positivas. El plugin analiza el contenido del post y cuenta cuantas palabras malsonantes y positivas hay en el contenido. Si hay más palabras malsonantes que positivas, el plugin devuelve un mensaje de error y no permite publicar el post.

- **Función de "renym_wordpress_typo_fix":**

  Esta función toma un parámetro "$text", que se supone es el contenido del post o la página del WP. Dentro de esta, reemplazamos las palabras consideradas ofensivas por un asterisco. Para esto, utilizamos la función "str_replace" de PHP, que toma tres parámetros: el primero es la palabra que queremos reemplazar, el segundo es la palabra por la que queremos reemplazarla y el tercero es el texto en el que queremos hacer el reemplazo. En este caso, el primer parámetro es la palabra que queremos reemplazar, el segundo es la que deseamos usar como sustituto. El tercer parámetro es el texto que queremos analizar, que en este caso es el contenido del post o la página.

```php

function renym_wordpress_typo_fix( $text ) {
    global $swearingWords, $nonSwearingWords;
    return str_replace($swearingWords, $nonSwearingWords, $text);
}
```

- **Función de "add_filter":**
  Esta agrega un filtro al contenido de WP utilizando la función "renym_wordpress_typo_fix" que creamos anteriormente. Esto se hace utilizando la función "add_filter" de WP, que toma dos parámetros: el primero es el nombre del filtro que queremos agregar y el segundo es la función que queremos utilizar como filtro. En este caso, el primer parámetro es "the_content" que es el nombre del filtro que queremos agregar y el segundo es la función que creamos anteriormente.

```php

add_filter( 'the_content', 'renym_wordpress_typo_fix' );
```

- **The_Content:**
  En WordPress, the_content es un gancho (hook) que se utiliza para mostrar el contenido principal de un post o página. Este gancho se emplea en el ciclo de WordPress para mostrar el contenido del post en el área de contenido de la página. Este gancho se utiliza principalmente para agregar contenido adicional antes o después del contenido del post.

---

# ACTUALIZACIÓN: Uso de la Base De Datos

Para mejorar la aplicación, la lista de palabras malsonantes y positivas se guardará en una base de datos. Para esto, se utilizará la base de datos de WordPress.

## Creación de la tabla en la base de datos

Para crear la tabla en la base de datos, se utiliza la función "dbDelta" de WP. Esta función toma un parámetro, que es la consulta SQL que queremos ejecutar.

```php
function createTable(){
    global $wpdb;
    //Nombre de la tabla
    $table_name = $wpdb->prefix . "words";
    //Charset de la tabla
    $charset_collate = $wpdb->get_charset_collate();
    //Sentencia SQL
    $sql = "CREATE TABLE IF NOT EXISTS $table_name (
        id mediumint(9) NOT NULL AUTO_INCREMENT,
        palabrotas varchar(255) NOT NULL,
        eufemismo varchar(255) NOT NULL,
        PRIMARY KEY  (id)
    ) $charset_collate;";
    //Incluir el fichero para poder ejecutar dbDelta
    require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
    dbDelta( $sql );
}

add_action("plugins_loaded", "createTable");
```

Tanto la variable **"wpdb"** como **"charset_collate"** son variables de WP. La variable **"wpdb"** es una instancia de la clase **"wpdb"** que se utiliza para conectarse a la base de datos. La variable **"charset_collate"** es una variable que contiene el charset de la base de datos.

También cabe destacar que se utiliza la función **"require_once"** para incluir el fichero que contiene la función **"dbDelta"**. Esta función se utiliza para ejecutar la sentencia SQL que creamos anteriormente.

El uso de **"add_action"** es para que la función se ejecute cuando se cargue el plugin.

## Inserción de datos en la base de datos

Para insertar datos en la base de datos, se utiliza la función "insert" de WP. Esta función toma dos parámetros, el primero es el nombre de la tabla (o bien columna) en la que queremos insertar los datos y el segundo es un array asociativo con los datos que queremos insertar. En este caso, se insertan los datos en la columna de palabras malsonantes y en la columna de palabras positivas.

```php
function insertData(){
    global $wpdb, $swearingWords, $nonSwearingWords;
    $table_name = $wpdb->prefix . "words";
    $flag = $wpdb->get_results("SELECT * FROM $table_name");
    if (count($flag)==0){
        for ($i = 0; $i < count($swearingWords); $i++){
            $wpdb->insert(
                $table_name,
                array(
                    'palabrotas' => $swearingWords[$i],
                    'eufemismo' => $nonSwearingWords[$i]
                )
            );
        }
    }
}

add_action("plugins_loaded", "insertData");
```

Con una bandera, se verifica si la tabla está vacía. Si la tabla está vacía, se insertan los datos en la tabla (esto evita que la lista se vuelva a añadir, duplicando las columnas con la misma información).

## Obtención de datos de la base de datos y su uso en el plugin

Para obtener los datos de la base de datos, se utiliza la función "get_results" de WP. Esta función toma un parámetro, que es la consulta SQL que queremos ejecutar.

```php
function selectData(){
    global $wpdb;
    $table_name = $wpdb->prefix . "words";
    $results = $wpdb->get_results("SELECT * FROM $table_name");
    return $results;
}
```

Finalmente, se recoge dicho contenido en una variable alojada en la función que previamente conociamos (**"renym_wordpress_typo_fix"**).


```php
function renym_wordpress_typo_fix( $text ) {
    $words = selectData();
    foreach ($words as $result){
        $swearingWords[] = $result->palabrotas;
        $nonSwearingWords[] = $result->eufemismo;
    }
    return str_replace($swearingWords, $nonSwearingWords, $text);
}

add_filter( 'the_content', 'renym_wordpress_typo_fix' );
```

Se realiza un "for each" recorriendo una a una las filas con el contenido que nos devuelve la función **"selectData"**. De esta manera, se recogen las palabras malsonantes y las palabras positivas en dos arrays diferentes. Para ubicar la columna deseada al guardarlos en los arrays respectivos, usamos una función flecha (->), que nos permite acceder a la columna deseada de la fila que estamos recorriendo.

De esta manera, podemos utilizar las palabras malsonantes y positivas que se encuentran en la base de datos para analizar el contenido del post o la página y determinar si el contenido es positivo o negativo!

## Dale 10 a esta actualización si te gustó! :smile: