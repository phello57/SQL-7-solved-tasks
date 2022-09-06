## MySQL
Модель
<br>
<img width="500px" height="" src="https://user-images.githubusercontent.com/103268341/188669634-31d60599-4dc4-48e4-b314-cac6f77304c0.png"></img>
<br>
<br>
<strong>1.</strong> Сформируйте отчет, который содержит все счета,<br> относящиеся к продуктам типа ДЕПОЗИТ, принадлежащих клиентам, <br>у которых нет открытых продуктов типа КРЕДИТ.
<br>
<br>select *
<br>	from accounts
<br>	where (product_ref = 1 or product_ref = 2) AND close_date is null
<br>	group by client_ref
<br>	having product_ref != 1
<br>
<h2>Таблица:</h2>
<img width="900px" height="" src="https://user-images.githubusercontent.com/103268341/188656591-3fcfe498-508e-4493-958e-e85f56c070fe.png"></img>
<h2>Что отдает запрос</h2>
<img width="900px" height="" src="https://user-images.githubusercontent.com/103268341/188656842-8dd6607e-ac47-47b7-87a7-609c2cf65448.png"></img>
<br>
<br>
<strong>2.</strong>	Сформируйте выборку, который содержит движения по счетам в рамках<br> одного произвольного дня, в разрезе типа продукта.
<br>
<br>select accounts.id, accounts.name, accounts.acc_num,records.dt, records.sum, records.acc_ref, records.oper_date
<br>	from accounts
<br>	INNER JOIN records on records.acc_ref = accounts.id
<br>	where oper_date = '2015-10-01'
<h2>Таблицы:</h2>
<div style="display: flex">
  <img width="350px" height="" src="https://user-images.githubusercontent.com/103268341/188658784-cff8d055-491d-4e7f-88ec-963fbd8c70c3.png"></img>
  <img width="600px" height="" src="https://user-images.githubusercontent.com/103268341/188656591-3fcfe498-508e-4493-958e-e85f56c070fe.png"></img>
</div>
<h2>Что отдает запрос</h2>
<img width="800px" height="" src="https://user-images.githubusercontent.com/103268341/188659880-44023492-70e0-4569-ac1b-50f953b2bafe.png"></img>
<br>
<br>
<strong>3.</strong>	В результате сбоя в базе данных разъехалась информация между <br>остатками и операциями по счетам. Напишите нормализацию <br>(процедуру выравнивающую данные), которая найдет такие счета <br>и восстановит остатки по счету.
<br>
<br>UPDATE accounts as a1, records as a2
<br>SET a1.saldo = (select a3.sam 
<br>&nbsp;                        from (select sum(sum) as sam, dt, acc_ref 
<br>&nbsp;                              from records 
<br>    &nbsp;                          group by acc_ref, dt) as a3
<br>  &nbsp;                      where a3.dt = 0 AND a1.id = a3.acc_ref) 
<br>  &nbsp;                      - 
 <br>  &nbsp;                     (select a4.sumi 
<br>  &nbsp;                       from (select sum(sum) as sumi, dt, acc_ref 
<br>  &nbsp;                             from records 
<br> &nbsp;                              group by acc_ref, dt) as a4
<br>  &nbsp;                       where a4.dt = 1 AND a1.id = a4.acc_ref)
<br>where a1.id = a2.acc_ref;
<br>

<div style="display: flex">
  <img width="350px" height="" src="https://user-images.githubusercontent.com/103268341/188658784-cff8d055-491d-4e7f-88ec-963fbd8c70c3.png"></img>
  <br>Например я добавил каждому счету по 10 000:
  <img width="800px" height="" src="https://user-images.githubusercontent.com/103268341/188663676-afada670-0c5d-4678-b586-e3ea4e8e52df.png"></img>
</div>

<h2>Что отдает запрос</h2>
<img width="800px" height="" src="https://user-images.githubusercontent.com/103268341/188664286-6d7b02ab-4b2f-4022-b1a8-399f8cef2053.png"></img>
<br>
<br> В чем суть: в таблице records, в столбике dt указаны значения операций. 0 - на счет положили деньги. 1 - со счета сняли. <br>Я сгруппировал все пополнения и траты, а потом вычел второе из первого. У меня получились корректные данные на основе истории транзакций
<br>
<br>
<strong>4.</strong>	Сформируйте выборку, который содержит информацию о клиентах, <br>которые полностью погасили кредит, но при этом<br> не закрыли продукт и пользуются им дальше <br>(по продукту есть операция новой выдачи кредита).
<br>
<br>select *, count(1)
<br>from (select * from (select client_ref, name  from products where product_type_id = 1 AND close_date is not null group by client_ref, name) as a1
<br>        union all
<br>        select * from (select client_ref, name  from products where product_type_id = 1 AND close_date is null group by client_ref, name) as a2) as a3
<br>group by a3.client_ref, a3.name
<br>having count(1) > 1
<br>
<h2>Таблица:</h2>
<img width="800px" height="" src="https://user-images.githubusercontent.com/103268341/188667229-e385ec96-6db6-4315-8d95-e4575caffe10.png"></img>

<h2>Что отдает запрос</h2>
<img width="400px" height="" src="https://user-images.githubusercontent.com/103268341/188667495-ae99a504-6137-44db-b4d0-4c2b4879a4a8.png"></img>

<br>Решение: я сформировал две выборки подзапросами.(Погашенных кредитов может быть много) Первая собирала всех клиентов, которые погасили кредит и группировала их в одну строку. <br> Вторая собирала всех клиентов с открытыми кредитами и делала тоже самое. <br>Далее я соединяю эти таблицы вместе и делаю группировку. <br>И если count(1) > 1, то это означает что клиент  имеет погашенный кредит и выплачивает новый. 








// <img width="800px" height="" src=""></img>
