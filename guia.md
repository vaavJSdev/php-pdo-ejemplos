# Conectando a PDO


// En mi caso me gusta usar un archivo de configuracion el cual posea las credenciales para conectar a la base de datos.
// Este archivo si se abre en el navegador saldra en blanco, es un modo de proteger tus credenciales:

// __config.ini.php
```php
// Si abre con navegador se ejecutara el retrn y saldra todo en blancl
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

// Ahora vamos a obtener ese archivo y a convertirlo a codigo leible por PHP


<?php
// obtenemos el archivo
$file = '__config.ini.php';
// guardamos el resultado
$config = parse_ini_file($file, true);
// asignamos los valores a las variables
$host = $config['database']['host'];
$user = $config['database']['username'];
$pass = $config['database']['password'];
$schema = $config['database']['schema'];
$encode = $config['database']['encode'];



$dsn = "mysql:host=$host;dbname=$schema;charset=$encode";
$opt = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES   => false,
];
$conexion = new PDO($dsn, $user, $pass, $opt);
?>
