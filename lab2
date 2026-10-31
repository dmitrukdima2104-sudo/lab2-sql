# Звіт до лабораторної роботи №2: Створення складних SQL запитів

**Студент:** Дмитрук Дмитро (група ІПЗ-31)  
**База даних:** `technomart` (supabase — `dmitrukdima2104-sudo's Project`)

---

## 🎯 Мета роботи
Набути практичних навичок створення складних SQL запитів для аналізу даних, опанувати техніки з'єднання таблиць, використання агрегатних функцій, групування даних, підзапитів та віконних функцій у реляційних базах даних.

---

## 📑 Хід виконання роботи та результати запитів

### Рівень 1: З'єднання таблиць, агрегатні функції та базові підзапити

#### 1. INNER JOIN — Список товарів із категоріями та постачальниками
* **Опис:** Отримання детальної інформації про товари із зазначенням їхньої категорії та компанії-постачальника.
* **SQL Код:**
```sql
SELECT p.product_name, c.category_name, s.company_name AS supplier_name, p.unit_price
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN suppliers s ON p.supplier_id = s.supplier_id
ORDER BY c.category_name, p.product_name;
```
* **Скріншот результату:** `![Виклик 1.1](./виклик1.1.png)`

#### 2. LEFT JOIN — Клієнти та їхні замовлення
* **Опис:** Вибірка всіх клієнтів (включно з тими, хто не робив замовлень) з підрахунком кількості замовлень.
* **SQL Код:**
```sql
SELECT c.contact_name, c.customer_type, r.region_name,
      COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN regions r ON c.region_id = r.region_id
GROUP BY c.customer_id, c.contact_name, c.customer_type, r.region_name
ORDER BY order_count DESC;
```
* **Скріншот результату:** `![Виклик 1.2](./виклик1.2.png)`

#### 3. Множинне з'єднання (6 таблиць) — Деталізація замовлень
* **Опис:** З'єднання таблиць `orders`, `customers`, `order_items`, `products`, `categories` та `suppliers` для формування повного звіту.
* **SQL Код:**
```sql
SELECT 
    o.order_id,
    o.order_date,
    cu.contact_name AS customer_name,
    p.product_name,
    c.category_name,
    s.company_name AS supplier_name,
    oi.quantity,
    oi.unit_price
FROM orders o
JOIN customers cu ON o.customer_id = cu.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
JOIN suppliers s ON p.supplier_id = s.supplier_id
ORDER BY o.order_date DESC;
```
* **Скріншот результату:** `![Виклик 1.3](./виклик1.3.png)`

#### 4. Агрегатні функції та GROUP BY — Статистика по категоріях
* **Опис:** Підрахунок кількості товарів, середньої, мінімальної та максимальної ціни в розрізі категорій.
* **SQL Код:**
```sql
SELECT c.category_name,
      COUNT(p.product_id) AS product_count,
      AVG(p.unit_price) AS avg_price,
      MIN(p.unit_price) AS min_price,
      MAX(p.unit_price) AS max_price
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY product_count DESC;
```
* **Скріншот результату:** `![Виклик 1.4](./виклик1.4.png)`

#### 5. Фільтрація груп через HAVING — Продажі за регіонами та активні постачальники
* **Опис:** Розрахунок загального виторгу за регіонами з урахуванням знижок та фільтрація через `HAVING`. Також вибірка постачальників із кількістю товарів > 2.
* **SQL Код (продажі по регіонах):**
```sql
SELECT 
    r.region_name,
    SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS total_sales
FROM orders o
JOIN customers cu ON o.customer_id = cu.customer_id
LEFT JOIN regions r ON cu.region_id = r.region_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY r.region_name
HAVING SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) > 0
ORDER BY total_sales DESC;
```
* **SQL Код (постачальники з >2 товарами):**
```sql
SELECT 
    s.company_name,
    COUNT(p.product_id) AS products_count
FROM suppliers s
JOIN products p ON s.supplier_id = p.supplier_id
GROUP BY s.supplier_id, s.company_name
HAVING COUNT(p.product_id) > 2
ORDER BY products_count DESC;
```
* **Скріншот результату:** `![Виклик 1.5](./виклик1.5.png)`, `![Виклик 1.6](./виклик1.6.png)`

#### 6. Базові та корельовані підзапити
* **Товари дорожчі за середню ціну у своїй категорії (`виклик1.7.png`):**
```sql
SELECT p.product_name, p.unit_price, c.category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
WHERE p.unit_price > (
    SELECT AVG(p2.unit_price)
    FROM products p2
    WHERE p2.category_id = p.category_id
)
ORDER BY c.category_name, p.unit_price DESC;
```
* **Клієнти з замовленнями у 2024 році через IN (`виклик1.8.png`):**
```sql
SELECT customer_id, contact_name, customer_type
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    WHERE EXTRACT(YEAR FROM order_date) = 2024
);
```
* **Підзапит у SELECT для загальної кількості продажів (`виклик1.9.png`):**
```sql
SELECT 
    p.product_name,
    p.unit_price,
    (
        SELECT COALESCE(SUM(oi.quantity), 0)
        FROM order_items oi
        WHERE oi.product_id = p.product_id
    ) AS total_sold_quantity
FROM products p
ORDER BY total_sold_quantity DESC;
```

---

### Рівень 2: Складні з'єднання та віконні функції

#### 7. RIGHT JOIN — Аналіз категорій
* **SQL Код (`виклик2.1.png`):**
```sql
SELECT c.category_name,
      COUNT(p.product_id) AS products_count,
      COALESCE(AVG(p.unit_price), 0) AS avg_price
FROM products p
RIGHT JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY products_count DESC;
```

