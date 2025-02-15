# Розбір фінансування медичних установ у 2023 році. 
# Зміст
 - Опис проекту
 - Імпорт та огляд вихідного файлу CSV
 - Обробка даних на SQL
   - Загальний обсяг фінансування у 2023 році
   - Топ-5 медичних установ за сумою виплат
   - Ковзна середня виплат за 3 місяці
   - Виплати за пакетами медичних послуг
   - Частка (у відсотках) виплат кожної установи від загальної суми
   - Динаміка виплат за місяцями у КНП ММР №1 м.Миколаїв
 - Візуалізація даних у BI-системі та висновки
# Опис проекту
Цей пет-проект присвячений аналізу виплат медичним установам за програмою медичних гарантій за 2023 рік. В рамках проекту буде вивчено структуру фінансування, визначено ключові тенденції та проведено візуалізацію даних за допомогою BI-інструментів.    
**Цілі проекту:**
 - Проаналізувати розподіл виплат між медичними установами.
 - Визначити основні отримувачі фінансування та типи послуг.
 - Дослідити динаміку виплат за періодами.
 - Виявити аномалії чи цікаві закономірності у даних.
 - Підготувати візуалізації, які допоможуть краще зрозуміти фінансування медицини у 2023 році.
 - Причина невиплати заробітної плати за січень у миколаївській лікарні.
# Імпорт та огляд вихідного файлу CSV
 - legal_entity_edrpou – код ЄДРПОУ організації
 - legal_entity_name – назва медичного закладу
 - period_month, period_year – місяць та рік платежу
 - package_id – ідентифікатор пакета послуг
 - pay_package – сума платежу за пакетом
 - pay_all – загальна сума платежу
 - contract_number – номер контракту
 - pay_date – дата платежу
 - pay_type, kekv – тип платежу і класифікація витрат
 - referral – опис послуги

# Обробка даних на SQL
# **Загальний обсяг фінансування у 2023 році**
```SQL 
SELECT 
    period_year, 
    ROUND(SUM(pay_all),2) AS total_payment
FROM payments
GROUP BY period_year;
```
Результат:
| period_year  | total_payment|
|-------------| ------------|
| 2023    | 652477823252.08|

# **Топ-5 медичних установ за сумою виплат у 2023 (Ранжування виплат по установам)**
```SQL 
SELECT 
    legal_entity_name, 
    SUM(pay_all) AS total_payment,
    RANK() OVER (ORDER BY SUM(pay_all) DESC) AS payment_rank
FROM payments
GROUP BY legal_entity_name
ORDER BY 3 ASC;
```
Результат:
| legal_entity_name  | total_payments | payment_rank|
|--------------------|----------------| ------------|
|КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО "ЛЬВІВСЬКЕ ТЕРИТОРІАЛЬНЕ МЕДИЧНЕ ОБЄДНАННЯ" БАГАТОПРОФІЛЬНА КЛІНІЧНА ЛІКАРНЯ ІНТЕНСИВНИХ МЕТОДІВ ЛІКУВАННЯ ТА ШВИДКОЇ МЕДИЧНОЇ ДОПОМОГИ"                 | 22290627674.779972     | 1|
| КП "ДНІПРОПЕТРОВСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ ІМ. І.І. МЕЧНИКОВА" ДНІПРОПЕТРОВСЬКОЇ ОБЛАСНОЇ РАДИ"| 9857854499.2800045   | 2 |
| "ПОЛТАВСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ ІМ. М.В.СКЛІФОСОВСЬКОГО ПОЛТАВСЬКОЇ ОБЛАСНОЇ РАДИ"| 8260268009.3300133      | 3 |
| КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО "МІСЬКА КЛІНІЧНА ЛІКАРНЯ № 4" ДНІПРОВСЬКОЇ МІСЬКОЇ РАДИ|6197921656.629981 | 4 |
| "ВОЛИНСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ" ВОЛИНСЬКОЇ ОБЛАСНОЇ РАДИ| 6186519065.9900045 | 5 |
**ВИСНОВОК**: Бачимо топ медичних установ з найбільшим обсягом фінансування у 2023 році. 

# **Динаміка виплат за місяцями (Як змінювалися виплати за місяцями.)**
```SQL 
SELECT 
    period_year, 
    period_month, 
    ROUND(SUM(pay_all),2) AS total_payment
FROM payments
GROUP BY period_year, period_month
ORDER BY 1, 2 ;
```
Результат:
| period_year  | period_month | total_payment|
|--------------------|----------------| ------------|
| 2023| 1  | 46505525970.75|
| 2023| 2  | 47499380533.38 |
| 2023| 3  | 51529774515.43|
| 2023| 4  | 54612141580.17|
| 2023| 5  | 54100574965.56 |
|...|...|...|

# **Ковзна середня виплат за 3 місяці**
```SQL 
SELECT 
    period_year, 
    period_month, 
    ROUND(SUM(pay_all),2) AS monthly_payment,
    AVG(SUM(pay_all)) OVER (
        ORDER BY period_year, period_month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg
FROM payments
GROUP BY period_year, period_month
ORDER BY period_year, period_month;
```
Результат:
| period_year  | period_month | monthly_payment| moving_avg  |
|--------------------|----------------| ------------| ------------|
| 2023| 1  | 46505525970.75| 46505525970.749794 |
| 2023| 2  | 47499380533.38 | 47002453252.0648 |
| 2023| 3  | 51529774515.43| 48511560339.853287 |
| 2023| 4  | 54612141580.17| 51213765542.99324 |
| 2023| 5  | 54100574965.56 |  53414163687.053207 |
|...|...|...| ... |

 # **Виплати за пакетами медичних послуг**
