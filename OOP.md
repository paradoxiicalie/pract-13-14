# pract-13-14
```text
Тикунов Алексей Александрович, ЕТ-143
```
```text
Вариант 22
```


```text 
А) абстрактный тип данных = нужно на базе массива (все пункты а на базе массива делается), но этот абстрактный тип данных содержит еще способы получения данных (придумать способ отстведить все способы доступы к элементам и тд) проверка есть данные во множесте = тру фолс и дальше на основе ее стоить
```
```cpp
#include <iostream>
using namespace std;

class Set
{
    int* data; // указатель на динам массив
    int size; // тек колво элтов в массиве
    int cap; //вместимость
    
    void resize()
    {
        cap *= 2;
        int* newData = new int[cap];
        
        for (int i = 0; i<size ; i++)
        {
            newData[i] = data[i];
        }
        
        delete[] data; // старый массив убираем
        data = newData; // перенаправляем указатель на новый
    }
    
public:
    Set()
    {
        size = 0;
        cap = 5;
        data = new int[cap];
    }
    
    ~Set()
    {
        delete[] data;
    }
    
    Set(const Set& other)
    {
        size = other.size;
        cap = other.cap;
        data = new int[cap];
        for (int i = 0 ; i < size ; i++)
        {
            data[i] = other.data[i];
        }
    }
    
    Set& operator=(const Set& other)
    {
        if (this == &other) return *this;
        delete[] data;
        
        size = other.size;
        cap = other.cap;
        data = new int[cap];
        
        for(int i = 0; i < size ; i++)
        {
            data[i] = other.data[i];
        }
        return *this;
    }
    
    
    void add(int value) //добавление элемента с проверкой на уникальность
    {
        if (*this > value) return; // если элемент уже есть, ничего не делаем
        if (size == cap) resize(); // если массив заполнен, расширяем его
        
        data[size] = value;
        size++;
    }
    
    
    bool operator>(int value) const
    {
        for (int i = 0 ; i < size ; i++)
        {
            if (data[i] == value) return true;
        }
        return false;
    }
    
    Set operator*(const Set& other) const
    {
        Set result;
        
        for (int i = 0; i < size ; i++)
        {
            if (other > data[i])
            {
                result.add(data[i]);
            }
        }
        return result;
    }
    
    bool operator<(const Set& other) const
    {
        for (int i = 0 ; i < size ; i++)
        {
            if (!(other > data[i])) // если хотя бы одного нашего элта нет во втором множестве
            {
                return false;
            }
        }
        return true;
    }
    
    void print() const
    {
        cout << "{ ";
        for (int i = 0 ; i < size ; i++)
        {
            cout << data[i] << " ";
        }
        cout << "}" << endl;
    }
};

int main()
{
    Set A;
    A.add(1);
    A.add(2);
    A.add(3);
    A.add(4);
    A.add(4);
    
    Set B;
    B.add(3);
    B.add(3);
    B.add(4);
    B.add(5);
    B.add(6);
    
    cout << "Множество А:"; A.print();
    cout << "Множество B:"; B.print();
    cout << "-------------------" << endl << endl;
    
    
    // >
    int test = 3;
    if (A > test)
    {
        cout << test << " принаджелит множеству А" << endl;
    }
    else
        cout << test << " Не принадлежит множеству А" << endl;
    
    // *
    Set C = A * B;
    cout << "Пересечение A и B: ";
    C.print();
    
    // <
    Set subA;
    subA.add(2);
    subA.add(3);
    
    cout << "Множество subA: "; subA.print();
    
    if (subA < A)
        cout << "subA является подмножеством А" << endl;
    else
        cout << "subA НЕ является подмножество А" << endl;
    
    return 0;
}
```

