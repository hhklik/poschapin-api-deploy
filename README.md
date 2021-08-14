<p align="center"><a href="https://poschapin.com/"><img width="500"src="https://user-images.githubusercontent.com/37667605/90585553-5a056800-e192-11ea-9ff1-77eb97a86c64.png" alt="WooCommerce"></a></p>

# Deployment poschapin-api for PHP
Este tutorial esta hecho para la integracion de API POSchapin en `PHP`, estamos trabajando para otros deployment con los lenguajes mas populares 

### Configurar public_key y private_key
- Solicitar creación de partner(Socio de negocios) en **admin@poschapin.com** Si ya es partner(Socio de negocios) salte este paso
- Con acceso a panel de control de POSchapin, hacer lo siguiente `wallet > Elegir wallet y dar click en EDIT` y en esta parte al entrar se visualizara el **public_key y private_key** para recolectar pagos para dicha billetera.
  - Si no logras procesar tu pago, en panel de control de POSchapin en `wallet > Elegir wallet y dar click en EDIT` vas a encontrar el siguiente icono <img src="https://poschapin.com/wp-content/assets/img/key_reload.svg" width="25" alt="reload_keys"> dando click en este icono y luego actualizar se podra crear otra **private_key** o **public_key**


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

`<orderid>|<amount>|<response>|<transactionid>|<avsresponse>|<cvvresponse>|<time>|<privatekey>`

Las llaves se puede obtener en la sección Llaves de seguridad de la billeta en su panel de control de POSchapin

## Transaction POST URL REDIRECT
Nota: No se puede utilizar CURL (PHP), dado que se tiene que recolectar la ip del cliente final, todo se trabaja CON HttpPOST
```
https://pos-chapin.appspot.com/api/transaccion/redirect
```
#### Ruta Deprecada
si usted utiliza la siguiente ruta:
```
@deprecated https://pos-chapin.appspot.com/transaccion/json
```
Actualmente se da soporte para esta ruta `@deprecated https://pos-chapin.appspot.com/transaccion/json`, pero cualquier cambio a futuro, no lo contendra

Sustituya por:
```
https://pos-chapin.appspot.com/api/transaccion/redirect
```
Y listo ya puede comenzar a recolectar pagos.


## Modulo de Prueba
Para acceder al `modulo de prueba` tiene ingresar al `panel de control de poschapin` buscar su `wallet` dar EDITAR, luego cambiar el desplegable `modo de pruebas` y seleccionar `HttpPost AND redirect`.
NOTA: si esta utilizando *HttpPost Direct* (respuestas en JSON) Tambien seleccionar `HttpPost AND redirect` para modulo de pruebas.
 |Tipo de tarjeta|numero de tarjeta|
| :---    |  :---  |
|Visa Card				| 4111111111111111|
|MasterCard Card 		| 5431111111111111|
|DiscoverCard Card 	| 6011601160116611|
|American Express Card 	| 341111111111111|
|Dinner club				| 36463664750005|

||Fecha de exipiracion|
| :---    |  :---  |
|Credit Card Expiration 	| Cualquier fecha de vencimiento válida|

|CVV|response_code|response_text|
| :---    | :---: | :---  |
|000| 200| Transaction decline|
|0000| 200| Transaction decline|
|001| 300| Error de puerta de enlace|
|0001| 300| Error de puerta de enlace|
|Cualquier CVV que no sean los anteriores| 100| Transaction aproved|


### Variables de Transaccion

| Variable name | required* | format | description | 
| :---         |     :---:      |     :---:     |  :---  |
| key_public   | requerido     |     | ID de clave de seguridad del panel de control del socio |
| amount     | requerido      | X.XX      | Importe total a cobrar (es decir 10.00) | 
| redirect     | requerido      | https://...      | La URL a la que se redirigirá al titular de la tarjeta. Esta URL también debe analizar y responder a la variables de respuesta incluidas en la consulta GET sobre la redirección. | 
| orderid     | requerido      |  `<texto>_<numero>` , `<numero>`  | valor numemrico o alfanumerico ej. orden_100 , 105 |
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


# Procedimiento para anular una transacccion

## Parámetros de autenticación para Anular una transacccion

- operationid
  - Este parametro es devuelto al a la hora de procesar un pago con el API de POSchapin

- time 
  - Un valor de tiempo también se asocia con la solicitud para proteger contra hashes
