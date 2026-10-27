# Pix Payment (Mercado Pago)

Uma biblioteca simples e tipada para criação e consulta de pagamentos via **PIX** utilizando a API do Mercado Pago. Suporta modo emulador para testes locais.

Biblioteca TypeScript distribuída como módulo ES. Para executar, use um ambiente JavaScript com `fetch` global (por exemplo, Node.js 18 ou superior). Para compilar o código-fonte, são necessários Node.js e npm.

---

## 📦 Instalação

```bash
npm install git+https://github.com/eliezer-dev-software-enginner/pix-payment.git
```

---

## ⚙️ Configuração

A biblioteca recebe um objeto `PixConfig` com o access token e, opcionalmente, a configuração do emulador.

Obtenha o token na sua conta do Mercado Pago e mantenha-o fora do código versionado. Mesmo no modo emulador, o construtor exige um `accessToken` não vazio.

```ts
import { PixService } from 'pix_generator';

const pixService = new PixService({
  accessToken: 'SEU_ACCESS_TOKEN',
});
```

---

## 🧪 Modo emulador

Para testes locais, ative o emulador. Quando habilitado, as requisições são redirecionadas para o [mercadopago-pix-emulator](https://github.com/eliezer-dev-software-enginner/mercadopago-pix-emulator) em vez da API real do Mercado Pago.

```ts
const pixService = new PixService({
  accessToken: 'SEU_ACCESS_TOKEN',
  emulator: {
    enabled: true,
    url: 'http://localhost:3001', // opcional, este é o default
  },
});
```

Uso recomendado com variável de ambiente:

```ts
const pixService = new PixService({
  accessToken: process.env.MP_ACCESS_TOKEN!,
  emulator: {
    enabled: process.env.NODE_ENV === 'development',
    ...(process.env.PIX_EMULATOR_URL && { url: process.env.PIX_EMULATOR_URL }),
  },
});
```

Defina `MP_ACCESS_TOKEN` e, se necessário, `PIX_EMULATOR_URL` no ambiente da aplicação. A biblioteca lê apenas os valores passados ao construtor; ela não carrega arquivos `.env` automaticamente.

---

## 🚀 Criando um pagamento PIX

```ts
const result = await pixService.createPixPayment({
  email: 'cliente@email.com',
  firstName: 'João',
  lastName: 'Silva',
  description: 'Pagamento do pedido #123',
  externalRef: 'pedido-123',
  value: 100,
  notificationUrl: 'https://seu-app.com/api/checkout/webhook', // opcional
  metadata: { userId: 'abc', romId: 'xyz' }, // opcional
});
```

---

## 📄 Resposta

### Sucesso

```ts
{
  success: true,
  data: {
    paymentId: "123456",
    qrCodeBase64: "base64...",
    qrCode: "copia e cola...",
    status: "pending"
  },
  error: null
}
```

### Erro

```ts
{
  success: false,
  data: null,
  error: "mensagem de erro"
}
```

---

## 🔎 Consultar pagamento

```ts
const payment = await pixService.getPaymentById('PAYMENT_ID');
```

O retorno é a resposta completa do pagamento (`PaymentResponse` do SDK), e erros de consulta são lançados como exceções. No emulador, a consulta usa `GET /v1/payments/{id}`.

---

## 🧱 Tipagens

### PixConfig

```ts
type PixConfig = {
  accessToken: string;
  emulator?: {
    enabled: boolean;
    url?: string; // default: http://localhost:3001
  };
};
```

### PaymentData

```ts
type PaymentData = {
  email: string;
  description: string;
  firstName: string;
  lastName: string;
  externalRef: string;
  value: number;
  notificationUrl?: string;
  metadata?: Record<string, any>;
};
```

### PixPaymentResult

```ts
type PixPaymentResult = {
  success: boolean;
  data: {
    paymentId: string;
    qrCodeBase64: string | null;
    qrCode: string | null;
    status: string;
  } | null;
  error: string | null;
};
```

---

## ⚠️ Regras e validações

- `accessToken` não pode ser vazio
- `value` deve ser maior que zero

Essas validações lançam exceções. Falhas ao criar o pagamento retornam `success: false` com uma mensagem em `error`.

---

## 🧠 Como funciona

A biblioteca encapsula o SDK do Mercado Pago e simplifica:

- Criação de pagamento PIX
- Extração de QR Code
- Consulta de status por ID

Em modo emulador, as requisições são feitas diretamente para o servidor configurado; a criação usa `POST /v1/payments`. O emulador é um projeto separado, indicado no link acima.

---

## 📌 Dependências

- `mercadopago`

---

## 📄 Licença e autoria

O `package.json` informa licença ISC, mas este repositório não inclui um arquivo de licença. Os termos de distribuição precisam ser confirmados pelo mantenedor.

Autor informado no `package.json`: Eliezer Assunção de Paulo | Eliezer Dev.
