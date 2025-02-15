# Розбір фінансування медичних установ у 2023 році 
# Зміст
 - [Опис проекту](#Опис-проекту)
 - [Імпорт та огляд вихідного файлу CSV](#Імпорт-та-огляд-вихідного-файлу-CSV)
 - [Обробка даних на SQL з візуалізацією даних у BI-системі та висновки](#Обробка-даних-на-SQL-з-візуалізацією-даних-у-BI-системі-та-висновки)
   - [Загальний обсяг фінансування у 2023 році](#Загальний-обсяг-фінансування-у-2023-році)
   - [Топ-5 медичних установ за сумою виплат](#Топ-5-медичних-установ-за-сумою-виплат)
   - [Динаміка виплат за місяцями](#Динаміка-виплат-за-місяцями)
   - [Ковзна середня виплат за 3 місяці](#Ковзна-середня-виплат-за-3-місяці)
   - [Частка виплат кожної установи від загальної суми](#Частка-виплат-кожної-установи-від-загальної-суми)
   - [Динаміка виплат за місяцями у КНП ММР №1 м.Миколаїв](#динаміка-виплат-за-місяцями-у-кнп-ммр-1-ммиколаїв)
 - [Заключення](#Заключення)
   - Підсумки виконаної роботи
 
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

# Обробка даних на SQL з візуалізацією даних у BI-системі та висновки
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

# Топ-5 медичних установ за сумою виплат
```SQL 
SELECT 
    legal_entity_name, 
    ROUND(SUM(pay_all),2) AS total_payment,
    RANK() OVER (ORDER BY SUM(pay_all) DESC) AS payment_rank
FROM payments
GROUP BY legal_entity_name
ORDER BY 3 ASC;
```
Результат:
| legal_entity_name  | total_payments | payment_rank|
|--------------------|----------------| ------------|
|КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО "ЛЬВІВСЬКЕ ТЕРИТОРІАЛЬНЕ МЕДИЧНЕ ОБЄДНАННЯ" БАГАТОПРОФІЛЬНА КЛІНІЧНА ЛІКАРНЯ ІНТЕНСИВНИХ МЕТОДІВ ЛІКУВАННЯ ТА ШВИДКОЇ МЕДИЧНОЇ ДОПОМОГИ"                 | 22290627674.78   | 1|
| КП "ДНІПРОПЕТРОВСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ ІМ. І.І. МЕЧНИКОВА" ДНІПРОПЕТРОВСЬКОЇ ОБЛАСНОЇ РАДИ"| 9857854499.28   | 2 |
| "ПОЛТАВСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ ІМ. М.В.СКЛІФОСОВСЬКОГО ПОЛТАВСЬКОЇ ОБЛАСНОЇ РАДИ"| 8260268009.33      | 3 |
| КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО "МІСЬКА КЛІНІЧНА ЛІКАРНЯ № 4" ДНІПРОВСЬКОЇ МІСЬКОЇ РАДИ|6197921656.63 | 4 |
| "ВОЛИНСЬКА ОБЛАСНА КЛІНІЧНА ЛІКАРНЯ" ВОЛИНСЬКОЇ ОБЛАСНОЇ РАДИ| 6186519065.99 | 5 |

**ВИСНОВОК**: Бачимо топ медичних установ з найбільшим обсягом фінансування у 2023 році. 

# **Динаміка виплат за місяцями**
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

![Tableau Dashboard](https://github.com/ImranAmietov/Pet-project1/blob/5a75d80713c46c3839ba12c2df498390107d4a1d/%D0%94%D0%B8%D0%BD%D0%B0%D0%BC%D1%96%D0%BA%D0%B0%20%D0%B2%D0%B8%D0%BF%D0%BB%D0%B0%D1%82%20%D0%B7%D0%B0%20%D0%BC%D1%96%D1%81%D1%8F%D1%86%D1%8F%D0%BC%D0%B8.png)

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

![Tableau Dashboard](https://github.com/ImranAmietov/Pet-project1/blob/30194e3075cd7196623a089926c1a3dd1223ff24/%D0%92%D0%B8%D0%BF%D0%BB%D0%B0%D1%82%D0%B8%20%D0%B7%D0%B0%20%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%B0%D0%BC%D0%B8%20%D0%BC%D0%B5%D0%B4%D0%B8%D1%87%D0%BD%D0%B8%D1%85%20%D0%BF%D0%BE%D1%81%D0%BB%D1%83%D0%B3.png)

**ВИСНОВОК**: Надання вторинної,третинної,паліат.мед.доп.,мед.реаб-ії, мед.доп.дітям до 16  р, вагітність і пологи є найбільш оплачуваним пакетом медичних послуг.

# **Частка виплат кожної установи від загальної суми**
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

**ВИСНОВОК**: КОМУНАЛЬНЕ НЕКОМЕРЦІЙНЕ ПІДПРИЄМСТВО " ЛЬВІВСЬКЕ ТЕРИТОРІАЛЬНЕ МЕДИЧНЕ ОБЄДНАННЯ " БАГАТОПРОФІЛЬНА КЛІНІЧНА ЛІКАРНЯ ІНТЕНСИВНИХ МЕТОДІВ ЛІКУВАННЯ ТА ШВИДКОЇ МЕДИЧНОЇ ДОПОМОГИ" топ заклад охорони здоров'я за часткою виплат від загальної суми.

# Динаміка виплат за місяцями у КНП ММР №1 м.Миколаїв
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

![Tableau Dashboard](https://github.com/ImranAmietov/Pet-project1/blob/beb57f098415e0e764d73f94e123393ef8c3641d/%D0%94%D0%B8%D0%BD%D0%B0%D0%BC%D1%96%D0%BA%D0%B0%20%D0%B2%D0%B8%D0%BF%D0%BB%D0%B0%D1%82%20%D0%B7%D0%B0%20%D0%BC%D1%96%D1%81%D1%8F%D1%86%D1%8F%D0%BC%D0%B8%20%D1%83%20%D0%9A%D0%9D%D0%9F%20%D0%9C%D0%9C%D0%A0%20%D0%BC.%D0%9C%D0%B8%D0%BA%D0%BE%D0%BB%D0%B0%D1%97%D0%B2.png)

**ВИСНОВОК**: Причина невиплати заробітної плати за січень у миколаївській лікарні недофінансування закладу охорони здоров'я у січні. Також цікава тенденція з виплатами лікарям інтернам, які різко зменшуються у липні. Ця тенденція прослідуються через дві причини: 1.В липні закінчують інтернатуру спеціальності з роками навчання 2 чи 3 роки. 2.Липень є місяцем відпустки на інтернатурі. 

# Заключення
**Підсумки виконаної роботи**
 - Фінансування розподілено між різними установами, але з переважною концентрацією великих організацій.
 - Динаміка виплат показує, що в медицині сезонні коливання не є тенденцією.
 - Є певні закономірності у розподілі виплат за категоріями послуг та регіонами.
 - Головна проблема: відсутність фінансування у січні місяці багатьох установ охорони здоров'я.





