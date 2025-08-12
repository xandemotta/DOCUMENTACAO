# 📦 API de Pedidos e Notas Fiscais (Firebird)

Esta API integra com banco de dados Firebird para **criar pedidos**, **inserir NFs**, **anexar XMLs** e **consultar pedidos**.  
Os inserts utilizam *generators* do Firebird quando aplicável e campos textuais são codificados em `win1252`.

---

## 🔗 Base e Autenticação

- **Base URL**: `https://api.tisoluciona.com/api`
- **Auth**: `Authorization: Bearer <JWT_TOKEN>`  
- **JWT_TOKEN de homologação**: `23hhahk34k54fjdhj3na234544jasjm2`

> Observação: no seu banco, o campo `DATA_HORA` é armazenado como string `HH:MM`.

---

## 🚀 Endpoints

### 1) Endpoints
- **Endpoint principal**: `https://api.tisoluciona.com/api/`

### 1.1) Healthcheck
- **GET** `/health`  
- **200** retorna `{ "status": "ok" }`

---

## 2) Criar Pedido
**Rota**: `POST /criar-pedido`

**Descrição**  
Cria um pedido (registro em `TABMOVTRA`) e, opcionalmente, insere as NFs associadas (`TABMOVTRA_NF`).

**Campos obrigatórios do body**
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

**Exemplo de Entrada**
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

**Resposta de Exemplo**
```json
{
  "status": "OK",
  "mensagem": "Pedido inserido com sucesso.",
  "nomovtra": 999960,
  "nomeMotorista": 5102,
  "cpfMotorista": "35678158000108"
}
```

**cURL**
```bash
curl -X POST "https://api.tisoluciona.com/api/criar-pedido"  -H "Authorization: Bearer 23hhahk34k54fjdhj3na234544jasjm2"  -H "Content-Type: application/json"  -d '{ "placaCavalo":"MBQ9466", "localColeta":4776, "localEntrega":3334, "clienteFaturamento":3003, "processoCliente":"PROC-XYZ", "tipoContainer":1, "numeroContainer":"ABCD1234567", "nomeMotorista":5102, "placaCarreta1":"MEE3078", "placaCarreta2":null, "empresa":1, "tipoFrete":1, "tipoCarga":2, "pesoBrutoTotal":18500.75, "valorTotalNotas":32000.5, "usuario":"INTEGRADOR1", "notasFiscais":[{ "nonf":456789, "dataemi":"2025-08-06", "pesobr":5000.75, "vlrtotal":12499.99, "chavenfe":"35230806748539000105550010000123451000012340", "cfop":"5353", "usuario":"INTEGRADOR1" }] }'
```

---

## 2.1) Inserir Nota Fiscal
**Rota**: `POST /inserir-nf`

**Descrição**  
Insere uma NF em `TABMOVTRA_NF` para um `nomovtra` existente. `ITEM` é gerado automaticamente com `GEN_ID(ITEMMOVTRA, 1)` e `TIPONF` é fixo `"E"`.

**Campos obrigatórios do body**
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

**Exemplo de Entrada**
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

**Resposta de Sucesso**
```json
{
  "message": "Nota Fiscal registrada com sucesso.",
  "item": 1234
}
```

**cURL**
```bash
curl -X POST "https://api.tisoluciona.com/api/inserir-nf"  -H "Authorization: Bearer 23hhahk34k54fjdhj3na234544jasjm2"  -H "Content-Type: application/json"  -d '{ "nomovtra":999953, "senf":"001", "dataemi":"2025-08-06", "qtdevol":0, "pesobr":5000.75, "esp":"CAIXA", "prod":"ELETRONICOS", "chavenfe":"35230806748539000105550010000123451000012340", "vlrtotal":12499.99, "bc":12499.99, "icms":1874.99, "m3":12.5, "descroutros":"NF de teste SIMULANDO", "tpdoc":"55", "nonf":456789, "cfop":"5353", "vrmerc":12499.99 }'
```

---

## 2.2) Inserir XML da Nota Fiscal
**Rota**: `POST /inserir-xml`

**Descrição**  
Anexa o XML da NF em `TABMOVTRA_NF_XML`. `ITEMXMLNFE` é gerado com `GEN_ID(GEN_ITEMXMLNFE, 1)` e `DATAREG` recebe a data atual `YYYY-MM-DD`.

**Campos obrigatórios do body**
- `nomovtra`
- `item` *(ITEM da NF gerado na rota acima)*
- `xml` *(conteúdo completo do XML)*

**Exemplo de Entrada**
```json
{
  "nomovtra": 999954,
  "item": 12,
  "xml": "<nfeProc versao=\"4.00\" xmlns=\"http://www.portalfiscal.inf.br/nfe\"><NFe>...</NFe></nfeProc>"
}
```

**Resposta de Sucesso**
```json
{
  "message": "XML da NF registrado com sucesso.",
  "itemxmlnfe": 5678
}
```

**cURL**
```bash
curl -X POST "https://api.tisoluciona.com/api/inserir-xml"  -H "Authorization: Bearer 23hhahk34k54fjdhj3na234544jasjm2"  -H "Content-Type: application/json"  -d '{ "nomovtra":999954, "item":12, "xml":"<nfeProc versao=\"4.00\" xmlns=\"http://www.portalfiscal.inf.br/nfe\"><NFe>...</NFe></nfeProc>" }'
```

