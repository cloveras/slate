---
title: Vipps Invoice API
language_tabs:
  - shell: Shell
  - http: HTTP
  - javascript: JavaScript
  - javascript--nodejs: Node.JS
  - ruby: Ruby
  - python: Python
  - java: Java
  - go: Go
toc_footers: []
includes: []
search: true
highlight_theme: darkula
headingLevel: 2

---

<h1 id="Vipps-Invoice-API">Vipps Invoice API v0.2.8</h1>

> Scroll down for code samples, example requests and responses. Select a language for code samples from the tabs above or the mobile navigation menu.

# Welcome
The api described by this document is currently under _very active_ development. Vipps has opened this API for selected partners, you, and we are happy to receive your feedback.<br/>
Since this product is under development we want to point out that the API is still subject to change and **breaking changes have to be expected**.
We nevertheless hope that early access to the API helps you and us to polish the required functionality and to end up with a nice and usable workflow.
Since 0.1.8 we have done a major redesign, but wanted to share the Swagger definition with you as soon as possible for input. This means that none of the described endpoints are implemented. **Meaning that any testing of the endpoints is currently not possible**. We will update the Swagger continously as endpoints are implemented.
## ISP and IPP?
Throughout this specification we use the terms ISP and IPP to categorize two groups of users.
* ISP, the invoice service providers, are all actors who submit invoices.
  Either for themselves or on behalf of their clients.
* IPP, the invoice payment providers, are all actors who handle invoices
  for the invoice recipients and execute payments, e.g. banks, the vipps
  app.

In this test environment all endpoints are available for everyone. When we reach production a subset of the endpoints will be made available, based on the role you have assigned. ISP, IPP or both.
# Core Functionality
## Send, receive and pay invoices.
Since version 0.1.8, we have changed the workflow. Previously, we followed the classical approach of working with batches. However, we wanted to simplify the workflow and be more explicit and have changed that approach in favour of a simple \"submit a single invoice\" approach. This means that invoices have to be posted one by one, each invoice in a separate http call.
Although it might seem to be an \"inefficient\" approach, we believe that the benefits far outweight performance considerations. In addition, our tests have shown that the performance is well on par with a batch approach, if multiple threads are used to push invoices to our system.
The main benefits are that it becomes much easier to ensure and verify idempotency of the endpoint. We insert received invoices _synchronously_. In case of problems (5xx return codes or network issues), it is possible to just repeatedly submit invoices until a 2xx status code is returned. We guarantee that any invoice is exactly inserted once.
The validation of the invoice will still be an asynchronous process since we have no possibibility to guarantee or even estimate the response times for all required validation and risk checks we have to perform.
Therefore, the invoice will be in a \"pending\" state once it is inserted. In this state the invoice will not be visible to anyone. First, when all validation steps are passed, the invoice will be shown to the recipients.
ISPs who provide the invoice will have to monitor the state. We provide two ways to do that.
## Managing/Paying Invoices
Invoice payment providers will mainly require to work with the resource `invoices` directly. The typical use case will be to fetch all invoices for a use/recipient, identified by his national identification number. This is provided by `GET: /invoices`.
If a user approves an invoice, the payment provider has to mark this individual invoice as processed so that the invoice is not displayed as an open invoice in other services.
## Debt Collection
All invoices contain information about the _invoice type_, i.e. whether it is a regular invoice, reminder or other. This enables payment providers to filter the allowed payment methods according to Norwegian inkasso rules.

Base URLs:

* <a href="https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1">https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1</a>

* <a href="http://vippsinvoice-test.westeurope.cloudapp.azure.com/v1">http://vippsinvoice-test.westeurope.cloudapp.azure.com/v1</a>

<h1 id="Vipps-Invoice-API-IPP">IPP</h1>

Invoice Payment Provider. Endpoints for those who display and process invoices on behalf of recipients.

## get__invoices

> Code samples

```shell
# You can also use wget
curl -X GET https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices \
  -H 'Accept: application/json' \
  -H 'vippsinvoice-token: string'

```

```http
GET https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com

Accept: application/json
vippsinvoice-token: string

```

