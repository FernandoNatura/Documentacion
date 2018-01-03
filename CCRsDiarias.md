### CCRs diarias  

Script utilizado para la generación diaria de CCRs para el cierre
diario de las cobranzas en contabilidad.

#### Requisitos:
    - Cargar todas las CCRs en la tabla Gera.CCR obtenido a la fecha.
    - Ejecutar este script.


### Script

```SQL
USE [TEMPORAL]

Declare @DesdeFecha Date
Declare @HastaFecha Date 

Set @HastaFecha = Convert(Date,Getdate())


Select @DesdeFecha = DATEADD(month, DATEDIFF(month, 0, GETDATE()), 0) 


Set @HastaFecha = Dateadd(day, -1, @HastaFecha)



IF  EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[Natura].[CCRxPPNF]') AND type in (N'U'))
DROP TABLE [Natura].[CCRxPPNF]

IF  EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[Natura].[CCR]') AND type in (N'U'))
DROP TABLE [Natura].[CCR]

/************************************** Pedidos Sin Factura **********************************************/

Declare @T Table(CodigoTitulo Int, CodigoPedido Int, CodigoPessoa Int, --Operacao Int, 
		FechaPedido  Date,
		--DataVencimiento date, 
		Deuda Decimal(10,2), 
		--SaldoPrincipal Decimal(10,2), 
		--Origen varchar(10),
		--FechaAlta datetime, 
		AjusteInicial Decimal(10,2), FechaPagoParcial Date, PagoParcial Decimal(10,2), FechaPagoTotal Date, PagoTotal Decimal(10,2))

insert into @T
Select		
	CodigoTitulo,
	CodigoPedido,
	CodigoPessoa,
	--Operacao,
	DataOperacao,
	--DataVencimento,
	ValorPrincipalOperacao,
	--SaldoPrincipal,
	--Origen,
	--TimeStamp,
	AjusteInicial = 0,
	FechaPagoParcial  = null,
	PagoParcial = 0,
	FechaPagoTotal  = null,
	PagoTotal = 0
From		INTERMEDIA.Deudas.TituloPadraoERPOut_v1
WHERE		Cast(DataOperacao as date) between @DesdeFecha And @HastaFecha
And			operacao = 1



/***  Todas las deudas anuladas en el mismo mes  ***/
Delete From	 D
From		@T D
Inner Join	INTERMEDIA.Deudas.TituloPadraoERPOut_v1 T
On		D.CodigoPedido = T.CodigoPedido
And		T.CodigoTitulo = D.CodigoTitulo
Where		T.Operacao = 2
And		Cast(T.DataOperacao as date) between @DesdeFecha And @HastaFecha



/** Todas las deudas facturadas el mismo mes ***/

Delete From	 D
From	@T D
Inner Join	INTERMEDIA.Facturas.NotaFiscalBoliviaPadraoERPOut_v1 F
On		D.CodigoPedido = F.CodigoPedido
Where	F.Situacao = 1
And		F.Data between @DesdeFecha And @HastaFecha



Update		D
Set			D.AjusteInicial = T.ValorPrincipalOperacao
From		@T D
Inner Join	INTERMEDIA.Deudas.TituloPadraoERPOut_v1 T
On			D.CodigoPedido = T.CodigoPedido
And			D.CodigoTitulo = T.CodigoTitulo
Where		T.Operacao = 13




Update		D
Set			D.PagoParcial = T.ValorPrincipalOperacao,
			D.FechaPagoParcial = Cast(T.DataOperacao as date)
From		@T D
Inner Join	INTERMEDIA.Deudas.TituloPadraoERPOut_v1 T
On			D.CodigoPedido = T.CodigoPedido
And			D.CodigoTitulo = T.CodigoTitulo
Where		T.Operacao in (4, 5)
And			Cast(T.dataoperacao as date) <= @HastaFecha



Update		D
Set			D.PagoTotal = T.ValorPrincipalOperacao,
			D.FechaPagoTotal = Cast(T.DataOperacao as date)
From		@T D
Inner Join	INTERMEDIA.Deudas.TituloPadraoERPOut_v1 T
On			D.CodigoPedido = T.CodigoPedido
And			D.CodigoTitulo = T.CodigoTitulo
Where		T.Operacao in (6)
And			Cast(T.dataoperacao as date) <= @HastaFecha



Select		
			T.*, ValorPPNF = (PagoParcial + PagoTotal), FF = Cast(F.Data as date)
into	Natura.CCRxPPNF
From		@T T
Left Join	INTERMEDIA.Facturas.NotaFiscalBoliviaPadraoERPOut_v1 F
On		T.CodigoPedido = F.CodigoPedido
Where	(T.PagoTotal  + T.PagoParcial) <> 0
order by	CodigoPessoa


/************************************** CCRs **********************************************/

Select		
			CodigoCCR, 
			Origen,
			Tipo,
			Situacion,
			FechaCreacionCCR,
			CodigoPessoa,
			Persona,
			Debito = Case When Origen = 'Débito' Then Valor Else 0 End,
			Credito = Case When Origen = 'Crédito' Then Valor Else 0 End,
			PedidoUtilizador,
			Notificacion,
			CodigoTitulo,
			CCROrigen,
			FechaPedido = (
				Select	Cast(Max(Dataoperacao) as Date)
				From	INTERMEDIA.Deudas.TituloPadraoERPOut_v1 T
				Where	C.PedidoUtilizador = T.CodigoPedido
				And		T.Operacao = 1
			),
			FechaFactura = (
				Select	Cast(Max(Data) as Date)
				From	INTERMEDIA.Facturas.NotaFiscalBoliviaPadraoERPOut_v1 F
				Where	C.PedidoUtilizador = F.CodigoPedido		
			)
into		Natura.CCR
From		Gera.CCR C
Where		FechaCreacionCCR <= @HastaFecha


/*** 
Actualizar a situacion = Activa  todas las CCRs con fecha creacion antes que @HastaFecha que esten anuladas por la creacion de 2 CCRs y que la creacion de las 
2 CCRs sea posterior a la @HastaFecha ***/

Update		C1
Set			C1.Situacion = 'Activa'
From		Natura.CCR C1
Inner Join	Gera.CCR C2
On			C1.CodigoCCR = C2.CCROrigen
Where		C2.fechaCreacionCCR > @HastaFecha
And			C1.Situacion = 'Anulada'


/***  Quitar pedido utilizador (habilitar) para las CCRs que tengan pedido utilizador posterior a @HastaFecha  
Son ajustes iniciales aplicados posterior a la @HastaFecha  
***/


Update		Natura.CCR 
Set			PedidoUtilizador = null
Where		FechaPedido > @HastaFecha



/*** Quitar pedido utilizador (habilitar) para las CCRs que tengan Factura del pedido posterior a @HastaFecha
son ajustes iniciales por pedido no facturado  ***/

Update		Natura.CCR
Set			PedidoUtilizador = null
Where		FechaFactura > @HastaFecha
And			PedidoUtilizador is not null

/*** Eliminar todas las CCRs con factura anterior a la @HastaFecha  ***/

Delete	
From		Natura.CCR
--Where		FechaFactura <='30/06/2017'
Where		FechaFactura <=@HastaFecha

/*** Eliminar todas las CCRs con fechaPedido anterior a la @HastaFecha  ***/

Delete	
From		Natura.CCR
--Where		FechaFactura <='30/06/2017'
Where		FechaFactura <=@HastaFecha

Delete	
From		Natura.CCR
Where		situacion = 'Anulada'


Delete	
From		Natura.CCR
Where		situacion = 'Inactiva'


Delete	
From		Natura.CCR
Where		FechaCreacionCCR > @HastaFecha
--Where		FechaCreacionCCR > '30/06/2017'
And			PedidoUtilizador is null

/*************************************   Insertar PPNFs ******************************/

insert into	Natura.CCR
Select		CodigoCCR = null,
			Origen = null,
			tipo = 'Credito CCR Por PPNF', 
			Situacion = null,
			FechaCreacionCCR = FechaPedido,
			CodigoPessoa,
			Persona = '',
			Debito = 0,
			Credito = ValorPPNF,
			PedidoUtilizador = null,
			Notificacion = null,
			CodigoTitulo = null,
			CCROrigen = null,
			FechaPedido = null,
			FechaFactura = null
From		Natura.CCRxPPNF


Select		Debito = Sum(Debito), --- 6197.84
			Credito = Sum(Credito) -- 135809.13
From		Natura.CCR

/*************************************   CCR Union PPNF  ******************************/

Select		Tipo, 
			FechaCreacionCCR, 
			codigoPessoa, 
			Persona, 
			Debito, 
			Credito
From		Natura.CCR
order by	fechaCreacionCCR
```
