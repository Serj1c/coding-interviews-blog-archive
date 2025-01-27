Find Median From Data Stream
============================

Oct 19, 2020 · 3 min read

Медиана — значение, которое делит набор данных на две половины так, что значения в одной меньше, чем в другой. Есть поток данных из которого мы читаем числа, надо в любой момент найти медиану среди прочитанных чисел.

Надо реализовать следующий интерфейс:

    // получает новый элемент из потока
    void addNum(int num)
    // находит медиану среди «прошедших через addNum значений»
    double findMedian()
    

Пример:

    addNum(1)
    addNum(2)
    // среднее значение между [1,2] –> 1.5
    findMedian() -> 1.5
    addNum(3) 
    // среднее значение между [1,2,3] –> 2
    findMedian() -> 2
    

[Задача на LeetCode](https://leetcode.com/problems/find-median-from-data-stream/).

Решение [#](#решение)
---------------------

Очевидное брутфорсное решение — складывать числа в коллекцию, и, когда просят найти медиану, сортировать и находить центр. Однако, в условии не просто так сказано про поток: со временем данных в коллекции будет всё больше и больше, и сортировать её всё дольше и дольше.

По сути, нам и не нужны данные в отсортированном виде, просто у нас нет пока нет подходящей _структуры данных_, которая бы позволяла найти «центральный элемент».

Хорошая практика при решении задач — найти «бутылочное горлышко». В чем оно заключается сейчас? Когда приходит очередной число надо понять в какую половину оно относится, по определению медианы. Однако, нам не нужен строгий порядок между _всеми числами_.

Представим, что у мы каким-то образом поделили числа на две половины (спойлер: прямо так и просится слово «куча» сюда). Приходит новое число. Смотрим прищуренным взглядом на кучи и понимаем, это число явно идёт в большую или меньшую половины. Нам не надо пересортировать все числа друг относительно друга.

Есть специальная структура данных, которая так и называется **heap** (куча) — дерево, которое в корне хранит наибольшее или наименьшее значение среди всех узлов. Сложность вставки и удаления элементов: `O(log N)`.

Здесь мы и избавляемся от «бутылочного горлышка» — `log N` вместо `N * log N` при сортировке. Отношения порядка между элементами нет, важно поставить куда надо только один элемент (минимальный или максимальный).

![](/images/find-median-from-data-stream--heaps.jpg)

Это именно то, что нужно! Заводим две кучи и поддерживаем их размеры отличающимися не более чем на единицу.

Далее прямо по определению медианы. Либо размеры куч одинаковые, т.е. количество чисел чётное, и тогда надо взять вершины деревьев (за `O(1)`) и найти среднее арифметическое. Либо размеры куч отличаются на единицу, т.е. количество чисел нечётное, и тогда надо взять вершину бо́льшего дерева.

> Написать Heap отличное упражнение, но на собеседовании стоит воспользоваться стандартной библиотекой. В случае с JavaScript традиционно можно сказать «представим, что реализован следующий интерфейс». Главное обсудить как это работает.

Моя реализация вышесказанного через очереди с приоритетом на C++.

    class MedianFinder {
    private:
        // делим все числа на две кучи: maxH и minH,
        // одна держит максимальный элемент в корне,
        // а другая соответственно — минимальный
        priority_queue<int> maxH;
        priority_queue<int, vector<int>, greater<int>> minH;
    public:
        void addNum(int num) {
            // не теряя общности закинем
            // новое число в одну половину
            maxH.push(num);
            // взамен перекинем вершину
            // в другую половину
            minH.push(maxH.top());
            // не забываем удалить элемент,
            // т.к. по смыслу он
            // теперь в другой куче
            maxH.pop();
    
            // ни одна куча не должна расти быстрее другой,
            // т.к. мы кидаем элементы в minH нужно убедиться,
            // что maxH при этом не пустеет
            if (minH.size() > maxH.size()) {
                maxH.push(minH.top());
                minH.pop();
            }
        }
        
        double findMedian() {
            // чётное количество:
            // возьмём среднее арифметическое
            if (minH.size() == maxH.size()) {
                return (minH.top() + maxH.top()) / 2.0;
            }
            // нечётное количество:
            // возьмём вершину большего дерева
            // на данном этапе мы знаем,
            // что в левой должно быть ровно на 1 больше
            return maxH.top();
        }
    };
    

Красота! По-моему, довольно компактное решение.

Сложность — логарифм по времени и линия по памяти. У задачи есть два любопытных развития:

*   что если все числа в диапазоне от 0 до 100?
*   что если 99% чисел в диапазоне от 0 до 100?

> Спойлер: bucket sort

PS. Обсудить можно в [телеграм-чате](https://t.me/ctci_chat_ru) любознательных программистов. Welcome! 🤗

Подписывайтесь на [мой твитер](https://twitter.com/vitkarpov) или [канал в телеграме](https://t.me/coding_interviews), чтобы узнавать о новых разборах задач.