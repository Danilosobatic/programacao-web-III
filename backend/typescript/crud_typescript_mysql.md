# CRUD Completo com TypeScript Utilizando MySQL

Este guia explica como criar um CRUD completo utilizando TypeScript com MySQL como banco de dados.

---

## Configurar o Ambiente

1. **Criar o Projeto:**
    ```bash
    mkdir meu-crud-mysql
    cd meu-crud-mysql
    npm init -y
    ```

2. **Instalar as Dependências:**
    ```bash
    npm install typescript mysql2
    npm install --save-dev @types/node
    ```

3. **Inicializar o TypeScript:**
    ```bash
    npx tsc --init
    ```

4. **Configurar o `tsconfig.json` (opcional):**
    Certifique-se de que o arquivo `tsconfig.json` contém:
    ```json
    {
        "compilerOptions": {
            "target": "ES6",
            "module": "CommonJS",
            "outDir": "dist",
            "rootDir": "src",
            "strict": true
        }
    }
    ```

---

## Estrutura do Projeto

Crie a seguinte estrutura de arquivos:

```
meu-crud-mysql/
├── src/
│   ├── database.ts
│   ├── crud.ts
│   └── index.ts
├── package.json
├── tsconfig.json
└── .gitignore
```

Adicione `node_modules` e `dist` no `.gitignore`.

---

## Configurar o Banco de Dados

### 1. Criar Conexão com o MySQL

No arquivo `src/database.ts`, configure a conexão com o banco de dados:

```typescript
import mysql from 'mysql2/promise';

const pool = mysql.createPool({
    host: 'localhost',
    user: 'root',
    password: 'sua_senha',
    database: 'meu_crud',
    waitForConnections: true,
    connectionLimit: 10,
    queueLimit: 0
});

export default pool;
```

---

## Implementar o CRUD

No arquivo `src/crud.ts`, implemente as operações do CRUD:

```typescript
import pool from './database';

// CREATE
export async function createItem(name: string): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [result] = await connection.query(`INSERT INTO items (name) VALUES (?)`, [name]);
        console.log('Item criado com sucesso:', result);
    } finally {
        connection.release();
    }
}

// READ
export async function readItems(): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [rows] = await connection.query(`SELECT * FROM items`);
        console.log('Itens:', rows);
    } finally {
        connection.release();
    }
}

// UPDATE
export async function updateItem(id: number, newName: string): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [result] = await connection.query(`UPDATE items SET name = ? WHERE id = ?`, [newName, id]);
        console.log('Item atualizado com sucesso:', result);
    } finally {
        connection.release();
    }
}

// DELETE
export async function deleteItem(id: number): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [result] = await connection.query(`DELETE FROM items WHERE id = ?`, [id]);
        console.log('Item deletado com sucesso:', result);
    } finally {
        connection.release();
    }
}
```

---

## Criar o Arquivo Principal

No arquivo `src/index.ts`, utilize as funções do CRUD para realizar testes:

```typescript
import { createItem, readItems, updateItem, deleteItem } from './crud';

async function main() {
    try {
        await createItem('Item 1');
        await createItem('Item 2');
        await readItems();

        await updateItem(1, 'Item Atualizado');
        await readItems();

        await deleteItem(2);
        await readItems();
    } catch (error) {
        console.error('Erro:', error);
    }
}

main();
```

---

## Configurar o Banco de Dados MySQL

1. **Criar o Banco de Dados:**
    Acesse o MySQL e crie o banco e a tabela:
    ```sql
    CREATE DATABASE meu_crud;

    USE meu_crud;

    CREATE TABLE items (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL
    );
    ```

2. **Ajustar as Configurações de Conexão:**
    Verifique as credenciais no arquivo `src/database.ts`.

---

## Compilar e Executar o Código

1. **Compile o Código:**
    ```bash
    npx tsc
    ```

2. **Execute o Arquivo Gerado:**
    ```bash
    node dist/index.js
    ```

---

## Código Completo

### `src/database.ts`
```typescript
import mysql from 'mysql2/promise';

const pool = mysql.createPool({
    host: 'localhost',
    user: 'root',
    password: 'sua_senha',
    database: 'meu_crud',
    waitForConnections: true,
    connectionLimit: 10,
    queueLimit: 0
});

export default pool;
```

### `src/crud.ts`
```typescript
import pool from './database';

export async function createItem(name: string): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [result] = await connection.query(`INSERT INTO items (name) VALUES (?)`, [name]);
        console.log('Item criado com sucesso:', result);
    } finally {
        connection.release();
    }
}

export async function readItems(): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [rows] = await connection.query(`SELECT * FROM items`);
        console.log('Itens:', rows);
    } finally {
        connection.release();
    }
}

export async function updateItem(id: number, newName: string): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [result] = await connection.query(`UPDATE items SET name = ? WHERE id = ?`, [newName, id]);
        console.log('Item atualizado com sucesso:', result);
    } finally {
        connection.release();
    }
}

export async function deleteItem(id: number): Promise<void> {
    const connection = await pool.getConnection();
    try {
        const [result] = await connection.query(`DELETE FROM items WHERE id = ?`, [id]);
        console.log('Item deletado com sucesso:', result);
    } finally {
        connection.release();
    }
}
```

### `src/index.ts`
```typescript
import { createItem, readItems, updateItem, deleteItem } from './crud';

async function main() {
    try {
        await createItem('Item 1');
        await createItem('Item 2');
        await readItems();

        await updateItem(1, 'Item Atualizado');
        await readItems();

        await deleteItem(2);
        await readItems();
    } catch (error) {
        console.error('Erro:', error);
    }
}

main();
```