#### 8. Самоз'єднання (Self-Join) — Ієрархія керівників
* **SQL Код (`виклик2.2.png`):**
```sql
SELECT e1.first_name || ' ' || e1.last_name AS employee,
      e1.title AS employee_title,
      e2.first_name || ' ' || e2.last_name AS manager,
      e2.title AS manager_title
FROM employees e1
LEFT JOIN employees e2 ON e1.reports_to = e2.employee_id
ORDER BY e2.last_name, e1.last_name;
```

#### 9. Віконні функції ранжування
* **SQL Код (`виклик2.3.png`):**
```sql
SELECT p.product_name,
      c.category_name,
      p.unit_price,
      RANK() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) AS price_rank,
      DENSE_RANK() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) AS price_dense_rank,
      ROW_NUMBER() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) AS row_num
FROM products p
JOIN categories c ON p.category_id = c.category_id
ORDER BY c.category_name, p.unit_price DESC;
```

---

## ❓ Відповіді на контрольні запитання

**1. В чому різниця між INNER JOIN та LEFT JOIN? Коли доцільно використовувати кожен з них в контексті навчальної бази даних?**
* **INNER JOIN** повертає лише ті рядки, для яких є збіги в обох об'єднуваних таблицях (наприклад, товари, які обов'язково належать до існуючої категорії). 
* **LEFT JOIN** повертає всі записи з лівої таблиці та відповідні їм з правої. Якщо збігу немає, на місці полів правої таблиці підставляється `NULL` (наприклад, вивести всіх клієнтів, навіть тих, хто ще не зробив жодного замовлення, як у *Виклику 1.2*).

**2. Чому не можна використовувати WHERE для фільтрації результатів агрегатних функцій? Яка роль HAVING при аналізі продажів товарів?**
* Запит виконується послідовно: секція `WHERE` фільтрує окремі рядки **до** того, як спрацює групування (`GROUP BY`). Агрегатні функції (`SUM`, `AVG`, `COUNT`) обчислюються вже на етапі сформованих груп. Тому для фільтрації результатів агрегатних функцій застосовується виключно `HAVING`, яка працює вже **після** групування (наприклад, відібрати регіони, де загальний обсяг продажів перевищує певне значення).

**3. Як працюють корельовані підзапити і чим вони відрізняються від звичайних? Наведіть приклад для знаходження товарів з ціною вище середньої по категорії.**
* Звичайний підзапит виконується незалежно один раз і передає результат зовнішньому запиту. **Корельований підзапит** виконується заново для кожного рядка зовнішнього запиту, оскільки він посилається на значення з цього зовнішнього рядка (через псевдонім таблиці). 
* *Приклад (з Виклику 1.7):*
  ```sql
  SELECT p1.product_name, p1.unit_price
  FROM products p1
  WHERE p1.unit_price > (
      SELECT AVG(p2.unit_price) FROM products p2 WHERE p2.category_id = p1.category_id
  );
  ```

**4. В яких ситуаціях доцільно використовувати віконні функції замість GROUP BY при аналізі замовлень клієнтів?**
* `GROUP BY` групує та «зхлопує» рядки, через що втрачається детальна інформація по кожному окремому замовленню чи товару. Віконні функції (наприклад, `RANK()`, `SUM() OVER(PARTITION BY...)`) дозволяють виконувати аналітичні обчислення (суми, ранги, порівняння), зберігаючи при цьому всі початкові рядки незмінними. Це зручно, коли поряд із деталями замовлення потрібно бачити загальний підсумок або місце замовлення в рейтингу.

**5. Що таке матеріалізовані представлення і які переваги вони дають для аналітичних запитів по продажах у навчальній базі даних?**
* **Матеріалізоване представлення (Materialized View)** — це віртуальна таблиця, яка не просто зберігає логіку запиту, а фізично зберігає результати його виконання на диску. Перевага полягає в тому, що складні аналітичні агрегації по великій кількості замовлень та позицій обчислюються лише в момент створення або оновлення (`REFRESH`), а не щоразу при зверненні користувача, що суттєво прискорює роботу системи.

**6. Як можна оптимізувати запити з множинними з'єднаннями таблиць orders, order_items, products, customers для покращення продуктивності?**
* Основні шляхи оптимізації:
  1. Створення індексів на зовнішніх ключах (`customer_id`, `product_id`, `order_id`).
  2. Фільтрація даних за допомогою `WHERE` якомога раніше (до або під час з'єднань), щоб зменшити обсяг оброблюваних даних.
  3. Використання `EXPLAIN ANALYZE` для аналізу плану виконання запиту та виявлення вузьких місць (наприклад, переходу від сканування індексу до повного сканування таблиці).

**7. Яка різниця між RANK() та DENSE_RANK() при ранжуванні товарів за ціною? Наведіть приклад з конкретними даними.**
* `RANK()` проставляє ранги з пропусками, якщо є однакові значення (наприклад: 1, 2, 2, 4).
* `DENSE_RANK()` проставляє ранги підряд без пропусків (наприклад: 1, 2, 2, 3).
* *Приклад:* Якщо ціни товарів складають `1000, 800, 800, 500`:
  * `RANK()` дасть: `1, 2, 2, 4` (номер 3 пропущено через дублювання другого місця).
  * `DENSE_RANK()` дасть: `1, 2, 2, 3` (без пропусків).

---

## 🎉 Висновок
Під час виконання лабораторної роботи було успішно освоєно складні SQL-запити, реалізовано багаторівневі з'єднання таблиць, групування, підзапити та віконні функції. Усі результати перевірені та задокументовані за допомогою практичних скріншотів у середовищі PostgreSQL.
