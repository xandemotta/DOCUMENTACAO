# 📦 API de Pedidos — Documentação para Consumo (Cliente)

Este documento orienta **como consumir a API**. Ele não aborda detalhes internos de banco ou implementação.

> **Alterações recentes (2025-08-13):**
> - Removidos os itens **2.5 ** e **2.6 **.
> - Atualizada a lógica do endpoint **2) Criar pedido** para resolução por **CNPJ** com fallback e resposta padronizada.

---

## 1) Informações Gerais

- **Base URL (produção):** `https://api.tisoluciona.com/api/`
- **Autenticação:** `Bearer <JWT_TOKEN>` no header `Authorization`
- **Content-Type:** `application/json; charset=utf-8`
- **Timezone de referência:** America/Sao_Paulo (datas podem ser enviadas no padrão do exemplo).

### 1.1) Exemplo de autorização
```
Authorization: Bearer SEU_JWT_AQUI
```

---

## 2) Criar pedido

**Rota:** `POST /criar-pedido`  
**Descrição (visão de consumo):**  
Cria um pedido e registra, opcionalmente, as notas fiscais associadas. A API **resolve CNPJs** informados em **coleta**, **entrega** e **cliente de faturamento** para identificar os respectivos cadastros, com **desempate automático** quando houver múltiplos cadastros, escolhendo o que teve **movimento mais recente**.

### 2.1) Regras de resolução e validações

- **Identificação por CNPJ (preferencial):**
  - `cnpjColeta`, `cnpjEntrega`, `cnpjCliente` são usados para localizar o cadastro correspondente.
  - Se não houver nenhum cadastro para o CNPJ informado → **HTTP 404** (campo não encontrado).
  - Se houver múltiplos cadastros e **nenhum** tiver movimento para desempate → **HTTP 409** (ambiguidade).

- **Fallbacks legados (opcional):**
  - Caso não envie os CNPJs, você **pode** informar:
    - `localColeta`
    - `localEntrega`
    - `clienteFaturamento`
  - Esses três **juntos** substituem a tríade `cnpjColeta`/`cnpjEntrega`/`cnpjCliente` se estes faltarem.
  - Se nem CNPJs nem fallbacks forem válidos/completos → **HTTP 400**.

- **Motorista:**
  - `nomeMotorista` deve ser o **código NOCLI** do motorista já cadastrado.
  - A resposta retorna:
    - `nomeMotorista`: **NOCLI** do motorista (numérico, sem formatação de nome)
    - `cpfMotorista`: documento do cadastro (somente dígitos, pode ser CPF ou CNPJ).

- **Notas fiscais:**
  - Envie em `notasFiscais` um array com itens `{ nonf, dataemi, pesobr, vlrtotal, chavenfe, cfop? }`.
  - `cfop` é opcional; se omitido, será salvo vazio.

### 2.2) Campos obrigatórios

Envie **uma** das combinações abaixo:

**A) Preferencial (por CNPJ):**
- `placaCavalo`
- `cnpjColeta`
- `cnpjEntrega`
- `cnpjCliente`
- `processoCliente`
- `tipoContainer`
- `numeroContainer`
- `nomeMotorista` (NOCLI do motorista)
- `placaCarreta1`
- `empresa`
- `tipoFrete`
- `tipoCarga`
- `pesoBrutoTotal`
- `valorTotalNotas`
- `notasFiscais` (array)

**OU B) Fallback legado (sem CNPJ), desde que envie os três:**
- `localColeta`
- `localEntrega`
- `clienteFaturamento`
- e os demais campos do grupo A.

### 2.3) Exemplo de requisição (preferencial por CNPJ)

```json
{
  "placaCavalo": "MBQ9466",
  "cnpjColeta": "36.975.508/0001-52",
  "cnpjEntrega": "02.023.797/0001-78",
  "cnpjCliente": "81.002.925/0001-73",
  "processoCliente": "PROC-XYZ",
  "tipoContainer": 1,
  "numeroContainer": "ABCD1234567",
  "nomeMotorista": "5102",
  "cpfMotorista": "5102",
  "placaCarreta1": "MEE3078",
  "placaCarreta2": null,
  "pesoBrutoTotal": 18500.75,
  "valorTotalNotas": 32000.5,
  "empresa": 1,
  "tipoFrete": 1,
  "tipoCarga": 2,
  "usuario": "INTEGRADOR1",
  "notasFiscais": [{
    "nonf": "12345",
      "pesobr": 5000.00,
      "vlrtotal": 10000.00,
      "chavenfe": "43250505638569000524570030000020921060219563",
      "cfop": "5353"
  }
  ]
}
```

### 2.4) Exemplo de resposta (201 Created)

