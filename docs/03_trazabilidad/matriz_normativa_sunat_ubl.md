# REGULATORY TRACEABILITY MATRIX (SUNAT PERÚ vs OASIS UBL 2.1 vs OPENAPI SCHEMAS)
**Company:** ABC Software Corp  
**Phase:** Phase 1 - Spec-First (Standardized in English / Peruvian Tax Alignment)  
**Base Standard:** OASIS UBL 2.1 (`UBL-Invoice-2.1.xsd`)  
**SUNAT Regulations:** RS N° 097-2012, RS N° 193-2020 y modificatorias; Ley N° 30884 (ICBPER).

---

## 1. Regulatory Context and Technical Guidelines (Contexto Normativo)

Esta matriz establece el mapeo estricto entre las exigencias tributarias de la **SUNAT**, las etiquetas del estándar internacional **OASIS UBL 2.1** y los campos estandarizados en inglés de la especificación **OpenAPI 3.0.3** (`spec/schemas/` y `openapi.yaml`).

### Puntos Obligatorios de Cumplimiento Normativo:
1. **`<cbc:UBLVersionID>`:** Debe contener de forma fija e invariable el valor **`2.1`** (campo `ublVersionId`).
2. **`<cbc:CustomizationID>`:** Debe contener de forma fija e invariable el valor **`2.0`** (campo `customizationId`, versión funcional de SUNAT).
3. **`<cbc:ID>` (Identificador Legal del Comprobante):** NO admite UUIDs ni identificadores aleatorios. Es estrictamente la concatenación determinística `{series}-{sequenceNumber:08d}` $\rightarrow$ **`F001-00000123`** (campo `invoiceNumber`).
4. **Afectación al IGV:** Se codifica mediante el **Catálogo 07 de SUNAT** (campo `igvTaxExemptionCode`, ej. `10` Gravado - Operación Onerosa, `20` Exonerado, `30` Inafecto).
5. **Sistema de Cálculo del ISC:** Se codifica mediante el **Catálogo 08 de SUNAT** (campo `iscCalculationSystemCode`, ej. `01` Sistema al Valor, `02` Sistema Específico, `03` Sistema al Valor según PVP).
6. **Términos Legales Peruanos:** Se preservan intactas las siglas y conceptos jurídicos de Perú: **SUNAT, RUC, IGV, ISC, ICBPER, Ubigeo**.

---

## 2. Comprehensive Traceability Matrix (Matriz de Mapeo Normativo)

