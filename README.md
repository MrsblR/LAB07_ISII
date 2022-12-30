# Laboratorio 7: Refactoring y SonarLint

## 1. Identificación de Code Smells

Los errores indican que:

- No se deben usar expresiones iguales en ambos lados de un operador binario.
- La complejidad cognitiva de las funciones no debe ser demasiado alta.
- Los strings que son literales no deben duplicarse.
- Las declaraciones if plegables deben fusionarse.
- Define una constante en lugar de duplicar esta literal.
- Importa sólo los nombres necesarios o importa el módulo y luego utiliza sus miembros

![](https://github.com/MrsblR/LAB07_ISII/blob/final/Evidences/Problems.PNG)

## 2. Adicionar casos de prueba faltantes (TDD para la nueva funcionalidad)

```python
import unittest
from gilded_rose import GildedRose, Item


class GildedRoseTest(unittest.TestCase):
    def updater(obj = GildedRose, times = 1):
        for i in range(times):
            obj.update_quality()

    def test_normal_item(self):
        # Set up the item
        item = Item("+5 Dexterity Vest", 10, 20)
        gilded_rose = GildedRose([item])

        # One day after
        gilded_rose.update_quality()
        self.assertEqual(item.sell_in, 9)
        self.assertEqual(item.quality, 19)
        
        # Sell by date passed
        GildedRoseTest.updater(gilded_rose, item.sell_in + 1)
        self.assertEqual(item.sell_in, -1)

        # Quality degrades twice as fast now
        self.assertEqual(item.quality, 8)
        GildedRoseTest.updater(gilded_rose, 10)
        self.assertEqual(item.sell_in, -11)
        self.assertEqual(item.quality, 0)  # never be less than 0

    def test_aged_brie(self):
        # Set up the item
        item = Item("Aged Brie", 10, 20)
        gilded_rose = GildedRose([item])
        
        # One day after
        gilded_rose.update_quality()
        self.assertEqual(item.sell_in, 9)
        self.assertEqual(item.quality, 21)

        # Sell by date passed
        GildedRoseTest.updater(gilded_rose, item.sell_in + 1)
        self.assertEqual(item.sell_in, -1)

        # Quality increases twice as fast now
        self.assertEqual(item.quality, 32)
        GildedRoseTest.updater(gilded_rose, 10)
        self.assertEqual(item.sell_in, -11)
        self.assertEqual(item.quality, 50) 

    def test_sulfuras(self):
        # Set up the item
        item = Item("Sulfuras, Hand of Ragnaros", 20, 80)
        gilded_rose = GildedRose([item])

        # One day after
        gilded_rose.update_quality()
        self.assertNotEqual(item.sell_in, 19)
        self.assertEqual(item.sell_in, 20)
        self.assertEqual(item.quality, 80)
        
        # sell by date passed
        GildedRoseTest.updater(gilded_rose, item.sell_in + 1)
        self.assertEqual(item.sell_in, 20)
        self.assertNotEqual(item.sell_in, -1)

    def test_backstage_passes(self):
        # Set up the item
        item = Item("Backstage passes to a TAFKAL80ETC concert", 20, 20)
        gilded_rose = GildedRose([item])

        # One day after
        gilded_rose.update_quality()
        self.assertEqual(item.sell_in, 19)
        self.assertEqual(item.quality, 21)
        GildedRoseTest.updater(gilded_rose, 9)

        # Sell in 10 or less
        self.assertEqual(item.sell_in, 10)
        self.assertEqual(item.quality, 30)
        GildedRoseTest.updater(gilded_rose)
        self.assertEqual(item.sell_in, 9)
        self.assertEqual(item.quality, 32)

        # Sell in 5 or less
        GildedRoseTest.updater(gilded_rose, 4)
        self.assertEqual(item.sell_in, 5)
        self.assertEqual(item.quality, 40)
        GildedRoseTest.updater(gilded_rose)
        self.assertEqual(item.sell_in, 4)
        self.assertEqual(item.quality, 43)
        GildedRoseTest.updater(gilded_rose, 2)
        self.assertEqual(item.sell_in, 2)
        self.assertEqual(item.quality, 49)

        # Never gets beyond 50 and drops to 0 once sell in passed
        GildedRoseTest.updater(gilded_rose)
        self.assertEqual(item.sell_in, 1)
        self.assertEqual(item.quality, 50)

        GildedRoseTest.updater(gilded_rose, 2)
        self.assertEqual(item.sell_in, -1)
        self.assertEqual(item.quality, 0)

    def test_conjured(self):
        # Set up the item
        item = Item("Conjured Mana Cake", 20, 50)
        gilded_rose = GildedRose([item])

        # One day after
        gilded_rose.update_quality()
        self.assertEqual(item.sell_in, 19)
        self.assertEqual(item.quality, 48)
        GildedRoseTest.updater(gilded_rose, item.sell_in + 1)

        # sell by date passed -- will decrease twice as fast as normal items (-4)
        self.assertEqual(item.sell_in, -1)
        self.assertEqual(item.quality, 6)

        GildedRoseTest.updater(gilded_rose)
        self.assertEqual(item.sell_in, -2)
        self.assertEqual(item.quality, 2)

        GildedRoseTest.updater(gilded_rose)
        self.assertEqual(item.sell_in, -3)
        self.assertEqual(item.quality, 0)

if __name__ == '__main__':
    unittest.main()
```

## 3. Refactorizar el código (2 ó 3) [con ayuda de SonarLint]


La clase GildedRose tiene varios problemas de estructura y legibilidad:

- Método “ update_quality ” muy grande e ilegible ;
- Muchas condiciones y ramas (anidadas si);
- Falta de comentarios y documentación.


### Refactorización de la clase GildedRose
La descripción del problema deja en claro que no podemos cambiar la clase de artículo. Solo podemos cambiar la clase GildedRose y/o crear nuevas clases. Se puede crear una clase Updater que contenga las reglas de actualización y cambiar la clase GildedRose , haciéndola legible y escalable para nuevos elementos.

```python
class GildedRose(object):

    def __init__(self, items):
        self.items = items

    def update_quality(self):
        updater = Updater()

        for item in self.items:
            if item.name == 'Aged Brie':
                updater.aged_brie(item)
            elif item.name == 'Backstage passes to a TAFKAL80ETC concert':
                updater.backstage_passes(item)
            elif item.name == 'Sulfuras, Hand of Ragnaros':
                updater.sulfuras(item)
            elif item.name == 'Conjured Mana Cake':
                updater.conjured(item)
            else:
                updater.normal_item(item)
```

La nueva clase instancia la clase Updater , verifica el nombre del elemento y ejecuta la función adecuada para cada uno. Se eliminaron todas las ramas, lo que hizo que el código fuera más legible. Si es necesario crear un nuevo tipo de elemento, simplemente cree otra elif condición.


### Creandola clase Updater

Todas las reglas comerciales se encapsularán en esta clase, lo que facilitará el mantenimiento, la lectura y la escala del código.


```python
class Updater(object):
    
    MIN_QUALITY = 0
    MAX_QUALITY = 50

    def normal_item(self, item):
        if item.sell_in > 0: 
            depreciation = -1
        else:
            depreciation = -2
        item.quality = max((item.quality + depreciation), self.MIN_QUALITY)
        item.sell_in += -1

    def aged_brie(self, item):
        if item.sell_in > 0:
            appreciation = 1
        else:
            appreciation = 2
        item.quality = min((item.quality + appreciation), self.MAX_QUALITY)
        item.sell_in += -1
    
    def sulfuras(self, item):
        pass

    def backstage_passes(self, item):

        def get_quality(item, appreciation):
            item.quality = min(item.quality + appreciation , self.MAX_QUALITY)

        if item.sell_in > 10:
            appreciation = 1
            get_quality(item, appreciation)
        elif item.sell_in > 5:
            appreciation = 2
            get_quality(item, appreciation)
        elif item.sell_in > 0:
            appreciation = 3
            get_quality(item, appreciation)
        else:
            item.quality = 0
        item.sell_in += -1

    def conjured(self, item):
        if item.sell_in > 0:
            depreciation = -2
        else:
            depreciation = -4

        item.quality = max((item.quality + depreciation), self.MIN_QUALITY)
        item.sell_in += -1
```

La clase Updater establece valores mínimos y máximos constantes para la calidad del elemento. Se crearon funciones, una para cada tipo de ítem, facilitando el aislamiento de las reglas de negocio.

De esta forma, cada nuevo tipo de elemento se puede implementar creando:

- Una elif condición en la clase GildedRose ;
- Una función en la clase Updater que contiene las reglas para la actualización sell_iny qualitylos valores.


## 5. Implementar la nueva funcionalidad

Se creo una clase hija de Item llamada conjured_item, donde se actualizó el comportamiento del método actualizar item

```python
class conj_item (Item):
    def update_quality(self):
        if self.quality > 2:
            self.quality = self.quality - 2
        else:
            self.quality = 0

        self.sell_in = self.sell_in - 1
```

## 6. Verificar y validar las actualizaciones: SonarLint y Re-ejecución de Pruebas

Se puede observar que el código de esta vez no se detectó ninguna violación de reglas ni issues.

Por otro lado, también se re-ejecutarón las Pruebas para verificar que el código funcione correctamente.

Los resultados son los siguientes:

![](https://github.com/MrsblR/LAB07_ISII/blob/final/Evidences/Issues.PNG)
