## MySQL
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