utilizado durante un período prolongado de tiempo. La puerta de enlace rechazará los tiempos que se terminaron
15 minutos de edad. El tiempo debe pasarse en el campo 'tiempo' y debe representar
una marca de tiempo de estilo C / Unix (número de segundos desde el 1 de enero de 1970 **UTC**, época)
- privatekey
  - clave privada de la wallet que va utilizar para recolectar pagos
  - Esta la encuentra en `panel de control > wallet > edit wallet` alli podra observar tando `key_public` y `key_private`
- hash 
  - El hash debe ser el valor de las siguientes variables delimitadas por pipes `|` y
hash con un algoritmo md5.
  - `<operationid>|<time>|<key_private>`

### ruta para anular una transacccion

`https://pos-chapin.appspot.com/api/transaccion/void`


### Variables de Anulacion

| Variable name | required* | format | description | 
| :---         |     :---:      |     :---:     |  :---  |
| key_public   | requerido     |    | API key de wallet del socio de negocios de POSchapin |
| operationid   | requerido     | `<text>`_<# operacion>   | codigo de operacion de la orden |
| hash     | requerido      |      | MD5 variable Hash |
| time | requerido | Unix Time Stamp | Segundos desde el 1 de enero de 1970 (época Unix) |
  
## Variables de respuesta a la anulacion

### Respuesta estándar
  
| Variable name | format | description | 
| :---     |     :---:     |  :---  |
| response | 1 / 2 / 3 | 1 = Transaction Approved /// 2 = Transaction Declined /// 3 = Error in transaction data or system error |
|response
| | Textual response|
|authcode| | Transaction authorization code|
|transactionid || Payment Gateway transaction id|
|hash ||The hash should be the values of defined variables delimited by pipes and hashed with an md5 algorithm.|
|avsresponse |C| |AVS Response Code (See Appendix 1)|
|cvvresponse |C|CVV Response Code (See Appendix 2)|
|orderid||The original order id passed in the transaction request.|
|response_code| C | Numeric mapping of processor responses (See Appendix 3)|

### Hash de respuesta

Después de realizar una llamada a la API, se devolverá inmediatamente un hash único en el
respuesta. El hash de respuesta protege la autenticidad de los datos devueltos al
servidores del socio. Realizar una verificación md5 adicional en el hash de respuesta asegurará
que la respuesta es auténtica. Específicamente, los siguientes campos están protegidos por la
hash de respuesta:

- response
- transactionid
- operationid
- time
- hash
 
### Validar Hash de respuesta

  `MD5(<response>|<operationid>|<transactionid>|key_private) = <hash en la respuesta>`

  
## posibles respuestgas
 
### repuesta satisfactoria

```
  {
    "response": "1",
    "responsetext": "Transaction Void Successful",
    "response_code": "100",
    "authcode": "<#number>",
    "transactionid": "<#number>",
    "operationid": "poschapin_10329",
    "avsresponse": "",
    "cvvresponse": "",
    "orderid": "poschapin",
    "type": "void",
    "amount": "",
    "trace": "",
    "void_text": "La orden fue cancelada",
    "void_time": 1628946111,
    "hash": "32c1eb5096e68180e5463261e39d2135",
    "time": 1628946112
}
```
  
### La orden ya se encuentra anualda
```
  {
    "response": 3,
    "responsetext": "La orden u operacion ya se encuentra anulada || The order or operation is already canceled",
    "response_code": 2312,
    "authcode": "",
    "transactionid": "",
    "operationid": "",
    "avsresponse": "",
    "cvvresponse": "",
    "orderid": "",
    "type": "",
    "amount": "",
    "trace": "2312,2119,2202",
    "error": true
}
```
  
### codigos de error
  

| response_code | description | 
| :---    |  :---  |
|8004| No se encuentra la wallet con ese key_public|
|2310| La operacion enviada no pertenece a esta billetera|
|2105|Hash invalido|
|2101|hash expirado|
  
NOTA: 
- Si esta implementando, varias wallets en el mismo sistema , se recomienda crear una tabla par lista de billeteras que van utilizar, con el fin de identifcar las ordenes con la billetera correspondiente, por temas de seguridad, dentro del panel de control de poschapin se puede cambiar la llave `publica` y `privada` por lo cual al sistema tendran que cambiar las llaves, pero sus ordenes tiene que seguir amarradas un identificador dentro de su sistema y no al `llave publica` la cual puede cambiar a lo largo de tiempo por motivos se seguridad.
- Ahora si solo va implementar una sola billetera, solo asegurese de dejar configurable la `llave publica` y `llave privada` y no queda en su codigo, porque estas llaves pueden cambiar por motivos de seguridad, como solo tiene una billetera implementada no se complica porque para anular envia la nueva `llave publica` por si ha cambiado.