```text
B) Разработайте иерархию классов матричные структуры (приватные
 члены сконструировать самостоятельно). Добавьте статические члены по
 подсчету суммарного количества элементов. Разработайте не менее 2-х
 виртуальных функций.
```
```cpp
#include <iostream>
using namespace std;

class Matrix
{
private:
    
    int colums, rows;
    static int countEl; // статическое поле ; Переменная общая на всю программу, кот хранит общее колво элтов во всех созданных матрицах
    
protected: // чтобы наследники могли короче понять скок строк и колонок
    int Get_colums()  const {return colums; } // const тк методы не меняют переменные класса (ток чтение)
    int Get_Stroki()  const {return rows; }
    
public:
    
    Matrix(int r, int c ) // this указывает на текущий объект
    {
        this -> rows = r;
        this -> colums = c;
        
        countEl += this->rows * this -> colums; // строка на колону = все
    }
    
    virtual ~Matrix() // чтобы при удалении объекта через указатель сначала вызывался деструктор наслденика
    {
        countEl-=colums*rows;
        // при удалении матрицы вычитаем ее элементы из всего
    }
    
    static int getCountEl() {return countEl;} // static - делаем для всех общей даж когда ни одной матрицы еще нету
    
    virtual void print() const = 0; // = 0 - нет реализации в базовом классе ; те каждый наследний представляет свою версию print()
    virtual double sled() const = 0; // след матрицы = сумма элементов главной диагонали
};


class Diagonal_Matrix : public Matrix // ток элементы главной диагонали вроде
{
private:
    double* diag; //храним ток элементы главной диагонали ; * тк тут будет лежать массив
public:
    Diagonal_Matrix(int n , const double* values) : Matrix(n,n)
    {
        diag = new double[n]; // ручное выделение памяти под n элтов в куче
        for (int i = 0; i < n ; i++)
        {
            diag[i] = values[i];
        }
    }
 ~Diagonal_Matrix() override // деструктор ; override тк виртуальный и он сам используется
    {
        delete[] diag; // осв памяти
    }
    
    void print() const override // тк в базовом ф-ция константная ; ну и чтобы он делал как в родителе
    {
        int n = Get_Stroki();
        cout << "Диагональная матрица (" << n << "x" << n <<"):" << endl;
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (i==j) cout << diag[i] << "\t"; // строка = столбу - выводим иначе 0
                else cout << "0\t";
            }
            cout << endl;
        }
    }
    
    double sled() const override
    {
        double sum = 0;
        for (int i = 0 ; i<Get_Stroki(); i++)
        {
            sum+=diag[i];
        }
        return sum;
    }
};
    
class UsualMatrix : public Matrix
{
    double** data; // двумерный массив для хранения всех элтов ; указатель на указатель (кажд элт ссылается на другой массив)
public:
    UsualMatrix(int r, int c, const double** values) : Matrix(r, c)
    {
        data = new double*[r]; // выделяем массив указателей на строки
        for (int i = 0; i < r ; i++)
        {
            data[i] = new double[c]; //выделяем память под каждую столб
            for (int j = 0; j < c ; j++)
            {
                data[i][j] = values[i][j]; // прост заполняет таблицу
            }
        }
    }
    
    
    ~UsualMatrix() override // удаляемся в обратном порядке создания
    {
        for (int i = 0 ; i < Get_Stroki(); i++)
        {
            delete[] data[i]; // каждую строку
        }
        delete[] data; // массив указателей
    }
    
    
    void print() const override
    {
        cout << "Обычная матрица (" << Get_Stroki() << "x" << Get_colums() << "):" << endl;
        for (int i = 0; i < Get_Stroki() ; i++)
        {
            for (int j = 0; j < Get_colums() ; j++)
            {
                cout << data[i][j] << "\t";
            }
            cout << endl;
        }
    }
    
    double sled() const override
    {
        double sum = 0; // след имеет смысл ток для квадратной матрицы
        int min_dim = (Get_Stroki() < Get_colums()) ? Get_Stroki() : Get_colums(); // если строк меньше колонок, то берем строки инае столбы
        for (int i = 0 ; i < min_dim; i++)
        {
            sum+= data[i][i];
        }
        return sum;
    }
};
    
int Matrix::countEl = 0; // выделение памяти под статическую переменную

int main()
{
    cout << "Всего элементов: " << Matrix::getCountEl() << endl << endl;
    
    
    double diag_chisls[] = {5.5 , 3.2 , 8.1};
    
    double row1[] = {1.1, 2.2 , 3.3};
    double row2[] = {4.4 , 5.5 , 6.6};
    const double* ord_vals[] = {row1, row2};
    
    
    Matrix* m1 = new Diagonal_Matrix(3, diag_chisls); // 3 х 3 = 9 элтов ; new - память в объект наследника
    Matrix* m2 = new UsualMatrix(2,3,ord_vals); // 2 х 3 = 6 элтов
    
    
    //визовы вирт функций
    m1->print();
    cout << "След: " << m1->sled() << endl << endl;
     
    m2-> print();
    cout << "След: " << m2->sled() << endl << endl;
    
    //проверяем счеткик
    cout << "Элементов после создания: " << Matrix::getCountEl() << endl;
    
    //очистка
    delete m1;
    cout << "Колво элементов после удаления первой матрицы: " << Matrix::getCountEl() << endl;
    
    delete m2;
    cout << "Колво элементов после удаления второй матрицы: " << Matrix::getCountEl() << endl;
    
    return 0;
}
```
```text
C)  Создайте шаблонный класс множества матриц различных видов,
 множества символов чисел. Продемонстрируйте работу класса и все его
 операции.
```
```cpp
#include <iostream>

using namespace std;

class Matrix
{
private:
    int colums, rows;
    static int countEl;
    
protected:
    int Get_colums() const { return colums; }
    int Get_Stroki() const { return rows; }
    
public:
    Matrix(int r, int c)
    {
        this->rows = r;
        this->colums = c;
        countEl += this->rows * this->colums;
    }
    
    virtual ~Matrix()
    {
        countEl -= colums * rows;
    }
    
    static int getCountEl() { return countEl; }
    
    virtual void print() const = 0;
    virtual double sled() const = 0;
};

class Diagonal_Matrix : public Matrix
{
private:
    double* diag;
public:
    Diagonal_Matrix(int n, const double* values) : Matrix(n, n)
    {
        diag = new double[n];
        for (int i = 0; i < n; i++)
        {
            diag[i] = values[i];
        }
    }
    
    ~Diagonal_Matrix() override
    {
        delete[] diag;
    }
    
    void print() const override
    {
        int n = Get_Stroki();
        cout << "Диагональная матрица (" << n << "x" << n << "):" << endl;
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (i == j) cout << diag[i] << "\t";
                else cout << "0\t";
            }
            cout << endl;
        }
    }
    
    double sled() const override
    {
        double sum = 0;
        for (int i = 0; i < Get_Stroki(); i++)
        {
            sum += diag[i];
        }
        return sum;
    }
};

class UsualMatrix : public Matrix
{
private:
    double** data;
public:
    UsualMatrix(int r, int c, const double** values) : Matrix(r, c)
    {
        data = new double*[r];
        for (int i = 0; i < r; i++)
        {
            data[i] = new double[c];
            for (int j = 0; j < c; j++)
            {
                data[i][j] = values[i][j];
            }
        }
    }
    
    ~UsualMatrix() override
    {
        for (int i = 0; i < Get_Stroki(); i++)
        {
            delete[] data[i];
        }
        delete[] data;
    }
    
    void print() const override
    {
        cout << "Обычная матрица (" << Get_Stroki() << "x" << Get_colums() << "):" << endl;
        for (int i = 0; i < Get_Stroki(); i++)
        {
            for (int j = 0; j < Get_colums(); j++)
            {
                cout << data[i][j] << "\t";
            }
            cout << endl;
        }
    }
    
    double sled() const override
    {
        double sum = 0;
        int min_dim = (Get_Stroki() < Get_colums()) ? Get_Stroki() : Get_colums();
        for (int i = 0; i < min_dim; i++)
        {
            sum += data[i][i];
        }
        return sum;
    }
};

template <typename T>
class Set
{
    T* data; // Массив элтов неизв заранее типа т
    int size;
    int cap;
    
    void resize()
    {
        cap *= 2;
        T* newData = new T[cap];
        for (int i = 0 ; i < size ; i++) newData[i] = data[i];
        delete[] data;
        data = newData;
    }
public:
    Set()
    {
        size = 0;
        cap = 5;
        data = new T[cap];
    }
    
    
    ~Set() {delete[] data;}
    
    Set(const Set& other)
    {
        size = other.size;
        cap = other.cap;
        data = new T[cap];
        for (int i = 0 ; i < size ; i++) data[i] = other.data[i];
    }
    
    Set& operator=(const Set& other)
    {
        if (this == &other) return *this;
        delete[] data;
        size = other.size;
        cap = other.cap;
        data = new T[cap];
        for (int i = 0 ; i < size ; i++) data[i] = other.data[i];
        return *this;
    }
    
    void add(T value)
    {
        if (*this > value) return;
        if (size == cap) resize();
        data[size] = value;
        size++;
    }
    
    
    // принадлежность
    bool operator>(T value) const
    {
        for (int i = 0; i < size ; i++)
        {
            if (data[i] == value) return true;
        }
        return false;
    }
    
    
    // пересечение
    Set operator*(const Set& other) const
    {
        Set result;
        for (int i = 0; i < size ; i++)
        {
            if (other > data[i]) result.add(data[i]);
        }
        return result;
    }
    
    bool operator<(const Set& other) const
    {
        for (int i = 0; i < size; i++)
        {
            if (!(other > data[i])) return false;
        }
        return true;
    }
    
    void print() const
    {
        cout << "{ ";
        for (int i = 0 ; i < size ; i++)
        {
            cout << data[i] << " ";
        }
        cout << "}" << endl;
    }
    
};


int Matrix::countEl = 0;

int main()
{
    cout << "Множество чисел  " << endl;
    
    Set<int> numSet;
    numSet.add(10);
    numSet.add(20);
    numSet.add(30);
    numSet.add(20);
    
    cout << "Числа: ";
    numSet.print();
    
    cout <<"\nМножество символов" << endl;
    
    Set<char> charSet1;
    charSet1.add('A');
    charSet1.add('B');
    charSet1.add('C');
    
    Set<char> charSet2;
    charSet2.add('B');
    charSet2.add('C');
    charSet2.add('X');
    
    cout << "Символы 1: "; charSet1.print();
    cout << "Символы 2: "; charSet2.print();
    
    Set<char> charInt = charSet1 * charSet2;
    cout << "Пересечение ( 1 *  2): ";
    charInt.print();
    
    cout << "\nМножество матриц" << endl;
    
    double diag_vals[] = {1.1 , 2.2 , 3.3};
    double row1[] = {1,2};
    double row2[] = {3,4};
    const double* ord_vals[] = {row1, row2};
    
    Matrix* m1 = new Diagonal_Matrix(3,diag_vals);
    Matrix* m2 = new UsualMatrix(2,2,ord_vals);
    
    Set<Matrix*> mastrixSet;
    mastrixSet.add(m1);
    mastrixSet.add(m2);
    
    cout << "Множество матриц (выводятся адреса указателей): " << endl;
    mastrixSet.print();
    
    delete m1;
    delete m2;
    
    return 0;
}
```
