---
title: "Cómo hacer integración con Tpaga API usando Python"
date: 2016-08-18 08:53:45 -0500
excerpt_separator: <!-- more -->
comments: true
featured: true
categories:
- Python
---

[Tpaga](https://tpaga.co/) es una plataforma que permite recibir pagos electrónicos. Tiene una estructura sencilla para entender y fácil para usar.

Para obtener nuestros claves de acceso y conectarnos con el API de Tpaga, creamos una cuenta en el "sandbox" de la plataforma: [sandbox.tpaga.co](https://sandbox.tpaga.co/).

![](/images/tpaga_sandbox_login.png)

Al registrarnos podemos ver que ahora tenemos dos claves que podemos usar para la autenticación: **Private Api Key** y **Public Api Key**:

![](/images/tpaga_sandbox_dashboard.png)

<!-- more -->

Tpaga tiene unos modelos básicos que nos permitirán organizar nuestros datos: **Customers** (Clientes), **Credit Cards** (Tarjetas de crédito) asociados a los Clientes y **Charges** (Transacciónes o cobros por tarjeta de crédito).

Ahora, cuando entendemos la estructura, podemos empezar a escribir nuestro programa en Python. Primero instalamos la librería `requests` que nos permitirá hacer peticiones HTTP.

```bash
$ pip install requests
```

Tpaga, como muchos otros sitios web, acepta la autenticación mediante HTTP Basic Auth. La librería `requests` provee una forma fácil de usarla:

```python
>>> import requests
>>> TPAGA_PRIVATE_TOKEN = 'd13fr8n7vhvkuch3lq2ds5qhjnd2pdd2'
>>> tpaga_url = 'https://sandbox.tpaga.co/api/customer'
>>> requests.post(tpaga_url, json={}, auth=(TPAGA_PRIVATE_TOKEN, ''))
<Response [201]>
```

Escribimos un código sencillo que nos permitirá conectarse al API de Tpaga y mandar peticiones:

```python
from urllib import parse as urlparse
import requests

TPAGA_PRIVATE_TOKEN = 'd13fr8n7vhvkuch3lq2ds5qhjnd2pdd2'
TPAGA_API_URL = 'https://sandbox.tpaga.co/api/'

class TpagaTestClient:
    def __init__(
            self,
            private_token=TPAGA_PRIVATE_TOKEN,
            base_url=TPAGA_API_URL,
    ):
        self.base_url = base_url
        self.private_token = private_token

    def api_post(self, path, data, token=None):
        if not token:
            token = self.private_token
        return requests.post(
            urlparse.urljoin(self.base_url, path),
            json=data, auth=(token, ''),
        )

    def fail(self, response):
        raise Exception(
            'Whoops, got\n\nSTATUS: {}\n\nHEADERS: {}\n\nCONTENT: {}'.format(
                response.status_code,
                response.headers,
                response.content,
            ))

    def json_from_response(self, response, expected_http_code=None):
        if not expected_http_code:
            expected_http_code = [201]
        if response.status_code not in expected_http_code:
            self.fail(response)
        if not response.content:
            return None
        return response.json()

```

El método `__init__` nos va a inicializar nuestro cliente, `api_post` - mandar peticiones POST a la ruta especificada (`path`) del API, `json_from_response` - obtener un objeto JSON de la respuesta de API, `fail` - imprimir los detalles de la respuesta si la petición no ha terminado con éxito.

### Crear un cliente

Para crear nuestro cliente, vamos a enviar una petición POST al endpoint `/customer`:

```python
class TpagaTestClient:
    # ...

    def create_customer(self, data):
        response = self.api_post('customer', data)
        return self.json_from_response(response)
```

```python
>> client = TpagaTestClient()
>> customer = client.create_customer({
    'firstName': 'Horns and Hoofs',
    'lastName': 'Perez',
    'email': 'hornsandhoofs@example.com',
    'phone': '012345678'
})
>> customer_token = customer['id']
>> print('customer_token', customer_token)
customer_token qoodmh04sh7ghpp58opn5g0hssg4slq0
```

Otros campos que podemos enviar para guardar nuestros clientes se puede encontrar aquí: [tpaga.co/docs/swaggers/v2#!/Customer/createCustomer](https://tpaga.co/docs/swaggers/v2#!/Customer/createCustomer).

En el dashboard de Tpaga podemos asegurarnos de que el ciente ["Horns and Hoofs"](https://en.wikipedia.org/wiki/The_Little_Golden_Calf#Cultural_influence) fue creado exitosamente:

![](/images/tpaga_dashboard_customers.png)

Teniendo un token de nuestro cliente, podemos agregarle una tarjeta de crédito.

### Registrar una tarjeta de crédito y asociarla al cliente

La creación de la tarjeta de crédito se realiza en dos pasos: tokenizar la tarjeta y asociarla un cliente.

Tpaga usa **tokenización**, que nos permite registrar las tarjetas de crédito de nuestros clientes de forma segura. Los clientes ingresan los datos en nuestro sitio web, y estos datos los enviamos directamente al API de Tpaga (desde el código JavaScript), allí serán tokenizados y Tpaga nos devuelve un token temporal con el que podemos proceder con el registro de la tarjeta (desde el código Python).

Creamos un formulario HTML para obtener los datos de la tarjeta de crédito:

![](/images/tpaga_credit_card_form.png)

Para hacer un formulario bonito, usamos [la plantilla de Bootstrap](http://bootsnipp.com/snippets/featured/credit-card-payment-with-stripe) y las librerías [jQuery](https://jquery.com/), [jQuery Validation Plugin](https://jqueryvalidation.org/) y [jQuery.payment](https://github.com/stripe/jquery.payment) para validar los datos.

```html
<form id="credit_card_form" method="post" name="credit_card_form">
    <div class="row">
      <div class="col-xs-12">
        <div class="form-group" id="div_id_primaryAccountNumber">
          <label class="control-label requiredField" for="id_primaryAccountNumber">Número de la tarjeta</label>
          <div class="controls">
            <div class="input-group">
              <input class="textinput textInput form-control" id="id_primaryAccountNumber" name="primaryAccountNumber" required="" type="text"> <span class="input-group-addon"><i class="fa fa-credit-card"></i></span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-xs-12">
        <div class="form-group" id="div_id_cardHolderName">
          <label class="control-label requiredField" for="id_cardHolderName">Nombre del tarjetahabiente</label>
          <div class="controls">
            <input class="textinput textInput form-control" id="id_cardHolderName" name="cardHolderName" required="" type="text">
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-xs-3">
        <div class="form-group" id="div_id_expirationMonth">
          <label class="control-label requiredField" for="id_expirationMonth">Fecha de expiración</label>
          <div class="controls">
            <select class="select form-control" id="id_expirationMonth" name="expirationMonth" required="">
              <option value="01">Enero</option>
              <option value="02">Febrero</option>
              <option value="03">Marzo</option>
              <option value="04">Abril</option>
              <option value="05">Mayo</option>
              <option value="06">Junio</option>
              <option value="07">Julio</option>
              <option value="08">Agosto</option>
              <option value="09">Septiembre</option>
              <option value="10">Octubre</option>
              <option value="11">Noviembre</option>
              <option value="12">Diciembre</option>
            </select>
          </div>
        </div>
      </div>
      <div class="col-xs-3">
        <div class="form-group" id="div_id_expirationYear">
          <label class="control-label requiredField" for="id_expirationYear">Año</label>
          <div class="controls">
            <select class="select form-control" id="id_expirationYear" name="expirationYear" required="">
              <option value="2016">2016</option>
              <option value="2017">2017</option>
              <option value="2018">2018</option>
              <option value="2019">2019</option>
              <option value="2020">2020</option>
              <option value="2021">2021</option>
              <option value="2022">2022</option>
              <option value="2023">2023</option>
              <option value="2024">2024</option>
              <option value="2025">2025</option>
            </select>
          </div>
        </div>
      </div>
      <div class="col-xs-3 pull-right">
        <div class="form-group" id="div_id_cvc">
          <label class="control-label requiredField" for="id_cvc">CVC</label>
          <div class="controls">
            <input class="textinput textInput form-control" id="id_cvc" maxlength="10" name="cvc" required="" type="password">
          </div>
        </div>
      </div>
    </div>
    <div class="buttonHolder">
      <input class="btn btn-primary bg-purple" id="submit-id-submit" name="submit" type="submit" value="Guardar">
    </div>
</form>
```

Al final, tenemos un formulario con los siguientes campos:

* `primaryAccountNumber` para el número de la tarjeta,
* `cardHolderName` para el nombre del tarjetahabiente,
* `expirationMonth` para el mes de expiración,
* `expirationYear` para el año de expiración,
* `cvc` para el código CVC.

Creamos otro formulario oculto que usaremos para enviar el token temporal de la tarjeta de crédito a nuestro servidor:

```html
<form id="associate_customer_cc_form" action="/asociar-cliente-tarjeta-credito" method="POST">
  <input type="hidden" name="tmp_cc_token">
</form>
```

En el código JavaScript obtenemos los datos de la tarjeta de crédito y los enviamos al endpoint de Tpaga `tokenize/credit_card`. Este endpoint convierte los datos sensibles de la tarjeta en un token, el cual será empleado para ejecutar el procesamiento de las transacciones sin necesidad que los datos sensibles del tarjetahabiente pasen por nuestro servidor. Si la información de la tarjeta tiene errores, Tpaga nos devuelve un JSON con el nombre de campo y el mensaje de error, y en el caso contrario - token temporal de la tarjeta.

```javascript
$(function () {
  'use strict';

  $.fn.serializeObject = function() {
      var o = {};
      var a = this.serializeArray();
      $.each(a, function() {
          if (o[this.name] !== undefined) {
              if (!o[this.name].push) {
                  o[this.name] = [o[this.name]];
              }
              o[this.name].push(this.value || '');
          } else {
              o[this.name] = this.value || '';
          }
      });
      return o;
  };

  function associate_customer_cc(data, text_status, request) {
      $('[name="tmp_cc_token"]').val(data.token);
      // Enviar el token temporal a nuestro servidor
      $('#associate_customer_cc_form').submit();
  }

  function show_errors(request, text_status, error_thrown) {
      // Mostrar errores de validación
      if (request.status == 401) {
          $('#credit_card_form').find('.payment-errors').closest('.row').show();
          $('#credit_card_form').find('.payment-errors').text('Error de autenticación a la plataforma de pagos.');
          return;
      }
      if (request.status == 422) {
          var data = JSON.parse(request.responseText);
          $('#credit_card_form').find('.payment-errors').closest('.row').show();
          $('#credit_card_form').find('.payment-errors').text('Datos erróneos en el campo ' + $.trim($('#credit_card_form label[for="id_' + data.errors[0].field + '"]').text()));
          return;
      }
  }

  function tokenize_credit_card() {
    $('#credit_card_form').find('.payment-errors').closest('.row').hide();
    $('#credit_card_form').find('.payment-errors').text('');

    var tpaga_public_key = 'pk_test_qvbvuthlvqpijnr0elmtg5jh';

    // Enviar los datos de la tarjeta directamente a Tpaga y obtener el token temporal
    $.ajax('https://sandbox.tpaga.co/api/tokenize/credit_card', {
      method: 'POST',
      beforeSend: function (xhr) {
          xhr.setRequestHeader('Authorization', 'Basic ' + btoa(tpaga_public_key + ':'));
      },
      username: tpaga_public_key,
      password: '',
      data: JSON.stringify($('#credit_card_form').serializeObject()),
      contentType: 'application/json',
      dataType: 'json',
      success: associate_customer_cc,
      error: show_errors
    });
    return false;
  }

  $('#credit_card_form').on('submit', tokenize_credit_card);

});
```

donde `tpaga_public_key` es **la llave PÚBLICA** que copiamos desde el dashboard de Tpaga.

Ahora usando el token temporal de la tarjeta (`tmp_cc_token`) podemos asociarla al cliente:

```python
class TpagaTestClient:
    # ...

    def assoc_cc_to_customer(self, customer_token, cc_temp_token=None):
        cdata = {'token': cc_temp_token }
        response = self.api_post(
            'customer/{}/credit_card_token'.format(customer_token), cdata
        )
        return self.json_from_response(response)
```

```python
>> cc_temp_token = request.POST['tmp_cc_token']
>> credit_card = client.assoc_cc_to_customer(
    customer_token=customer_token,
    cc_temp_token=cc_temp_token,
)
>> credit_card_token = credit_card['id']
print('credit_card_token', credit_card_token)
credit_card_token 2k54foql0hki0ot7avrg9nhpvbpqam55
```

Mirando la tabla de [sandbox.tpaga.co/merchantDashboard/cards](https://sandbox.tpaga.co/merchantDashboard/cards) vemos que nuestra tarjeta quedó registrada con Tpaga.

Opcionalmente, dependiendo de la configuración de nuestra cuenta en Tpaga, Tpaga puede hacer un cargo de prueba a la tarjeta de crédito al crearla, automáticamente. En este caso, el valor la respuesta `response['validationCharge']['successful']` nos indica si el pago fue exisoto, y `response['validationCharge']['errorCode']` tiene el código de error. Por ejemplo:

```python
{
    'id': 'r6ae7t2u7dmt7injv5c2bg9sqbh0krtr',
    'addressLine2': None,
    'addressCountry': None,
    'addressState': None,
    'expirationMonth': '03',
    'lastFour': '0004',
    'bin': '404000',
    'fingerprint': '812a1abf6c03db89bdf91025687fe5a77e24065a652860445e45e62fce3a2858',
    'addressCity': None,
    'addressPostalCode': None,
    'type': 'VISA',
    'cardHolderName': 'Иван Иваныч',
    'customer': 'qoodmh04sh7ghpp58opn5g0hssg4slq0',
    'validationCharge': {'successful': False, 'errorCode': '04'},
    'addressLine1': None,
    'expirationYear': '2018'
}
```

### Realizar el pago por la tarjeta de crédito

Con el token de la tarjeta de crédito, que obtuvimos en el paso anterior, ahora podemos realizar pagos. Para eso enviemos una petición POST al [addCreditCardCharge](https://tpaga.co/docs/swaggers/v2#!/Charge/addCreditCardCharge) endpoint con los siguientes parametros:

* `orderId` - nuestro id interno que asociamos al pago, que luego nos ayudaría a identificar la transacción en el dashboard de Tpaga;
* `amount`- cantidad de dinero para cobrar,
* `currency`- tipo de moneda, por ejemplo, 'COP',
* `creditCard` - token de la tarjeta de crédito.

```python
class TpagaTestClient:
    # ...

    def charge_cc(self, cc_token, order_id='BRG-2', amount=1000):
        cdata = {
              'orderId': order_id,
              'currency': 'COP',
              'taxAmount': 0,
              'description': 'One bridge in good condition.',
              'installments': 1,
              'amount': amount,
              'creditCard': cc_token,
        }
        response = self.api_post('charge/credit_card', cdata)
        return self.json_from_response(response)

    def refund_cc(self, cc_charge_id):
        cdata = {
            'id': cc_charge_id,
        }
        response = self.api_post('refund/credit_card', cdata)
        return self.json_from_response(response, expected_http_code=[202])
```

```python
>> charge_cc_response = client.charge_cc(cc_token=credit_card_token, amount=4500)
>> cc_charge_id = charge_cc_response['id']
print('charge_cc_response', charge_cc_response)
charge_cc_response {
    'id': '1rnfu463258eph0mlqli4105mjb85kut',
    'creditCard': 'ifmjd9rbe8peqdjh09pln702306nfniu',
    'thirdPartyId': None,
    'installments': 1,
    'tpagaFeeAmount': '868.00',
    'customer': 'gl01l74skk0po9afrjiaaclt0hr5acsh',
    'iacAmount': '0.00',
    'transactionInfo': {
        'authorizationCode': '723045',  # código de transacción del banco
        'status': 'authorized',  # Posibles valores: created, fraudulent,
                                 # settled, processor_declined, authorized,
                                 # voided
    },
    'netAmount': '3545.00',
    'tipAmount': '0.00',
    'reteIvaAmount': '0.00',
    'reteIcaAmount': '19.00',
    'paid': True,
    'reteRentaAmount': '68.00',
    'paymentTransaction': 'tta2hlk0e5n5dgr4kggm5j6vv8qoh3jP',
    'orderId': 'BRG-2',
    'description': 'One bridge in good condition.',
    'currency': 'COP',
    'errorMessage': 'Approved',
    'taxAmount': '0.00',
    'errorCode': '00',
    'amount': '4500.00'
}
```

Si el pago fue exitoso, el código de respuesta es `201` y en el JSON podemos ver que la llave `paid` es `True` y `amount` es igual al valor cobrado de la tarjeta.

En el caso cuando el código de respuesta es `402`, tendríamos fijarnos en los valores de `errorCode` y `errorMessage` para entender qué pasó con la transacción. Por ejemplo, el código de error `43` significa que el dueño de la tarjeta la reportó como robada, y `61` - que el monto máximo de tarjeta fue excedido.

En otros casos necesitaremos verificar que los datos que pasamos en la petición sean válidos y tengan todos los valores necesarios.

### Revertir el pago

Los bancos nos permiten revertir el pago dentro de 24 horas después de la transacción. Para hacerlo debemos mandar token de la transacción que queremos revertir al [refundCreditCardCharge](https://tpaga.co/docs/swaggers/v2#!/Credit_Card/refundCreditCardCharge) endpoint.

```python
>> refund_cc_response = client.refund_cc(cc_charge_id)
>> print('refund_cc_response', refund_cc_response)
refund_cc_response {
    'id': '1rnfu463258eph0mlqli4105mjb85kut',
    'creditCard': 'ifmjd9rbe8peqdjh09pln702306nfniu',
    'thirdPartyId': None,
    'installments': 1,
    'tpagaFeeAmount': '868.00',
    'customer': 'gl01l74skk0po9afrjiaaclt0hr5acsh',
    'iacAmount': '0.00',
    'transactionInfo': {'authorizationCode': '723045', 'status': 'voided'},
    'netAmount': '3545.00',
    'tipAmount': '0.00',
    'reteIvaAmount': '0.00',
    'reteIcaAmount': '19.00',
    'paid': False,
    'reteRentaAmount': '68.00',
    'paymentTransaction': 'tta2hlk0e5n5dgr4kggm5j6vv8qoh3jP',
    'orderId': 'BRG-2',
    'description': 'One bridge in good condition.',
    'currency': 'COP',
    'errorMessage': 'Approved',
    'taxAmount': '0.00',
    'errorCode': '00',
    'amount': '4500.00'
}
```

El JSON que nos devolvió Tpaga `transactionInfo.status` aparece como `voided` y el valor de `paid` ahora es falso:

![](/images/tpaga_dashboard_refund.png)

Enlaces:

* Documentación de Tpaga: [tpaga.co/docs/swaggers/v2](https://tpaga.co/docs/swaggers/v2)
* Tpaga FAQ: [tpaga.zendesk.com/hc/es](https://tpaga.zendesk.com/hc/es)
* Documentación para la librería [Requests: HTTP for Humans](http://docs.python-requests.org/en/master/)
* Un ejemplo de integración con Tpaga usando PHP/JavaScript: [Tpaga/tpaga-php-example-backend](https://github.com/Tpaga/tpaga-php-example-backend/)

Muchas gracias a [@jerojasro](https://twitter.com/jerojasro) por su ayuda y paciencia.

