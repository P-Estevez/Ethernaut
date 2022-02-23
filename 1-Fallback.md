# 1-Fallback

En este reto nos proponen cumplir con dos puntos:
1. Reclamar el ownership del contrato.
2. Dejar el balance del contrato en 0

El contrato a deployar es el siguiente:
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Fallback {

  using SafeMath for uint256;
  mapping(address => uint) public contributions;
  address payable public owner;

  constructor() public {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    owner.transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

Analicemos el contrato:

* Puede ser compilado con una versión 0.6.0 o superior de Solidity.</br>
* Importa la librería SafeMath de OpenZeppeling por lo que sabemos que las operaciones aritméticas es poco probable que fallen.</br>
* Posee el **_mapping_** contributions, que relaciona una dirección con un entero, en este caso una dirección a la cantidad de contribución que aportó.</br>
* Posee la variable owner tipo **_address_** que puede recibir pagos.</br>
* El constructor asigna a quien deploya el contrato como su dueño y mapea 1000 ethers de contribución al dueño.</br>
* Un modificador **_onlyOwner_**, valida que quien realiza la transacción sea el dueño del contrato.</br>
* La función **_contribute_**, es pública y puede recibir ether. Dentro de esta se valida que se envíe una cantidad menor a 0.001 ether en la transacción, se suma el valor a la contribución y se valida que si la contribución es mayor a la contribución del dueño
* se asigna como dueño el sender de la transacción.</br>
* La función **_getContribution_** es pública, es *view* por lo que no realiza acciones de escritura, y devuelve un entero. Este entero es la cantidad de ether contribuido por la dirección que llama a la función.</br>
* La función **_withdraw_**, es pública y sólo puede llamarla el dueño del contrato. Transfiere los fondos del contrato, a la dirección del dueño del contrato.</br>
* La función **_receive_**, es externa (de Fallback) puede recibir ether. Requiere que el valor enviado en la transacción sea mayor a 0 y la contribución de quien llama a la función también sea mayor a 0.
Si esto se cumple, asigna al sender como nuevo dueño del contrato. La particularidad de esta función, es que si nosotros enviamos
ether a la dirección del contrato, sin ninguna función especial (como contribute), se termina llamando a esta función receive.

Para resolver este reto, tenemos dos opciones:
* realizamos muchas transacciones menores a 0,001 ethers hasta superar los 1000 ethers de contribución.
* lograr llamar a la función recieve para poder reclamar el ownership del contrato.

Por lo que para resolver este challenge debemos:
1. Contribuir con una cantidad menor a 0.001 ether.
2. Enviar una cantidad mayor a 0 al contrato sin especificar ninguna función.
3. Retirar los fondos del contrato.

Entonces:</br>
En la consola del navegador:
1. ```await contract.contribute({value:toWei("0.0001")}) ```
2. ```web3.eth.sendTransaction({from:player,to:instance , value: toWei("0.0001")})```
   - Luego validamos que seamos el owner ```await contract.owner()```
3. ```contract.withdraw()```
4. Realizamos el submit de la instacia y listo! Segundo reto completado.
