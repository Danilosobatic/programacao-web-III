# CRUD com API RESTful usando Node.js e Express.js

Este é um exemplo de como criar uma API RESTful com operações CRUD (Create, Read, Update, Delete) usando **Node.js** e o framework **Express.js**. Vamos criar um servidor básico e implementar as rotas para gerenciar recursos.

### Estrutura do Projeto



/crud-api ├── /node_modules ├── /controllers │ └── userController.js ├── /routes │ └── userRoutes.js ├── /models │ └── userModel.js ├── app.js ├── package.json

## Passos para criar o projeto:

1. **Iniciar o projeto Node.js**:

```bash
npm init -y


```

2. **Instalar as dependências:**




```bash
 
npm install express

```

# Crud Esemplo

1. app.js - Arquivo principal
Este é o arquivo principal do servidor, onde as rotas são configuradas e o servidor é iniciado.



```javascript
 
// Importando o pacote express
const express = require('express');
const app = express();

// Middleware para parser de JSON
app.use(express.json());

// Importando as rotas
const userRoutes = require('./routes/userRoutes');

// Usando as rotas no servidor
app.use('/api/users', userRoutes);

// Definindo a porta em que o servidor vai rodar
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});

```

2. controllers/userController.js - Controlador


Este arquivo contém as funções para lidar com as requisições de cada operação CRUD.



```javascript
 
// Importando o modelo de usuário
const User = require('../models/userModel');

// Função para criar um novo usuário
exports.createUser = (req, res) => {
  const { name, email } = req.body;

  // Criando um novo usuário
  const newUser = { id: Date.now(), name, email };

  // Simulando a criação no banco de dados (em memória)
  users.push(newUser);

  // Respondendo com sucesso
  res.status(201).json(newUser);
};

// Função para obter todos os usuários
exports.getAllUsers = (req, res) => {
  res.status(200).json(users);
};

// Função para obter um usuário por ID
exports.getUserById = (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));

  if (!user) {
    return res.status(404).json({ message: 'User not found' });
  }

  res.status(200).json(user);
};

// Função para atualizar um usuário
exports.updateUser = (req, res) => {
  const { id } = req.params;
  const { name, email } = req.body;

  let user = users.find(u => u.id === parseInt(id));

  if (!user) {
    return res.status(404).json({ message: 'User not found' });
  }

  user.name = name || user.name;
  user.email = email || user.email;

  res.status(200).json(user);
};

// Função para excluir um usuário
exports.deleteUser = (req, res) => {
  const { id } = req.params;

  const userIndex = users.findIndex(u => u.id === parseInt(id));

  if (userIndex === -1) {
    return res.status(404).json({ message: 'User not found' });
  }

  users.splice(userIndex, 1);

  res.status(200).json({ message: 'User deleted successfully' });
};

```


3. routes/userRoutes.js - Rotas
Este arquivo define as rotas que irão mapear para as funções do controlador.



```javascript
 
// Importando o express
const express = require('express');
const router = express.Router();

// Importando o controlador
const userController = require('../controllers/userController');

// Rota para criar um novo usuário (POST)
router.post('/', userController.createUser);

// Rota para obter todos os usuários (GET)
router.get('/', userController.getAllUsers);

// Rota para obter um usuário específico (GET by ID)
router.get('/:id', userController.getUserById);

// Rota para atualizar um usuário (PUT)
router.put('/:id', userController.updateUser);

// Rota para deletar um usuário (DELETE)
router.delete('/:id', userController.deleteUser);

 
```
4. models/userModel.js - Modelo (em memória)
Este é o modelo do usuário. Em vez de usar um banco de dados, vamos armazenar os dados em memória para simplificação.


```javascript

// Simulando um banco de dados com um array de usuários
let users = [
  { id: 1, name: 'John Doe', email: 'john.doe@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane.smith@example.com' }
];


```

# Como executar

1. Criar e rodar o servidor:
No terminal, execute o seguinte comando:


```bash
 
node app.js
```


2. Testar a API:
A API estará disponível em http://localhost:3000/api/users.

Você pode testar as operações CRUD usando ferramentas como Postman ou Insomnia, ou também usando o curl.

* POST /api/users - Criar um novo usuário

Exemplo de corpo da requisição:
 
```json
 
{
  "name": "Alice Brown",
  "email": "alice.brown@example.com"
}
```


* GET /api/users - Obter todos os usuários

*GET /api/users/:id - Obter um usuário específico

Exemplo de URL: http://localhost:3000/api/users/1

* PUT /api/users/:id - Atualizar um usuário

Exemplo de corpo da requisição]
 
```json
Copiar código
{
  "name": "Alice B.",
  "email": "alice.b@example.com"
}
```

* DELETE /api/users/:id - Deletar um usuário

## Conclusão
Este é um exemplo básico de como criar uma API RESTful usando Node.js e Express.js com operações CRUD. O código usa um armazenamento simples em memória, mas você pode facilmente integrar um banco de dados (como MongoDB, MySQL, PostgreSQL, etc.) para persistir os dados de forma permanente.

 