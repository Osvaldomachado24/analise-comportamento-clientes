consultas_negocio.sql

select*from customer limit 20

--1. qual  é a receita total gerada por clientes do sexo masculino vs. feminino?
select
	genero,
	to_char(sum(valor_compra_usd),'FM999G999G999')as "Receita total"
from customer
group by genero
order by sum(valor_compra_usd) desc;


--2.quais clientes usaram um desconto mas ainda assim gastaram mais do que o valor médio de compras
select
	id_cliente, valor_compra_usd
from customer
where desconto_aplicado='Yes'and valor_compra_usd>=(select avg(valor_compra_usd)from customer)

--3.Quais são os 5 produtos com a maior média de avaliação?
select
	item_comprado,
	round(avg(avaliacao)::numeric,2)as "média produto Raking"
from customer
group by item_comprado
order by avg(avaliacao) desc
limit 5

 --4.Compare os valores médios de compra entre o envio padrão e o envio expresso
 select
 	tipo_envio,
	round(avg(valor_compra_usd),2)as comparaçao
from customer 
where tipo_envio in ('Express','Standard','Next Day Air')
group by tipo_envio

--5.Os clientes assinantes gastam mais? Compare o gasto médio e a receita total entre assinantes e não assinantes
select
	status_assinatura,
	count(id_cliente)as total_cliente,
	round(avg(valor_compra_usd),2)as Média_gastos,
	to_char(sum(valor_compra_usd),'FM999G999G999')as total_receita
from customer
group by status_assinatura
order by total_receita,Média_gastos desc;

 --6.Quais 5 produtos têm a maior porcentagem de compras com descontos aplicados?
 select 
 item_comprado, 
 round( (sum(case when desconto_aplicado = 'Yes' then 1 else 0 end)::numeric / count(*) * 100), 2 ) as desconto_apli
 from customer
 group by item_comprado 
 order by desconto_apli desc 
 limit 5;

--7 Segmente os clientes em Novos, Retornando e Fiéis com base no número total 
--de compras anteriores, e mostre a distribuição.
select
	id_cliente,compras_anteriores,
	case
		when compras_anteriores=1 then 'novo'
		when compras_anteriores between 2 and 10 then 'Retornado'
		else 'Fiel'
	end as cliente_segmentados
	from customer;

--Agrupando pela segmentação
	select 
	CASE 
	WHEN compras_anteriores = 1 THEN 'Novo'
	WHEN compras_anteriores BETWEEN 2 AND 10 THEN 'Retornando' 
	ELSE 'Fiel' END as "Segmento Cliente", 
	count(*) as "Número de Clientes" 
	from customer 
	group by "Segmento Cliente"
	order by "Número de Clientes" desc; 

	--8. Quais são os 3 produtos mais comprados dentro de cada categoria?
	
with contagem_itens as (
-- aqui você monta uma subconsulta
select 
	categoria, item_comprado,
	count(id_cliente) as total_pedidos,
row_number() over(partition by categoria order by count(id_cliente) desc) as posicao
from customer 
group by categoria, item_comprado 
)
select *
from contagem_itens 
where posicao <= 3;

--10.Clientes recorrentes (com mais de 5 compras anteriores) têm maior probabilidade de subscrever o serviço?
select
	status_assinatura,
	count(id_cliente)as "clientes recorrente"
from customer
where compras_anteriores >5
group by status_assinatura;

--11.Qual é a contribuição da receita de cada faixa etária?
select
	faixa_etaria,
	to_char(sum(valor_compra_usd),'FM999G999G999')as Receita_total
from customer
group by faixa_etaria
order by Receita_total desc;