```javascript
var headers = {
  'Accept':'application/json',
  'vippsinvoice-token':'string'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices',
  method: 'get',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');

const headers = {
  'Accept':'application/json',
  'vippsinvoice-token':'string'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json',
  'vippsinvoice-token' => 'string'
}

result = RestClient.get 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json',
  'vippsinvoice-token': 'string'
}

r = requests.get('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
        "vippsinvoice-token": []string{"string"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /invoices`

*List invoices*

List all invoices for a user identified by the national identification number. May optionally be filtered by the described query parameters.
* The invoices are not guaranteed to be sorted.
* Currently the number of returned invoices is not limited. I.e. there
  is currently no paging.

<h3 id="get__invoices-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|vippsinvoice-token|header|string|true|Recipient token. Issued by vipps.|
|state|query|string|false|**Not yet implemented!** Filter invoices by state. Default is "open", which returns both "new" and "fetched" invoices.|

#### Enumerated Values

|Parameter|Value|
|---|---|
|state|all|
|state|open|
|state|pending|
|state|new|
|state|fetched|
|state|processed|
|state|deleted|
|state|rejected|

> Example responses

> 200 Response

```json
[
  {
    "invoiceId": "orgno-no.999999999.20180001",
    "paymentInformation": {
      "type": "kid",
      "value": "1234567890128",
      "account": "12345678903"
    },
    "invoiceType": "invoice",
    "due": "2023-03-13T16:00:00+01:00",
    "amount": 25043,
    "minAmount": 25043,
    "subject": "Bompasseringer",
    "issuerName": "Lister Bompengeselskap",
    "recipient": {
      "nin": "12045678903",
      "mobile": 4740040040
    },
    "commercialInvoice": [
      {
        "mimeType": "application/pdf",
        "url": "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/comminv.pdf"
      }
    ],
    "attachmentUrls": [
      "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/vedlegg.pdf"
    ],
    "issuerIconUrl": "https://www.example.com/logos/lister.png",
    "status": {
      "lastModified": "2018-07-07T13:30:51Z",
      "state": "pending",
      "due": "2018-07-07T13:30:51Z",
      "amount": 0
    }
  }
]
```

