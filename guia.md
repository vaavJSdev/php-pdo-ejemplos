# Conectando a PDO


// En mi caso me gusta usar un archivo de configuracion el cual posea las credenciales para conectar a la base de datos.
// Este archivo si se abre en el navegador saldra en blanco, es un modo de proteger tus credenciales:

# config.ini.php

```php
// Si abre con navegador se ejecutara el return y saldra todo en blanclo
// Si se abre con la funcion de php se ignora el codigo php y se obtienen solos los datos
// Desde [database] hasta abajo.
<?php return; ?> ; 
[database]
driver="mysql"
host="localhost"
port="3306"
username="root"
schema="omd_2017"
password="" 
encode="utf8" 
```

__Ahora vamos a obtener ese archivo y a convertirlo a codigo legible por PHP

# conexion.php
```php
<?php
// obtenemos el archivo
$file = 'config.ini.php';
// guardamos el resultado
$config = parse_ini_file($file, true);
// asignamos los valores a las variables
$host = $config['database']['host'];
$user = $config['database']['username'];
$pass = $config['database']['password'];
$schema = $config['database']['schema'];
$encode = $config['database']['encode'];

// Una vez tengamos listas las credenciales vamos a conectar PDO, este se compone de tres elementos, primero los DSN
//aqui especificamos el motor de la base de datos, en este caso mysql, luego la base de datos, el servidor y el tipo de encodificaion

$dsn = "mysql:host=$host;dbname=$schema;charset=$encode";

// Seguido tenemos las opciones del PDO, en este caso que los errores devuelvan excepciones (investiga sobre esto), segundo que el moto //de recolectar (fetch) los datos sea asociativo, es decir, "campoBaseDatos" = "valorDeEseCampo"
// Tercero que NO emule las sentencias preparadas.
// Investiga que son sentencias preparadas.

$opt = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES   => false,
];
// Luego creamos el objeto pdo de conexionl pasando el dsn, el usuario, la contraseña y las opciones
$conexion = new PDO($dsn, $user, $pass, $opt);
?>
```

Ahora ese archivo lo vamos a requerir en todos nuestros formularios para tener la conexion lista.

Pero antes veamos como funciona una sentencia preparada

1) Escribes la consulta sql.
2) Preparamos nuestra sentencia, esto quiere decir, enviarle a el motor MySQL primero la sentencia y luego mandarle los valores, de modo que no se pueda alterar la sentencia.
3) Asignamos las variables a la sentencia.
4) Ejecutamos la consulta, en este paso PHP se encarga de unir las variables a la sentencia, en este paso los valores que entraron no podran cambiar, NI tampoco la sentencia, siendo asi mas seguro.
5) Obtememmos (fetch) los datos, segun el modo en que lo establecimos en la conexion

# Nota, esto es para las sentencias preparadas, en el caso de que NO tengan un WHERE en tu sql, no se preparan, revisa este enlace:

[Enlace](https://es.stackoverflow.com/questions/78554/diferencia-entre-execute-y-query-en-mysql/78560#78560)


Veamos ahora un ejemplo:

# Ejemplos
```php
// Requerimos la conexon
require_once 'conexion.php';
// Creamos nuestra consulta SQL -- Para saber el porque del  ? revisa el enlace anterior.
$sql = "SELECT * FROM usuario WHERE id=?"
// Obtenemos las variables con POST, ejemplo
$id = 5;
//creamos la sentencia , se usa $stmt por costumbre , pero puedes colocarle cualquier nombre
$stmt = $conexion->prepare(SQL);
//Asignamos los parametros, si se usa ? colocamos numeros, si se usan nombre :asiPorEjemplo, colocamos el nombre
$stmt->bindValue(1,$id,PDO::PARAM_INT);
// En este caso el PDO::PARAM_INT es para establecer el parametro esperado, en este caso se espera un ENTERO, existen otros como _STR, _FLOAT
// Ahora ejecutamos
$stmt->execute();
// Como establecimos en la conexion devolvera un array asociativo {"llave":"valor"} 
// Para leerlo podemos usar dos metodos:
// 1 Con un while, obtiendo los resultados en la varaible $resultado
while ($resultado = $stmt -> fetch()) {
    $nombre = $resultado["nombre"];
    $contrasena = $resultado["contrasena"];
}
// 2 Con un for in
