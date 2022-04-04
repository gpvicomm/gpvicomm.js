# GpViComm
===================

GpViComm JS es una biblioteca que permite a los desarrolladores conectar fácilmente con el API de TARJETAS DE CRÉDITO de Global Payments ViComm.

[Revise el ejemplo funcionando](https://developers.gpvicomm.com/docs/payments/#javascript)

## Instalación

Primero necesitas incluir jQuery y los archivos `payment_stable.min.js` y `payment_stable.min.css` dentro de tu página web especificando "UTF-8" como charset.

```html
<script src="https://code.jquery.com/jquery-1.11.3.min.js" charset="UTF-8"></script>

<link href="https://cdn.gpvicomm.com/ccapi/sdk/payment_stable.min.css" rel="stylesheet" type="text/css" />
<script src="https://cdn.gpvicomm.com/ccapi/sdk/payment_stable.min.js" charset="UTF-8"></script>
```

## Uso

### Utilizando el Form de GpViComm

Cualquier elemento con la clase `payment-form` será automáticamente convertido en una entrada de tarjeta de crédito básica con fecha de vencimiento y el check de CVC.

La manera más fácil de comenzar con GpViCommForm es insertando el siguiente pedazo de código:

```html
<div class="payment-form" id="my-card" data-capture-name="true"></div>
```
Para obtener el objeto `Card` de la instancia `PaymentForm`, pregunte al formulario por su tarjeta.


```javascript
let myCard = $('#my-card');
let cardToSave = myCard.PaymentForm('card');
if(cardToSave == null){
  alert("Invalid Card Data");
}
```

Si la tarjeta (`Card`) regresada es null, el estado de error mostrará los campos que necesitan ser arreglados.

Una vez que obtengas un objeto no null de la tarjeta (`Card`) del widget, podrás llamar [addCard](#addcard-agregar-una-tarjeta).


### Biblioteca Init

Siempre debes inicializar la biblioteca.

```javascript
/**
  * Init library
  *
  * @param env_mode `prod`, `stg`, `local` para cambiar ambiente. Por defecto es `stg`
  * @param client_app_code proporcionado por Global Payments ViComm.
  * @param client_app_key proporcionado por Global Payments ViComm.
  */
Payment.init('stg', 'CLIENT_APP_CODE', 'CLIENT_APP_KEY');
```

### addCard Agregar una Tarjeta

La función addCard convierte los datos confidenciales de una tarjeta, en un token que puede pasar de forma segura a su servidor, para realizar el cobro al usuario.

Esta funcionalidad consume el servicio de nuestro API pero de manera segura para comercios que no cuentan con certificación PCI.
[Aquí](https://developers.gpvicomm.com/api/#metodos-de-pago-tarjetas-agregar-una-tarjeta) podras encontrar la descripción de cada campo en la respuesta.

```javascript
/*
 * @param uid Identificador del usuario. Este es el id que usas del lado de tu aplicativo.
 * @param email Email del usuario para iniciar la compra. Usar el formato válido para e-mail.
 * @param card La tarjeta que se desea tokenizar.
 * @param success_callback Funcionalidad a ejecutar cuando el servicio de la pasarela responde correctamente. (Incluso si se recibe un estado diferente a "valid")
 * @param failure_callback Funcionalidad a ejecutar cuando el servicio de la pasarela responde con un error.
 */
Payment.addCard(uid, email, cardToSave, successHandler, errorHandler);

let successHandler = function(cardResponse) {
  console.log(cardResponse.card);
  if(cardResponse.card.status === 'valid'){
    $('#messages').html('Tarjeta correctamente agregada<br>'+
                  'Estado: ' + cardResponse.card.status + '<br>' +
                  "Token: " + cardResponse.card.token + "<br>" +
                  "Referencia de transacción: " + cardResponse.card.transaction_reference
                );
  }else if(cardResponse.card.status === 'review'){
    $('#messages').html('Tarjeta en revisión<br>'+
                  'Estado: ' + cardResponse.card.status + '<br>' +
                  "Token: " + cardResponse.card.token + "<br>" +
                  "Referencia de transacción: " + cardResponse.card.transaction_reference
                );
  }else if(cardResponse.card.status === 'pending'){
    $('#messages').html('Tarjeta pendiente de aprobar<br>'+
                  'Estado: ' + cardResponse.card.status + '<br>' +
                  "Token: " + cardResponse.card.token + "<br>" +
                  "Referencia de transacción: " + cardResponse.card.transaction_reference
                );
  }else{
    $('#messages').html('Error<br>'+
                  'Estado: ' + cardResponse.card.status + '<br>' +
                  "Mensaje: " + cardResponse.card.message + "<br>"
                );
  }
  submitButton.removeAttr("disabled");
  submitButton.text(submitInitialText);
};

let errorHandler = function(err) {
  console.log(err.error);
  $('#messages').html(err.error.type);
  submitButton.removeAttr("disabled");
  submitButton.text(submitInitialText);
};
```

El tercer argumento para el addCard es un objeto Card que contiene los campos requeridos para realizar la tokenización.


### getSessionId

El Session ID es un parámetro que Global Payments ViComm utiliza para fines del antifraude.
Llame este método si usted desea recolectar la información del dispositivo del usuario.

```javascript
let session_id = Payment.getSessionId();
```

Una vez que tenga el Session ID, puedes pasarlo a tu servidor para realizar el cargo al usuario.


## PaymentForm Referencia Completa

### Inserción Manual

Si desea alterar manualmente los campos utilizados por PaymentForm para añadir clases adicionales, el placeholder o id. Puedes rellenar previamente los campos del formulario como se muestra a continuación.

Esto podría ser útil en caso de que desees procesar el formulario en otro idioma (de forma predeterminada, el formulario se representa en español), o para hacer referencia a alguna entrada por nombre o id.

Por ejemplo si desea mostrar el formulario en Inglés y añadir una clase personalizada para el _card_number_
```html
<div class="payment-form">
  <input class="card-number my-custom-class" name="card-number" placeholder="Card number">
  <input class="name" id="the-card-name-id" placeholder="Card Holders Name">
  <input class="expiry-month" name="expiry-month">
  <input class="expiry-year" name="expiry-year">
  <input class="cvc" name="cvc">
</div>
```


### Seleccionar Campos
Puedes determinar los campos que mostrará el formulario.

| Field                             | Description                                                      |
| :-------------------------------- | :--------------------------------------------------------------- |
| data-capture-name                 | Input para nombre del Tarjetahabiente, requerido para tokenizar  |
| data-capture-email                | Input para email del usuario                                     |
| data-capture-cellphone            | Input para teléfono celular del usuario                          |
| data-icon-colour                  | Color de los íconos                                              |
| data-use-dropdowns                | Utiliza una lista desplegable para establecer la fecha de expiración de la tarjeta   |
| data-exclusive-types              | Define los tipos de tarjetas permitidos                          |
| data-invalid-card-type-message    | Define un mensaje personalizado para mostrar cuando se registre una tarjeta no permitida  |

El campo 'data-use-dropdowns' puede resolver el problema que se presenta con la mascara de expiración en dispositivos móviles antiguos.

Se integra en el form de manera simple, como se muestra a continuación:
```html
<div class="payment-form"
id="my-card"
data-capture-name="true"
data-capture-email="true"
data-capture-cellphone="true"
data-icon-colour="#569B29"
data-use-dropdowns="true">
```

### Tipos de tarjetas específicos
Si deseas especificar los tipos de tarjetas permitidos en el formulario, como Exito o Alkosto. Puedes configurarlo como en el siguiente ejemplo:
Cuando una tarjeta de un tipo no permitido es capturado, el formulario se reinicia, bloqueando las entradas y mostrando un mensaje
*Tipo de tarjeta invalida para está operación.*


```html
<div class="payment-form"
id="my-card"
data-capture-name="true"
data-exclusive-types="ex,ak"
data-invalid-card-type-message="Tarjeta invalida. Por favor ingresa una tarjeta Exito / Alkosto."
>
```
Revisa todos los [tipos de tarjetas](https://developers.gpvicomm.com/api/#metodos-de-pago-tarjetas-marcas-de-tarjetas) permitidos por Global Payments ViComm.


### Leyendo los Valores

PaymentForm proporciona funcionalidad que le permite leer los valores del campo de formulario directamente con JavaScript.

Genera un elemento de PaymentForm y asigne un id único (en este ejemplo `my-card`)

```html
<div class="payment-form" id="my-card" data-capture-name="true"></div>
```
El siguiente javascript muestra cómo leer cada valor del formulario en variables locales.


```javascript
let myCard = $('#my-card');

let cardType = myCard.PaymentForm('cardType');
let name = myCard.PaymentForm('name');
let expiryMonth = myCard.PaymentForm('expiryMonth');
let expiryYear = myCard.PaymentForm('expiryYear');
let fiscalNumber = myCard.PaymentForm('fiscalNumber');
```


### Funciones

Para llamar a una función en un elemento PaymentForm, sigue el patrón a continuación.
Remplace el texto 'function' con el nombre de la función que desea llamar.

```javascript
$('#my-card').PaymentForm('function')
```

Las funciones disponibles se enumeran a continuación

| Function          | Description                                    |
| :---------------- | :--------------------------------------------- |
| card              | Obtiene la tarjeta del objeto                  |
| cardType          | Obtiene el tipo de tarjeta que se capturó      |
| name              | Obtiene el nombre capturado                    |
| expiryMonth       | Obtiene el mes de expiración de la tarjeta     |
| expiryYear        | Obtiene el año de expiración de la tarjeta     |
| fiscalNumber      | Obtiene el número fiscal del usuario / cédula  |



#### Función CardType

La función `cardType` devolverá una cadena según el número de tarjeta ingresado. Si no se puede determinar el tipo de tarjeta, se le dará una cadena vacía.

[Marcas permitidas](https://developers.gpvicomm.com/api/#metodos-de-pago-tarjetas-marcas-de-tarjetas)