<h3 id="get__invoices-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad request|[Error](#schemaerror)|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Invoice not found|None|
|500|[Internal Server Error](https://tools.ietf.org/html/rfc7231#section-6.6.1)|Internal server error|Inline|

<h3 id="get__invoices-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[invoiceOut](#schemainvoiceout)]|false|none|none|
|» invoiceId|string|false|none|none|
|» paymentInformation|object|false|none|none|
|»» type|string|false|none|none|
|»» value|string|false|none|none|
|»» account|string|false|none|none|
|» invoiceType|string|false|none|none|
|» due|string(date-time)|false|none|When an invoice is due|
|» amount|integer|false|none|Amount by lowest subdivision(øre)|
|» minAmount|integer|false|none|Minimum allowed amount to pay by lowest subdivision(øre). If this is set, it indicates that this invoice can be paid with any amount between andincluding minAmount and amount.|
|» subject|string|false|none|A textual subject for the receipient of the invoice|
|» issuerName|string|false|none|Organization name of issuer|
|» recipient|object|false|none|none|
|»» nin|string|false|none|National Identification Number (Only if given when requesting recipientToken)|
|»» mobile|integer|false|none|Mobile number formatted as MSISDN (Only if given when requesting recipientToken).|
|» commercialInvoice|[object]|false|none|The issuer is advised to provide the commercial invoice in multiple MIME types if possible. This will make it easier for the user to read the invoice.|
|»» mimeType|string|false|none|MIME type of commercial invoice document|
|»» url|string|false|none|URL of commercial invoice|
|» attachmentUrls|[string]|false|none|URLs to invoice attachments. Max 100 attachments.|
|» issuerIconUrl|string|false|none|URL to invoice issuer's logo|
|» status|object|false|none|none|
|»» lastModified|string(date-time)|false|none|none|
|»» state|string|false|none|Status of the invoice. Fetched means at least one payment provider has fetched the invoice. Processed means a payment provider has marked the invoice as processed.|
|»» due|string(date-time)|false|none|When the user has set the payment to be due|
|»» amount|integer|false|none|Amount the user has set to be paid by lowest subdivision(øre)|

#### Enumerated Values

|Property|Value|
|---|---|
|type|kid|
|type|message|
|invoiceType|invoice|
|invoiceType|paymentReminder|
|invoiceType|debtCollectionWarning|
|invoiceType|debtCollectionNotice|
|invoiceType|noticeOfLegalProceedings|
|invoiceType|debtCollectionReminder|
|invoiceType|creditNote|
|state|pending|
|state|new|
|state|fetched|
|state|processed|
|state|deleted|
|state|rejected|

Status Code **500**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» Error|string|false|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## InvoiceCount

<a id="opIdInvoiceCount"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count \
  -H 'Accept: application/json' \
  -H 'vippsinvoice-token: string'

```

```http
GET https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com

Accept: application/json
vippsinvoice-token: string

```

```javascript
var headers = {
  'Accept':'application/json',
  'vippsinvoice-token':'string'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count',
  method: 'get',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');

const headers = {
  'Accept':'application/json',
  'vippsinvoice-token':'string'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json',
  'vippsinvoice-token' => 'string'
}

result = RestClient.get 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json',
  'vippsinvoice-token': 'string'
}

r = requests.get('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
        "vippsinvoice-token": []string{"string"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/count", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /invoices/count`

*Count invoices for a user*

Returns the number of open invoices for a user.

<h3 id="invoicecount-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|vippsinvoice-token|header|string|true|Recipient token. Issued by vipps.|
|state|query|string|false|**Not yet implemented!** Filter invoices by state. Default is "open", which returns both "new" and "fetched" invoices.|

#### Enumerated Values

|Parameter|Value|
|---|---|
|state|all|
|state|open|
|state|new|
|state|fetched|
|state|processed|
|state|deleted|

> Example responses

> 200 Response

```json
42
```

<h3 id="invoicecount-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|integer|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad request|[Error](#schemaerror)|
|500|[Internal Server Error](https://tools.ietf.org/html/rfc7231#section-6.6.1)|Internal server error|Inline|

<h3 id="invoicecount-responseschema">Response Schema</h3>

Status Code **500**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» Error|string|false|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## get__invoices_{invoiceId}

> Code samples

```shell
# You can also use wget
curl -X GET https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId} \
  -H 'Accept: application/json'

```

```http
GET https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId} HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com

Accept: application/json

```

```javascript
var headers = {
  'Accept':'application/json'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
  method: 'get',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');

const headers = {
  'Accept':'application/json'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /invoices/{invoiceId}`

*Get a single invoice*

Returns a single invoice identified by its unique id. This is used to verify the state of an invoice, e.g. if it has been validated and now is available for recipients.

<h3 id="get__invoices_{invoiceid}-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|invoiceId|path|string|true|The unique invoice id.|

> Example responses

> 200 Response

```json
{
  "invoiceId": "orgno-no.999999999.20180001",
  "paymentInformation": {
    "type": "kid",
    "value": "1234567890128",
    "account": "12345678903"
  },
  "invoiceType": "invoice",
  "due": "2023-03-13T16:00:00+01:00",
  "amount": 25043,
  "minAmount": 25043,
  "subject": "Bompasseringer",
  "issuerName": "Lister Bompengeselskap",
  "recipient": {
    "nin": "12045678903",
    "mobile": 4740040040
  },
  "commercialInvoice": [
    {
      "mimeType": "application/pdf",
      "url": "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/comminv.pdf"
    }
  ],
  "attachmentUrls": [
    "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/vedlegg.pdf"
  ],
  "issuerIconUrl": "https://www.example.com/logos/lister.png",
  "status": {
    "lastModified": "2018-07-07T13:30:51Z",
    "state": "pending",
    "due": "2018-07-07T13:30:51Z",
    "amount": 0
  }
}
```

<h3 id="get__invoices_{invoiceid}-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[invoiceOut](#schemainvoiceout)|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Invoice not found|None|

<aside class="success">
This operation does not require authentication
</aside>

## put__invoices_{invoiceId}_status_new

> Code samples

```shell
# You can also use wget
curl -X PUT https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new \
  -H 'Accept: application/json'

```

```http
PUT https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com

Accept: application/json

```

```javascript
var headers = {
  'Accept':'application/json'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new',
  method: 'put',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');

const headers = {
  'Accept':'application/json'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new',
{
  method: 'PUT',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.put 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.put('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("PUT");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("PUT", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/new", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`PUT /invoices/{invoiceId}/status/new`

*Mark the invoice as new*

Mark an invoice as new. For example if a payment fails, the status can be set to new so that the invoice become visible for the recipient again.

<h3 id="put__invoices_{invoiceid}_status_new-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|invoiceId|path|string|true|The unique invoice id.|

> Example responses

> 400 Response

```json
{
  "error": "error message ..."
}
```

<h3 id="put__invoices_{invoiceid}_status_new-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|None|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad request|[Error](#schemaerror)|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Invoice not found|None|

<aside class="success">
This operation does not require authentication
</aside>

## put__invoices_{invoiceId}_status_processed

> Code samples

```shell
# You can also use wget
curl -X PUT https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
PUT https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com
Content-Type: application/json
Accept: application/json

```

```javascript
var headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed',
  method: 'put',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');
const inputBody = '{
  "due": "2023-03-13T16:00:00+01:00",
  "amount": 25043
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed',
{
  method: 'PUT',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.put 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.put('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("PUT");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("PUT", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}/status/processed", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`PUT /invoices/{invoiceId}/status/processed`

*Mark the invoice as processed.*

Mark an invoice as processed. 

> Body parameter

```json
{
  "due": "2023-03-13T16:00:00+01:00",
  "amount": 25043
}
```

<h3 id="put__invoices_{invoiceid}_status_processed-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|invoiceId|path|string|true|The unique invoice id.|
|body|body|object|true|none|
|» due|body|string(date-time)|false|When the user has set the payment to be due|
|» amount|body|integer|false|Amount the user has set to be paid by lowest subdivision(øre)|

> Example responses

> 400 Response

```json
{
  "error": "error message ..."
}
```

<h3 id="put__invoices_{invoiceid}_status_processed-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|201|[Created](https://tools.ietf.org/html/rfc7231#section-6.3.2)|OK|None|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad request|[Error](#schemaerror)|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Invoice not found|None|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="Vipps-Invoice-API-ISP">ISP</h1>

Invoice Service Provider. Endpoints for those who send invoices on behalf of issuers.

## post__recipients_tokens_msisdn

> Code samples

```shell
# You can also use wget
curl -X POST https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com
Content-Type: application/json
Accept: application/json

```

```javascript
var headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn',
  method: 'post',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');
const inputBody = '{
  "msisdn": 4748831893
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/msisdn", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /recipients/tokens/msisdn`

*Request a recipient token*

Request a `recipientToken` by providing the recipients mobile number in MSISDN format.

> Body parameter

```json
{
  "msisdn": 4748831893
}
```

<h3 id="post__recipients_tokens_msisdn-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|true|none|
|» msisdn|body|integer|false|mobile phone number in MSISDN format|

> Example responses

> 200 Response

```json
{
  "recipientToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9"
}
```

<h3 id="post__recipients_tokens_msisdn-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Recipient token|[recipientToken](#schemarecipienttoken)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request|[Error](#schemaerror)|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Recipient not Found|None|
|500|[Internal Server Error](https://tools.ietf.org/html/rfc7231#section-6.6.1)|Internal Server Error|[Error](#schemaerror)|

<aside class="success">
This operation does not require authentication
</aside>

## post__recipients_tokens_nin-no

> Code samples

```shell
# You can also use wget
curl -X POST https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com
Content-Type: application/json
Accept: application/json

```

```javascript
var headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no',
  method: 'post',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');
const inputBody = '{
  "nin-no": "03086423412"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/recipients/tokens/nin-no", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /recipients/tokens/nin-no`

*Request a recipient token*

Request a `recipientToken` by providing the recipients norwegian national identification number.

> Body parameter

```json
{
  "nin-no": "03086423412"
}
```

<h3 id="post__recipients_tokens_nin-no-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|true|none|
|» nin-no|body|integer|false|norwegian national identification number (fødselsnummer)|

> Example responses

> 200 Response

```json
{
  "recipientToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9"
}
```

<h3 id="post__recipients_tokens_nin-no-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Recipient found.|[recipientToken](#schemarecipienttoken)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request|[Error](#schemaerror)|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Recipient not Found|None|
|500|[Internal Server Error](https://tools.ietf.org/html/rfc7231#section-6.6.1)|Internal Server Error|[Error](#schemaerror)|

<aside class="success">
This operation does not require authentication
</aside>

## put__invoices_{invoiceId}

> Code samples

```shell
# You can also use wget
curl -X PUT https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId} \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
PUT https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId} HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com
Content-Type: application/json
Accept: application/json

```

```javascript
var headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
  method: 'put',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');
const inputBody = '{
  "recipientToken": "string",
  "paymentInformation": {
    "type": "kid",
    "value": "1234567890128",
    "account": "12345678903"
  },
  "invoiceType": "invoice",
  "due": "2023-03-13T16:00:00+01:00",
  "amount": 25043,
  "minAmount": 25043,
  "subject": "Bompasseringer",
  "issuerName": "Lister Bompengeselskap",
  "commercialInvoice": [
    {
      "mimeType": "application/pdf",
      "url": "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/comminv.pdf"
    }
  ],
  "attachmentUrls": [
    "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/vedlegg.pdf"
  ],
  "issuerIconUrl": "https://www.example.com/logos/lister.png"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
{
  method: 'PUT',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.put 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.put('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("PUT");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("PUT", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`PUT /invoices/{invoiceId}`

*Send an invoice*

This endpoint adds the provided invoice to our system. To submit invoices into our system this endpoint has to be used.
We will accept any invoice as long as the body is well formed json. Any validation errors of the invoice are picked up by workers in the background and are reported back to the caller as a response body of the feed or GET:/invoices/{invoiceId}.

## Idempotency
The endpoint is idempotent. If an invoice with the same unique identifiers (organisation number and invoice reference) is submitted and the invoice exists already in the system, it is rejected. Invoices cannot be changed.

> Body parameter

```json
{
  "recipientToken": "string",
  "paymentInformation": {
    "type": "kid",
    "value": "1234567890128",
    "account": "12345678903"
  },
  "invoiceType": "invoice",
  "due": "2023-03-13T16:00:00+01:00",
  "amount": 25043,
  "minAmount": 25043,
  "subject": "Bompasseringer",
  "issuerName": "Lister Bompengeselskap",
  "commercialInvoice": [
    {
      "mimeType": "application/pdf",
      "url": "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/comminv.pdf"
    }
  ],
  "attachmentUrls": [
    "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/vedlegg.pdf"
  ],
  "issuerIconUrl": "https://www.example.com/logos/lister.png"
}
```

<h3 id="put__invoices_{invoiceid}-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|invoiceId|path|string|true|The `invoiceId` must be constructed as `orgno-no.{issuerOrgno}.{invoiceRef}` where {invoiceRef} is a URL-safe reference that is unique for each issuer.|
|body|body|[invoiceIn](#schemainvoicein)|true|A single invoice.|

> Example responses

> 400 Response

```json
{
  "error": "Invalid JSON data"
}
```

<h3 id="put__invoices_{invoiceid}-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|An invoice was successfully created.|None|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad request|Inline|
|409|[Conflict](https://tools.ietf.org/html/rfc7231#section-6.5.8)|Conflict|Inline|
|500|[Internal Server Error](https://tools.ietf.org/html/rfc7231#section-6.6.1)|Internal server error|None|

<h3 id="put__invoices_{invoiceid}-responseschema">Response Schema</h3>

Status Code **400**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» error|string|false|none|none|

Status Code **409**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» error|string|false|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## processInvoice

<a id="opIdprocessInvoice"></a>

> Code samples

```shell
# You can also use wget
curl -X DELETE https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId} \
  -H 'Accept: application/json'

```

```http
DELETE https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId} HTTP/1.1
Host: vippsinvoice-test.westeurope.cloudapp.azure.com

Accept: application/json

```

```javascript
var headers = {
  'Accept':'application/json'

};

$.ajax({
  url: 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
  method: 'delete',

  headers: headers,
  success: function(data) {
    console.log(JSON.stringify(data));
  }
})

```

```javascript--nodejs
const request = require('node-fetch');

const headers = {
  'Accept':'application/json'

};

fetch('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
{
  method: 'DELETE',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.delete 'https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}', params={

}, headers = headers)

print r.json()

```

```java
URL obj = new URL("https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("DELETE");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
        
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("DELETE", "https://vippsinvoice-test.westeurope.cloudapp.azure.com/v1/invoices/{invoiceId}", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`DELETE /invoices/{invoiceId}`

*Mark the invoice as deleted*

Mark an invoice as deleted. The invoice is not actually deleted, but marked as "deleted by recipient". Should be used in cases of incorrect invoices that no longer should be prosessed or viewable.
*This endpoint is idempotent and may be called multiple times*.

<h3 id="processinvoice-parameters">Parameters</h3>

|Parameter|In|Type|Required|Description|
|---|---|---|---|---|
|invoiceId|path|string|true|The unique invoice id.|

> Example responses

> 400 Response

```json
{
  "error": "error message ..."
}
```

<h3 id="processinvoice-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|204|[No Content](https://tools.ietf.org/html/rfc7231#section-6.3.5)|OK|None|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad request|[Error](#schemaerror)|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Invoice not found|None|

<aside class="success">
This operation does not require authentication
</aside>

# Schemas

<h2 id="tocSrecipienttoken">recipientToken</h2>

<a id="schemarecipienttoken"></a>

```json
{
  "recipientToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|recipientToken|string|false|none|none|

<h2 id="tocSinvoicein">invoiceIn</h2>

<a id="schemainvoicein"></a>

```json
{
  "recipientToken": "string",
  "paymentInformation": {
    "type": "kid",
    "value": "1234567890128",
    "account": "12345678903"
  },
  "invoiceType": "invoice",
  "due": "2023-03-13T16:00:00+01:00",
  "amount": 25043,
  "minAmount": 25043,
  "subject": "Bompasseringer",
  "issuerName": "Lister Bompengeselskap",
  "commercialInvoice": [
    {
      "mimeType": "application/pdf",
      "url": "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/comminv.pdf"
    }
  ],
  "attachmentUrls": [
    "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/vedlegg.pdf"
  ],
  "issuerIconUrl": "https://www.example.com/logos/lister.png"
}

```

*Invoice document which is sent to our servers.*

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|recipientToken|string|true|none|token received is obtained by calling the endpoint to resolve a recipient|
|paymentInformation|object|true|none|none|
|» type|string|true|none|none|
|» value|string|true|none|none|
|» account|string|true|none|none|
|invoiceType|string|true|none|none|
|due|string(date-time)|true|none|When an invoice is due|
|amount|integer|true|none|Amount by lowest subdivision(øre)|
|minAmount|integer|false|none|Minimum allowed amount to pay by lowest subdivision(øre)|
|subject|string|true|none|A textual subject for the receipient of the invoice|
|issuerName|string|false|none|Organization name of issuer|
|commercialInvoice|[object]|false|none|The issuer is advised to provide the commercial invoice in multiple MIME types if possible. This will make it easier for the user to read the invoice.|
|» mimeType|string|false|none|MIME type of commercial invoice document|
|» url|string|false|none|URL of commercial invoice|
|attachmentUrls|[string]|false|none|URLs to invoice attachments. Max 100 attachments.|
|issuerIconUrl|string|false|none|URL to invoice issuer's logo|

#### Enumerated Values

|Property|Value|
|---|---|
|type|kid|
|type|message|
|invoiceType|invoice|
|invoiceType|paymentReminder|
|invoiceType|debtCollectionWarning|
|invoiceType|debtCollectionNotice|
|invoiceType|noticeOfLegalProceedings|
|invoiceType|debtCollectionReminder|
|invoiceType|creditNote|

<h2 id="tocSinvoiceout">invoiceOut</h2>

<a id="schemainvoiceout"></a>

```json
{
  "invoiceId": "orgno-no.999999999.20180001",
  "paymentInformation": {
    "type": "kid",
    "value": "1234567890128",
    "account": "12345678903"
  },
  "invoiceType": "invoice",
  "due": "2023-03-13T16:00:00+01:00",
  "amount": 25043,
  "minAmount": 25043,
  "subject": "Bompasseringer",
  "issuerName": "Lister Bompengeselskap",
  "recipient": {
    "nin": "12045678903",
    "mobile": 4740040040
  },
  "commercialInvoice": [
    {
      "mimeType": "application/pdf",
      "url": "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/comminv.pdf"
    }
  ],
  "attachmentUrls": [
    "https://www.example.com/08fd5360-e218-4658-894f-4f37649e7df7/vedlegg.pdf"
  ],
  "issuerIconUrl": "https://www.example.com/logos/lister.png",
  "status": {
    "lastModified": "2018-07-07T13:30:51Z",
    "state": "pending",
    "due": "2018-07-07T13:30:51Z",
    "amount": 0
  }
}

```

*Invoice document which is returned from our servers.*

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|invoiceId|string|false|none|none|
|paymentInformation|object|false|none|none|
|» type|string|false|none|none|
|» value|string|false|none|none|
|» account|string|false|none|none|
|invoiceType|string|false|none|none|
|due|string(date-time)|false|none|When an invoice is due|
|amount|integer|false|none|Amount by lowest subdivision(øre)|
|minAmount|integer|false|none|Minimum allowed amount to pay by lowest subdivision(øre). If this is set, it indicates that this invoice can be paid with any amount between andincluding minAmount and amount.|
|subject|string|false|none|A textual subject for the receipient of the invoice|
|issuerName|string|false|none|Organization name of issuer|
|recipient|object|false|none|none|
|» nin|string|false|none|National Identification Number (Only if given when requesting recipientToken)|
|» mobile|integer|false|none|Mobile number formatted as MSISDN (Only if given when requesting recipientToken).|
|commercialInvoice|[object]|false|none|The issuer is advised to provide the commercial invoice in multiple MIME types if possible. This will make it easier for the user to read the invoice.|
|» mimeType|string|false|none|MIME type of commercial invoice document|
|» url|string|false|none|URL of commercial invoice|
|attachmentUrls|[string]|false|none|URLs to invoice attachments. Max 100 attachments.|
|issuerIconUrl|string|false|none|URL to invoice issuer's logo|
|status|object|false|none|none|
|» lastModified|string(date-time)|false|none|none|
|» state|string|false|none|Status of the invoice. Fetched means at least one payment provider has fetched the invoice. Processed means a payment provider has marked the invoice as processed.|
|» due|string(date-time)|false|none|When the user has set the payment to be due|
|» amount|integer|false|none|Amount the user has set to be paid by lowest subdivision(øre)|

#### Enumerated Values

|Property|Value|
|---|---|
|type|kid|
|type|message|
|invoiceType|invoice|
|invoiceType|paymentReminder|
|invoiceType|debtCollectionWarning|
|invoiceType|debtCollectionNotice|
|invoiceType|noticeOfLegalProceedings|
|invoiceType|debtCollectionReminder|
|invoiceType|creditNote|
|state|pending|
|state|new|
|state|fetched|
|state|processed|
|state|deleted|
|state|rejected|

<h2 id="tocSerrors">Errors</h2>

<a id="schemaerrors"></a>

```json
[
  {
    "error": "error message ..."
  }
]

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[Error](#schemaerror)]|false|none|none|

<h2 id="tocSerror">Error</h2>

<a id="schemaerror"></a>

```json
{
  "error": "error message ..."
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|error|string|false|none|none|

