# CRUD Node.js com MySQL

Este projeto implementa um CRUD (Create, Read, Update, Delete) simples utilizando Node.js e MySQL. O backend foi desenvolvido com Node.js e Express, e a comunicação com o banco de dados é feita com o pacote `mysql2`.

## Tecnologias Utilizadas

- Node.js
- Express
- MySQL
- mysql2
- body-parser

## Passo 1: Instalar Dependências

1. Crie um novo diretório para o projeto e inicialize o projeto Node.js:

    ```bash
    mkdir crud-nodejs-mysql
    cd crud-nodejs-mysql
    npm init -y
    ```

2. Instale as dependências necessárias:

    ```bash
    npm install express mysql2 body-parser
    ```

## Passo 2: Criar o Banco de Dados e a Tabela no MySQL

1. Acesse o MySQL e crie o banco de dados e a tabela que irá armazenar os dados:

    ```sql
    CREATE DATABASE cruddb;

    USE cruddb;

    CREATE TABLE users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(100),
      email VARCHAR(100),
      idade INT
    );
    ```

2. Se você já tiver a tabela com a coluna `age`, altere o nome da coluna para `idade`:

    ```sql
    ALTER TABLE users CHANGE age idade INT;
    ```

## Passo 3: Código do Servidor Node.js com Express

Crie o arquivo `app.js` e insira o código abaixo para configurar o servidor Express e realizar operações CRUD com o MySQL.

```js
const express = require('express');
const mysql = require('mysql2');
const bodyParser = require('body-parser');

const app = express();
const port = 3000;

// Configuração do parser para JSON
app.use(bodyParser.json());

// Conexão com o MySQL
const db = mysql.createConnection({
  host: 'localhost',
  user: 'root',
  password: '',  // Adicione sua senha aqui
  database: 'cruddb'
});

// Conectar ao banco de dados MySQL
db.connect(err => {
  if (err) {
    console.error('Erro ao conectar ao banco de dados:', err);
    return;
  }
  console.log('Conectado ao banco de dados MySQL');
});

// Rota para criar um novo usuário (Create)
app.post('/users', (req, res) => {
  const { name, email, idade } = req.body;
  const query = 'INSERT INTO users (name, email, idade) VALUES (?, ?, ?)';
  db.query(query, [name, email, idade], (err, result) => {
    if (err) {
      res.status(500).json({ error: err.message });
      return;
    }
    res.status(201).json({ id: result.insertId, name, email, idade });
  });
});

// Rota para obter todos os usuários (Read)
app.get('/users', (req, res) => {
  const query = 'SELECT * FROM users';
  db.query(query, (err, results) => {
    if (err) {
      res.status(500).json({ error: err.message });
      return;
    }
    res.json(results);
  });
});

// Rota para atualizar um usuário (Update)
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const { name, email, idade } = req.body;
  const query = 'UPDATE users SET name = ?, email = ?, idade = ? WHERE id = ?';
  db.query(query, [name, email, idade, id], (err, result) => {
    if (err) {
      res.status(500).json({ error: err.message });
      return;
    }
    if (result.affectedRows === 0) {
      return res.status(404).json({ message: 'Usuário não encontrado' });
    }
    res.json({ id, name, email, idade });
  });
});

// Rota para excluir um usuário (Delete)
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  const query = 'DELETE FROM users WHERE id = ?';
  db.query(query, [id], (err, result) => {
    if (err) {
      res.status(500).json({ error: err.message });
      return;
    }
    if (result.affectedRows === 0) {
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


## Passo 4: Testar a Aplicação

Agora você pode testar as rotas CRUD utilizando ferramentas como Postman ou cURL.

Exemplos de Requisições:
1. **Criar Usuário (POST)**

json
```POST /users
{
  "name": "João Silva",
  "email": "joao@example.com",
  "idade": 30
}

```

2. **Listar Usuários (GET)**

```
json
GET /users
```

3. **Atualizar Usuário (PUT)**
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

4. **Excluir Usuário (DELETE)**
```
json
Copiar código
DELETE /users/1
```

##  Passo 5: Conclusão
Este é um exemplo simples de como criar um CRUD em Node.js utilizando MySQL para manipulação de dados. Ele permite realizar as operações de criação, leitura, atualização e exclusão de usuários com o banco de dados.