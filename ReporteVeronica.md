```SQL
Create LOCAL TEMPORARY TABLE #Vero( "TransId" Integer, "TaxDate" Timestamp, "BaseRef" nvarchar(11), "TransType" nvarchar(20), "Origen" nvarchar(100), "Concepto" nvarchar(50),
						"FACTURA" smallint, "PAGO" smallint, "GN_MOV_DIA" smallint, "GN_INTERES" smallint, "GN_VENTAS" smallint,
						 "RECONCILIACION" smallint, "OTROS" smallint  );
--Select * from #Vero
--Drop table #Vero

Insert into #Vero ("TransId", "TaxDate", "BaseRef" ,"TransType", "Origen", "Concepto", 
				"FACTURA", "PAGO", "GN_MOV_DIA", "GN_INTERES", "GN_VENTAS", "RECONCILIACION", "OTROS")
Select 	"TransId",
		"TaxDate" ,
 		"BaseRef" ,
 		"TransType",
 		Case "TransType"
		      when 10000071 then '10000071 Contabilización de Stock - PA'
		      when 13       then '13       Facturas deudores - RF'
		      when 14       then '14       Nota de Credito Clientes - RC'
		      when 18       then '18       Factura Proveedores - TT'
		      when 19       then '19       Nota de Credito Proveedores - PC'
		      when 20       then '20       Pedido de Entrada de Mercancias - EP'
		      when 24       then '24       Pago Recibido - PR'
		      when 25       then '25       Deposito - DP'
		      when 30       then '30       Asiento - AS'
		      when 321      then '321      Reconciliacion Interna - ID'
		      when 46       then '46       Pago Proveedor - PP'
		      when 59       then '59       Entrada de Mercancias - EM'
		      when 60       then '60       Salida de Mercancias - OA'
		      when 67       then '67       Traslados - IM'
		      when 69       then '69       Precio de Entrega - DI' End as "Origen",
	  "Memo" as "Concepto",
	  Case when "TransType" In(13,18) then 1 else 0 end "FACTURA",
	  Case when "TransType" In(24,46) then 1 else 0 end "PAGO",
	  Case when "TransType" = 30 And "Memo" Like '[GETNAT]%Movimientos%' then 1 else 0 end "GN_MOV_DIA",
	  Case when "TransType" = 30 And "Memo" Like '[GETNAT]%Facturas Interes%' then 1 else 0 end "GN_INTERES",
      Case when "TransType" = 30 AND "Memo" Like '[GETNAT]Facturas Validas%' then 1 else 0 end "GN_VENTAS",
      Case when "TransType" <> 321 And "Memo" like '%reconciliaci%' then 1 else 0 end "RECONCILIACION",
      Case When "TransType" in(60,67,59,19,20,25,69,321,10000071,14) Then 1 Else 0 End "OTROS"
from 	OJDT 
Where "TaxDate" Between '20180101' And '20181231'
And 	"BaseRef" between 3000000 and 3000109
--And	"TaxDate" between '20180102' and '20180102'	
Order by "BaseRef";


Update #Vero Set "OTROS"=1 Where "FACTURA" + "PAGO" + "GN_MOV_DIA" + "GN_INTERES" + "GN_VENTAS" + "RECONCILIACION" + "OTROS" = 0;

Select 		'' As "Fecha Entrega", 
			Cast("TaxDate" as Date) As "Fecha Emisión",
			"BaseRef" As "Asiento Nro.",
			"Origen",
			"Concepto",
			Case When "FACTURA" = 1 Then 'X' Else '' End "FACTURA",
			Case When "PAGO" = 1 Then 'X' Else '' End "PAGO",
			Case When "GN_MOV_DIA" = 1 Then 'X' Else '' End "GN/MOV.DIA",
			Case When "GN_INTERES" = 1 Then 'X' Else '' End "GN/INTERES",
			Case When "GN_VENTAS" = 1 Then 'X' Else '' End "GN/VENTAS",
			Case When "RECONCILIACION" = 1 Then 'X' Else '' End "RECONCILIACION",
			Case When "OTROS" = 1 Then 'X' Else '' End "OTROS",
			'' "Recibí Conforme",
			'' "Observaciones",
			'' "Elaborado Por"
From		#Vero 

```
