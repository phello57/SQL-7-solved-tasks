## MySQL
Модель
<br>
<img width="500px" height="" src="https://user-images.githubusercontent.com/103268341/188669634-31d60599-4dc4-48e4-b314-cac6f77304c0.png"></img>
<br>
<br>
## 1.
Сформируйте отчет, который содержит все счета, относящиеся к продуктам <br>типа ДЕПОЗИТ, принадлежащих клиентам, у которых нет открытых продуктов типа КРЕДИТ.<br>
  
<br>select * from (
<br>&nbsp;	***select * from accounts
<br>&nbsp;&nbsp;&nbsp;		where (product_ref = 1 or product_ref = 2) AND close_date is null
<br>&nbsp;&nbsp;&nbsp;		Order by client_ref, product_ref***) as acc
<br>&nbsp;	group by acc.client_ref
<br>&nbsp;	having acc.product_ref != 1;
##### Решение: <br> Подзапросом я получаю всех клиентов, у кого есть открытые депозиты и открытые кредиты.<br> Группирую. Группировка оставляет у клиента одну запись<br> Если у клиента не было кредитов, у него останется 2(Депозит) в product_ref.
<br>Что отдает подзапрос
![Screenshot_5](https://user-images.githubusercontent.com/113181404/189364761-6b48e952-d97b-4451-b52c-e60d96eb0040.png)
<br>Результат
![Screenshot_3](https://user-images.githubusercontent.com/113181404/189364460-59fa03ff-6f25-4603-9769-b2d39191686d.png)

## 2.
Сформируйте выборку, которая содержит средние движения по счетам в рамках одного произвольного дня, в разрезе типа продукта.
<br>
<br>select accounts.id, accounts.name, accounts.acc_num, records.acc_ref, records.oper_date, accounts.product_ref
<br>&nbsp;&nbsp;&nbsp;		from accounts INNER JOIN records on records.acc_ref = accounts.id
<br>&nbsp;&nbsp;&nbsp;	where oper_date = '2015-10-01' and product_ref = 1;
<br>
##### Решение: <br> Cклеил две таблицы inner join и отсортировал их по дате и типу продукта.
Результат<br>
![Screenshot_3](https://user-images.githubusercontent.com/113181404/189371394-755477d7-864a-44fd-b16a-4769f5f63fb5.png)

## 3.
Сформируйте выборку, в который попадут клиенты, у которых были операции по счетам <br>за прошедший месяц от текущей даты. Выведите клиента и сумму операций за день в разрезе даты.
<br>
<br>select accounts.id, accounts.name, records.oper_date, count(1) as Количество операций
<br>&nbsp;&nbsp;	from accounts
<br>&nbsp;&nbsp;	INNER JOIN records on records.acc_ref = accounts.id
<br>&nbsp;&nbsp;	where to_days(now()) - to_days(oper_date) <= 30
<br>&nbsp;&nbsp;	group by accounts.name, records.oper_date
##### Решение: соединяю две таблицы. Каждую дату вычитаю из сегодняшней, получаю число. Добавляю count(1) и группирую
<br>Результат<br>
![Screenshot_3](https://user-images.githubusercontent.com/113181404/189374042-777379f0-3782-4841-bd9f-09461f573cb3.png)
## 4.
В результате сбоя в базе данных разъехалась информация между остатками и операциями <br>по счетам. Напишите нормализацию (процедуру выравнивающую данные), которая найдет такие <br>счета и восстановит остатки по счету.
<br>
<br>select * from accounts;
<br>UPDATE accounts as a1, records as a2
<br>SET a1.saldo = (select a3.sam 
<br>&nbsp;&nbsp;                        from (select sum(sum) as sam, dt, acc_ref 
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                              from records 
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                             group by acc_ref, dt) as a3
<br>&nbsp;&nbsp;                       where a3.dt = 0 AND a1.id = a3.acc_ref) 
<br>&nbsp;&nbsp;                        - 
<br>&nbsp;&nbsp;                        (select a4.sumi 
<br>&nbsp;&nbsp;                         from (select sum(sum) as sumi, dt, acc_ref 
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                               from records 
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                               group by acc_ref, dt) as a4
<br>&nbsp;&nbsp;                         where a4.dt = 1 AND a1.id = a4.acc_ref)
<br>where a1.id = a2.acc_ref; <br><br>
***Решение: в таблице records, в столбике dt указаны значения операций. 0 - пополнение. 1 - снятие. Я сгруппировал все пополнения и снятия, а потом вычел второе из первого. У меня получились корректные данные на основе истории транзакций***<br>
<br>Добавил всем по 10000 
![Screenshot_2](https://user-images.githubusercontent.com/113181404/189375913-8b80df72-2ca5-4e5d-84a5-9938f0d7676b.png)
<br>Результат
![Screenshot_5](https://user-images.githubusercontent.com/113181404/189376034-22b9ac7a-9755-4b44-872a-6c98ec0ecbcb.png)
## 5.
Сформируйте выборку, который содержит информацию о клиентах, которые полностью погасили кредит, но при этом не закрыли продукт и пользуются им дальше (по продукту есть операция новой выдачи кредита).
<br>
<br>select *
<br>&nbsp;&nbsp;from (select * from (select client_ref, name  from products where product_type_id = 1 AND close_date is not null group by client_ref, <br>&nbsp;&nbsp;name) as a1
<br>&nbsp;&nbsp;&nbsp;&nbsp;        union all
<br>&nbsp;&nbsp;        select * from (select client_ref, name  from products where product_type_id = 1 AND close_date is null group by client_ref, name) <br>&nbsp;&nbsp;as a2) as a3
<br>group by a3.client_ref, a3.name
<br>having count(1) > 1
##### Решение:<br> В первом подзапросе клиенты с закрытым кредитом. Во втором - с открытым кредитом. Кто попадает в обе группы - результат
<br>Объединенная таблица union all<br>
![Screenshot_2](https://user-images.githubusercontent.com/113181404/189382534-8f98e997-4f70-4629-ab9e-66685f39bd81.png)
<br>Результат<br>
![Screenshot_3](https://user-images.githubusercontent.com/113181404/189382540-13adfaeb-ee0e-4cf6-9c72-6b069e3557a0.png)

## 6.
Закройте возможность открытия (установите дату окончания действия) для типов продуктов, по счетам продуктов которых, не было движений более одного месяца.<br>
<br>update accounts as a1
<br>set a1.close_date = CURRENT_DATE(); 
<br>where to_days(NOW()) - TO_DAYS(
<br>&nbsp;&nbsp;&nbsp;&nbsp;            (select a2.oper
<br>&nbsp;&nbsp;&nbsp;&nbsp;            from (select max(oper_date) as oper,acc_ref from records group by acc_ref) as a2
<br>&nbsp;&nbsp;&nbsp;&nbsp;            where  a1.id = a2.acc_ref)) >= 30;
<br>
##### Решение:<br> Из таблицы с последней датой достаем дату по каждому счету. Вычитаем её из сегодняшней даты. Если получ. число больше 30 - закрываем счет.
<br>Таблица с последней датой по каждому счету<br>
![Screenshot_8](https://user-images.githubusercontent.com/113181404/189384110-7914ef70-fa01-4f49-9d1f-b64fdfd66ccc.png)
<br>Результат<br>
![Screenshot_6](https://user-images.githubusercontent.com/113181404/189383839-2485f5c2-4f57-4876-b816-73b0401c5e46.png)
 ## 7.
Закройте продукты (установите дату закрытия равную текущей) типа КРЕДИТ, у которых произошло полное погашение, но при этом не было повторной выдачи.<br>
<br>
<br>update accounts as a1
<br>set a1.close_date = CURRENT_DATE()
<br>where id in (select * from (select a2.id from (select * from accounts where saldo = 0 and close_date is null) as a2
<br>where not exists (select * from accounts as a3 where saldo < 0 and a2.client_ref = a3.client_ref AND a3.close_date is null))as a4)
<br>

##### Решение:<br> Нужно найти всех тех, у кого saldo = 0 и close_date = null это первая таблица в подзапросе.Во второй таблице все клиенты с непогашенными кредитами. Мы ищем во второй таблице сходства с первой по каждому клиенту. Если клиента нет во второй таблице(но есть в первой) - меняем у него дату на сегодняшнюю.<br>
<br>Что отдает подзапрос<br>
![Screenshot_2](https://user-images.githubusercontent.com/113181404/189486930-d845dd92-cde4-4132-8513-2efebbdcaff0.png)
<br>Начальная таблица<br>
![Screenshot_5](https://user-images.githubusercontent.com/113181404/189486955-6271ec7b-ec96-4770-a10b-3deb46436b8a.png)
<br>Результат<br>
![Screenshot_6](https://user-images.githubusercontent.com/113181404/189487094-293181be-84f9-45b3-a316-7bcf601b5d90.png)
<br>

 
