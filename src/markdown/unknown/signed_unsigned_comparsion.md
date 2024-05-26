## Сравнение знаковых и беззнаковых типов
Когда люди приходят изучать C++, то не используют беззнаковые типы, а полагаются лишь на int. Так люди могут увидеть такое предупреждение от компилятора `comparison of integer expressions of different signedness: 'std::vector<int>::size_type' {aka 'long unsigned int'} and 'int'`.
А что может пойти не так? Рассмотрим следующий код.
```cpp
#include <iostream>
#include <format>

int main() {
    int a1 = -1;
    unsigned int a2 = 1;

    int b1 = -1;
    unsigned long b2 = 4294967296 + 1;

    std::cout << std::format(
        "{} > {} is {} \t {} > {} is {}",
        a1, a2, (a1 > a2),
        b1, b2, (b1 > b2)
    ) << std::endl;
}
```
Вроде бы всё логично и в итоге результат сравнения должен быть как false и false, однако не всё так просто. Чтобы сравнить int и unsigned int компилятор приводит int к unsigned int. Однако -1 для unsigned int тоже самое, что и 4294967295 (из-за модульной арифметики)`[p.s. надо будет написать про различия unsigned & signed арифметики]`, а следовательно -1 будет больше 1 при таких типах. Тогда возникает вопрос, а как всё же сравнивать знаковые и беззнаковые типы данных? Решение появилось в стандартой библиотеке с 20 года в хедере `<utility>` и чуть более подробно можно ознакомиться с ними на [cppreference](https://en.cppreference.com/w/cpp/utility/intcmp).
Ниже [пример](https://godbolt.org/z/KEK63a1E6), который показывает разницу, между тем как хочется и как надо.
```cpp
#include <iostream>

#include <format>
#include <utility>

int main() {
    int a1 = -1;
    unsigned int a2 = 1;

    int b1 = -1;
    unsigned long b2 = 4294967296 + 1;

    std::cout << std::format(
        "{} > {} is {} \t {} > {} is {}",
        a1, a2, (a1 > a2),
        b1, b2, (b1 > b2)
    ) << std::endl;

    std::cout << std::format(
        "{} > {} is {} \t {} > {} is {}",
        a1, a2, std::cmp_greater(a1, a2),
        b1, b2, std::cmp_greater(b1, b2)
    ) << std::endl;
    /* output:
        -1 > 1 is true      -1 > 4294967297 is true
        -1 > 1 is false     -1 > 4294967297 is false
    */
}
```
