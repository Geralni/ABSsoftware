# DOCUMENTO DE ARQUITECTURA DE SOFTWARE (MVC CON VISIÓN HEXAGONAL)
**Proyecto:** ABC Software Corp  
**Módulo:** Fase 1 - Spec-First  
**Arquitectura:** Monolito Limpio Modular MVC con Puertos y Adaptadores (Hexagonal)  
**Salida Multiplataforma:** Windows (.MSI) y Android (.APK)  

---

## 1. Visión y Estrategia Arquitectónica

El sistema de facturación electrónica para **ABC Software Corp** inicia como un **Monolito Limpio Modular** fundamentado en el patrón **Modelo-Vista-Controlador (MVC)**, pero estructurado rigurosamente bajo los principios de la **Arquitectura Hexagonal (Puertos y Adaptadores)** y **Clean Architecture**.

### ¿Por qué esta decisión arquitectónica?
1. **Simplicidad Operativa Inicial (Time-to-Market):** Un monolito modular elimina la sobrecarga operativa de microservicios (latencia de red, transacciones distribuidas, orquestación de contenedores) en una empresa que viene de un sistema de almacén informal.
2. **Desacoplamiento Radical (Fronteras Claras):** La lógica del negocio (cálculo de impuestos peruanos, reglas de validación de SUNAT, estados del comprobante) es agnóstica de frameworks web, motores de base de datos y protocolos de transporte.
3. **Migración Futura sin Reescribir:** Al interactuar exclusivamente a través de **Puertos (Interfaces)**, el núcleo puede evolucionar a microservicios distribuidos sin modificar una sola línea de lógica de dominio.

---

## 2. Salida Multiplataforma: Clientes Nativos

Para atender las necesidades operativas de la empresa (mostrador de venta fijo y atención móvil en almacén), la capa de presentación se desacopla en dos clientes nativos:

```
                  ┌───────────────────────────────┐
                  │      Terminal de Venta        │
                  │        (Mostrador)            │
                  └──────────────┬────────────────┘
                                 │
                  ┌──────────────▼────────────────┐
                  │   Cliente Windows (.MSI)      │
                  │   - C# / .NET / WPF           │
                  │   - Drivers ESC/POS (USB)     │
                  │   - Lector de código de barras│
                  └──────────────┬────────────────┘
                                 │ HTTP/JSON
                                 │ (OpenAPI 3.0.3)
      ┌──────────────────────────┴──────────────────────────┐
      │                                                     │
┌─────▼─────────────────────────┐             ┌─────────────▼─────────────────┐
│ Servidor / Monolito Local POS │             │  Cliente Android (.APK)       │
│ - Controladores REST          │◄────────────┤  - Kotlin / Jetpack Compose   │
│ - Dominio Tributario SUNAT    │  HTTP/JSON  │  - Impresoras Móviles BLE/WiFi│
│ - UBLGenerator (XML UBL 2.1)  │             │  - Cámara / Escaneo de barras │
└───────────────────────────────┘             └───────────────────────────────┘
```

1. **Cliente Windows (.MSI):**
   * **Tecnología:** C# / .NET / WPF o Tauri / Rust Desktop.
   * **Empaquetado:** Instalador `.MSI` corporativo con soporte para instalación desatendida.
   * **Propósito:** Puestos de caja fijos de alta velocidad. Control directo de hardware mediante Win32/USB (gaveta portamonedas eléctrica, impresoras térmicas ESC/POS de 80mm, lectores de barras ópticos).
2. **Cliente Android (.APK):**
   * **Tecnología:** Kotlin / Jetpack Compose o Flutter nativo.
   * **Empaquetado:** Paquete `.APK` firmado para terminales POS portátiles (Sunmi, Clover, PAX o tablets comerciales).
   * **Propósito:** Facturación móvil en patio de carga, ventas en pasillo de almacén y preventa. Integración con Bluetooth Low Energy (BLE) para impresoras de cinturón.

Ambos clientes consumen de forma unificada la especificación REST definida en el contrato `openapi.yaml`.

---

## 3. El Componente Clave: `UBLGenerator` (El Puente)

El estándar **UBL 2.1 (Universal Business Language)** de OASIS adaptado por SUNAT es extremadamente exigente a nivel sintáctico y de seguridad criptográfica.

Para evitar que el modelo de dominio se contamine con la complejidad de esquemas XML, se crea un componente especializado denominado **`UBLGenerator`**:

