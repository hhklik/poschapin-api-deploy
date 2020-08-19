<p align="center"><a href="https://poschapin.com/"><img width="500"src="https://user-images.githubusercontent.com/37667605/90585553-5a056800-e192-11ea-9ff1-77eb97a86c64.png" alt="WooCommerce"></a></p>

# Deployment poschapin-api for PHP
Este tutorial esta hecho para la integracion de API POSchapin en `PHP`, estamos trabajando para otros deployment con los lenguajes mas populares 

## Parámetros de autenticación de API
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
## Hash de respuesta

Después de realizar una llamada a la API, se devolverá inmediatamente un hash único en el
respuesta. El hash de respuesta protege la autenticidad de los datos devueltos al
servidores del socio. Realizar una verificación md5 adicional en el hash de respuesta asegurará
que la respuesta es auténtica. Específicamente, los siguientes campos están protegidos por la
hash de respuesta:

- OrderID
- Amount
- Response
- TransactionID
- AVS_Response
- CVV_Response

### Campos adicionales devueltos:

- time
  - La puerta de enlace generará una hora en el formato descrito anteriormente. La aplicación del socio puede utilizar este tiempo para determinar si la respuesta es obsoleta.
- amount
  - La cantidad se repetirá en la respuesta.
- hash
  - El hash debe ser los valores de las siguientes variables delimitadas por pipes `|` y hash con un algoritmo md5.

`orderid|amount|response|transactionid|avsresponse|cvvresponse|time|privatekey`

Las llaves se puede obtener en la sección Llaves de seguridad de la billeta en su panel de control de POSchapin

## Transaction POST URL
nota: No se puede utilizar CUrl (PHP), dado que se tiene que recolectar la ip del cliente final, todo se trabaja CON HttpPOST
```
https://pos-chapin.appspot.com/transaccion/json
```

### Variables de Transaccion

| Variable name | required* | format | description | 
| :---         |     :---:      |     :---:     |  :---  |
| key_public   | requerido     |     | ID de clave de seguridad del panel de control del socio |
| amount     | requerido      | X.XX      | Importe total a cobrar (es decir 10.00) | 
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
| response | 1 / 2 / 3 | 1 = Transaction Approved /// 2 = Transaction Declined /// 3 = Error in transaction data or system error |
|responsetext| | Textual response|
|authcode| | Transaction authorization code|
|transactionid || Payment Gateway transaction id|
|hash ||The hash should be the values of defined variables delimited by pipes and hashed with an md5 algorithm.|
|avsresponse |C| |AVS Response Code (See Appendix 1)|
|cvvresponse |C|CVV Response Code (See Appendix 2)|
|orderid||The original order id passed in the transaction request.|
|response_code| C | Numeric mapping of processor responses (See Appendix 3)|

## informacion de prueba
||| 
|:---| :---|
|Visa Credit Card | 4111111111111111|
|MasterCard Card | 5431111111111111|
|DiscoverCard Card| 6011601160116611|
|American Express Card | 341111111111111|
|Credit Card Expiration| Cualquier fecha de vencimiento válida
|Amount| >1.00|

## Como saber si la conexion esta lisa?
Cuando el sistema retorne tarjeta declinada, se sabra que ya el API esta respondiendo correctamente, si no es asi consultar al equipo de tecnico de POSchapin


## Appendix 1 – AVS Response Codes
||| 
|:---| :---|
|X| Exact match, 9-character numeric ZIP|
|Y |Exact match, 5-character numeric ZIP|
|D| “|
|M| “|
|A |Address match only|
|B |“|
|W |9-character numeric ZIP match only|
|Z |5-character Zip match only|
|P |“|
|L| “|
|N |No address or ZIP match|
|C |“|
|U| Address unavailable|
|G |Non-U.S. Issuer does not participate|
|I |“|
|R |Issuer system unavailable|
|E| Not a mail/phone order|
|S |Service not supported|
|0 |AVS Not Available|
|O| “|
|B |“|

## Appendix 2 – CVV Response Codes
||| 
|:---| :---|
|M |CVV2/CVC2 Match|
|N |CVV2/CVC2 No Match|
|P |Not Processed|
|S| Merchant has indicated that CVV2/CVC2 is not present on card|
|U |Issuer is not certified and/or has not provided Visa encryption keys|


## Gateway response
|||
|:---|:---|
|100|Transaction was approved |
|200|Transaction was declined by processor|
|300|Transaction was rejected by gateway|
|400|Transaction Error Returned by processor |

|||
|:---|:---|
|612|Parameters missing see your guide|
|613|expired hash|
|614|amount format incorrect|
|615| Invalid Hash |
|616|transaction error returned by processor|
|617|stop data invalid, go get a code for children|
|6311|ccexp invalid|
|6312|ccexp not available|
|6313|cvv invalid|
|6314|cvv not available|
|6315|card invalid|
|6316|card not available|







