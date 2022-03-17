# 8-Vault

## Objetivo
Desbloquear el contrato

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) public {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```
## Análisis del contrato
* Posee la variable locker del tipo **_booleana_** y es pública
* Posee la variable passwor del tipo **_bytes32_** y es privada
* El constructor recibe por parámetro una variable del tipo **_bytes32_**. Asigna true a la variable locked, y a la variable password le asigna el valor recibido por parámetro
* La función unlock(), recibe por parámetro una variable del tipo **_bytes32_**. Si la variable recibida por parámetro es igual a la variable password del contrato, asigna
el valor false a la variable locked.

## Solución
Para resolver este reto, lo que debemos averiguar es el valor de la variable password del contrato, pero al ser privada no podemos acceder a ella desde otro contrato/Remix.
Sin embargo todo lo que está almacenado en la blockchain puede ser visualizado, por lo que podemos acceder al slot de memoria de este contrato donde se guarda esta variable.

Entonces:

Para accceder al slot de memoria donde se encuentra la variable: ```await web3.eth.getStorageAt(instance,1)```. Las variables se almacenan en slots de memoria de 32bytes en este caso
la variables locked ocupa todo el slot 0, ya que no se puede dividir la información binaria de las variables entre los slots de memoria, y la variable password se encuentra en el slot 1. [Documentación](https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_storage.html)

Nos devuelve el siguiente código en Hexadecimal:
```'0x412076657279207374726f6e67207365637265742070617373776f7264203a29'```

Si queremos validar qué contiene podemos ejecutar lo siguiente: 
```await web3.utils.hexToString("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")```

Por último llamamos a la función unlock con el valor del password (en hexa porque recibe como parámetro una variable del tipo bytes32):
```contract.unlock("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")```

Envíamos la instancia y listo!, reto resuelto.
