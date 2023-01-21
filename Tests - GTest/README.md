# dps-laboratory2

Preconfiguración de la máquina Ubuntu: He recibido un fallo en la ejecución de GTest. Se requería el paquete build-essential. Estos paquetes son la forma de meta-paquetes esenciales para compilar software. Contienen la colección de compiladores GNU/g++, el depurador GNU y algunas bibliotecas y herramientas más útiles para compilación.
```
sudo apt-get install build-essential
```


## GTEST & CMAKE

Para lanzar GTest, primero de todo hay que configurarlo. Google test (GTest) es un framework para tests de C++.

### Configuración de GTest

1. Se comienza por instalar el paquete de desarrollo de gtest

```
sudo apt-get install libgtest-dev
```

2. Se necesita instalar cmake utilizando los siguientes comandos:

```
sudo apt-get install cmake
```
3. Una vez instalado se localiza la raíz de los archivos Gtest (en mi caso):

```
cd /usr/src/gtest
```
4. Se compilan los archivos utilizando cmake:

```
sudo cmake CMakeLists.txt
sudo make
```
5. Se copian las librerías 'libgtest.a' y 'libgtest_main.a' desde el directorio actual (/usr/src/gtest) al directorio /usr/lib:

```
sudo cp *.a /usr/lib
```

Una vez se ha compilado Gtest, se pueden realizar los tests oportunos. 

6. Se vuelve al directorio del proyecto y se procede a compilar y generar los archivos correspondientes con CMakeLists.txt

```
sudo cmake CMakeLists.txt
sudo make
```
7. Se debe haber generado el archivo binario 'runTests' y se ejecuta:
```
sudo ./runTests
```

Output obtenido tras la ejecución de 'runTests':
```
[==========] Running 6 tests from 3 test suites.
[----------] Global test environment set-up.
[----------] 2 tests from wrapAddFunctionTest
[ RUN      ] wrapAddFunctionTest.NonWrappingNums
[       OK ] wrapAddFunctionTest.NonWrappingNums (0 ms)
[ RUN      ] wrapAddFunctionTest.WrappingNums
[       OK ] wrapAddFunctionTest.WrappingNums (0 ms)
[----------] 2 tests from wrapAddFunctionTest (0 ms total)

[----------] 2 tests from wrapMulFunctionTest
[ RUN      ] wrapMulFunctionTest.NonWrappingMulNums
[       OK ] wrapMulFunctionTest.NonWrappingMulNums (0 ms)
[ RUN      ] wrapMulFunctionTest.WrappingMulNums
[       OK ] wrapMulFunctionTest.WrappingMulNums (0 ms)
[----------] 2 tests from wrapMulFunctionTest (0 ms total)

[----------] 2 tests from wrapShiftFunctionTest
[ RUN      ] wrapShiftFunctionTest.NonWrappingMulBNums
[       OK ] wrapShiftFunctionTest.NonWrappingMulBNums (0 ms)
[ RUN      ] wrapShiftFunctionTest.WrappingMulBNums
[       OK ] wrapShiftFunctionTest.WrappingMulBNums (0 ms)
[----------] 2 tests from wrapShiftFunctionTest (0 ms total)

[----------] Global test environment tear-down
[==========] 6 tests from 3 test suites ran. (0 ms total)
[  PASSED  ] 6 tests.
```

Todos los tests han sido pasados.

## MODIFICACIÓN DE LAS FUNCIONES

1. Para la operación 'wrapFunctionAdd()' se requiere verificar la regla INT30-C (Resultado: 0). Este ejemplo puede dar como resultado un ajuste de entero sin signo durante la suma de los operandos.
```
unsigned int wrapFunctionAdd(unsigned int ui_a, unsigned int uiu_b){
```
Una solución compatible sería realizar un comando de exclusión:
```
if (uint_max - ui_a < ui_b) { /* Handle Error */ }
```


2. Para la operación 'wrapFunctionMul()' se requiere verificar la regla INT30-C
```
unsigned int wrapFunctionMul(unsigned int ui_a, unsigned int uiu_b){
```



3. Para la operación 'wrapFunctionShift()' se requiere verificar la regla INT34-C
```
unsigned int wrapFunctionShift(unsigned int ui_a, unsigned int uiu_b){
```
Como solución podemos tener en cuenta una división de esta operación: left Shift y right Shift.

  3.1. Left Shift

El siguiente código definiría la función para left shift:
```
void func(unsigned int ui_a, unsigned int ui_b) {
  unsigned int uresult = ui_a << ui_b;
  /* ... */
}
```
Para solucionarlo se elimina la posibilidad de desplazamiento mayor o igual que el número de bits que existen en la precisión del operando izquierdo:
```
extern size_t popcount(uintmax_t);
#define PRECISION(x) popcount(x)
  
void func(unsigned int ui_a, unsigned int ui_b) {
  unsigned int uresult = 0;
  if (ui_b >= PRECISION(UINT_MAX)) { /* Handle error */ }
```
Los comandos PRECISION(x) y popcount(x) definen la correcta precisión de cualquier parámetro tipo entero.

  3.2. Right Shift
  
El siguiente ejemplo de código no prueba si el operando derecho es mayor o igual que la precisión del operando izquierdo:
```
void func(unsigned int ui_a, unsigned int ui_b) {
  unsigned int uresult = ui_a >> ui_b;
  /* ... */
}
```
La siguiente solución elimina esta posibilidad:
```
extern size_t popcount(uintmax_t);
#define PRECISION(x) popcount(x)
  
void func(unsigned int ui_a, unsigned int ui_b) {
  unsigned int uresult = 0;
  if (ui_b >= PRECISION(UINT_MAX)) {
    /* Handle error */
  } else {
    uresult = ui_a >> ui_b;
  }
  /* ... */
}
```
