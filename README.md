# poschapin-api
Tutorial para realizar un pago por medio de la API REST poschapin 

### Parámetros de autenticación de API
Para proteger completamente la autenticidad de las solicitudes y respuestas de la API de un socio,
Se deben usar llaves de seguridad cuando se usa el método de redireccionamiento del navegador. No puedes usar
un nombre de usuario y contraseña para la autenticación de API de redireccionamiento del navegador.
El método apropiado de autenticación se describe a continuación.

- ordenid
  - Es la orden que genera la plataforma partner de poschapin
  - Puede ser un valor alfanunumero ej. `<texto>`_`<Numero>` 
  - Puede serun valor numerico ej, `<Numero>`
  - Este valor se devolvera en Vairables GET, para actualizar la orden
- amount
  - Monto que se va cobrar
  - formato: 125.00, 200.00; si no lleva este formato tu transaccion sera rechazada
- time 
  - Un valor de tiempo también se asocia con la solicitud para proteger contra hashes
utilizado durante un período prolongado de tiempo. La puerta de enlace rechazará los tiempos que se terminaron
15 minutos de edad. El tiempo debe pasarse en el campo 'tiempo' y debe representar
una marca de tiempo de estilo C / Unix (número de segundos desde el 1 de enero de 1970 **UTC**, época)
- privatekey
  - clave privada de la wallet que va utilizar para recolectar pagos
- hash 
  - El hash debe ser el valor de las siguientes variables delimitadas por pipes `|` y
hash con un algoritmo md5.

```PHP
  $privatekey = (string) trim($this->get_option('privatekey',true));
  date_default_timezone_set('UTC');
  $time = time();

  $string_hash = $orderid . '|' . $amount . '|' . $time . '|' .  $privatekey;
  $hash = md5($string_hash);
  
  <?php $time = time(); ?>
  <form action="https://pos-chapin.appspot.com/transaccion/json">
  ...
  <input type=text name=key_public value=a74524fdcccf575b9219572978855df8>
  <input type=text name=hash value=<?php echo $hash?>>
  ...
  <form>
```

## Transaction POST URL
nota: No se puede utilizar CUrl (PHP), dado que se tiene que recolectar la ip del cliente final, todo se trabaja con redirecciones
```
https://pos-chapin.appspot.com/transaccion/json
```

### Transaction Variables

| Variable name | required* | format | description | 
| :---         |     :---:      |     :---:     |  :---  |
| key_public   | requerido     |     | ID de clave de seguridad del panel de control del socio |
| amount     | requerido      | X.XX      | Importe total a cobrar (es decir, 10,00) | 
| redirect     | requerido      | https://...      | La URL a la que se redirigirá al titular de la tarjeta. Esta URL también debe analizar y responder a la variables de respuesta incluidas en la consulta GET sobre la redirección. | 
| hash     | requerido      |      | MD5 variable Hash |
| time | requerido | Unix Time Stamp | Segundos desde el 1 de enero de 1970 (época Unix) |
|email| recomendado| | Billing email address|
|card_number| requerido | |Número de tarjeta de crédito|
|ccexp| requerido |MMYY| Vencimiento de la tarjeta de crédito (es decir, 0705 = 7/2005) |
|cvv| recomendado | | Código de Seguridad de la Tarjeta|
|first_name|recomendado| | Nombre del titular de la tarjeta|
|last_name|recomendado| | apellido del titular de la tarjeta|
|address1|recomendado | |Dirección de facturación de la tarjeta|
|city| recomendado | | Ciudad de facturación de la tarjeta|
|state|recomendado|CC| Estado de facturación de la tarjeta (abreviatura de 2 caracteres)|
|zip|recomendado| | Código postal de facturación de la tarjeta|
|country|recomendado |CC (ISO-3166)| País de facturación de la tarjeta (es decir, US)|
|phone|recomendado| | Número de teléfono de facturación|

## Variables de respuesta a la transacción

### Respuesta estándar

| Variable name | format | description | 
| :---     |     :---:     |  :---  |
| response | 1 / 2 / 3 | - 1 = Transaction Approved - 2 = Transaction Declined - 3 = Error in transaction data or system error |
