```SQL 
SELECT  
    referral, 
    SUM(pay_all) AS total_payment
FROM payments
GROUP BY package_id, referral
ORDER BY total_payment DESC;
```
Результат:
| referral | total_payment| 
|----------------| ------------| 
| Надання вторинної,третинної,паліат.мед.доп.,мед.реаб-ії, мед.доп.дітям до 16  р, вагітність і пологи| 606902180039.84| 
| Надання первинної медичної допомоги | 23363031893.84 | 
| Надання екстреної медичної допомоги  | 11155252995.2 | 
| Проведення розрахунків за надані медичні послуги за грудень 2023 року | 1584447433.17| 
| Забезпечення збереження кадрового потенціалу для мед. допомоги населенню, яке перебуває на ТОТ  | 54100574965.56 |  
|Забезпечення кадрового потенціалу СОЗ, надання медичної допомоги із залученням лікарів-інтернів|598057873.92| 

# **Частка (у відсотках) виплат кожної установи від загальної суми**
```SQL 
SELECT 
    legal_entity_name, 
    SUM(pay_all) AS total_payment,
    ROUND(SUM(pay_all) * 100.0 / SUM(SUM(pay_all)) OVER (),2) AS percentage_of_total
FROM payments
GROUP BY legal_entity_name
ORDER BY percentage_of_total DESC;
```
Результат:
| legal_entity_name  | total_payment | percentage_of_total| 
|--------------------|----------------| ------------| 
| КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО " ЛЬВІВСЬКЕ ТЕРИТОРІАЛЬНЕ МЕДИЧНЕ ОБЄДНАННЯ " БАГАТОПРОФІЛЬНА КЛІНІЧНА ЛІКАРНЯ ІНТЕНСИВНИХ МЕТОДІВ ЛІКУВАННЯ ТА ШВИДКОЇ МЕДИЧНОЇ ДОПОМОГИ"| 22290627674.78  | 3.42| 
| КП "ДНІПРОПЕТРОВСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ ІМ. І.І. МЕЧНИКОВА" ДНІПРОПЕТРОВСЬКОЇ ОБЛАСНОЇ РАДИ"| 9857854499.28 | 1.51 |
| "ПОЛТАВСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ ІМ. М.В.СКЛІФОСОВСЬКОГО ПОЛТАВСЬКОЇ ОБЛАСНОЇ РАДИ" |8260268009.33 | 1.27| 
| "ВОЛИНСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ" ВОЛИНСЬКОЇ ОБЛАСНОЇ РАДИ| 6186519065.99  | 0.95| 
| КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО "МІСЬКА КЛІНІЧНА ЛІКАРНЯ № 4" ДНІПРОВСЬКОЇ МІСЬКОЇ РАДИ| 6197921656.63  | 0.95 | 
|...|...|...|

**ВИСНОВОК**: Заклади охорони здоров'я відсортовані за часткою виплат від загальної суми.

# **Динаміка виплат за місяцями у КНП ММР №1 м.Миколаїв**
```SQL 
SELECT DISTINCT(REGEXP_REPLACE(legal_entity_name, '^КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО','КНП')),
 referral, pay_date, ROUND(SUM(pay_all),2) as total_payment
FROM payments 
WHERE legal_entity_name LIKE "%МИКОЛАЇВСЬКОЇ_МІСЬКОЇ_РАДИ__МІСЬКА_ЛІКАРНЯ_№1%"
GROUP BY legal_entity_name, referral, pay_date
ORDER BY pay_date ASC;
```
Результат:
| legal_entity_name  | referral | pay_date | total_payment |
|--------------------|----------------| ------------| ------------|
| КНП МИКОЛАЇВСЬКОЇ МІСЬКОЇ РАДИ "МІСЬКА ЛІКАРНЯ №1"| Надання вторинної,третинної,паліат.мед.доп.,мед.реаб-ії, мед.доп.дітям до 16  р, вагітність і пологи| 2023-02-14| 6355930.53| 
| КНП МИКОЛАЇВСЬКОЇ МІСЬКОЇ РАДИ "МІСЬКА ЛІКАРНЯ №1"| Надання вторинної,третинної,паліат.мед.доп.,мед.реаб-ії, мед.доп.дітям до 16  р, вагітність і пологи | 2023-02-28 | 30339766.77|
| КНП МИКОЛАЇВСЬКОЇ МІСЬКОЇ РАДИ "МІСЬКА ЛІКАРНЯ №1"| Забезпечення кадрового потенціалу СОЗ, надання медичної допомоги із залученням лікарів-інтернів| 2023-03-01| 57218.0 |
| КНП МИКОЛАЇВСЬКОЇ МІСЬКОЇ РАДИ "МІСЬКА ЛІКАРНЯ №1"| Надання вторинної,третинної,паліат.мед.доп.,мед.реаб-ії, мед.доп.дітям до 16  р, вагітність і пологи  | 2023-03-08| 6355930.53|
| КНП МИКОЛАЇВСЬКОЇ МІСЬКОЇ РАДИ "МІСЬКА ЛІКАРНЯ №1"| Забезпечення кадрового потенціалу СОЗ, надання медичної допомоги із залученням лікарів-інтернів| 2023-03-09 | 57218.0 |
|...|...|...| ... |

**ВИСНОВОК**: недофінансування закладу охорони здоров'я у січні.
## Интерактивный дашборд

[▶ Открыть Tableau Dashboard](https://github.com/ImranAmietov/Pet-project1/blob/bcdd0d3fb2ad7e2e21034aff2284a7e115d2040f/tableau.html).





# Візуалізація даних у BI-системі та висновки




