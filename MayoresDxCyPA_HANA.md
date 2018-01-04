### Mayores Documentos x Cobrar y Pagos anticipados HANA  
Este script obtiene los mayores de las cuentas mencionadas por consultora
hasta la fecha.

Se ejecuta en el servidor 192.168.15.200 en el Hana studio.

```SQL
Select		
		"ShortName",
		A."FormatCode",
		--A."Segment_0",
		A."AcctName",
		AC."TransId",
		AL."Line_ID",
		AL."LineMemo",
		TO_CHAR(AC."TaxDate", 'dd/mm/yyyy') As "TaxDate",
		TO_CHAR(AC."CreateDate", 'dd/mm/yyyy') As "CreateDate",
		"U_NPedidoGera" As "PedidoGera",
		--Cast(AL."Debit" as Decimal(10,2)) As "Debit" ,
		--Cast(AL."Credit" as Decimal(10,2)) as "Credit" ,
		Cast(Coalesce((Case When A."Segment_0" like '113020%' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit_DXC",
	    Cast(Coalesce((Case When A."Segment_0" like '113020%' then Coalesce("Credit",0) End),0) as Decimal(10,2)) as "Credit_DXC", 
		Cast(Coalesce((Case When A."Segment_0" Like '212050%' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit_PA",
		Cast(Coalesce((Case When A."Segment_0" Like '212050%' then Coalesce("Credit",0) End),0) as Decimal(10,2)) As "Credit_PA"
FROM	OJDT AC
Inner Join	JDT1 AL
On		AC."TransId" = AL."TransId"
Inner Join	OACT A
On		AL."Account" = A."AcctCode" 
Where	"ShortName" = 'C52720'
And		AC."TaxDate" >= '20170531'
Order by	AC."TaxDate", AC."TransId", AL."Line_ID"
    

```


