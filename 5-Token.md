# 5-Token

## Objetivo
Al deployar este contrato vamos a recibir 20 tokens, para poder superar este nivel, tenemos que poseer más de 20 tokens (muchos más que 20).

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

## Análisis de contrato
* Puede ser compilado con una versión 0.6.0 o superior de Solidity.
* Posee el **_mappping_** balances, que mapea un address a un entero.
* Posee un entero que guarda el total supply.
* El constructor recibe como parámetro un entero. Setea el entero recibido por parámetro a la variable total supply y al balance de quien deploya el contrato.
* La función transfer, es pública, recibe como parámetro un address y un entero y retorna un booleano.
Se valida que la resto entre el valor recibido por parámetro y el balance de quien envía la transacción sea mayor o igual a 0. Si esto se cumple, se resta el
valor enviado por parámetro al balance del sender, y se suma al balance del address recibido por parámetro. Luego devuelve true.
* La función balanceOf, es pública y view, recibe un address por parámetro y retorna un entero. Devuelve el balance del address recibida por parámetro.

## Solución
En Solidity los enteros son sin signo (siempre positivos) y de caracter fijo, es decir poseen una cantidad de bits definida en memoria. Al ser este tamaño fijo, tienen un rango
de valores que pueden tomar, por lo que un uint8 tiene un rango de 0 a 255. Si a este 255 le sumamos 1, esta variable vuelve a 0. Esto se ve mejor en una suma binaria:

1111 1111 + 0000 0001 debería ser 1 0000 0000, pero como sólo dispone de 8 bits y el primero es el menos significativo se pierde quedando 0000 0000.

Vamos a utilizar esto para resolver el reto. Básicamente lo que debemos realizar es llamar a la función transfer y que resten al balance de nuestro address una cantidad superior a
la que poseemos para generar un overflow negativo.

```Javascript
contract.transfer("0x0000000000000000000000000000000000000000",21)
```

En este caso utilicé el address 0, pero se puede utilizar cualquier address que no sea el address que envía la transacción.
Podemos validar el balance de nuestro address ```contract.balanceOf("tu address")```

Realizamos el submit de la instancia y listo!, sexto reto superado.
