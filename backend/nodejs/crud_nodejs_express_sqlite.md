# CRUD Node.js com SQLite

Este projeto implementa um CRUD (Create, Read, Update, Delete) simples utilizando Node.js e SQLite. O backend foi desenvolvido com Node.js e Express, e a comunicação com o banco de dados SQLite é feita com o pacote `sqlite3`.

## Tecnologias Utilizadas

- Node.js
- Express
- SQLite
- sqlite3
- body-parser

## Passo 1: Instalar Dependências

1. Crie um novo diretório para o projeto e inicialize o projeto Node.js:

    ```bash
    mkdir crud-nodejs-sqlite
    cd crud-nodejs-sqlite
    npm init -y
    ```

2. Instale as dependências necessárias:

    ```bash
    npm install express sqlite3 body-parser
    ```

## Passo 2: Criar o Banco de Dados SQLite

1. No diretório do projeto, crie um arquivo chamado `database.js` para configurar o banco de dados SQLite e a tabela:

    ```js
    const sqlite3 = require('sqlite3').verbose();

    // Cria o banco de dados ou abre o existente
    const db = new sqlite3.Database('./cruddb.sqlite3', (err) => {
        if (err) {
            console.error('Erro ao abrir o banco de dados:', err.message);
        } else {
            console.log('Banco de dados SQLite conectado');
        }
    });

    // Cria a tabela users
    db.serialize(() => {
        db.run(`
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT NOT NULL,
                idade INTEGER NOT NULL
            )
        `);
    });

    module.exports = db;
    ```

## Passo 3: Código do Servidor Node.js com Express

Crie um arquivo `app.js` para configurar o servidor Express e realizar as operações CRUD com o SQLite.

```js
const express = require('express');
const bodyParser = require('body-parser');
const db = require('./database');

const app = express();
const port = 3000;

// Configuração do parser para JSON
app.use(bodyParser.json());

// Rota para criar um novo usuário (Create)
app.post('/users', (req, res) => {
    const { name, email, idade } = req.body;
    const query = 'INSERT INTO users (name, email, idade) VALUES (?, ?, ?)';
    db.run(query, [name, email, idade], function (err) {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        res.status(201).json({
            id: this.lastID,
            name,
            email,
            idade
        });
    });
});

// Rota para obter todos os usuários (Read)
app.get('/users', (req, res) => {
    const query = 'SELECT * FROM users';
    db.all(query, [], (err, rows) => {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        res.json(rows);
    });
});

// Rota para obter um usuário específico (Read)
app.get('/users/:id', (req, res) => {
    const { id } = req.params;
    const query = 'SELECT * FROM users WHERE id = ?';
    db.get(query, [id], (err, row) => {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        if (!row) {
            return res.status(404).json({ message: 'Usuário não encontrado' });
        }
        res.json(row);
    });
});

// Rota para atualizar um usuário (Update)
app.put('/users/:id', (req, res) => {
    const { id } = req.params;
    const { name, email, idade } = req.body;
    const query = 'UPDATE users SET name = ?, email = ?, idade = ? WHERE id = ?';
    db.run(query, [name, email, idade, id], function (err) {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        if (this.changes === 0) {
            return res.status(404).json({ message: 'Usuário não encontrado' });
        }
        res.json({ id, name, email, idade });
    });
});

// Rota para excluir um usuário (Delete)
app.delete('/users/:id', (req, res) => {
    const { id } = req.params;
    const query = 'DELETE FROM users WHERE id = ?';
    db.run(query, [id], function (err) {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        if (this.changes === 0) {
            return res.status(404).json({ message: 'Usuário não encontrado' });
        }
        res.status(204).send();
    });
});

// Iniciar o servidor
app.listen(port, () => {
    console.log(`Servidor rodando na porta ${port}`);
});

```
#Passo 4: Testar a Aplicação
Agora você pode testar as rotas CRUD utilizando ferramentas como Postman ou cURL.

Exemplos de Requisições:
1. **Criar Usuário (POST)**
```
json
Copiar código
POST /users
{
  "name": "João Silva",
  "email": "joao@example.com",
  "idade": 30
}
```
2. **Listar Usuários (GET)**
```
json
Copiar código
GET /users
```
3. **Obter um Usuário Específico (GET)**
```
json
Copiar código
GET /users/1
```
4. **Atualizar Usuário (PUT)**
```
json
Copiar código
PUT /users/1
{
  "name": "João Silva Jr.",
  "email": "joaojr@example.com",
  "idade": 31
}
```
5. **Excluir Usuário (DELETE)**
```
json
Copiar código
DELETE /users/1
```
#Passo 5: Conclusão
Este é um exemplo simples de como criar um CRUD em Node.js utilizando SQLite para manipulação de dados. Ele permite realizar as operações de criação, leitura, atualização e exclusão de usuários com o banco de dados.