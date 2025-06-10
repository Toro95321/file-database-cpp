# Файловая база данных на C++

Консольное приложение для работы с простой файловой базой данных в виде таблицы.

## Техническое задание

### Требования к функционалу:
- Табличное представление данных с границами
- Операции:
  - Добавление/удаление колонок
  - Добавление/удаление записей
  - Просмотр всей таблицы
  - Очистка всех данных
- Автоматическое сохранение в файл после каждого изменения

### Особенности реализации:
- Данные хранятся в CSV-файле
- Используются STL-контейнеры (vector, string)
- Реализовано красивое форматирование вывода
- Простое консольное меню

## Сборка и запуск

g++ main.cpp -o filedb
./filedb

## Структура проекта

file-database-cpp/
├── main.cpp       # Основной исходный код
├── README.md      # Этот файл
├── .gitignore     # Игнорируемые файлы
└── database.csv   # Файл базы данных (создается автоматически)


## Коммиты

Желательная структура коммитов:
1. `feat`: Добавление нового функционала
2. `fix`: Исправление ошибок
3. `docs`: Обновление документации
4. `refactor`: Рефакторинг кода без изменения функционала

### Код:

#include <iostream>  // Для ввода/вывода (cin, cout)
#include <fstream>   // Для работы с файлами (ifstream, ofstream)
#include <vector>    // Для использования динамических массивов (vector)
#include <string>    // Для работы со строками (string)
#include <iomanip>   // Для форматированного вывода (setw, left)
#include <limits>    // Для работы с пределами типов (numeric_limits)

using namespace std; // Используем стандартное пространство имен

class Database {
private:
    vector<vector<string>> data; // Двумерный вектор для хранения данных таблицы (строки и столбцы)
    vector<string> columns;     // Вектор для хранения названий столбцов
    string filename;            // Имя файла для сохранения/загрузки БД

    // Метод для сохранения данных в файл
    void saveToFile() {
        ofstream file(filename); // Создаем объект для записи в файл
        if (!file.is_open()) {   // Проверяем открылся ли файл
            cerr << "Ошибка при сохранении файла" << endl;
            return;
        }

        // Сохраняем заголовки столбцов
        for (size_t i = 0; i < columns.size(); ++i) {
            file << columns[i];  // Записываем название столбца
            if (i != columns.size() - 1) file << ","; // Добавляем запятую если не последний столбец
        }
        file << endl; // Переход на новую строку после заголовков

        // Сохраняем данные таблицы
        for (const auto& row : data) {      // Для каждой строки в данных
            for (size_t i = 0; i < row.size(); ++i) { // Для каждого элемента в строке
                file << row[i];             // Записываем значение
                if (i != row.size() - 1) file << ","; // Добавляем запятую если не последний элемент
            }
            file << endl; // Переход на новую строку после каждой строки данных
        }

        file.close(); // Закрываем файл
    }

    // Метод для загрузки данных из файла
    void loadFromFile() {
        ifstream file(filename); // Создаем объект для чтения из файла
        if (!file.is_open()) {   // Если файл не открылся (например не существует)
            return;             // Просто выходим создадим файл при первом сохранении
        }

        data.clear();    // Очищаем существующие данные
        columns.clear(); // Очищаем существующие столбцы

        string line; // Переменная для хранения строки из файла

        // Читаем первую строку - заголовки столбцов
        if (getline(file, line)) {
            size_t pos = 0; // Позиция для поиска разделителя
            // Разбиваем строку по запятым
            while ((pos = line.find(',')) != string::npos) {
                string token = line.substr(0, pos); // Выделяем название столбца
                columns.push_back(token);         // Добавляем в вектор столбцов
                line.erase(0, pos + 1);           // Удаляем обработанную часть строки
            }
            columns.push_back(line); // Добавляем последний столбец (после последней запятой)
        }

        // Читаем остальные строки  данные таблицы
        while (getline(file, line)) {
            vector<string> row; // Создаем вектор для строки данных
            size_t pos = 0;
            // Разбиваем строку по запятым
            while ((pos = line.find(',')) != string::npos) {
                string token = line.substr(0, pos); // Выделяем значение
                row.push_back(token);              // Добавляем в строку данных
                line.erase(0, pos + 1);            // Удаляем обработанную часть строки
            }
            row.push_back(line); // Добавляем последнее значение строки
            data.push_back(row);  // Добавляем строку в данные таблицы
        }

        file.close(); // Закрываем файл
    }

public:
    // Конструктор - создает объект БД и загружает данные из указанного файла
    Database(const string& dbFilename) : filename(dbFilename) {
        loadFromFile(); // Загружаем данные при создании объекта
    }

    // Метод для добавления нового столбца
    void addColumn(const string& columnName) {
        columns.push_back(columnName); // Добавляем название в вектор столбцов
        // Добавляем пустую ячейку в каждую существующую строку
        for (auto& row : data) {
            row.push_back("");
        }
        saveToFile(); // Сохраняем изменения в файл
    }

    // Метод для удаления столбца по индексу
    void removeColumn(size_t index) {
        if (index >= columns.size()) { // Проверка корректности индекса
            cerr << "Неверный индекс столбца!" << endl;
            return;
        }

        columns.erase(columns.begin() + index); // Удаляем название столбца
        // Удаляем соответствующий элемент из каждой строки
        for (auto& row : data) {
            if (index < row.size()) {
                row.erase(row.begin() + index);
            }
        }
        saveToFile(); // Сохраняем изменения в файл
    }

