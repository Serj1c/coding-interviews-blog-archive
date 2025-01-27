Jump Game VI
============

Jan 18, 2021 · 5 min read

Дан массив чисел `nums` и целое число `k`.

Игра начинается в начале массива. За один ход можно сделать прыжок на не более чем `k` шагов вперёд (не выходя за границу массива). Задача добраться до конца массива так, чтобы получить максимальное количество очков, которые берутся из значений массива.

Пример: `nums = [1,-1,-2,4,-7,3], k = 2`

![](/images/jump-game-vi--ex.jpg)

На рисунке отмечены «оптимальные» прыжки, которые принесут максимальное количество очков.

[Задача на LeetCode](https://leetcode.com/problems/jump-game-vi/).

Решение [#](#решение)
---------------------

Первое, что приходит в голову — жадный алгоритм. То есть искать максимум среди первых k чисел, делать прыжок туда (взяв это значение в сумму очков), и продолжать поиск с нового индекса.

На LeetCode первые тесты специально подобраны так, чтобы они проходили, и только после сдачи уже выяснилось, что это не работает.

Это отлично показывает важный принцип «сперва протестируй алгоритм, прежде чем писать код».

На собеседовании стоит подобрать контр-пример для жадинки и сразу понять, что не стоит идти этим путём. Хороший интервьюер даст контр-пример сам, если кандидат всё же решает написать жадинку, чтобы не тратить время.

Итак, наивное решение.

    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var maxResult = function(nums, k) {
      const n = nums.length;
      let sum = nums[0];
      let i = 0;
    
      while (i < n - 1) {
        // ищем максимум среди следующих k чисел
        const [value, j] = findMaxInRange(nums, i + 1, Math.min(i + k, n - 1));
        // решаем, что будем прыгать на индекс с максимальным количеством очков,
        // начиная поиск в следующий раз с этого индекса
        i = j;
        sum += value;
      }
      return sum;
    };
    
    function findMaxInRange(nums, start, end) {
      let maxValue = nums[start];
      let maxIndex = start;
    
      for (let i = start + 1; i <= end; i++) {
        if (nums[i] > maxValue) {
          maxValue = nums[i];
          maxIndex = i;
        }
      }
      return [maxValue, maxIndex];
    }
    

![](/images/jump-game-vi--greedy.jpg)

Давайте разберём контр-пример.

![](/images/jump-game-vi--greedy-fail.jpg)

Как видно, ошибка в том, что оптимальный путь лежит через -2, а не -1. В первом интервале из трёх чисел -1 оказывается наибольшим, поэтому жадный алгоритм берёт его, а не -2.

В чем главная ошибка наивного алгоритма? Он не учитывает числа, которые будут дальше. То есть не знает, что если выбрать -2, то среди следующей тройки попадётся большое положительное число, которое компенсирует этот выбор. И в данном контр-примере это как раз хорошо видно.

Ключ к решению — в исправлении именно этой ошибки.

Давайте пойдём с конца массива и подумаем над решением через динамическое программирование.

Обычно для дп выбирают ответ на вопрос задачи. В данном случае просят найти максимальной количество очков, которое можно собрать в массиве длины `n`, соответственно, начиная с базового случая (массив длины 1) нужно последовательно строить `dp`.

Пусть `dp[i + 1]` показывает _максимальное количество очков, которое можно собрать начиная с `i + 1`\-го индекса и до конца массива_. Как вычислить `dp[i]`?

> По форме задача похожа на [размен монет](/posts/coin-change.md)

    dp[i] = nums[i] + findMaxInRange(dp, i + 1, i + k)
    

Похоже на то, что и было, но в отличии от наивного алгоритма — `dp` рассчитывается с конца к началу, и правильно учитывает «хвост».

    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var maxResult = function(nums, k) {
      const n = nums.length;
      const dp = new Array(n);
      dp[n - 1] = nums[n - 1];
    
      for (let i = n - 2; i >= 0; i--) {
        // теперь максимум ищем внутри dp, а не nums!
        const max = findMaxInRange(dp, i + 1, Math.min(i + k, n - 1))
        dp[i] = nums[i] + max;
      }
      return dp[0];
    };
    
    function findMaxInRange(nums, start, end) {
      let result = nums[start];
    
      for (let i = start + 1; i <= end; i++) {
        result = Math.max(result, nums[i]);
      }
      return result;
    }
    

![](/images/jump-game-vi--tle.jpg)

На большом тесте данное решение показывает TLE. Все-таки алгоритм квадратный (`O(n^2)`), и при текущих ограничениях `1 <= nums.length, k <= 10^5` – получается долго.

Где лишняя работа?

Для поиска максимума приходится бегать туда-сюда по всему _скользящему окну_ и каждый раз искать максимум снова.

На самом деле, как только возникает слово «скользящее окно», сразу вспоминается задача [поиска максимума в окне](/posts/sliding-window-maximum.md) — ровно ту же идею можно использовать и здесь, чтобы избавиться от бутылочного горлышка.

    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var maxResult = function(nums, k) {
      const n = nums.length;
      const dp = new Array(n);
      // в очереди будем хранить индексы,
      // которые указывают на максимумальный элемент в dp
      const q = [n - 1];
      dp[n - 1] = nums[n - 1];
    
      for (let i = n - 2; i >= 0; i--) {
        // убираем лишние элементы, которые больше не в окне,
        // окно — [i, i + k]
        while (q.length > 0 && q[0] > i + k) {
          q.shift();
        }
        // в начале очереди всегда лежит индекс,
        // который указывает на максимальное значение в dp
        dp[i] = nums[i] + dp[q[0]];
        // убираем с конца очереди все элементы,
        // которые уже точно не станут максимумами
        while (q.length > 0 && dp[q[q.length - 1]] < dp[i]) {
          q.pop();
        }
        // кладём в очередь новое начало окна
        q.push(i);
      }
      return dp[0];
    };
    

Мы никогда не используем один и тот же элемент более двух раз (положить и убрать из очереди), соответсвенно сложность сокращается до `O(n)`.

![](/images/jump-game-vi--result.jpg)

Слегка можно ускориться, если написать «условно честную очередь» — чтобы удаление элементов с начала было за `O(1)` (т.е. без `shift`).

    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var maxResult = function(nums, k) {
      const n = nums.length;
      const dp = new Array(n);
      // в очереди будем хранить индексы,
      // которые указывают на максимумальный элемент в dp
      const q = [n - 1];
      dp[n - 1] = nums[n - 1];
    + let start = 0;
    
      for (let i = n - 2; i >= 0; i--) {
        // убираем лишние элементы, которые больше не в окне,
        // окно — [i, i + k]
    -   while (q.length > 0 && q[0] > i + k) {
    +   while (q.length > start && q[start] > i + k) {
    -     q.shift();
    +     start++;
        }
        // в начале очереди всегда лежит индекс,
        // который указывает на максимальное значение в dp
        dp[i] = nums[i] + dp[q[0]];
        // убираем с конца очереди все элементы,
        // которые уже точно не станут максимумами
    -   while (q.length > 0 && dp[q[q.length - 1]] < dp[i]) {
    +   while (q.length > start && dp[q[q.length - 1]] < dp[i]) {
          q.pop();
        }
        // кладём в очередь новое начало окна
        q.push(i);
      }
      return dp[0];
    };
    

![](/images/jump-game-vi--result2.jpg)

PS. Обсудить можно в [телеграм-чате](https://t.me/ctci_chat_ru) любознательных программистов. Welcome! 🤗

Подписывайтесь на [мой твитер](https://twitter.com/vitkarpov) или [канал в телеграме](https://t.me/coding_interviews), чтобы узнавать о новых разборах задач.