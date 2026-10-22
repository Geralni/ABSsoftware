# MATRIZ DE TRAZABILIDAD NORMATIVA (SUNAT PERÚ vs OASIS UBL 2.1 vs OPENAPI SCHEMAS)
**Proyecto:** ABC Software Corp  
**Módulo:** Fase 1 - Spec-First (Peruanización Completa)  
**Estándar Base:** OASIS UBL 2.1 (`UBL-Invoice-2.1.xsd`)  
**Regulaciones SUNAT:** RS N° 097-2012, RS N° 193-2020 y modificatorias; Ley N° 30884 (ICBPER).

---

## 1. Propósito y Lineamientos Técnicos de SUNAT Perú

Esta matriz define el mapeo formal entre los requisitos de la normativa tributaria peruana, los nodos XML del estándar **OASIS UBL 2.1** y los campos peruanizados de los esquemas **OpenAPI 3.0.3** (`openapi.yaml` y carpeta `spec/schemas/`).

### Puntos Obligatorios de Cumplimiento Normativo:
1. **`<cbc:UBLVersionID>`:** Debe contener de forma fija e invariable el valor **`2.1`**.
2. **`<cbc:CustomizationID>`:** Debe contener de forma fija e invariable el valor **`2.0`** (versión de las especificaciones funcionales de SUNAT).
3. **`<cbc:ID>` (Identificador Legal del Comprobante):** NO es un identificador aleatorio ni UUID. Es estrictamente la concatenación determinística `{serie}-{correlativo:08d}` (ej. `F001-00000123`).
4. **Afectación al IGV:** Se codifica mediante el **Catálogo 07 de SUNAT** (e.g. `10` Gravado - Operación Onerosa, `20` Exonerado, `30` Inafecto).
5. **Sistema de Cálculo del ISC:** Se codifica mediante el **Catálogo 08 de SUNAT** (e.g. `01` Sistema al Valor, `02` Sistema Específico, `03` Sistema al Valor según Precio de Venta al Público).
6. **Campos Peruanizados:** Uso explícito de `rucEmisor`, `tipoDocumentoIdentidad`, `ubigeo`, `codigoAfectacionIgv`, `codigoSistemaIsc`, `numeroComprobante`.

---

## 2. Matriz Exhaustiva de Mapeo Normativo

