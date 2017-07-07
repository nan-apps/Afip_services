# Clases para interactuar con servicios afip

1. `Auth` permite interactuar con el web service de autenticacion WSAA
2. `Biller` permite interactuar con el servicio de facturacion electronica WSFEV1

## Carpeta Resources

En carpeta Resources debe haber: 

1. `cert.pem` este .pem nos lo da la afip.
2. `cert.key` es la llave con el cual generamos nuestro CSR. El CSR es lo que le enviamos a la afip para obtener el `.pem`

Mas info aca -> http://www.afip.gob.ar/ws/WSASS/WSASS_manual.pdf

## Carpeta Temp
Debe tener permisos de escritura

## Ejemplo de uso con urls de testing/homologacion

```php
	$conf = [		
        'cuit' => 'xxxxxxxxxxx',
        'auth' => [
            'passprhase' => '', //opcional
            'wsdl'       => 'https://wsaahomo.afip.gov.ar/ws/services/LoginCms?wsdl',
            'end_point'  => 'https://wsaahomo.afip.gov.ar/ws/services/LoginCms'
        ],
        'biller' => [
            'wsdl'      => 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx?wsdl',
            'end_point' => 'https://wswhomo.afip.gov.ar/wsfev1/service.asmx'
        ]    	
	];

    $auth_conf = $conf['auth'];            
    $biller_conf = $conf['biller'];            

    try{

		/**
		 * Servicio de autenticación
		 */ 
		$auth = new Afip\Services\Auth( 
		    Afip\SoapClientFactory::create( $auth_conf['wsdl'], $auth_conf['end_point'] ),                                 
		    $auth_conf['passprhase'] 
		);        

		/**
		 * Servicio de facturacion, recibe como parametro el servicio de autenticación 
		 */
		$biller = new Afip\Services\Biller( 
		    Afip\SoapClientFactory::create( $biller_conf['wsdl'], $biller_conf['end_point'] ), 
		    $auth, 
		    new Afip\AccessTicket( $conf['cuit'] ) 
		);

		/**
		 * Solicitar cae y cae_validdate y otros datos [ string    'cae' => '',  
         *                                               \DateTime 'cae_validdate' => null, 
         *                                               int       'invoice_number' => 0, 
         *                                               string    'tax_id' => '', 
         *                                               \DateTime 'invoice_date' => null
         *                                               stdClass  'full_response' => null ]
		 * $params debe ser un array con los datos de facturacion a enviar a la afip.  
		 * Ej mas abajo y data completa en manual de F.E. 
		 */		
		$data = $biller->requestCAE( $params );


    } catch ( WSException $e ) {
            
        var_dump([
        	'description' => "{$e->getService()->getServiceName()}: {$e->getMessage()}",
        	'log_api_response' => $e->getWSResponse()
    	]);

	}

```

------------------------------------------------------------------------

### Ejemplo de datos

```php
	$params = [
            'Cuit' => 'xxxxxxxxx',
            'CantReg' => 1,
            'PtoVta' => 1,
            'CbteTipo' => 2, //B
            'Concepto' => 2, //servicios
            'DocTipo' => 80, //80=CUIL
            'DocNro' => $contact->getContactDni(),
            'CbteDesde' => null, //si es null lo obtiene Biller consultando a la afip
            'CbteHasta' => null, 
            'CbteFch' => '20170505',
            'ImpNeto' => 0,
            'ImpTotConc' => 0, 
            'ImpIVA' => 0,
            'ImpTrib' => 0,
            'ImpOpEx' => 0,
            'ImpTotal' => $amount, 
            'FchServDesde' => '20170401', 
            'FchServHasta' => '20170431', 
            'FchVtoPago' => '20170531',
            'MonId' => 'PES', //PES 
            'MonCotiz' => 1, //1 
        ];
```

--------------------------------------------------------------------------
**Manuales AFIP**

1. Auth: http://www.afip.gob.ar/ws/WSAA/Especificacion_Tecnica_WSAA_1.2.2.pdf

2. F.E.: http://www.afip.gob.ar/fe/documentos/manual_desarrollador_COMPG_v2_9.pdf