| N° | Requisito Normativo SUNAT | Jerarquía y Etiqueta en XML UBL 2.1 | Campo en Schema OpenAPI (JSON) | Catálogo SUNAT Aplicable | Regla Técnica y Transformación |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **1** | **Versión de UBL** | `/Invoice/cbc:UBLVersionID` | `ublVersionId` | **Estándar OASIS UBL 2.1** | **Valor fijo '2.1'**. Exigido por SUNAT para comprobantes electrónicos UBL 2.1. |
| **2** | **Versión de Estructura SUNAT** | `/Invoice/cbc:CustomizationID` | `customizationId` | **RS 097-2012/SUNAT** | **Valor fijo '2.0'**. Representa la versión de especificaciones funcionales de SUNAT. |
| **3** | **Serie y Correlativo (ID Legal)** | `/Invoice/cbc:ID` | `invoiceNumber`<br>*(o `{series}-{sequenceNumber}`)* | **RS 097-2012/SUNAT** | **REGLA ESTRICTA:** No admite UUIDs. Debe ser `{series}-{sequenceNumber:08d}` (ej. `F001-00000123`). Serie inicia con `'F'` y correlativo con 8 dígitos rellenado con ceros a la izquierda. Coincide exactamente con el nombre del XML (`20123456789-01-F001-00000123.xml`). |
| **4** | **Fecha de Emisión** | `/Invoice/cbc:IssueDate` | `issueDate` | **Norma SUNAT / ISO 8601** | Formato `YYYY-MM-DD`. Plazo máximo de hasta 3 días hábiles anteriores a la fecha de remisión a SUNAT. |
| **5** | **Hora de Emisión** | `/Invoice/cbc:IssueTime` | `issueTime` | **Norma SUNAT / ISO 8601** | Formato militar `HH:MM:SS`. Hora de generación del comprobante. |
| **6** | **Tipo de Comprobante de Pago** | `/Invoice/cbc:InvoiceTypeCode` | `invoiceTypeCode` | **Catálogo 01** *(Tipo de comprobante)* | Código `'01'` correspondiente a Factura Electrónica. |
| **7** | **Tipo de Operación Comercial** | `/Invoice/cbc:InvoiceTypeCode/@listID` | `operationTypeCode` | **Catálogo 51** *(Tipo de operación)* | Atributo `@listID` con valor `'0101'` ("Venta Interna"). |
| **8** | **Código de Moneda** | `/Invoice/cbc:DocumentCurrencyCode` | `currencyCode` | **Catálogo 02** *(Tipo de moneda)* | Código alfabético ISO 4217: `'PEN'` (Soles) o `'USD'` (Dólares). Afecta el atributo `@currencyID` en montos monetarios. |
| **9** | **RUC de la Empresa Emisora** | `/Invoice/cac:AccountingSupplierParty`<br>`/cac:Party/cac:PartyIdentification/cbc:ID`<br>`@schemeID="6"` | `issuerRuc` | **Catálogo 06** *(Tipo de documento)* | RUC de 11 dígitos de ABC Software Corp. Inyectado con `@schemeID="6"`. |
| **10** | **Razón Social de la Emisora** | `/Invoice/cac:AccountingSupplierParty`<br>`/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `issuerLegalName` | **Padrón RUC SUNAT** | Razón social oficial: `"ABC SOFTWARE CORP S.A.C."`. Se envuelve en CDATA en el XML. |
| **11** | **Ubigeo y Domicilio Fiscal Emisor** | `/Invoice/cac:AccountingSupplierParty`<br>`/cac:Party/cac:PostalAddress/cbc:ID` | `issuerUbigeo` | **Catálogo INEI / SUNAT** | Código numérico de 6 dígitos del distrito (ej. `'150131'` para San Isidro, Lima). |
| **12** | **Tipo Documento Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PartyIdentification/cbc:ID`<br>`@schemeID` | `customer.identityDocumentType` | **Catálogo 06** *(Tipo de documento)* | Mandatorio `'6'` (RUC) para Facturas Electrónicas (`01`). |
| **13** | **Número de RUC del Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PartyIdentification/cbc:ID` | `customer.identityDocumentNumber` | **Catálogo 06 / Módulo 11** | 11 dígitos numéricos validados con algoritmo Módulo 11 de SUNAT (estado Activo y Habido). |
| **14** | **Razón Social del Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `customer.legalName` | **Padrón RUC SUNAT** | Razón social oficial del cliente registrada en SUNAT. |
| **15** | **Ubigeo y Domicilio Adquirente** | `/Invoice/cac:AccountingCustomerParty`<br>`/cac:Party/cac:PostalAddress/cbc:ID` | `customer.ubigeo` | **Catálogo INEI / SUNAT** | Código de 6 dígitos del distrito del adquirente (ej. `'150103'` para Ate, Lima). |
| **16** | **Monto Operaciones Gravadas** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cbc:TaxableAmount` *(para tributo 1000)* | `taxableOperationsAmount`<br>*(o `items[].salesValue`)* | **Catálogo 05** *(Tipos de tributos)* | Sumatoria de bases imponibles de ítems con afectación gravada onerosa (`10`). |
| **17** | **Total IGV (18%)** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cac:TaxCategory/cac:TaxScheme/cbc:ID`<br>`= "1000"` con `cbc:TaxAmount` | `totalIgv`<br>`items[].igvAmount` | **Catálogo 05** *(Tipos de tributos)* | Bloque de impuesto con `cbc:ID` = `1000`, `cbc:Name` = `'IGV'`, `cbc:TaxTypeCode` = `'VAT'`. Tasa 18%. |
| **18** | **Código Afectación IGV por Ítem** | `/Invoice/cac:InvoiceLine/cac:TaxTotal`<br>`/cac:TaxSubtotal/cac:TaxCategory`<br>`/cbc:TaxExemptionReasonCode` | `items[].igvTaxExemptionCode` | **Catálogo 07** *(Afectación al IGV)* | **CATÁLOGO 07:** `'10'` (Gravado Oneroso), `'20'` (Exonerado), `'30'` (Inafecto), `'11'` (Retiro por premio). |
| **19** | **Total ISC (Impuesto Selectivo)** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cac:TaxCategory/cac:TaxScheme/cbc:ID`<br>`= "2000"` con `cbc:TaxAmount` | `totalIsc`<br>`items[].iscAmount` | **Catálogo 05** *(Tipos de tributos)* | Tributo código `2000`, `cbc:Name` = `'ISC'`, `cbc:TaxTypeCode` = `'EXC'`. |
| **20** | **Sistema Cálculo ISC por Ítem** | `/Invoice/cac:InvoiceLine/cac:TaxTotal`<br>`/cac:TaxSubtotal/cac:TaxCategory`<br>`/cac:TaxScheme/cbc:ID = "2000"` | `items[].iscCalculationSystemCode` | **Catálogo 08** *(Sistema cálculo ISC)* | **CATÁLOGO 08:** `'01'` (Sistema al Valor), `'02'` (Sistema Específico), `'03'` (Al Valor según PVP). |
| **21** | **Impuesto Bolsas Plásticas (ICBPER)** | `/Invoice/cac:TaxTotal/cac:TaxSubtotal`<br>`/cac:TaxCategory/cac:TaxScheme/cbc:ID`<br>`= "7152"` con `cbc:TaxAmount` | `totalIcbper`<br>`items[].icbperAmount` | **Catálogo 05 / Ley 30884** | Código tributo `7152`, categoría `'OTH'`. Tarifa fija de S/ 0.50 por bolsa plástica. |
| **22** | **Unidad de Medida Comercial** | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity`<br>`@unitCode` | `items[].unitOfMeasure` | **Catálogo 03** *(Unidades de medida)* | Código internacional: `'NIU'` (Unidades), `'KGM'` (Kilogramos), `'ZZ'` (Servicios), `'MTR'` (Metros), `'BX'` (Cajas). |
| **23** | **Precio de Venta con Impuesto** | `/Invoice/cac:InvoiceLine/cac:PricingReference`<br>`/cac:AlternativeConditionPrice/cbc:PriceAmount` | `items[].unitPriceWithIgv` | **Catálogo 16** *(Tipo de precio)* | Con atributo `@PriceTypeCode="01"` correspondiente a "Precio unitario comercial incluye IGV". |
| **24** | **Valor Total de Venta (Base Neta)** | `/Invoice/cac:LegalMonetaryTotal`<br>`/cbc:LineExtensionAmount` | `taxableOperationsAmount` | **Cuadre Contable UBL 2.1** | Sumatoria de bases imponibles netas antes de impuestos y descuentos globales. |
| **25** | **Importe Total a Pagar** | `/Invoice/cac:LegalMonetaryTotal`<br>`/cbc:PayableAmount` | `totalPayableAmount` | **RS 097-2012/SUNAT** | Importe final de la transacción: `LineExtensionAmount` + `TaxTotal` - Descuentos. `@currencyID="PEN"`. |
| **26** | **Firma Digital XML-DSig y Hash** | `/Invoice/ext:UBLExtensions/.../ds:DigestValue` | `digestValue`<br>`signatureValue` | **Estándar W3C XML-DSig / SUNAT** | Hash SHA-256 Base64 extraído para la generación obligatoria del código QR y PDF. |

---

## 3. Concrete Example: JSON Payload vs SUNAT UBL 2.1 XML

### JSON Input Payload (`InvoiceInput`):
```json
{
  "ublVersionId": "2.1",
  "customizationId": "2.0",
  "invoiceTypeCode": "01",
  "series": "F001",
  "sequenceNumber": 123,
  "issueDate": "2026-10-22",
  "issueTime": "10:15:30",
  "currencyCode": "PEN",
  "operationTypeCode": "0101",
  "issuerRuc": "20123456789",
  "issuerLegalName": "ABC SOFTWARE CORP S.A.C.",
  "issuerUbigeo": "150131",
  "issuerFiscalAddress": "Av. Las Camelias 820, San Isidro, Lima",
  "customer": {
    "identityDocumentType": "6",
    "identityDocumentNumber": "20601234567",
    "legalName": "DISTRIBUIDORA LOGISTICA DEL CENTRO S.A.C.",
    "fiscalAddress": "Av. Industrial 450, Urb. Vulcano, Ate, Lima",
    "ubigeo": "150103"
  },
  "paymentTerms": {
    "type": "Contado",
    "netPendingAmount": 0.00
  },
  "items": [
    {
      "lineNumber": 1,
      "productCode": "PRD-TUB-01",
      "description": "Tubo PVC Presión 1/2 pulgada x 5 metros",
      "unitOfMeasure": "NIU",
      "quantity": 10.0,
      "unitValue": 15.00,
      "unitPriceWithIgv": 17.70,
      "igvTaxExemptionCode": "10",
      "igvRate": 18.00,
      "igvAmount": 27.00,
      "iscCalculationSystemCode": "01",
      "iscRate": 0.0,
      "iscAmount": 0.0,
      "appliesIcbper": false,
      "plasticBagQuantity": 0,
      "icbperAmount": 0.00,
      "salesValue": 150.00
    }
  ]
}
```

### Generated Official SUNAT UBL 2.1 XML (`UBLGenerator`):
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

  <!-- SUNAT MANDATORY HEADER ATTRIBUTES -->
  <cbc:UBLVersionID>2.1</cbc:UBLVersionID>
  <cbc:CustomizationID>2.0</cbc:CustomizationID>
  <cbc:ID>F001-00000123</cbc:ID>
  <cbc:IssueDate>2026-10-22</cbc:IssueDate>
  <cbc:IssueTime>10:15:30</cbc:IssueTime>
  <cbc:InvoiceTypeCode listID="0101">01</cbc:InvoiceTypeCode>
  <cbc:DocumentCurrencyCode>PEN</cbc:DocumentCurrencyCode>

  <!-- ISSUER DATA (ABC Software Corp) -->
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

  <!-- CUSTOMER DATA -->
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

  <!-- GLOBAL TAX TOTALS (IGV TAX 1000 VAT) -->
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

  <!-- LEGAL MONETARY TOTALS -->
  <cac:LegalMonetaryTotal>
    <cbc:LineExtensionAmount currencyID="PEN">150.00</cbc:LineExtensionAmount>
    <cbc:TaxInclusiveAmount currencyID="PEN">177.00</cbc:TaxInclusiveAmount>
    <cbc:PayableAmount currencyID="PEN">177.00</cbc:PayableAmount>
  </cac:LegalMonetaryTotal>

  <!-- INVOICE LINE ITEM -->
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
          <!-- IGV TAX EXEMPTION CODE: SUNAT CATALOG 07 -->
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

---

## 4. Critical Technical Rule: System ID (UUID) vs `<cbc:ID>` (SUNAT UBL 2.1)

En el diseño de sistemas de facturación electrónica en el Perú:

1. **Separación de Responsabilidades:**
   * **`id` en API REST / Base de Datos:** Se utiliza un UUID v4 (ej. `550e8400-e29b-41d4-a716-446655440000`) para garantizar inmutabilidad, no colisión distribuida y trazabilidad técnica en la capa de software.
   * **`<cbc:ID>` en XML UBL 2.1:** Representa exclusivamente el **Identificador Legal y Tributario del Comprobante** (`invoiceNumber`). Está compuesto obligatoriamente por la concatenación:
     $$\text{cbc:ID} = \text{series} + \text{"-"} + \text{sequenceNumber (formateado con 8 dígitos)}$$
     $$\text{Ejemplo: } \mathbf{F001-00000123}$$

2. **Reglas de Validación Sintáctica (SUNAT RS 097-2012):**
   * **`series` (4 caracteres):** Para Facturas debe iniciar obligatoriamente con la letra `F` seguida de 3 caracteres alfanuméricos en mayúsculas (Regex: `^[F][A-Z0-9]{3}$`, ej. `F001`, `FA01`).
   * **`sequenceNumber`:** Número entero positivo correlativo secuencial entre `1` y `99999999` rellenado con ceros a la izquierda a 8 dígitos (`F001-00000123`).

3. **Trazabilidad Cruzada con el Archivo Físico:**
   * El valor contenido en `<cbc:ID>` debe ser idéntico al componente de numeración en el nombre del archivo XML y del archivo comprimido ZIP enviado al Web Service SOAP:
     * Nombre del XML: `[RUC_EMISOR]-[TIPO_COMPROBANTE]-[SERIE]-[CORRELATIVO].xml`  
       $\rightarrow$ `20123456789-01-F001-00000123.xml`
     * Valor en `/Invoice/cbc:ID`: `F001-00000123`
   * Si existe la más mínima discrepancia entre el nombre del archivo y el contenido de `<cbc:ID>`, SUNAT/OSE emitirá el **código de error de rechazo 2014** (*"El número de comprobante del archivo no coincide con el contenido del XML"*).
