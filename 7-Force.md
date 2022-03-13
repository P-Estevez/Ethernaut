# 7-Force
## Objetivo
Lograr que el balance del contrato sea mayor a cero
## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```
## Análisis del contrato
Es un contrato vacío, no posee constructor, ni variables o funciones.

## Solución
Para resolver este método vamos a utilizar el comando [selfdestruct](https://docs.soliditylang.org/en/develop/introduction-to-smart-contracts.html?#deactivate-and-self-destruct) que poseen todos los smartcontracts.
Este comando envia el balance del contrato que se va a destruir a la dirección que se envíe por parámetro, y borra el código y el almacenamiento del contrato de la EVM (el contrato puede seguir siendo llamado porque forma
parte de la blockchain, pero queda inutilizado y al utilizar sus funciones no devuelve error).

Deployamos el siguiente contrato con una cantidad de ETH mínima
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract hackForce {

  constructor () public payable{}

  function hack() public {
    selfdestruct(Dirección de contrato Force);
  }
}
```
Llamamos a su función hack(), que terminará ejecutando el comando selfdestruct y transferirá los fondos al contrato Force.

Enviamos la instancia y listo!, reto completado.