```
                       UBLGenerator (El Puente)
 ┌────────────────────────────────────────────────────────────────────────┐
 │                                                                        │
 │  DTO / Entidad Factura           Árbol DOM XML conforme                │
 │  (JSON / Domain Model)           a UBL-Invoice-2.1.xsd                 │
 │  ┌─────────────────────────┐     ┌──────────────────────────────────┐  │
 │  │ - ublVersionId: '2.1'   │     │ <Invoice xmlns="urn:oasis...">   │  │
 │  │ - customizationId: '2.0'│     │   <cbc:UBLVersionID>2.1</...>    │  │
 │  │ - rucEmisor: 20123456789│ ──> │   <cbc:CustomizationID>2.0</...> │  │
 │  │ - serie: F001           │     │   <cbc:ID>F001-00000123</cbc:ID> │  │
 │  │ - correlativo: 123      │     │   <cac:TaxTotal>...</cac:TaxTot> │  │
 │  │ - afectacionIgv: Cat. 07│     │ </Invoice>                       │  │
 │  │ - sistemaIsc: Cat. 08   │     └────────────────┬─────────────────┘  │
 │  │ - ubigeo: '150131'      │                      │                    │
 │  └─────────────────────────┘             Canonicalización (C14N)       │
 │                                                   │                    │
 │                                          Firma Digital XML-DSig        │
 │                                          (Certificado X.509 v3)        │
 │                                                   │                    │
 │                                          ┌────────▼─────────────────┐  │
 │                                          │ Inyección en             │  │
 │                                          │ ext:UBLExtensions        │  │
 │                                          └────────┬─────────────────┘  │
 │                                                   │                    │
 │                                     ┌─────────────┴────────────┐       │
 │                                     ▼                          ▼       │
 │                             XML Firmado Final           DigestValue    │
 │                             (2012...-01-F001.xml)     (Hash para QR)   │
 └────────────────────────────────────────────────────────────────────────┘
```

