# Фінальний проект

## 1. Завантажте дані:
### a) Створіть схему pandemic у базі даних за допомогою SQL-команди.
```
CREATE SCHEMA pandemic;
```
### b) Оберіть її як схему за замовчуванням за допомогою SQL-команди.
```
 USE pandemic;
```
<img width="1228" alt="p1_1-2" src="https://github.com/user-attachments/assets/f6bc7893-716f-4a92-a30d-b1816ca4992c">

### c) Імпортуйте дані за допомогою Import wizard так, як ви вже робили це у темі 3.
<img width="1229" alt="p1_3" src="https://github.com/user-attachments/assets/67508b2d-d785-410f-9ae8-871a400cbb9b">

### d) Продивіться дані, щоб бути у контексті.
```
SELECT * FROM infectious_cases
LIMIT 10;
```
```
SELECT DISTINCT Entity, Code
FROM infectious_cases;
```
<img width="943" alt="p1_4" src="https://github.com/user-attachments/assets/e13f18d0-ebc1-4ce4-8dcf-d1b9cd3343a1">

## 2. Нормалізуйте таблицю infectious_cases до 3ї нормальної форми. Збережіть у цій же схемі дві таблиці з нормалізованими даними.
```
CREATE TABLE entities ( id INT AUTO_INCREMENT PRIMARY KEY, Entity VARCHAR(255), Code VARCHAR(10) );
```
<img width="1229" alt="p2_1" src="https://github.com/user-attachments/assets/5281457d-8024-412f-8881-820da55d38bc">

```
INSERT INTO entities (Entity, Code)
SELECT DISTINCT Entity, Code
FROM infectious_cases;
```
<img width="1225" alt="p2_2" src="https://github.com/user-attachments/assets/ce6b5ecd-a4d9-474f-a59b-7e92bf11064d">

```
ALTER TABLE infectious_cases ADD entity_id INT;
```
```
SET SQL_SAFE_UPDATES = 0;
UPDATE infectious_cases
SET entity_id = (
    SELECT id FROM entities
    WHERE entities.Entity = infectious_cases.Entity
    AND entities.Code = infectious_cases.Code
);
SET SQL_SAFE_UPDATES = 1;
```
<img width="1009" alt="p2_3" src="https://github.com/user-attachments/assets/17183691-c574-40b0-92a2-370731bd485d">

```
ALTER TABLE infectious_cases
DROP COLUMN Entity,
DROP COLUMN Code;
```
<img width="1008" alt="p2_4" src="https://github.com/user-attachments/assets/8df75d3b-91b1-471c-9db7-8b268a195bf4">

### Результат (entities)
<img width="1072" alt="p2_result_entities" src="https://github.com/user-attachments/assets/71322bef-171b-4b37-aa63-9f51195be5f4">

### Результат (infectious_cases)
<img width="1069" alt="p2_result_3NF" src="https://github.com/user-attachments/assets/89114dbd-432e-4efe-b332-8993e7a01877">

## 3. Проаналізуйте дані:
Для кожної унікальної комбінації Entity та Code або їх id порахуйте середнє, мінімальне, максимальне значення та суму для атрибута Number_rabies.
Результат відсортуйте за порахованим середнім значенням у порядку спадання.
Оберіть тільки 10 рядків для виведення на екран.
```
SELECT 
    e.Entity, 
    e.Code, 
    AVG(ic.Number_rabies) AS avg_rabies,
    MIN(ic.Number_rabies) AS min_rabies,
    MAX(ic.Number_rabies) AS max_rabies,
    SUM(ic.Number_rabies) AS sum_rabies
FROM infectious_cases ic
JOIN entities e ON ic.entity_id = e.id
WHERE ic.Number_rabies IS NOT NULL AND ic.Number_rabies != ''
GROUP BY e.Entity, e.Code
ORDER BY avg_rabies DESC
LIMIT 10;
```
<img width="1003" alt="p3" src="https://github.com/user-attachments/assets/b074fb3d-66e9-4e68-b6cb-7db1555d4d56">

## 4. Побудуйте колонку різниці в роках.
### Для оригінальної або нормованої таблиці для колонки Year побудуйте з використанням вбудованих SQL-функцій:
- атрибут, що створює дату першого січня відповідного року,
- атрибут, що дорівнює поточній даті,
- атрибут, що дорівнює різниці в роках двох вищезгаданих колонок.
```
ALTER TABLE infectious_cases ADD start_date DATE; 
ALTER TABLE infectious_cases ADD date_current DATE;
ALTER TABLE infectious_cases ADD year_difference INT;
```
```
SET SQL_SAFE_UPDATES = 0;

UPDATE infectious_cases
SET start_date = STR_TO_DATE(CONCAT(Year, '-01-01'), '%Y-%m-%d'),
    date_current = CURDATE(),
    year_difference = TIMESTAMPDIFF(YEAR, STR_TO_DATE(CONCAT(Year, '-01-01'), '%Y-%m-%d'), CURDATE());

SET SQL_SAFE_UPDATES = 1
```
<img width="1227" alt="p4" src="https://github.com/user-attachments/assets/3c0de526-0994-4345-9d84-cc411394a2dd">

### Результати
<img width="895" alt="p4_results" src="https://github.com/user-attachments/assets/03e4eb30-7396-4b27-9fb6-d31a321e3828">

## 5. Побудуйте власну функцію.
### Створіть і використайте функцію, що будує такий же атрибут, як і в попередньому завданні: функція має приймати на вхід значення року, а повертати різницю в роках між поточною датою та датою, створеною з атрибута року (1996 рік → ‘1996-01-01’).
```
DELIMITER //
CREATE FUNCTION calculate_year_difference(input_year INT)
RETURNS INT
NO SQL
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, STR_TO_DATE(CONCAT(input_year, '-01-01'), '%Y-%m-%d'), CURDATE());
END //
DELIMITER ;
```
```
SELECT 
    Year, 
    calculate_year_difference(Year) AS year_diff
FROM infectious_cases;
```
<img width="892" alt="p5" src="https://github.com/user-attachments/assets/8fa99065-6aa1-400c-8080-a0583a0f1129">

### Функція, що рахує кількість захворювань за певний період.
```
DELIMITER //
CREATE FUNCTION calculate_cases_per_period(total_cases FLOAT, period INT)
RETURNS FLOAT
NO SQL
BEGIN
    RETURN total_cases / period;
END //
DELIMITER ;
```

```
SELECT 
    Year,
    Number_rabies,
    calculate_cases_per_period(Number_rabies, 12) AS avg_monthly_cases
FROM infectious_cases
WHERE Number_rabies IS NOT NULL AND Number_rabies != '';
```
<img width="959" alt="p5_additional" src="https://github.com/user-attachments/assets/e2f1fba3-428a-40a9-8b65-bbad1e7de50a">

