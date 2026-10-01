# 🚀 Node.js + TypeScript Boilerplate (API REST)

Template base para inicialização rápida de APIs REST com **Node.js**, **TypeScript**, **Express**, **Sequelize (PostgreSQL)** e testes automatizados com **Jest**.

Use este repositório como fundação para novos projetos. Ele já conta com a arquitetura padrão em camadas (Routes → Controllers → Models / Database), suporte a TypeScript, variáveis de ambiente e recarregamento automático no desenvolvimento.

---

## 📌 Índice

- [Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [Estrutura de Pastas](#-estrutura-de-pastas)
- [Pré-requisitos](#-pré-requisitos)
- [Configuração Passo a Passo](#-configuração-passo-a-passo)
- [Scripts Disponíveis](#-scripts-disponíveis)
- [Guia: Como Adicionar uma Nova Funcionalidade](#-guia-como-adicionar-uma-nova-funcionalidade)
- [Endpoints Disponíveis (Exemplo Base)](#-endpoints-disponíveis-exemplo-base)
- [Checklist ao Iniciar um Novo Projeto](#-checklist-ao-iniciar-um-novo-projeto)
- [Dicas e Boas Práticas](#-dicas-e-boas-práticas)

---

## 🛠 Tecnologias Utilizadas

| Tecnologia | Descrição |
| :--- | :--- |
| **Node.js** | Ambiente de execução JavaScript server-side. |
| **TypeScript** | Superset de JavaScript que adiciona tipagem estática e segurança ao código. |
| **Express** | Framework web rápido e minimalista para criação de rotas e middlewares. |
| **Sequelize** | ORM (Object-Relational Mapping) baseado em Promises para comunicação com o banco. |
| **PostgreSQL (`pg`, `pg-hstore`)** | Sistema gerenciador de banco de dados relacional. |
| **Dotenv** | Carregamento de variáveis de ambiente a partir do arquivo `.env`. |
| **Jest & ts-jest** | Framework de testes unitários e de integração configurado para TypeScript. |
| **Nodemon** | Monitora alterações nos arquivos `.ts` e reinicia o servidor automaticamente. |
| **CORS** | Middleware para habilitar requisições cross-origin. |

---

## 📂 Estrutura de Pastas

```text
node-test/
├── .env                  # Variáveis de ambiente locais (credenciais de banco, porta)
├── .env.example          # Modelo de variáveis de ambiente para novos projetos
├── .gitignore            # Arquivos ignorados pelo Git (node_modules, logs, etc.)
├── jest.config.js        # Configurações de teste com Jest e ts-jest
├── package.json          # Metadados do projeto, scripts e dependências
├── tsconfig.json         # Configuração do compilador TypeScript
└── src/
    ├── server.ts         # Ponto de entrada da aplicação (configura Express, rotas e servidor)
    ├── routes/           # Definição dos endpoints e mapeamento de rotas
    │   └── api.ts        # Rotas da API (registro, login, listagem, etc.)
    ├── controllers/      # Regras de negócio e tratamento de req/res
    │   └── apiController.ts
    ├── models/           # Schemas e definições de tabelas do banco de dados (Sequelize)
    │   ├── User.ts       # Modelo de Usuário
    │   └── User.test.ts  # Testes unitários do modelo
    └── instances/        # Instâncias de serviços externos e conexões
        └── pg.ts         # Conexão com o banco PostgreSQL via Sequelize
```

---

## ⚙️ Pré-requisitos

Antes de iniciar, certifique-se de possuir instalado:
1. **Node.js** (versão 16+ ou LTS recomendada)
2. **PostgreSQL** instalado e em execução (ou container Docker)
3. Gerenciador de pacotes **npm** ou **yarn**

---

## 🚀 Configuração Passo a Passo

### 1. Clonar ou copiar o template
Copie esta pasta para o diretório do seu novo projeto e abra o terminal nele.

### 2. Instalar as dependências
```bash
npm install
```

### 3. Configurar as Variáveis de Ambiente (`.env`)
Verifique o arquivo `.env` na raiz do projeto e ajuste com as credenciais do seu banco de dados:

```env
PORT=4000

PG_DB=auths
PG_USER=postgres
PG_PASSWORD=1234
PG_PORT=5432
```

> **Atenção:** Certifique-se de que o banco de dados definido em `PG_DB` (ex: `auths`) já existe no seu PostgreSQL. Caso não exista, crie-o:
> ```sql
> CREATE DATABASE auths;
> ```

---

## 💻 Scripts Disponíveis

| Comando | Descrição |
| :--- | :--- |
| `npm start` | Inicia o servidor em modo de desenvolvimento com **nodemon** e recarregamento automático a cada alteração em arquivos `.ts` ou `.json`. |
| `npm test` | Executa os testes automatizados com **Jest** em série (`--runInBand`) e com `NODE_ENV=test`. |

---

## 🏗 Guia: Como Adicionar uma Nova Funcionalidade

Siga o padrão em 3 camadas sempre que for criar um novo recurso (ex: produtos, pedidos, clientes):

### 1º Passo: Criar o Model (`src/models/Product.ts`)
Defina a interface TypeScript e a tabela no Sequelize:
```typescript
import { Model, DataTypes } from 'sequelize';
import { sequelize } from '../instances/pg';

export interface ProductInstance extends Model {
    id: number;
    title: string;
    price: number;
}

export const Product = sequelize.define<ProductInstance>('Product', {
    id: {
        primaryKey: true,
        autoIncrement: true,
        type: DataTypes.INTEGER
    },
    title: {
        type: DataTypes.STRING
    },
    price: {
        type: DataTypes.FLOAT
    }
}, {
    tableName: 'products',
    timestamps: false
});
```

### 2º Passo: Criar o Controller (`src/controllers/productController.ts`)
Implemente a lógica de requisição e resposta:
```typescript
import { Request, Response } from 'express';
import { Product } from '../models/Product';

export const getAll = async (req: Request, res: Response) => {
    const products = await Product.findAll();
    res.json({ products });
};

export const create = async (req: Request, res: Response) => {
    const { title, price } = req.body;
    if (title && price) {
        const newProduct = await Product.create({ title, price });
        res.status(201).json({ product: newProduct });
        return;
    }
    res.status(400).json({ error: 'Dados incompletos.' });
};
```

### 3º Passo: Conectar na Rota (`src/routes/api.ts`)
Adicione o endpoint chamando o método do controller:
```typescript
import * as ProductController from '../controllers/productController';

router.get('/products', ProductController.getAll);
router.post('/products', ProductController.create);
```

---

## 📡 Endpoints Disponíveis (Exemplo Base)

| Método | Rota | Descrição | Parâmetros (Body / URL) | Retorno Sucesso |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/ping` | Teste de conectividade da API | Nenhum | `{ "pong": true }` |
| `POST` | `/register` | Cadastra um novo usuário | `email`, `password` | `201: { "id": 1 }` |
| `POST` | `/login` | Valida credenciais do usuário | `email`, `password` | `{ "status": true }` |
| `GET` | `/list` | Lista os emails cadastrados | Nenhum | `{ "list": ["email@exemplo.com"] }` |

---

## ✅ Checklist ao Iniciar um Novo Projeto

Quando copiar este projeto para uma nova aplicação, lembre-se de:

1. [ ] Alterar o campo `"name"` no `package.json` para o nome do seu novo projeto.
2. [ ] Configurar um novo repositório Git (`git init` ou alterar a `remote origin`).
3. [ ] Ajustar o nome do banco no `.env` (`PG_DB=meu_novo_banco`).
4. [ ] Se a sua API receber dados no formato JSON via requisições POST/PUT, garanta que o middleware `server.use(express.json());` esteja ativo no `src/server.ts`.
5. [ ] Remover ou adaptar as rotas de exemplo (`apiController.ts`, `User.ts`) de acordo com as necessidades do seu sistema.

---

## 💡 Dicas e Boas Práticas

- **Tipos e Interfaces:** Sempre declare a interface do modelo que herda de `Model` do Sequelize (como feito em `UserInstance`) para ter auto-complete no TypeScript.
- **Variáveis de Ambiente:** Nunca envie senhas de produção ou chaves de API secretas para o repositório Git público.
- **Tratamento de Erros:** O arquivo `server.ts` já possui um middleware de fallback para rotas inexistentes (404) e um `errorHandler` genérico para capturar exceções não tratadas.
- **Sincronização do Banco:** Se quiser que o Sequelize crie as tabelas automaticamente em ambiente de teste ou desenvolvimento inicial, você pode usar `sequelize.sync()`.