    // Метод для добавления новой записи (строки)
    void addRecord(const vector<string>& record) {
        if (record.size() != columns.size()) { // Проверка соответствия количества значений
            cerr << "Количество значений не соответствует количеству столбцов" << endl;
            return;
        }
        data.push_back(record); // Добавляем новую строку в данные
        saveToFile();           // Сохраняем изменения в файл
    }

    // Метод для удаления записи по индексу
    void removeRecord(size_t index) {
        if (index >= data.size()) { // Проверка корректности индекса
            cerr << "Неверный индекс записи" << endl;
            return;
        }
        data.erase(data.begin() + index); // Удаляем строку
        saveToFile();                     // Сохраняем изменения в файл
    }

    // Метод для полной очистки данных (но не структуры столбцов)
    void clearData() {
        data.clear();  // Очищаем все данные
        saveToFile(); // Сохраняем изменения в файл
    }

    // Метод для отображения таблицы в консоли
    void display() const {
        if (columns.empty()) { // Если нет столбцов  таблица пуста
            cout << "База данных пуста" << endl;
            return;
        }

        // Вычисляем ширину каждого столбца для красивого вывода
        vector<size_t> widths(columns.size(), 0); // Вектор для хранения ширины каждого столбца
        for (size_t i = 0; i < columns.size(); ++i) {
            widths[i] = columns[i].length(); // Начинаем с длины названия столбца
            // Ищем максимальную длину значения в столбце
            for (const auto& row : data) {
                if (i < row.size() && row[i].length() > widths[i]) {
                    widths[i] = row[i].length();
                }
            }
            widths[i] += 2; // Добавляем отступы по бокам
        }

        // Выводим верхнюю границу таблицы
        cout << "+";
        for (size_t w : widths) {
            cout << string(w, '-') << "+"; // Рисуем горизонтальную линию
        }
        cout << endl;

        // Выводим заголовки столбцов
        cout << "|";
        for (size_t i = 0; i < columns.size(); ++i) {
            // Форматированный вывод с выравниванием по левому краю
            cout << " " << setw(widths[i] - 1) << left << columns[i] << "|";
        }
        cout << endl;

        // Выводим разделитель между заголовками и данными
        cout << "+";
        for (size_t w : widths) {
            cout << string(w, '-') << "+";
        }
        cout << endl;

        // Выводим данные таблицы
        for (const auto& row : data) {
            cout << "|";
            for (size_t i = 0; i < columns.size(); ++i) {
                // Берем значение или пустую строку, если значения нет
                string value = (i < row.size()) ? row[i] : "";
                // Форматированный вывод значения
                cout << " " << setw(widths[i] - 1) << left << value << "|";
            }
            cout << endl;
        }

        // Выводим нижнюю границу таблицы
        cout << "+";
        for (size_t w : widths) {
            cout << string(w, '-') << "+";
        }
        cout << endl;
    }

    // Метод для получения количества столбцов
    size_t getColumnCount() const {
        return columns.size();
    }
};

// Функция для отображения меню пользователя
void showMenu() {
    cout << "Меню:\n";
    cout << "1. Добавить колонку\n";
    cout << "2. Удалить колонку\n";
    cout << "3. Добавить запись\n";
    cout << "4. Удалить запись\n";
    cout << "5. Вывести таблицу\n";
    cout << "6. Очистить таблицу\n";
    cout << "7. Выход\n";
    cout << "Выберите действие: ";
}

// Главная функция программы
int main() {
    setlocale(LC_ALL, "Ru");// Русификатор
    Database db("database.csv"); // Создаем объект БД с указанием имени файла

    int choice; // Переменная для выбора пользователя
    do {
        showMenu(); // Показываем меню
        cin >> choice; // Читаем выбор пользователя
        cin.ignore(numeric_limits<streamsize>::max(), '\n'); // Очищаем буфер ввода

        // Обрабатываем выбор пользователя
        switch (choice) {
        case 1: { // Добавление колонки
            cout << "Введите название новой колонки: ";
            string columnName;
            getline(cin, columnName); // Читаем название колонки
            db.addColumn(columnName); // Добавляем колонку
            break;
        }
        case 2: { // Удаление колонки
            cout << "Введите индекс колонки для удаления (начиная с 0): ";
            size_t index;
            cin >> index; // Читаем индекс колонки
            db.removeColumn(index); // Удаляем колонку
            break;
        }
        case 3: { // Добавление записи
            size_t colCount = db.getColumnCount(); // Получаем количество колонок
            if (colCount == 0) {
                cout << "Сначала добавьте колонки!\n";
                break;
            }

            vector<string> record; // Вектор для новой записи
            cout << "Введите " << colCount << " значений через Enter:\n";
            for (size_t i = 0; i < colCount; ++i) {
                string value;
                cout << "Значение " << i << ": ";
                getline(cin, value); // Читаем каждое значение
                record.push_back(value); // Добавляем в запись
            }
            db.addRecord(record); // Добавляем запись в БД
            break;
        }
        case 4: { // Удаление записи
            cout << "Введите индекс записи для удаления (начиная с 0): ";
            size_t index;
            cin >> index; // Читаем индекс записи
            db.removeRecord(index); // Удаляем запись
            break;
        }
        case 5: { // Вывод таблицы
            db.display(); // Отображаем таблицу
            break;
        }
        case 6: { // Очистка таблицы
            db.clearData(); // Очищаем данные
            cout << "Таблица очищена\n";
            break;
        }
        case 7: { // Выход
            cout << "Выход из программы\n";
            break;
        }
        default: { // Неверный ввод
            cout << "Неверный выбор\n";
            break;
        }
        }
    } while (choice != 7); // Пока пользователь не выберет выход

    return 0;
}
