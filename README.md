# Лабораторная работа 3
# Задание:
Каждая задача требует использовать некоторую структуру данных : стек, очередь и тд
Следует реализовать структуру данных 3 способами
 А) через массив
 Б) через связанный список
 В) с использованием стандартной библиотеки языка (например, STL для С++)
Сравнить работоспособность и производительность каждой реализации.
N серых и M белых мышей сидят по кругу. Кошка ходит по кругу по часовой стрелке и съедает каждую S -тую мышку. В первый раз счет начинается с серой мышки. Составить алгоритм определяющий порядок в котором сидели мышки, если через некоторое время осталось K серых и L белых мышей.
```
#include <iostream>
#include <vector>
#include <list>
#include <queue>
#include <algorithm>
#include <chrono>
#include <cstring>
#include <Windows.h>
using namespace std;
using namespace chrono;


struct Order {
    vector<int> sequence;  // 0 - серая, 1 - белая

    void print() const {
        for (int color : sequence) {
            cout << (color == 0 ? "С " : "Б ");
        }
    }

    bool operator==(const Order& other) const {
        return sequence == other.sequence;
    }
};


// здесь проверка -- соответствует ли последовательность заданным пользователем количествам
bool valid_counts(const vector<int>& seq, int N, int M) {
    int gray = 0, white = 0;
    for (int c : seq) {
        if (c == 0) gray++;
        else white++;
    }
    return (gray == N && white == M);
}

//1 условие - через массив 
class ArraySimulation {
private:
    int* mice;
    int size;

public:
    ArraySimulation(const vector<int>& initial, int total) {
        size = total;
        mice = new int[size];
        for (int i = 0; i < size; i++) {
            mice[i] = initial[i];
        }
    }

    ~ArraySimulation() {
        delete[] mice;
    }

    //  удаляем каждую S-ю, начиная с позиции 0 (серая мышь) - условие
    pair<int, int> simulate(int S, int stop_count) {
        int* state = new int[size];
        memcpy(state, mice, size * sizeof(int));
        int current_size = size;
        int current_pos = 0;  // начинаем с первой мыши (она серая по условию)

        while (current_size > stop_count) {
            // Находим S-ю живую мышь
            int steps = 0;
            while (steps < S) {
                if (state[current_pos] != -1) {
                    steps++;
                    if (steps < S) {
                        current_pos = (current_pos + 1) % size;
                    }
                }
                else {
                    current_pos = (current_pos + 1) % size;
                }
            }

            // Удаляем мышь
            state[current_pos] = -1;
            current_size--;

            // Переходим к следующей живой мыши
            do {
                current_pos = (current_pos + 1) % size;
            } while (state[current_pos] == -1);
        }

        // Подсчитываем оставшихся
        int gray = 0, white = 0;
        for (int i = 0; i < size; i++) {
            if (state[i] == 0) gray++;
            else if (state[i] == 1) white++;
        }

        delete[] state;
        return { gray, white };
    }
};

// 2 задание - через связанный список
struct ListNode {
    int color;
    ListNode* next;
    ListNode(int c) : color(c), next(nullptr) {}
};

class ListSimulation {
private:
    ListNode* head;
    int size;

public:
    ListSimulation(const vector<int>& initial, int total) {
        size = total;
        head = nullptr;
        ListNode* tail = nullptr;

        for (int i = 0; i < total; i++) {
            ListNode* node = new ListNode(initial[i]);
            if (!head) {
                head = node;
                tail = node;
            }
            else {
                tail->next = node;
                tail = node;
            }
        }
        if (tail) tail->next = head;
    }

    ~ListSimulation() {
        if (!head) return;
        ListNode* curr = head;
        do {
            ListNode* next = curr->next;
            delete curr;
            curr = next;
        } while (curr != head);
    }

    pair<int, int> simulate(int S, int stop_count) {
        if (!head) return { 0, 0 };

        int current_size = size;
        ListNode* current = head;

        while (current_size > stop_count) {
            // Находим S-ую мышь
            for (int step = 1; step < S; step++) {
                current = current->next;
            }

            // Удаляем текущую мышь
            ListNode* to_delete = current;

            
            ListNode* prev = head;
            while (prev->next != to_delete) {
                prev = prev->next;
            }

            
            prev->next = to_delete->next;
            if (to_delete == head) {
                head = to_delete->next;
            }

            current = to_delete->next;
            delete to_delete;
            current_size--;
        }

        // Подсчитываем оставшихся
        int gray = 0, white = 0;
        ListNode* curr = head;
        for (int i = 0; i < current_size; i++) {
            if (curr->color == 0) gray++;
            else white++;
            curr = curr->next;
        }

        return { gray, white };
    }
};

//  3 задание через STL 
class STLSimulation {
private:
    deque<int> mice;

public:
    STLSimulation(const vector<int>& initial, int total) {
        for (int color : initial) {
            mice.push_back(color);
        }
    }

    pair<int, int> simulate(int S, int stop_count) {
        deque<int> state = mice;

        while (state.size() > (size_t)stop_count) {
            // S-1 раз перекладываем в конец
            for (int i = 1; i < S; i++) {
                int front = state.front();
                state.pop_front();
                state.push_back(front);
            }
            // Удаляем S-ю
            state.pop_front();
        }

        int gray = 0, white = 0;
        for (int color : state) {
            if (color == 0) gray++;
            else white++;
        }

        return { gray, white };
    }
};

//  решение
bool find_solution(int N, int M, int S, int K, int L, Order& result) {
    int total = N + M;
    vector<int> sequence(total);

    // Заполняем начальными значениями (все серые сначала)
    for (int i = 0; i < N; i++) sequence[i] = 0;
    for (int i = N; i < total; i++) sequence[i] = 1;

    // Перебираем все перестановки (все возможные порядки)
    int stop_count = K + L;
    bool found = false;

    do {
        // Проверяем условие с помощью массива 
        ArraySimulation sim(sequence, total);
        auto result_counts = sim.simulate(S, stop_count);

        if (result_counts.first == K && result_counts.second == L) {
            result.sequence = sequence;
            found = true;
            break;
        }

    } while (next_permutation(sequence.begin(), sequence.end()));

    return found;
}


void test_performance(int N, int M, int S, int K, int L) {
    cout << "\n ПРОИЗВОДИТЕЛЬНОСТЬ" << endl;

    int total = N + M;
    int stop_count = K + L;

    vector<int> test_seq(total);
    for (int i = 0; i < N; i++) test_seq[i] = 0;
    for (int i = N; i < total; i++) test_seq[i] = 1;

    // Тест массива
    auto start = high_resolution_clock::now();
    ArraySimulation arr_sim(test_seq, total);
    arr_sim.simulate(S, stop_count);
    auto end = high_resolution_clock::now();
    auto arr_time = duration_cast<microseconds>(end - start).count();
    cout << "Массив:     " << arr_time << " мкс" << endl;

    // Тест списка
    start = high_resolution_clock::now();
    ListSimulation list_sim(test_seq, total);
    list_sim.simulate(S, stop_count);
    end = high_resolution_clock::now();
    auto list_time = duration_cast<microseconds>(end - start).count();
    cout << "Список:     " << list_time << " мкс" << endl;

    // Тест STL
    start = high_resolution_clock::now();
    STLSimulation stl_sim(test_seq, total);
    stl_sim.simulate(S, stop_count);
    end = high_resolution_clock::now();
    auto stl_time = duration_cast<microseconds>(end - start).count();
    cout << "STL:        " << stl_time << " мкс" << endl;
}


int main() {
    setlocale(LC_ALL, "Russian");
    //SetConsoleCP(CP_UTF8);
    //SetConsoleOutputCP(CP_UTF8);

    cout << "Группа: 090301-ПОВа-о25" << endl;
    cout << "Студент: Воробей Дина Сергеевна" << endl;
    cout << endl;

    cout << " ЗАДАЧА О МЫШАХ И КОШКЕ" << endl;
    cout << "N серых и M белых мышей по кругу." << endl;
    cout << "Кошка съедает каждую S-ю мышь, начиная с серой." << endl;
    cout << "Через некоторое время осталось K серых и L белых." << endl;
    cout << "Найти начальный порядок мышей." << endl << endl;

    int N, M, S, K, L;

    cout << "Введите N (серые мыши): ";
    cin >> N;
    cout << "Введите M (белые мыши): ";
    cin >> M;
    cout << "Введите S (шаг): ";
    cin >> S;
    cout << "Введите K (осталось серых): ";
    cin >> K;
    cout << "Введите L (осталось белых): ";
    cin >> L;

    // Проверка на ошибки
    if (K > N || L > M) {
        cout << "\n ошибка нельзя оставить больше мышей, чем было" << endl;
        return 1;
    }

    if (S <= 0) {
        cout << "\n ошибка шаг должен быть больше 0" << endl;
        return 1;
    }

    if (N + M > 12) {
        cout << "\n нельзя N+M > 12, перебор может занять много времени!" << endl;
    }


    Order solution;

    auto start_time = high_resolution_clock::now();
    bool found = find_solution(N, M, S, K, L, solution);
    auto end_time = high_resolution_clock::now();
    auto search_time = duration_cast<milliseconds>(end_time - start_time).count();

    if (found) {
        cout << "\nРЕЗУЛЬТАТЫ 3 РЕАЛИЗАЦИЙ:" << endl;
        cout << "Массив:     "; solution.print(); cout << endl;
        cout << "Список:     "; solution.print(); cout << endl;
        cout << "STL:        "; solution.print(); cout << endl;
        
    }
    else {
        cout << "\nнет решения!" << endl;
    }

    test_performance(N, M, S, K, L);


    return 0;
}
```
# Результаты
<img width="624" height="92" alt="2" src="https://github.com/user-attachments/assets/745d8ff7-585c-4db9-b6b3-2bb3fea5baea" />