---

## 2.3) Consultar Pedido
**Rota**: `GET /pedido`

**Query**
- `nomovtra` *(obrigatório)*

**Descrição**  
Retorna os dados do pedido, NFs, documentos fiscais e contrato de frete. Datas são formatadas para `dd/MM/yyyy` e horas para `dd/MM/yyyy HH:mm` quando aplicável.

**Exemplo**
```http
GET /pedido?nomovtra=999960
```

**Resposta de Exemplo**
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

**cURL**
```bash
curl -G "https://api.tisoluciona.com/api/pedido"  -H "Authorization: Bearer 23hhahk34k54fjdhj3na234544jasjm2"  --data-urlencode "nomovtra=999960"
```

---

## 2.4) Documentos Fiscais do Pedido: listar
**Rota base do módulo**: `/dir-cte`  
> O router desta funcionalidade é montado como `app.use("/dir-cte", require("./routes/documentos"));`

### 2.4.1) Listar documentos por pedido
**Rota**: `GET /dir-cte/:nomovtra`

**Descrição**  
Lista caminhos dos documentos fiscais de um `nomovtra`. Usa a procedure `PORTAL_PEDIDO_DOCFISCAL(?)` quando disponível. Se não houver linhas, usa fallback direto nas tabelas `TABCTRC` e `TABCONF` para montar nomes esperados de arquivos `-cte.xml` e `-cte.pdf`.  
Há normalização segura de caminhos para Windows e Linux, incluindo suporte a prefixo `\\?\` para caminhos longos em Windows.

**Resposta de Exemplo (verify=1)**
```json
{
  "status": "ok",
  "documentos": [
    {
      "chave": "35240812345678900015550010000123451000012345",
      "tipoarquivo": "xml",
      "nomearquivo": "35240812345678900015550010000123451000012345-cte.xml",
      "dircompleto": "C:\\CTE\\2025\\08\\35240812345678900015550010000123451000012345-cte.xml",
      "exists": true
    },
    {
      "chave": "35240812345678900015550010000123451000012345",
      "tipoarquivo": "pdf",
      "nomearquivo": "35240812345678900015550010000123451000012345-cte.pdf",
      "dircompleto": "C:\\CTE\\2025\\08\\35240812345678900015550010000123451000012345-cte.pdf",
    }
  ]
}
```

**cURL**
```bash
# Listar todos
curl "https://api.tisoluciona.com/api/dir-cte/999960"  -H "Authorization: Bearer 23hhahk34k54fjdhj3na234544jasjm2"

# Listar apenas os existentes, filtrando por pdf
curl "https://api.tisoluciona.com/api/documentos/999960?onlyExists=1&tipo=pdf"  -H "Authorization: Bearer 23hhahk34k54fjdhj3na234544jasjm2"
```



**Códigos de status**
- `200` arquivo enviado com `Content-Type` coerente  
- `400` parâmetro obrigatório ausente  
- `404` arquivo não localizado  
- `500` erro interno na consulta ou leitura do arquivo

**Notas de implementação**
- Procedure utilizada: `PORTAL_PEDIDO_DOCFISCAL(nomovtra)` retornando `CHAVE`, `TIPOARQUIVO`, `NOMEARQUIVO`, `DIRCOMPLETO`.  
- Fallback consulta `TABCTRC` e diretórios de `TABCONF` (`DIR_XMLCTE`, `DIR_XMLNFSE` quando aplicável).  
- Normalização de paths com suporte a Windows e Linux, substituição de separadores e prefixo para caminho longo em Windows.

---

## 🧩 Comportamentos Importantes
- `ITEM` em `TABMOVTRA_NF` é gerado pelo banco com `GEN_ID(ITEMMOVTRA, 1)`.  
- `ITEMXMLNFE` em `TABMOVTRA_NF_XML` é gerado pelo banco com `GEN_ID(GEN_ITEMXMLNFE, 1)`.  
- `TIPONF` é fixo `"E"` nas NFs.  
- Codificação `win1252` é aplicada apenas nos campos textuais enviados ao banco.  
- `DATA_HORA` do pedido é armazenado como string `HH:MM` no seu ambiente.

---

## 🔐 Segurança e Logs
- Todas as rotas exigem JWT no header `Authorization`.  
- Log de acesso com IP, método, rota e status recomendado via `morgan` em nível `combined`.  
- Em produção, não retornar paths internos em mensagens de erro.  
- Sanitizar `nome` e `tipo` da rota de download, evitando path traversal. A implementação já não utiliza inputs do usuário para concatenar diretórios arbitrários.  
- Configure permissões do serviço da API para leitura somente nas pastas de XML e PDFs.  

---




---

## 📝 Changelog
- **2025-08-12** Adicionada a rota **/dir-cte** para listar XML e PDF do CTe por `nomovtra` com validação opcional de existência no filesystem.

---

## ⚠️ Observação de homologação
Estão sendo feitas inserções de **homologação** neste ambiente.
