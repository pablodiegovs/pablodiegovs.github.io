---
layout: post
title:  "Utilizando APIs de Cisco ISE"
description: Script en Python para habilitar usuarios masivos con fechas de expiración utilizando las API de Cisco ISE.
date:   2022-02-12 21:03:36 +0530
categories: Python3 API ISE
---
Con el propósito de optimizar los tiempos operativos, un problema habitual en la administracion de un AD sobre un Cisco ISE es la cantidad de solicitudes día a día de habilitación de usuarios con fechas límite de expiración, este proceso de configuración a través de la interfaz web (GUI) de un ISE es algo morosa y la configuración es uno a uno. Por lo tanto, el script trata de optimizar un poco el proceso habilitando con fechas de expiración grupos de usuarios.

![texture theme preview](https://github.com/pablodiegovs/pablodiegovs.github.io/raw/main/assets/images/ISE-Phyton.jpg)

Es importante que previo al despliegue de los scripts, validar la versión de Cisco ISE a utilizar y las APIs soportadas en el mismo. El Cisco ISE en general nos presenta dos APIs disponibles: ERS APIs y OPEN APIs. Más información del mismo les dejo el link de la página de [Developer-Cisco][dev-cisco]

Utilizaremos las siguientes librerías, "request" para ejecutar las funciones Rest, "json" para especificar el tipo de formato de datos utilizado en la comunicación con la API del ISE, y "getpass" para ocultar los caracteres del password al momento de ingresar los credenciales.

```javascript

import requests
import json
import getpass
```

Solicitaremos las credenciales, para utilizar un autenticación de tipo BasicAuth para tener acceso a la APIs del Cisco ISE. El usuario por utilizar debe tener permisos de administración asociados al grupo ERS API que utilizamos en el ejemplo.

```javascript

login=input("Ingrese su usuario login de administracion: ")
pswd=getpass.getpass(prompt="Ingrese su password: ")
basicauth=(login,pswd)
```

La primera funcion a utilizar nos valida el estado de un usuario en específico y el tiempo permitido de uso. 

```javascript

def user_state(user):
    url = "https://<ise-ipaddress>:<port>/ers/config/internaluser/name/{}".format(user)
    payload={}
    headers = {
        'Accept': 'application/json'
    }
    response = requests.request("GET", url, auth=basicauth, headers=headers, data=payload, verify=False)
    dataresp = json.loads(response.text)
    if dataresp['InternalUser']['enabled']==True and dataresp['InternalUser']['expiryDateEnabled']==True:
      print("  - " + user + " se encuetra activo hasta la fecha " + dataresp['InternalUser']['expiryDate'] + " \n")
    elif dataresp['InternalUser']['enabled']==True and dataresp['InternalUser']['expiryDateEnabled']==False:
      print("  - " + user + " se encuentra activo sin fecha de expiracion \n")
    elif dataresp['InternalUser']['enabled']==False:
      print("  - " + user + " se encuentra inactivo \n")

```

La segunda funcion obtenemos el o los grupos de permisos de autorización donde se encuentra habilitado un usuario en específico.

```javascript
def get_uig(user):
  url = "https://<ise-ipaddress>:<port>/ers/config/internaluser/name/{}".format(user)
  payload={}
  headers = {
    'Accept': 'application/json'
  }
  response = requests.request("GET", url, auth=basicauth, headers=headers, data=payload, verify=False)
  dataresp=json.loads(response.text)
  uig=dataresp["InternalUser"]["identityGroups"]
  return (uig)
```

Posteriormente, especificamos una función para habilitar con fechas de expiración un usuario en específico, la función tendrá de parámetros: el usuario, la fecha de expiración y el/los user-groups asociados al usuario en específico. Todo esto enviaremos a través de una función PUT hacia la API del ISE.

```javascript
def enable_user(user,date,uig):
    url = "https://<ise-ipaddress>:<port>/ers/config/internaluser/name/{}".format(user)
    payload="{\n    \"InternalUser\": {\n        \"enabled\": true,\n        \"expiryDateEnabled\": true,\n        \"expiryDate\": \"%s\",\n        \"identityGroups\": \"%s\",\n        \"link\": {\n            \"rel\": \"self\",\n            \"href\": \"https://<ise-ipaddress>:<port>/ers/config/internaluser/name/%s\",\n            \"type\": \"application/json\"\n        }\n    }\n}" % (date, uig, user)
    headers = {
      'Accept': 'application/json',
      'Content-Type': 'application/json',
    }
    response = requests.request("PUT", url, auth=basicauth, headers=headers, data=payload, verify=False)
    print("Usuario: ",user)
    print(response.text)
```

Utilizamos en este caso un archivo de texto con la lista de usuarios por modificar "usuarios.txt", que enviaremos esos datos a una lista para una mejor manipulación.

```javascript
users=[]
file=open('usuarios.txt','r')
for item in file:
  item=item.strip()
  users.append(item)
file.close()
```

Inicialmente, procedemos a validar el estado actual de los usuarios en la lista utilizando la función "user_state".

```javascript
print("\nEstado actual de los siguientes usuarios: ", users)
print("\n")
for user in users:
    user_state(user)
```

Por último, si no existen inconvenientes procedemos a habilitar los usuarios de la lista utilizando la función "enable_user".

```javascript
counter=0
user_error=""
print("Los siguientes usuarios se habilitaran: ", users)
date=input("Ingrese fecha de expiracion (formato: AAAA-MM-DD): ")
for user in users:
  try:
    uig=get_uig(user)
    enable_user(user,date,uig)  
    counter+=1
  except:
    user_error=user_error + "- " + user + "\n"
print("Total usuarios habilitados :", counter)
print("Usuarios con error: \n", user_error)
```

Las salidas nos muestran la cantidad total de usuarios habilitados, y la cantidad de usuarios que presentaron algún tipo de error en el proceso de habilitación.

Espero que les sirva de ayuda o referencia.

[dev-cisco]: https://developer.cisco.com/docs/identity-services-engine/v1/#!cisco-ise-api-framework/introduction
