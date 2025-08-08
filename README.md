# 📦 API de Pedidos e Notas Fiscais (Firebird)

Esta API integra com banco de dados Firebird para **criar pedidos**, **inserir NFs**, **anexar XMLs** e **consultar pedidos**.  
Os inserts utilizam *generators* do Firebird quando aplicável e campos textuais são codificados em `win1252`.

---

## 🚀 Endpoints


**Endpoints:**
**api.tisoluciona.com/api/**


### 1) Criar Pedido
**Rota:** `POST /criar-pedido`

**Descrição:**  
Cria um pedido (registro em `TABMOVTRA`) e, opcionalmente, insere as NFs associadas (`TABMOVTRA_NF`).  
> Observação: no seu banco, o campo `DATA_HORA` é armazenado como string `HH:MM`.

**Campos obrigatórios do body:**
- `placaCavalo`
- `localColeta`
- `localEntrega`
- `clienteFaturamento`
- `processoCliente`
- `tipoContainer`
- `numeroContainer`
- `nomeMotorista`
- `placaCarreta1`
- `empresa`
- `tipoFrete`
- `tipoCarga`
- `pesoBrutoTotal`
- `valorTotalNotas`
- `notasFiscais` *(array; pode estar vazia, mas o campo deve existir)*

**Exemplo de Entrada:**
```json
{
  "placaCavalo": "MBQ9466",
  "localColeta": 4776,
  "localEntrega": 3334,
  "clienteFaturamento": 3003,
  "processoCliente": "PROC-XYZ",
  "tipoContainer": 1,
  "numeroContainer": "ABCD1234567",
  "nomeMotorista": 5102,
  "placaCarreta1": "MEE3078",
  "placaCarreta2": null,
  "empresa": 1,
  "tipoFrete": 1,
  "tipoCarga": 2,
  "pesoBrutoTotal": 18500.75,
  "valorTotalNotas": 32000.5,
  "usuario": "INTEGRADOR1",
  "notasFiscais": [
    {
      "nonf": 456789,
      "dataemi": "2025-08-06",
      "pesobr": 5000.75,
      "vlrtotal": 12499.99,
      "chavenfe": "35230806748539000105550010000123451000012340",
      "cfop": "5353",
      "usuario": "INTEGRADOR1"
    }
  ]
}
```

**Resposta de Exemplo:**
```json
{
  "status": "OK",
  "mensagem": "Pedido inserido com sucesso.",
  "nomovtra": 999960,
  "nomeMotorista": 5102,
  "cpfMotorista": "35678158000108"
}
```

---

### 2) Inserir Nota Fiscal
**Rota:** `POST /inserir-nf`

**Descrição:**  
Insere uma **NF** em `TABMOVTRA_NF` para um `nomovtra` existente.  
O campo `ITEM` é gerado automaticamente pelo banco com `GEN_ID(ITEMMOVTRA, 1)` e `TIPONF` é fixo `"E"`.

**Campos obrigatórios do body:**
- `nomovtra`
- `senf`
- `dataemi`
- `qtdevol`
- `pesobr`
- `esp`
- `prod`
- `chavenfe`
- `vlrtotal`
- `bc`
- `icms`
- `m3`
- `descroutros`
- `tpdoc`
- `nonf`
- `cfop`
- `vrmerc`

**Exemplo de Entrada:**
```json
{
  "nomovtra": 999953,
  "senf": "001",
  "dataemi": "2025-08-06",
  "qtdevol": 0,
  "pesobr": 5000.75,
  "esp": "CAIXA",
  "prod": "ELETRONICOS",
  "chavenfe": "35230806748539000105550010000123451000012340",
  "vlrtotal": 12499.99,
  "bc": 12499.99,
  "icms": 1874.99,
  "m3": 12.5,
  "descroutros": "NF de teste SIMULANDO",
  "tpdoc": "55",
  "nonf": 456789,
  "cfop": "5353",
  "vrmerc": 12499.99
}
```

**Resposta de Sucesso:**
```json
{
  "message": "Nota Fiscal registrada com sucesso.",
  "item": 1234
}
```

---

### 3) Inserir XML da Nota Fiscal
**Rota:** `POST /inserir-xml`

**Descrição:**  
Anexa o **XML da NF** em `TABMOVTRA_NF_XML`.  
O campo `ITEMXMLNFE` é gerado automaticamente pelo banco com `GEN_ID(GEN_ITEMXMLNFE, 1)` e `DATAREG` é a data atual (`YYYY-MM-DD`).

**Campos obrigatórios do body:**
- `nomovtra`
- `item` *(ITEM da NF gerado na rota acima)*
- `xml` *(conteúdo completo do XML)*

**Exemplo de Entrada:**
```json
{
  "nomovtra": 999954,
  "item": 12,
  "xml": "<nfeProc versao=\"4.00\" xmlns=\"http://www.portalfiscal.inf.br/nfe\"><NFe>...</NFe></nfeProc>"
}
```

**Resposta de Sucesso:**
```json
{
  "message": "XML da NF registrado com sucesso.",
  "itemxmlnfe": 5678
}
```

---

### 4) Consultar Pedido
**Rota:** `GET /pedido`  
**Parâmetros de Query:**  
- `nomovtra` *(obrigatório)* — Número do movimento de transporte.

**Descrição:**  
Retorna os dados do pedido, NFs, documentos fiscais e contrato de frete.  
Datas são formatadas para `dd/MM/yyyy` e horas para `dd/MM/yyyy HH:mm` quando aplicável.

**Exemplo de Entrada:**
```http
GET /pedido?nomovtra=999960
```

**Resposta de Exemplo:**
```json
{
  "numeroOrdem": 999960,
  "nomeMotorista": "João da Silva",
  "cpfMotorista": "12345678900",
  "placaCavalo": "AAA1234",
  "dataOperacao": "07/08/2025",
  "horaOperacao": "07/08/2025 14:30",
  "localColeta": 4776,
  "localEntrega": 3334,
  "clienteFaturamento": 3003,
  "processoCliente": "PROC-XYZ",
  "tipoContainer": 1,
  "numeroContainer": "ABCD1234567",
  "placaCarreta1": "BBB1234",
  "placaCarreta2": null,
  "empresa": 1,
  "tipoFrete": 1,
  "tipoCarga": 2,
  "notasFiscais": [],
  "cfopsUtilizados": [],
  "documentosFiscais": [],
  "contratoFrete": {}
}
```

---

## 🧩 Comportamentos Importantes
- `ITEM` em `TABMOVTRA_NF` é **sempre** gerado pelo banco: `SELECT GEN_ID(ITEMMOVTRA, 1) ...`  
- `ITEMXMLNFE` em `TABMOVTRA_NF_XML` é **sempre** gerado pelo banco: `SELECT GEN_ID(GEN_ITEMXMLNFE, 1) ...`  
- `TIPONF` é fixo `"E"` nas NFs.  
- Codificação `win1252` é aplicada apenas nos campos textuais enviados ao banco.  
- `DATA_HORA` do pedido é armazenado como string `HH:MM` no seu ambiente.

---

## ⚙️ Requisitos
- Node.js + Express
- Firebird (serviço em `services/firebirdService`)
- `iconv-lite` para encoding

