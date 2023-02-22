---
layout: post
title:  "Maintainability and Modularity in Network Automation"
description: Atributos de mantenibilidad, modularidad y programación orientada a objetos en automatización de redes
date:   2023-02-22 18:52:36
categories: Python3 NetworkAutomation QualityAttributes
---
En mi proceso de estudio de DevNet, una de las consideraciones más importantes en el proceso de coding de aplicaciones de automatización de redes, como en cualquier proceso de desarrollo de software, es la mantenibilidad; y uno de los factores que más contribuyen a tener un diseño efectivo de mantenibilidad es la modularidad. La modularidad es otro atributo de calidad o requerimiento no funcional, que consiste en separar una arquitectura en pequeños bloques de código con funciones e interacciones claras entre módulos.

![texture theme preview](https://github.com/pablodiegovs/pablodiegovs.github.io/raw/main/assets/images/mantenibilidad.jpg)


Desde la perspectiva de networking, podría yo relacionar conceptos de mantenibilidad y modularidad, a procesos de segmentación y microsegmentación para poder hacer más llevadera la gestión de una infraestructura de red y aislar posibles fallas que surjan en sectores sin necesidad que afecte al funcionamiento de otros sectores (principio básico de segmentación en diseño de redes).

Volviendo al tema de tratar de entender los conceptos de Mantenibilidad y Modularidad en desarrollo de software, y plasmarlo en la construcción en códigos de automatización de redes, aterrizo en una recomendación de mejores prácticas de mantenibilidad conocido como "SOLID principle". Pero para profundizar en estas best-pratices, necesite profundizar los conceptos de diseño de Programación Orientada a Objetos (POO) en los que está basado este principio de diseño.

Entendí por POO a una manera de realizar código de forma estructurada, ordenada, modular y sobre todo fácil de modificar y mantener. La POO en si consiste en realizar modelos de abstracciones reutilizables para distintos objetos que vayamos a utilizar en el proceso de construcción de nuestro código. Podemos definir como objeto algo visible (ej. un switch, un router) o algo conceptual (una conexión SSH). Un objeto tiene atributos y métodos. Definimos como atributo a las características de un objeto en particular, en el caso de un Switch podemos definir algunos ejemplos: hostname, vendor, ip_address, port_numbers. Un método define lo que el objeto puede hacer o un acción que puede realizar entre los métodos de un switch por ejemplo: set_ip_add(), get_int_status(). Llevando esto a código Python tendríamos de la siguiente manera:


```python
 class Switch:
	#definiendo el método constructor
	def __init__(self, hostname, vendor, ip_add):
		self.hostname = hostname
		self.vendor = vendor
		self.ip_add = ip_add
		self._port_density = 24
	
	#definiendo otros métodos (output referencial)
	def set_ip_add(self, ip_add):
		self.ip_add = ip_add
	def get_int_status(self):
		print("output of interface status")

def main():
	switch1 = Switch("SW1"," Cisco"," 192.168.0.1")
	switch2 = Switch("SW2", "Juniper", "192.168.0.2")
	
	print(switch1.hostname)
	#out: SW1
	
	switch2.set_ip_vlan("192.168.0.20")
	print(switch2.ip_add)
	#out: "192.168.0.20"

if __name__ == '__main__':
  	main()
```

El principio SOLID para el diseño de programación orientada a objetos es la abreviación de las iniciales de 5 directrices:
	- Single responsability principle (SRP)
	- Open-closed principle (OCP)
	- Liskov's substituion principle (LSP)
	- Interface segregation principle (ISP)
	- Dependency inversion principle (DIP)

1. SRP: se basa en el principio de que una clase debería tener una única responsabilidad o trabajo, Si un módulo tiene multiples funciones , modificar una de ellas afectaría a las otras y afectaría al sistema en general. Un error común sería incluir un método en la clase, que su modificación afecte a toda la clase en si, siguiendo un ejemplo de la guía:

```python
#not-easy-to-maintain-code:
 class Switch:
	def __init__(self, hostname, vendor):
		self.hostname = hostname
		self.vendor = vendor
		
	def set_hostname(self, hostname):
		self.hostname = hostname
	def set_vendor(self, vendor):
		self.vendor = vendor
	
	def deploy_switch(self):
    datacenter.deploy(self.hostname, self.vendor)
```

Como se observa en el ejemplo anterior, el método "deploy_switch" tiene funciones adicionales que cualquier modificación afectaría a toda la clase en general, por lo tanto como best-practice y siguiendo la directriz de SRP, asignamos el servicio "deploy_switch" a otra clase, asi mantenemos una única responsabilidad a toda la class que es la de hacer un setting de parámetros.

```python
 class Switch:
	def __init__(self, hostname, vendor):
		self.hostname = hostname
		self.vendor = vendor
		
	def set_hostname(self, hostname):
		self.hostname = hostname
	def set_vendor(self, vendor):
		self.vendor = vendor

class SwitchDeployer:
	def deploy_switch(self, service):
    datacenter.deploy(service.hostname, self.vendor)
```

2. OCP: indica que todos los componentes (class, modules, functions) deberían admitir extensión pero no modificación. Existinsibilidad se conceptualiza como la capacidad de permitir a un sistema o aplicación de extender sus features sin modificaciones.
Uno de las características que nos brinda la programación orientada a objetos es la de "Herencia". Donde una clase hija se relaciona heredando características de la clase padre. En el ejemplo tenemos una clase objeto "Switch" (clase padre), que engloba todos los switch devices de nuestra red. Si necesito distinguir un switch de acceso, de un switch de agregación, y asignar diferentes rangos de IPs de gestión dependiendo de la funcionalidad de cada switch, entonces tendría que modificar mi clase Switch, adicionar una condicionante que dependiendo su función jerarquica asignar cierto rango de IP. Esto viola el principio OCP del que estamos hablando, porque si bien estamos extendiendo funcionalidades, estamos modificando la clase en si. Para implementar en nuestro script esta funcionalidad sin violar el OCP, podemos utilizar herencia de la POO:

```python
#parent class
 class Switch:
	def __init__(self, hostname, vendor):
		self.hostname = hostname
		self.vendor = vendor

#child class
 class SwitchAgg(Switch):    
	def __init__(self):
		super().__init__(self)
		self.segment_ip_add = '192.168.100.0/24'
		
#child class
 class SwitchAcc(Switch):    
	def __init__(self):
		super().__init__(self)
self.segment_ip_add = '192.168.200.0/24'
```
En el ejemplo anterior se observa que un futuro si queremos agregar un objeto con un segmento para IPs de gestión para Switches Core, simplemente se adiciona una nueva clase hija, dependiente de la clase "Switch", de esta forma vamos de acuerdo al principio de OCP.

3. LSP: indica que un método de una clase hija, no debería reemplazar una función ya existente en la clase padre. Por ejemplo si en la clase padre tengo un método específico de obtener el estado de las interfaces de un Switch, desde una clase hija "SwitchAcc" no debería modificar el funcionamiento del método de la clase padre.

4. ISP: el principio recomienda no tener interfaces junto a otras interfaces no utilizadas en la misma superclase. Por ejemplo una utilización errónea sería la siguiente:

```python
 class SwitchAgg(Switch):    
	def get_model(self):
		return self.get_model()
	def get_AccModel(self):
		return NotAvailableError

 class SwitchAcc(Switch):    
	def get_AggModel(self):
		return NotAvailableError
	def get_AccModel(self):
    return db.get_AccModel(self)

```
Ejemplo respetando el principio de interface segregation:

```python
class Switch:    
	def get_model(self):
		return self.get_model()
		
class SwitchAgg(Switch):    
	def get_model(self):
		return db.get_AggModel(self)

class SwitchAcc(Switch):    
	def get_model(self):
    return db.get_AccModel(self)

```

5. DIP: habla de la dependencia de módulos basado en abstracciones y es la esencia del atributo de modularidad. La comunicación entre módulos debería realizar a través de interfaces como APIs, donde la modificación de módulos dependientes no afecte a módulos de más alto nivel.


Fuente:
- Python for network engineers by Natasha Samoylenko
- Cisco DEVCOR Official Certification Guide
- Platzi Courses
