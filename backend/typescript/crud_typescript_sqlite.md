# CRUD Completo com TypeScript Utilizando SQLite

Este guia explica como criar um CRUD completo utilizando TypeScript com SQLite como banco de dados.

---

## Configurar o Ambiente

1. **Criar o Projeto:**
    ```bash
    mkdir meu-crud-sqlite
    cd meu-crud-sqlite
    npm init -y
    ```

2. **Instalar as Dependências:**
    ```bash
    npm install typescript sqlite3
    npm install --save-dev @types/node @types/sqlite3
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
meu-crud-sqlite/
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

### 1. Criar Conexão com o SQLite

No arquivo `src/database.ts`, configure a conexão com o banco de dados:

```typescript
import sqlite3 from 'sqlite3';

const db = new sqlite3.Database(':memory:'); // Banco de dados em memória

// Criar tabela de exemplo
db.serialize(() => {
    db.run(`
        CREATE TABLE items (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL
        )
    `);
});

export default db;
```

---

## Implementar o CRUD

No arquivo `src/crud.ts`, implemente as operações do CRUD:

```typescript
import db from './database';

// CREATE
export function createItem(name: string): Promise<void> {
    return new Promise((resolve, reject) => {
        db.run(`INSERT INTO items (name) VALUES (?)`, [name], function (err) {
            if (err) return reject(err);
            console.log(`Item criado com ID: ${this.lastID}`);
            resolve();
        });
    });
}

// READ
export function readItems(): Promise<void> {
    return new Promise((resolve, reject) => {
        db.all(`SELECT * FROM items`, [], (err, rows) => {
            if (err) return reject(err);
            console.log('Itens:', rows);
            resolve();
        });
    });
}

// UPDATE
export function updateItem(id: number, newName: string): Promise<void> {
    return new Promise((resolve, reject) => {
        db.run(`UPDATE items SET name = ? WHERE id = ?`, [newName, id], function (err) {
            if (err) return reject(err);
            console.log(`Item atualizado: ${this.changes} registro(s) modificado(s).`);
            resolve();
        });
    });
}

// DELETE
export function deleteItem(id: number): Promise<void> {
    return new Promise((resolve, reject) => {
        db.run(`DELETE FROM items WHERE id = ?`, [id], function (err) {
            if (err) return reject(err);
            console.log(`Item deletado: ${this.changes} registro(s) removido(s).`);
            resolve();
        });
    });
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
import sqlite3 from 'sqlite3';

const db = new sqlite3.Database(':memory:');

db.serialize(() => {
    db.run(`
        CREATE TABLE items (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL
        )
    `);
});

export default db;
```

### `src/crud.ts`
```typescript
import db from './database';

export function createItem(name: string): Promise<void> {
    return new Promise((resolve, reject) => {
        db.run(`INSERT INTO items (name) VALUES (?)`, [name], function (err) {
            if (err) return reject(err);
            console.log(`Item criado com ID: ${this.lastID}`);
            resolve();
        });
    });
}

export function readItems(): Promise<void> {
    return new Promise((resolve, reject) => {
        db.all(`SELECT * FROM items`, [], (err, rows) => {
            if (err) return reject(err);
            console.log('Itens:', rows);
            resolve();
        });
    });
}

export function updateItem(id: number, newName: string): Promise<void> {
    return new Promise((resolve, reject) => {
        db.run(`UPDATE items SET name = ? WHERE id = ?`, [newName, id], function (err) {
            if (err) return reject(err);
            console.log(`Item atualizado: ${this.changes} registro(s) modificado(s).`);
            resolve();
        });
    });
}

export function deleteItem(id: number): Promise<void> {
    return new Promise((resolve, reject) => {
        db.run(`DELETE FROM items WHERE id = ?`, [id], function (err) {
            if (err) return reject(err);
            console.log(`Item deletado: ${this.changes} registro(s) removido(s).`);
            resolve();
        });
    });
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

---