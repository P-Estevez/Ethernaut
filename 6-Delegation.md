# 6-Delegation

## Objetivo
Reclamar el ownership del contrato

## Contratos
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Delegate {

  address public owner;

  constructor(address _owner) public {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) public {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```

## Análisis de contratos
### Delegate
* Posee la variable owner, del tipo **_address_**
* El constructor, setea a quién deploya el contrato como owner
* La función pwn() es pública, y setea a quién realiza el llamado como el owner

## Delegation
* Posee a variable owner, del tipo **_address_** pública.
* La variable *delegate* del tipo contrato.
* El constructor, setea en la variable delegate la dirección de donde fue deployado y también setea a quien deploya el contrato como su owner.
* La función fallback(), es externa y al ser una función fallback se ejecuta si no se llama a ninguna función dentro del contrato. Realiza un delegatecall al address almacenado en
 la variable delegate. Por último valida si se pudo ejecutar esta función y devuelve true o false.
 
## Solución
Para poder resolver este reto primero tenemos que conocer el concepto de [delegatecall](https://docs.soliditylang.org/en/develop/introduction-to-smart-contracts.html?#delegatecall-callcode-and-libraries). El delegatecall, permite ejecutar funciones de otro contrato en el entorno del contrato actual.
Es decir, utiliza las funciones de otro contrato, pero el estado y memoria a utilizar son las del contrato original.

Entonces: 
La idea es llamar a la función Fallback del contrato Delegation. De esta manera utilizamos el delegateCall que realiza el contrato contra Delegate y ejecutamos su función pwd() para
modificar el owner.
 
Debemos desde la consola ejecutar la siguiente función:
```contract.sendTransaction({data: web3.utils.sha3('pwn()').slice(0, 10)})```

*Explicación del comando: enviamos una transacción normal sin una función específica para que se ejecute el fallback, y le pasamos como data el string "pwn()" encriptado con el algoritmo
sha3. De ese string resultante tomamos los 10 primeros caracteres. Esto último se debe a que debemos utilizar el ID del método a llamar. El ID del método
se compone por los primeros 4 bytes del hash obtenido al cifrar el nombre de la función con sus parámetros con el método Keccak-256.* [ID método](https://docs.soliditylang.org/en/develop/abi-spec.html#function-selector)

Luego enviar la instancia del reto, y listo! Reto completado.