```json
{
  "status": "OK",
  "mensagem": "Pedido inserido com sucesso.",
  "nomovtra": 999970,
  "nomeMotorista": 5102,
  "cpfMotorista": "35678158000108",
  "coleta": {
    "nocli": 4906,
    "nomcli": "HAKA ARMAZENS GERAIS",
    "documento": "02.023.797/0001-78",
    "documentoSemMascara": "02023797000178",
    "tipoDocumento": "CNPJ",
    "cnpj": "02.023.797/0001-78"
  },
  "entrega": {
    "nocli": 4919,
    "nomcli": "BLOCOS GUARANI FABRICA DE ARTEFATOS",
    "documento": "81.002.925/0001-73",
    "documentoSemMascara": "81002925000173",
    "tipoDocumento": "CNPJ",
    "cnpj": "81.002.925/0001-73"
  },
  "cliente": {
    "nocli": 4912,
    "nomcli": "JAISON MELCHIORETTO",
    "documento": "073.459.879-30",
    "documentoSemMascara": "07345987930",
    "tipoDocumento": "CPF",
    "cnpj": "073.459.879-30"
  }
}
```

> Observação: os campos `cnpj` dentro de `coleta/entrega/cliente` são mantidos por **compatibilidade**, mas podem conter **CPF** (nome legado do campo). Utilize os campos padronizados `documento`, `documentoSemMascara` e `tipoDocumento` para lógica nova.

### 2.5) Erros comuns

| HTTP | Código interno (indicativo) | Causa | Correção sugerida |
|------|------------------------------|-------|-------------------|
| 400  | `Campos obrigatórios ausentes` | Nem CNPJs válidos nem os três fallbacks fornecidos | Envie `cnpjColeta/cnpjEntrega/cnpjCliente` ou `localColeta/localEntrega/clienteFaturamento` |
| 404  | `CNPJ ... não encontrado` | CNPJ não existe no cadastro | Confirme o CNPJ ou cadastre previamente |
| 409  | `Ambiguidade em ...` | Múltiplos cadastros para o CNPJ e nenhum movimento para desempate | Informe o código exato (`localColeta/localEntrega/clienteFaturamento`) |
| 409  | `Conflito de chave primária` | NOMOVTRA já existe (situação rara por reprocessamento) | Reenvie sem reutilizar o mesmo identificador |
| 422  | `Erro de conversão de dados` | Formato inválido (datas/números) | Ajuste para o formato do exemplo |
| 500  | `Erro interno ao salvar o pedido` | Falha inesperada | Reenvie; se persistir, acione o suporte com o payload |

### 2.6) cURL de exemplo

```bash
curl -X POST "https://api.tisoluciona.com/api/criar-pedido"   -H "Authorization: Bearer SEU_JWT_AQUI"   -H "Content-Type: application/json"   --data-raw '{
    "placaCavalo": "MBQ9466",
    "cnpjColeta": "02.023.797/0001-78",
    "cnpjEntrega": "81.002.925/0001-73",
    "cnpjCliente": "07.345.987/0001-00",
    "processoCliente": "PROC-XYZ",
    "tipoContainer": 1,
    "numeroContainer": "ABCD1234567",
    "nomeMotorista": 5102,
    "placaCarreta1": "MEE3078",
    "placaCarreta2": null,
    "pesoBrutoTotal": 18500.75,
    "valorTotalNotas": 32000.50,
    "empresa": 1,
    "tipoFrete": 1,
    "tipoCarga": 2,
    "usuario": "INTEGRADOR1",
    "notasFiscais": [{
      "nonf": "12345",
      "dataemi": "2025-08-06",
      "pesobr": 5000.00,
      "vlrtotal": 10000.00,
      "chavenfe": "43250505638569000524570030000020921060219563",
      "cfop": "5353"
    }]
  }'
```

---

## 3) Consultar pedido (básico)

**Rota:** `GET /pedido?nomovtra={numero}`  
**Descrição:** Retorna os dados do pedido criado, quando disponível.  
**Exemplo:** `GET /pedido?nomovtra=999970`

> Observação: campos e formato seguem a mesma convenção da resposta do **Criar pedido** quando aplicável.

---

## 4) Saúde da API

**Rota:** `GET /health` → `200 OK` quando o serviço está operacional.

---

## 5) Suporte

- Se ocorrer **409 Ambiguidade** ao criar um pedido com CNPJ, informe temporariamente os IDs legados `localColeta`, `localEntrega` e `clienteFaturamento` para desbloquear o processo enquanto o cadastro é unificado.
- Dúvidas sobre consumo: alexandremota@tisoluciona.com (assunto: **API Pedidos**).

---

## 6) Changelog

- **2025-08-13**:
  - Removidos **/noterm-by-local** e **/nomot-by-cpf**.
  - Atualizada a lógica de **/criar-pedido**: resolução por CNPJ com desempate por movimento mais recente, resposta padronizada com `documento`/`documentoSemMascara`/`tipoDocumento`, e manutenção dos campos legados por compatibilidade.
- **2025-08-12** Adicionadas as rotas **/noterm-by-local** e **/nomot-by-cpf**.  
- **2025-08-12** Adicionada a rota **/dir-cte** para listar XML e PDF do CTe por `nomovtra` com validação opcional de existência no filesystem.
