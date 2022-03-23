# RESTful API
id: 202203032121
__
    API — Application Programming Interface
	REST - REpresentational State Transfer
	API, для работы с IPAM/DCIM-системой.
    Структура сообщений HTTP
        Стартовая строка
		Req:
			Метод (GET, POST, PUT, DELETE)
			URI
			HTTP_VERSION (REST - 1.1.)
		Resp:
			HTTP_VERSION — версия HTTP (1.1).
			STATUS_CODE — (2XX, 3XX, 4XX, 5XX)
			REASON_PHRASE (OK, NOT FOUND, BAD GATEWAY, ...)
        Заголовки
		Req:
			Host: name.domain:{port}
			User-Agent: curl/7.54.0
			Accept: application/json; indent=4
		Resp:
			Server: nginx/1.14.0 (Ubuntu)
			Date: Tue, 21 Jan 2020 15:14:22 GMT
			Content-Type: application/json
			Content-Length: 1638
			Connection: keep-alive
        Тело HTTP-сообщения (JSON, XML, HTML, ...)
    Методы (глаголы) (CRUD — Create, Read, Update, Delete.)
        HTTP GET (?param1_name=parametr1_value&param2_name=parametr2_value)
        HTTP POST (-H Autorization: TOKEN XXXX... -d params) 201 Created (if success)
        HTTP PUT (full change)
        HTTP PATCH (part change)
        HTTP DELETE
    Способы работы с RESTful API
        CURL
        Postman (for import all endpoints to collections : Import (File->Import) - Import From Link : URL netbox.domain:{port}/api/docs/?format=openapi.)
        Python+Requests (for automate using api)
        SDK Pynebtbox (for use all endpoints as objects.. to automate url generation)
        SWAGGER (framework with full man and use and test cases)
    Критика REST и альтернативы
		gRPC
__

***Tags***
#pro #it

***Status***
#pro

***Type*** 
#it #job

***Prior***
#important #urgent #unimportant #unurgent #hard #simple #quick #slow #effective #useless #needs

__

***Source***
[[https://linkmeup.ru/blog/1266/]]

## Description


__

***Goal***

***Result***
__

***Mnemonika***
__

### Keys: 
[[REST]], [[RESTful]], [[API]], [[IPAM]], [[DCIM]]

