# 2-Fallout

## Objetivo
Este reto sólo nos pide que claimeemos el ownership del contrato

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```
## Análisis del contrato
* Puede ser compilado con una versión 0.6.0 o superior de Solidity.
* Importa la librería SafeMath de OpenZeppeling por lo que sabemos que las operaciones aritméticas es poco probable que fallen.
* Posee un **_mapping_** allocations, el cual asocia una dirección a un entero
* La variable owner del tipo **_address_** que permite recibir pagos.
* La función Fal1out, comentada como "constructor", setea a quien llama la función como owner, y mapea el valor enviado en la transacción con este address.
* Un modificador onlyOwner, valida que quien realiza la transacción sea el dueño del contrato.
* Función allocate, es pública y recibe pagos. Suma el valor que se envia en la transacción al entero que tiene mapeado la dirección del sender.
* Función sendAllocation, es pública. Recibe como parámetro un address. Valida que el entero mapeado al address recibida por parámetro sea mayor a 0.
* Función collectAllocations, es pública y solo puede ejecutarla el owner del contrato. Transfiere los fondos del contrato al caller de la función.
* Función allocatorBalance, es pública y view, devuelve un entero y recibe como parámetro un address. Devuelve el entero mapeado a ese address.

## Solución
Para versiones anteriores a la 0.6.0 el constructor del contrato, llevaba el mismo nombre del contrato.
Pero el "constructor" posee un error de tipeo, por lo que deja a la función Fal1out libre para que cualquiera pueda accederlo.</br>
Solo debemos llamar esta función desde la consola: ```await contract.Fal1out()``` </br>
Realizamos el envio de la instancia y listo!. Tercer reto completado.