### Subcomponentes de `UBLGenerator`:
* **`XMLMapper`:** Mapea las estructuras internas de la factura al esquema formal OASIS UBL 2.1 (`UBL-Invoice-2.1.xsd`), estructurando los nodos `cac:AccountingSupplierParty`, `cac:AccountingCustomerParty`, `cac:TaxTotal`, `cac:LegalMonetaryTotal` y las líneas `cac:InvoiceLine`.
* **`XMLDigitalSigner`:** Implementa la especificación XML-DSig (`enveloped`) bajo el estándar W3C. Lee el certificado digital tributario X.509 v3 (PKCS#12 `.pfx` o `.p12`), calcula el hash canónico del documento y genera el bloque criptográfico `ds:SignatureValue` y `ds:KeyInfo`.
* **`DigestValueExtractor`:** Extrae el valor resumen SHA-256 generado por la firma digital (`ds:DigestValue`), el cual es obligatorio para imprimir el código QR y generar la representación impresa en PDF.
* **`XSDValidator`:** Validador sintáctico estricto en memoria que comprueba el documento contra `UBL-Invoice-2.1.xsd` antes de autorizar el envío a SUNAT, previniendo rechazos en el OSE.

---

## 4. Diagrama Integral de Arquitectura (Mermaid)

```mermaid
flowchart TB
    %% CLIENTES MULTIPLATAFORMA
    subgraph Multiplataforma["CAPA CLIENTES MULTIPLATAFORMA"]
        WinMSI["🖥️ Cliente Nativo Windows (.MSI)<br/>WPF / .NET / C# Desktop<br/>(Impresoras ESC/POS USB, Gavetas, Lectores)"]
        AndroidAPK["📱 Cliente Nativo Android (.APK)<br/>Kotlin / Jetpack Mobile POS<br/>(Tablets de Almacén, Impresoras Bluetooth BLE)"]
    end

    %% ADAPTADORES DE ENTRADA (DRIVING ADAPTERS / REST)
    subgraph DrivingAdapters["ADAPTADORES DE ENTRADA (Driving Adapters / MVC Controllers)"]
        CtrlFactura["🎮 FacturaRestController<br/>POST /api/v1/facturas<br/>GET /api/v1/facturas/{id}<br/>GET /api/v1/facturas/{id}/xml"]
        CtrlAnular["🎮 AnulacionRestController<br/>POST /api/v1/facturas/{id}/anular"]
        CtrlMigrar["🎮 MigracionRestController<br/>POST /api/v1/migracion/legado"]
    end

    WinMSI -->|HTTP / JSON (OpenAPI)| CtrlFactura
    WinMSI -->|HTTP / JSON (OpenAPI)| CtrlAnular
    WinMSI -->|HTTP / JSON (OpenAPI)| CtrlMigrar
    
    AndroidAPK -->|HTTP / JSON (OpenAPI)| CtrlFactura
    AndroidAPK -->|HTTP / JSON (OpenAPI)| CtrlAnular

    %% PUERTOS DE ENTRADA (INPUT PORTS)
    subgraph InputPorts["PUERTOS DE ENTRADA (Input Ports / Use Case Interfaces)"]
        PortEmitir["📥 InputPort: EmitirFacturaUseCase"]
        PortAnular["📥 InputPort: AnularFacturaUseCase"]
        PortMigrar["📥 InputPort: MigrarLegadoUseCase"]
    end

    CtrlFactura --> PortEmitir
    CtrlAnular --> PortAnular
    CtrlMigrar --> PortMigrar

    %% CAPA DE APLICACIÓN (APPLICATION INTERACTORS)
    subgraph ApplicationServices["CAPA DE APLICACIÓN (Interactors / Application Services)"]
        SvcEmitir["⚙️ EmitirFacturaService<br/>(Orquestador de emisión y validación)"]
        SvcAnular["⚙️ AnularFacturaService<br/>(Orquestador de bajas tributarias)"]
        SvcMigrar["⚙️ MigracionLegadoService<br/>(Saneamiento de catálogo y clientes)"]
    end

    PortEmitir --> SvcEmitir
    PortAnular --> SvcAnular
    PortMigrar --> SvcMigrar

    %% CAPA DE DOMINIO (DOMAIN CORE)
    subgraph DomainCore["CAPA DE DOMINIO (Entidades, Value Objects y Servicios de Dominio)"]
        EntFactura["📦 Entidad: Factura"]
        EntDetalle["📦 Entidad: DetalleFactura"]
        VOCliente["💎 Value Object: Cliente (RUC)"]
        VOTributos["💎 Value Object: DesgloseTributario"]
        
        CalcTributos["🧠 Domain Service: CalculadorTributosPeruanos<br/>- Base Imponible Gravada (Cat. 07: 10)<br/>- IGV 18% (Cat. 05: 1000 VAT)<br/>- ISC (Cat. 08 y Cat. 05: 2000 EXC)<br/>- ICBPER (Ley 30884: 7152 OTH)"]
        ValidadorNormas["🧠 Domain Service: ValidadorNormativoSUNAT<br/>- Algoritmo Módulo 11 (RUC)<br/>- Estructura Serie F### / B###<br/>- Plazo máximo 3 días hábiles"]
    end

    SvcEmitir --> EntFactura
    SvcEmitir --> CalcTributos
    SvcEmitir --> ValidadorNormas
    SvcMigrar --> ValidadorNormas

    %% PUERTOS DE SALIDA (OUTPUT PORTS)
    subgraph OutputPorts["PUERTOS DE SALIDA (Output Ports / SPI Interfaces)"]
        PortUBL["🔌 OutputPort: UBLGeneratorPort"]
        PortRepo["🔌 OutputPort: FacturaRepositoryPort"]
        PortSunat["🔌 OutputPort: SunatClientPort"]
        PortStorage["🔌 OutputPort: StoragePort"]
    end

    SvcEmitir --> PortUBL
    SvcEmitir --> PortRepo
    SvcEmitir --> PortSunat
    SvcEmitir --> PortStorage
    SvcMigrar --> PortRepo

    %% COMPONENTE CLAVE: EL PUENTE (UBLGenerator)
    subgraph BridgeComponent["COMPONENTE CLAVE: EL PUENTE (UBLGenerator Adapter)"]
        subgraph UBLGeneratorAdapter["UBLGenerator (Transformador y Firmador)"]
            MapperUBL["🔄 XMLMapper<br/>Transforma Factura a árbol OASIS UBL-Invoice-2.1.xsd"]
            SignerX509["🔏 XMLDigitalSigner<br/>Firma enveloped XML-DSig con Certificado X.509 v3"]
            ExtractorHash["🔑 DigestValueExtractor<br/>Calcula y extrae Hash SHA-256 para Código QR"]
            ValidatorXSD["🛡️ XSDValidator<br/>Valida el XML contra esquemas oficiales OASIS"]
        end
    end

    PortUBL --> UBLGeneratorAdapter
    MapperUBL --> SignerX509
    SignerX509 --> ExtractorHash
    ExtractorHash --> ValidatorXSD

    %% ADAPTADORES DE SALIDA (DRIVEN ADAPTERS)
    subgraph DrivenAdapters["ADAPTADORES DE SALIDA (Driven Adapters / Infraestructura)"]
        RepoPostgres["🗄️ PostgresSqlFacturaRepository<br/>(Base de datos principal transaccional)"]
        RepoSqlite["🗄️ SqliteLocalRepository<br/>(Caché local offline para Windows y Android)"]
        ClientSunat["☁️ SunatOseSoapClientAdapter<br/>(Web Service BillService SOAP / MTOM / WS-Security)"]
        StorageXml["📁 FileSystem / S3StorageAdapter<br/>(Almacenamiento de XMLs firmados, CDRs y PDFs)"]
    end

    PortRepo --> RepoPostgres
    PortRepo -.-> RepoSqlite
    PortSunat --> ClientSunat
    PortStorage --> StorageXml

    %% COMUNICACIÓN EXTERNA
    ClientSunat <==>|SOAP XML-DSig ZIP / CDR Response| SunatOSE["🏛️ SUNAT / OSE (Servidores Oficiales)"]
```
