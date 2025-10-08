# 📘 Лекция: Работа с файлами с помощью потоков в C++

Работа с файлами — одна из важнейших задач при программировании.
Файлы позволяют сохранять результаты работы программы, хранить большие объёмы данных, обмениваться информацией между программами и пользователями.

В языке C++ для работы с файлами используется **механизм потоков (streams)**.
Поток — это абстракция, описывающая последовательность байтов, идущих **из** программы (запись) или **в** программу (чтение).

Для работы с файлами необходимо подключить заголовочный файл:

```cpp
#include <fstream>
```

## Классы для работы с файлами

В стандартной библиотеке определены три основных класса:

| Класс      | Назначение                           | Аналог в C         | Пример использования        |
| ---------- | ------------------------------------ | ------------------ | --------------------------- |
| `ifstream` | Чтение данных из файла               | `fopen(..., "r")`  | `ifstream in("data.txt");`  |
| `ofstream` | Запись данных в файл                 | `fopen(..., "w")`  | `ofstream out("log.txt");`  |
| `fstream`  | Чтение и запись в один и тот же файл | `fopen(..., "r+")` | `fstream file("data.txt");` |

## Открытие файлов

Файл можно открыть **при создании объекта** или **через метод `.open()`**.

### Примеры:

```cpp
std::ofstream out("hello.txt");                // сразу открываем файл для записи
std::ifstream in("input.txt");                 // открываем файл для чтения
std::fstream fs("data.txt", std::ios::in | std::ios::out); // чтение и запись
```

Или эквивалентно:

```cpp
std::ofstream out;
out.open("hello.txt");                         // открываем позднее
```

### Режимы открытия файлов

Режимы определяются флагами из пространства имён `std::ios`.

| Флаг          | Назначение                             |
| ------------- | -------------------------------------- |
| `ios::in`     | Открытие файла для чтения              |
| `ios::out`    | Открытие файла для записи              |
| `ios::app`    | Добавление в конец файла (append)      |
| `ios::binary` | Открытие файла в бинарном режиме       |
| `ios::trunc`  | Очистка содержимого файла при открытии |
| `ios::ate`    | Переход в конец файла после открытия   |

### Примеры:

```cpp
std::ofstream out("file1.txt", std::ios::out);                 // запись
std::ofstream log("file2.txt", std::ios::app);                 // дозапись
std::fstream data("file3.dat", std::ios::in | std::ios::out);  // чтение + запись
std::ofstream bin("file4.bin", std::ios::binary | std::ios::trunc); // бинарный режим
```

## Закрытие файлов

Закрывать файл следует после окончания работы:

```cpp
out.close();
```

> 💡 **Важно:** если объект потока выходит из области видимости, файл закрывается автоматически — вызывать `close()` вручную не обязательно, но полезно для читаемости кода.

## Последовательный доступ

### Запись данных

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main() {
    ofstream outFile("example.txt");
    if (!outFile) {
        cerr << "Ошибка открытия файла!" << endl;
        return 1;
    }

    outFile << "Hello, World!" << endl;
    outFile << "C++ File Handling Example" << endl;

    outFile.close();
    cout << "Данные успешно записаны." << endl;
}
```

### Чтение данных построчно

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main() {
    ifstream inFile("example.txt");
    if (!inFile) {
        cerr << "Ошибка открытия файла!" << endl;
        return 1;
    }

    string line;
    while (getline(inFile, line)) {
        cout << line << endl;
    }

    inFile.close();
}
```

### Посимвольное чтение

```cpp
ifstream file("text.txt");
if (file.is_open()) {
    char c;
    while (file.get(c)) { // чтение посимвольно
        cout << c;
    }
    file.close();
}
```

> 📘 **Применение:** последовательный доступ используется при работе с **текстовыми файлами**, логами, конфигурационными данными и т.д.


## Прямой доступ к данным (Random Access)

В отличие от последовательного доступа, **прямой доступ** позволяет перемещаться по файлу к нужной позиции и считывать или записывать данные напрямую.

Используются указатели:

| Функция      | Назначение                       |
| ------------ | -------------------------------- |
| `seekg(pos)` | Переместить **указатель чтения** |
| `seekp(pos)` | Переместить **указатель записи** |
| `tellg()`    | Узнать текущую позицию чтения    |
| `tellp()`    | Узнать текущую позицию записи    |
| `read()`     | Прочитать блок байтов            |
| `write()`    | Записать блок байтов             |

### Пример: запись и чтение бинарных данных

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main() {
    // Запись массива в бинарный файл
    int data[5] = {10, 20, 30, 40, 50};
    ofstream outFile("example.dat", ios::binary);
    outFile.write(reinterpret_cast<char*>(data), sizeof(data));
    outFile.close();

    // Чтение третьего элемента напрямую
    ifstream inFile("example.dat", ios::binary);
    if (!inFile) {
        cerr << "Ошибка открытия файла!" << endl;
        return 1;
    }

    int value;
    inFile.seekg(2 * sizeof(int), ios::beg); // переходим к 3-му элементу
    inFile.read(reinterpret_cast<char*>(&value), sizeof(value));

    cout << "Третий элемент: " << value << endl;
    inFile.close();
}
```

> 💡 **Пояснение:**
> `reinterpret_cast<char*>` используется для преобразования адреса массива к указателю на байты (`char*`), так как `write()` и `read()` работают именно с последовательностями байтов.

## Полезные приёмы

* Проверка успешности открытия файла:

  ```cpp
  if (!file.is_open()) { ... }
  ```

* Проверка конца файла:

  ```cpp
  while (!inFile.eof()) { ... }
  ```

  > ⚠️ Однако надёжнее использовать `while (getline(...))` или `while (inFile.read(...))`, так как `eof()` возвращает `true` **только после неудачного чтения**.

* Получение размера файла:

  ```cpp
  inFile.seekg(0, ios::end);
  size_t size = inFile.tellg();
  inFile.seekg(0, ios::beg);
  cout << "Размер файла: " << size << " байт\n";
  ```


## Рекомендуемая литература

* Metanit.com: [Файловые потоки. Открытие и закрытие](https://metanit.com/cpp/tutorial/8.2.php), [Чтение и запись текстовых файлов](https://metanit.com/cpp/tutorial/8.3.php)
* cppreference.com: [std::ifstream / std::ofstream / std::fstream](https://en.cppreference.com/w/cpp/io/basic_ifstream)