| N° | Requisito SUNAT | Jerarquía y Etiqueta en XML UBL 2.1 | Campo en Schema YAML (OpenAPI / JSON) | Catálogo SUNAT Aplicable | Regla Técnica y Transformación |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **1** | **Versión de UBL** | `/Invoice/cbc:UBLVersionID` | `ublVersionId` | **Estándar OASIS UBL 2.1** | **Valor fijo '2.1'**. Exigido por SUNAT para comprobantes emitidos bajo este estándar. |
| **2** | **Versión de Estructura SUNAT** | `/Invoice/cbc:CustomizationID` | `customizationId` | **RS 097-2012/SUNAT** | **Valor fijo '2.0'**. Representa la versión de las reglas funcionales de SUNAT. |
| **3** | **Serie y Número Correlativo (ID Legal)** | `/Invoice/cbc:ID` | `numeroComprobante`<br>*(concatenación de `serie` y `correlativo`)* | **RS 097-2012/SUNAT** | **REGLA ESTRICTA:** No admite UUIDs. Debe ser `{serie}-{correlativo:08d}` (ej. `F001-00000123`). Serie inicia con `'F'` y correlativo a 8 dígitos con ceros a la izquierda. Coincide exactamente con el nombre del XML: `20123456789-01-F001-00000123.xml`. |
| **4** | **Fecha de Emisión** | `/Invoice/cbc:IssueDate` | `fechaEmision` | **Norma Técnica SUNAT / ISO 8601** | Formato `YYYY-MM-DD`. Máximo 3 días calendario anteriores a la fecha de remisión al OSE/SUNAT. |
| **5** | **Hora de Emisión** | `/Invoice/cbc:IssueTime` | `horaEmision` | **Norma Técnica SUNAT / ISO 8601** | Formato `HH:MM:SS`. Hora militar de emisión del comprobante. |
| **6** | **Tipo de Comprobante de Pago** | `/Invoice/cbc:InvoiceTypeCode` | `tipoComprobante` | **Catálogo 01** *(Tipo de comprobante)* | Código `'01'` correspondiente a Factura Electrónica. |
| **7** | **Tipo de Operación Comercial** | `/Invoice/cbc:InvoiceTypeCode/@listID` | `tipoOperacion` | **Catálogo 51** *(Tipo de operación)* | Atributo `@listID` con valor `'0101'` correspondiente a "Venta Interna". |
| **8** | **Código de Moneda** | `/Invoice/cbc:DocumentCurrencyCode` | `codigoMoneda` | **Catálogo 02** *(Tipo de moneda)* | Código alfabético ISO 4217: `'PEN'` (Soles) o `'USD'` (Dólares). Afecta el atributo `@currencyID` de los importes. |
| **9** | **RUC de la Empresa Emisora** | `/Invoice/cac:AccountingSupplierParty`<br>`/cac:Party/cac:PartyIdentification/cbc:ID`<br>`@schemeID="6"` | `rucEmisor` | **Catálogo 06** *(Tipo de documento)* | RUC de 11 dígitos de ABC Software Corp. Se inyecta en `cbc:ID` con `@schemeID="6"`. |
| **10** | **Razón Social de la Emisora** | `/Invoice/cac:AccountingSupplierParty`<br>`/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `razonSocialEmisor` | **Padrón RUC SUNAT** | Nombre oficial: `"ABC SOFTWARE CORP S.A.C."`. Se envuelve en CDATA si contiene caracteres especiales. |
| **11** | **Ubigeo y Domicilio Fiscal Emisor** | `/Invoice/cac:AccountingSupplierParty`<br>`/cac:Party/cac:PostalAddress/cbc:ID` | `ubigeoEmisor` | **Catálogo INEI / SUNAT** | Código numérico de 6 dígitos del distrito (ej. `'150131'` para San Isidro, Lima). |
| **12** | **Tipo de Documento Identidad Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PartyIdentification/cbc:ID`<br>`@schemeID` | `cliente.tipoDocumentoIdentidad` | **Catálogo 06** *(Tipo de documento)* | Para Facturas (`01`), debe ser obligatoriamente `'6'` (RUC). No se admiten DNI ni documentos no autorizados. |
| **13** | **Número de RUC del Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PartyIdentification/cbc:ID` | `cliente.numeroDocumentoIdentidad` | **Catálogo 06 / Módulo 11** | 11 dígitos validados con algoritmo Módulo 11 de SUNAT. Debe encontrarse Activo y Habido en el padrón. |
| **14** | **Razón Social del Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `cliente.razonSocial` | **Padrón RUC SUNAT** | Razón social oficial del cliente registrada en SUNAT. |
| **15** | **Ubigeo y Domicilio Fiscal Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PostalAddress/cbc:ID` | `cliente.ubigeo` | **Catálogo INEI / SUNAT** | Código de 6 dígitos numéricos del distrito fiscal del adquirente (ej. `'150103'` para Ate, Lima). |
| **16** | **Monto Operaciones Gravadas (Base IGV)** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cbc:TaxableAmount` *(para tributo 1000)* | `montoOperacionesGravadas`<br>*(o `montoValorVenta`)* | **Catálogo 05** *(Tipos de tributos)* | Sumatoria de bases imponibles de líneas con afectación onerosa (`10`). |
| **17** | **Total Impuesto General a las Ventas (IGV)** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cac:TaxCategory/cac:TaxScheme/cbc:ID`<br>`= "1000"` con `cbc:TaxAmount` | `totalIgv`<br>`detalles[].montoIgv` | **Catálogo 05** *(Tipos de tributos)* | Bloque de impuesto con `cbc:ID` = `1000`, `cbc:Name` = `'IGV'`, `cbc:TaxTypeCode` = `'VAT'`. Tasa 18%. |
| **18** | **Código de Afectación al IGV por Ítem** | `/Invoice/cac:InvoiceLine/cac:TaxTotal`<br>`/cac:TaxSubtotal/cac:TaxCategory`<br>`/cbc:TaxExemptionReasonCode` | `detalles[].codigoAfectacionIgv` | **Catálogo 07** *(Afectación al IGV)* | **CATÁLOGO 07:** `'10'` (Gravado - Operación Onerosa), `'20'` (Exonerado), `'30'` (Inafecto), `'11'` (Retiro por premio). |
| **19** | **Total Impuesto Selectivo al Consumo (ISC)** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cac:TaxCategory/cac:TaxScheme/cbc:ID`<br>`= "2000"` con `cbc:TaxAmount` | `totalIsc`<br>`detalles[].montoIsc` | **Catálogo 05** *(Tipos de tributos)* | Tributo código `2000`, `cbc:Name` = `'ISC'`, `cbc:TaxTypeCode` = `'EXC'`. Aplica a bienes tipificados en el Apéndice IV. |
| **20** | **Sistema de Cálculo del ISC por Ítem** | `/Invoice/cac:InvoiceLine/cac:TaxTotal`<br>`/cac:TaxSubtotal/cac:TaxCategory`<br>`/cac:TaxScheme/cbc:ID = "2000"` | `detalles[].codigoSistemaIsc` | **Catálogo 08** *(Sistema cálculo ISC)* | **CATÁLOGO 08:** `'01'` (Sistema al Valor), `'02'` (Sistema Específico), `'03'` (Al Valor según PVP). |
| **21** | **Impuesto a las Bolsas Plásticas (ICBPER)** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cac:TaxCategory/cac:TaxScheme/cbc:ID`<br>`= "7152"` con `cbc:TaxAmount` | `totalIcbper`<br>`detalles[].montoIcbper` | **Catálogo 05** *(Tributos)* y **Ley N° 30884** | Código `7152`, `cbc:Name` = `'ICBPER'`, `cbc:TaxTypeCode` = `'OTH'`. Se multiplica cantidad de bolsas por S/ 0.50. |
| **22** | **Unidad de Medida Comercial** | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity`<br>`@unitCode` | `detalles[].unidadMedida` | **Catálogo 03** *(Unidades de medida)* | Código internacional: `'NIU'` (Unidades), `'KGM'` (Kilogramos), `'ZZ'` (Servicios), `'MTR'` (Metros), `'BX'` (Cajas). |
| **23** | **Precio de Venta Unitario (Con Impuesto)** | `/Invoice/cac:InvoiceLine/cac:PricingReference`<br>`/cac:AlternativeConditionPrice/cbc:PriceAmount` | `detalles[].precioUnitarioConIgv` | **Catálogo 16** *(Tipo de precio)* | Con atributo `@PriceTypeCode="01"` correspondiente a "Precio unitario comercial incluye IGV". |
| **24** | **Valor Total de Venta (Base Imponible Neta)** | `/Invoice/cac:LegalMonetaryTotal`<br>`/cbc:LineExtensionAmount` | `montoOperacionesGravadas` | **Cuadre Contable UBL 2.1** | Sumatoria de bases imponibles netas antes de impuestos y descuentos globales. |
| **25** | **Importe Total a Pagar** | `/Invoice/cac:LegalMonetaryTotal`<br>`/cbc:PayableAmount` | `montoTotalVenta` | **RS 097-2012/SUNAT** | Importe final de la transacción: `LineExtensionAmount` + `TaxTotal` - Descuentos. `@currencyID="PEN"`. |
| **26** | **Firma Digital XML-DSig y Hash SHA-256** | `/Invoice/ext:UBLExtensions/ext:UBLExtension`<br>`/ext:ExtensionContent/ds:Signature`<br>`/ds:SignedInfo/ds:Reference/ds:DigestValue` | `digestValue`<br>`signatureValue` | **Estándar W3C XML-DSig y SUNAT** | Hash SHA-256 Base64 extraído del `ds:DigestValue` para la generación obligatoria del código QR y PDF. |

---

## 3. Ejemplo Concreto: Payload JSON vs XML UBL 2.1 SUNAT

### Entrada JSON Peruanizada (`FacturaInput`):
```json
{
  "ublVersionId": "2.1",
  "customizationId": "2.0",
  "tipoComprobante": "01",
  "serie": "F001",
  "correlativo": 123,
  "fechaEmision": "2026-10-22",
  "horaEmision": "10:15:30",
  "codigoMoneda": "PEN",
  "tipoOperacion": "0101",
  "rucEmisor": "20123456789",
  "razonSocialEmisor": "ABC SOFTWARE CORP S.A.C.",
  "ubigeoEmisor": "150131",
  "direccionFiscalEmisor": "Av. Las Camelias 820, San Isidro, Lima",
  "cliente": {
    "tipoDocumentoIdentidad": "6",
    "numeroDocumentoIdentidad": "20601234567",
    "razonSocial": "DISTRIBUIDORA LOGISTICA DEL CENTRO S.A.C.",
    "direccionFiscal": "Av. Industrial 450, Urb. Vulcano, Ate, Lima",
    "ubigeo": "150103"
  },
  "formaPago": {
    "tipo": "Contado",
    "montoNetoPendiente": 0.00
  },
  "detalles": [
    {
      "linea": 1,
      "codigoProducto": "PRD-TUB-01",
      "descripcion": "Tubo PVC Presión 1/2 pulgada x 5 metros",
      "unidadMedida": "NIU",
      "cantidad": 10.0,
      "valorUnitario": 15.00,
      "precioUnitarioConIgv": 17.70,
      "codigoAfectacionIgv": "10",
      "porcentajeIgv": 18.00,
      "montoIgv": 27.00,
      "codigoSistemaIsc": "01",
      "porcentajeIsc": 0.0,
      "montoIsc": 0.0,
      "aplicaIcbper": false,
      "cantidadBolsas": 0,
      "montoIcbper": 0.00,
      "montoValorVenta": 150.00
    }
  ]
}
```

### Salida Generada por `UBLGenerator` (XML UBL 2.1 Oficial SUNAT):
```xml
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<Invoice xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
         xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
         xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
         xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
         xmlns:ext="urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2">
  <ext:UBLExtensions>
    <ext:UBLExtension>
      <ext:ExtensionContent>
        <ds:Signature Id="SignSUNAT">
          <ds:SignedInfo>
            <ds:CanonicalizationMethod Algorithm="http://www.w3.org/TR/2001/REC-xml-c14n-20010315"/>
            <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
            <ds:Reference URI="">
              <ds:Transforms>
                <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
              </ds:Transforms>
              <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
              <ds:DigestValue>4z0uW1eG+s/gZl8aA31k1M7ZfN0=</ds:DigestValue>
            </ds:Reference>
          </ds:SignedInfo>
          <ds:SignatureValue>j3jW4N3qKz+D83kld...</ds:SignatureValue>
          <ds:KeyInfo>
            <ds:X509Data>
              <ds:X509Certificate>MIIF...</ds:X509Certificate>
            </ds:X509Data>
          </ds:KeyInfo>
        </ds:Signature>
      </ext:ExtensionContent>
    </ext:UBLExtension>
  </ext:UBLExtensions>

  <!-- REGLAS DE CABECERA SUNAT -->
  <cbc:UBLVersionID>2.1</cbc:UBLVersionID>
  <cbc:CustomizationID>2.0</cbc:CustomizationID>
  <cbc:ID>F001-00000123</cbc:ID>
  <cbc:IssueDate>2026-10-22</cbc:IssueDate>
  <cbc:IssueTime>10:15:30</cbc:IssueTime>
  <cbc:InvoiceTypeCode listID="0101">01</cbc:InvoiceTypeCode>
  <cbc:DocumentCurrencyCode>PEN</cbc:DocumentCurrencyCode>

  <!-- DATOS DEL EMISOR (ABC Software Corp) -->
  <cac:AccountingSupplierParty>
    <cac:Party>
      <cac:PartyIdentification>
        <cbc:ID schemeID="6">20123456789</cbc:ID>
      </cac:PartyIdentification>
      <cac:PartyLegalEntity>
        <cbc:RegistrationName><![CDATA[ABC SOFTWARE CORP S.A.C.]]></cbc:RegistrationName>
      </cac:PartyLegalEntity>
      <cac:PostalAddress>
        <cbc:ID>150131</cbc:ID>
        <cac:AddressLine>
          <cbc:Line><![CDATA[Av. Las Camelias 820, San Isidro, Lima]]></cbc:Line>
        </cac:AddressLine>
      </cac:PostalAddress>
    </cac:Party>
  </cac:AccountingSupplierParty>

  <!-- DATOS DEL ADQUIRENTE -->
  <cac:AccountingCustomerParty>
    <cac:Party>
      <cac:PartyIdentification>
        <cbc:ID schemeID="6">20601234567</cbc:ID>
      </cac:PartyIdentification>
      <cac:PartyLegalEntity>
        <cbc:RegistrationName><![CDATA[DISTRIBUIDORA LOGISTICA DEL CENTRO S.A.C.]]></cbc:RegistrationName>
      </cac:PartyLegalEntity>
      <cac:PostalAddress>
        <cbc:ID>150103</cbc:ID>
        <cac:AddressLine>
          <cbc:Line><![CDATA[Av. Industrial 450, Urb. Vulcano, Ate, Lima]]></cbc:Line>
        </cac:AddressLine>
      </cac:PostalAddress>
    </cac:Party>
  </cac:AccountingCustomerParty>

  <!-- TOTALES TRIBUTARIOS GLOBALES (IGV TRIBUTO 1000) -->
  <cac:TaxTotal>
    <cbc:TaxAmount currencyID="PEN">27.00</cbc:TaxAmount>
    <cac:TaxSubtotal>
      <cbc:TaxableAmount currencyID="PEN">150.00</cbc:TaxableAmount>
      <cbc:TaxAmount currencyID="PEN">27.00</cbc:TaxAmount>
      <cac:TaxCategory>
        <cac:TaxScheme>
          <cbc:ID>1000</cbc:ID>
          <cbc:Name>IGV</cbc:Name>
          <cbc:TaxTypeCode>VAT</cbc:TaxTypeCode>
        </cac:TaxScheme>
      </cac:TaxCategory>
    </cac:TaxSubtotal>
  </cac:TaxTotal>

  <!-- TOTALES LEGALES MONETARIOS -->
  <cac:LegalMonetaryTotal>
    <cbc:LineExtensionAmount currencyID="PEN">150.00</cbc:LineExtensionAmount>
    <cbc:TaxInclusiveAmount currencyID="PEN">177.00</cbc:TaxInclusiveAmount>
    <cbc:PayableAmount currencyID="PEN">177.00</cbc:PayableAmount>
  </cac:LegalMonetaryTotal>

  <!-- DETALLE DE LÍNEA DE VENTA -->
  <cac:InvoiceLine>
    <cbc:ID>1</cbc:ID>
    <cbc:InvoicedQuantity unitCode="NIU">10.0</cbc:InvoicedQuantity>
    <cbc:LineExtensionAmount currencyID="PEN">150.00</cbc:LineExtensionAmount>
    <cac:PricingReference>
      <cac:AlternativeConditionPrice>
        <cbc:PriceAmount currencyID="PEN">17.70</cbc:PriceAmount>
        <cbc:PriceTypeCode>01</cbc:PriceTypeCode>
      </cac:AlternativeConditionPrice>
    </cac:PricingReference>
    <cac:TaxTotal>
      <cbc:TaxAmount currencyID="PEN">27.00</cbc:TaxAmount>
      <cac:TaxSubtotal>
        <cbc:TaxableAmount currencyID="PEN">150.00</cbc:TaxableAmount>
        <cbc:TaxAmount currencyID="PEN">27.00</cbc:TaxAmount>
        <cac:TaxCategory>
          <cbc:Percent>18.00</cbc:Percent>
          <!-- CÓDIGO DE AFECTACIÓN AL IGV: CATÁLOGO 07 -->
          <cbc:TaxExemptionReasonCode>10</cbc:TaxExemptionReasonCode>
          <cac:TaxScheme>
            <cbc:ID>1000</cbc:ID>
            <cbc:Name>IGV</cbc:Name>
            <cbc:TaxTypeCode>VAT</cbc:TaxTypeCode>
          </cac:TaxScheme>
        </cac:TaxCategory>
      </cac:TaxSubtotal>
    </cac:TaxTotal>
    <cac:Item>
      <cbc:Description><![CDATA[Tubo PVC Presión 1/2 pulgada x 5 metros]]></cbc:Description>
      <cac:SellersItemIdentification>
        <cbc:ID>PRD-TUB-01</cbc:ID>
      </cac:SellersItemIdentification>
    </cac:Item>
    <cac:Price>
      <cbc:PriceAmount currencyID="PEN">15.00</cbc:PriceAmount>
    </cac:Price>
  </cac:InvoiceLine>
</Invoice>
```
