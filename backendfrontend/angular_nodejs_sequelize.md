# CRUD Node.js, Angular e Sequelize

Este projeto implementa um CRUD utilizando **Node.js** no back-end, **Angular** no front-end e **Sequelize** para a interação com o banco de dados.

## Estrutura do Projeto

crud-node-angular-sequelize/ │ ├── backend/ # Diretório para o back-end (Node.js) │ ├── node_modules/ # Dependências do back-end │ ├── models/ # Modelos do Sequelize │ ├── config/ # Configurações do banco de dados │ ├── server.js # Arquivo principal do servidor Node.js │ ├── database.js # Arquivo para configurar o banco de dados │ └── package.json # Dependências do back-end │ └── frontend/ # Diretório para o front-end (Angular) ├── node_modules/ # Dependências do front-end ├── src/ │ ├── app/ │ │ ├── app.component.ts # Componente principal do Angular │ │ ├── app.module.ts # Módulo principal do Angular │ │ └── user.service.ts # Serviço para comunicação com o back-end │ └── index.html # Arquivo HTML principal └── angular.json # Configurações do Angular


---

## Passo 1: Criar o Back-End (Node.js e Sequelize)

### 1.1. Inicializar o Projeto Node.js

1. Crie a pasta para o back-end:

    ```bash
    mkdir backend
    cd backend
    npm init -y
    npm install express sequelize mysql2 body-parser cors
    ```

2. Crie a pasta `models/` para armazenar o modelo do banco de dados.

3. Crie o arquivo `database.js` para configurar o Sequelize:

    ```js
    const { Sequelize } = require('sequelize');

    const sequelize = new Sequelize('mysql://root:password@localhost:3306/mydatabase'); // Substitua com seus dados

    try {
      sequelize.authenticate();
      console.log('Conexão com o banco de dados realizada com sucesso.');
    } catch (error) {
      console.error('Erro ao conectar com o banco de dados:', error);
    }

    module.exports = sequelize;
    ```

4. Crie a pasta `models/` e dentro dela crie o arquivo `user.js` para definir o modelo de usuário:

    ```js
    const { DataTypes } = require('sequelize');
    const sequelize = require('../database');

    const User = sequelize.define('User', {
      name: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
      },
    });

    module.exports = User;
    ```

### 1.2. Criar o Servidor Express

Crie o arquivo `server.js` para configurar as rotas de API:

```js
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const sequelize = require('./database');
const User = require('./models/user');

const app = express();
const port = 3000;

app.use(cors());
app.use(bodyParser.json());

// Sincroniza o banco de dados
sequelize.sync().then(() => {
  console.log('Banco de dados sincronizado');
}).catch(err => {
  console.log('Erro ao sincronizar o banco de dados:', err);
});

// Rota para obter todos os usuários
app.get('/users', async (req, res) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (err) {
    res.status(500).json({ message: 'Erro ao obter usuários', error: err });
  }
});

// Rota para obter um usuário pelo ID
app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findByPk(req.params.id);
    if (user) {
      res.json(user);
    } else {
      res.status(404).json({ message: 'Usuário não encontrado' });
    }
  } catch (err) {
    res.status(500).json({ message: 'Erro ao obter usuário', error: err });
  }
});

// Rota para criar um novo usuário
app.post('/users', async (req, res) => {
  try {
    const { name, email } = req.body;
    const newUser = await User.create({ name, email });
    res.status(201).json(newUser);
  } catch (err) {
    res.status(500).json({ message: 'Erro ao criar usuário', error: err });
  }
});

// Rota para atualizar um usuário
app.put('/users/:id', async (req, res) => {
  try {
    const { name, email } = req.body;
    const user = await User.findByPk(req.params.id);
    if (user) {
      user.name = name;
      user.email = email;
      await user.save();
      res.json(user);
    } else {
      res.status(404).json({ message: 'Usuário não encontrado' });
    }
  } catch (err) {
    res.status(500).json({ message: 'Erro ao atualizar usuário', error: err });
  }
});

// Rota para excluir um usuário
app.delete('/users/:id', async (req, res) => {
  try {
    const user = await User.findByPk(req.params.id);
    if (user) {
      await user.destroy();
      res.status(204).send();
    } else {
      res.status(404).json({ message: 'Usuário não encontrado' });
    }
  } catch (err) {
    res.status(500).json({ message: 'Erro ao excluir usuário', error: err });
  }
});

app.listen(port, () => {
  console.log(`Servidor rodando na porta ${port}`);
});
```

Agora, inicie o servidor:

```bash
Copiar código

node server.js

```


### Passo 2: Criar o Front-End (Angular)
2.1. **Criar o Projeto Angular**
Crie um novo projeto Angular:

```bash
Copiar código
ng new frontend
cd frontend
Instale o HttpClient para realizar as requisições HTTP:

bash
Copiar código
npm install @angular/common/http
```
2.2. Criar o Serviço user.service.ts
Crie o serviço user.service.ts dentro do diretório src/app para interagir com o back-end Node.js.

Crie o arquivo user.service.ts com o seguinte conteúdo:

```ts
Copiar código
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'http://localhost:3000/users';

  constructor(private http: HttpClient) { }

  // Obter todos os usuários
  getUsers(): Observable<any> {
    return this.http.get(this.apiUrl);
  }

  // Obter um usuário pelo ID
  getUser(id: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/${id}`);
  }

  // Adicionar um novo usuário
  addUser(user: any): Observable<any> {
    return this.http.post(this.apiUrl, user);
  }

  // Atualizar um usuário
  updateUser(id: number, user: any): Observable<any> {
    return this.http.put(`${this.apiUrl}/${id}`, user);
  }

  // Excluir um usuário
  deleteUser(id: number): Observable<any> {
    return this.http.delete(`${this.apiUrl}/${id}`);
  }
}
```
### 2.3. Atualizar o Componente Principal app.component.ts
Crie o arquivo app.component.ts com o seguinte conteúdo:

```ts
Copiar código
import { Component, OnInit } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  users: any[] = [];
  newUser = { name: '', email: '' };
  editUser = { id: 0, name: '', email: '' };

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    this.userService.getUsers().subscribe(data => {
      this.users = data;
    });
  }

  addUser() {
    if (this.newUser.name && this.newUser.email) {
      this.userService.addUser(this.newUser).subscribe(() => {
        this.loadUsers();
        this.newUser = { name: '', email: '' }; // Limpar os campos
      });
    }
  }

  edit(id: number) {
    this.userService.getUser(id).subscribe(user => {
      this.editUser = { ...user };
    });
  }

  updateUser() {
    if (this.editUser.name && this.editUser.email) {
      this.userService.updateUser(this.editUser.id, this.editUser).subscribe(() => {
        this.loadUsers();
        this.editUser = { id: 0, name: '', email: '' }; // Limpar os campos
      });
    }
  }

  deleteUser(id: number) {
    this.userService.deleteUser(id).subscribe(() => {
      this.loadUsers();
    });
  }
}
```
### 2.4. Atualizar o HTML app.component.html
Crie o arquivo app.component.html com o seguinte conteúdo:

```html
Copiar código
<div>
  <h1>CRUD de Usuários</h1>

  <!-- Formulário para adicionar novo usuário -->
  <h3>Adicionar Novo Usuário</h3>
  <input [(ngModel)]="newUser.name" placeholder="Nome" />
  <input [(ngModel)]="newUser.email" placeholder="Email" />
  <button (click)="addUser()">Adicionar</button>

  <!-- Tabela de usuários -->
  <h3>Lista de Usuários</h3>
  <table>
    <tr>
      <th>ID</th>
      <th>Nome</th>
      <th>Email</th>
      <th>Ações</th>
    </tr>
    <tr *ngFor="let user of users">
      <td>{{ user.id }}</td>
      <td>{{ user.name }}</td>
      <td>{{ user.email }}</td>
      <td>
        <button (click)="edit(user.id)">Editar</button>
        <button (click)="deleteUser(user.id)">Excluir</button>
      </td>
    </tr>
  </table>

  <!-- Formulário para editar usuário -->
  <div *ngIf="editUser.id">
    <h3>Editar Usuário</h3>
    <input [(ngModel)]="editUser.name" placeholder="Nome" />
    <input [(ngModel)]="editUser.email" placeholder="Email" />
    <button (click)="updateUser()">Atualizar</button>
  </div>
</div>
```
2.5. Configurar o Módulo app.module.ts
Crie o arquivo app.module.ts com o seguinte conteúdo:

```ts
Copiar código
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';
import { FormsModule } from '@angular/forms';
import { AppComponent } from './app.component';
import { UserService } from './user.service';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    FormsModule
  ],
  providers: [UserService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
### Passo 3: Rodando o Projeto
1. **Rodando o Back-End (Node.js):**

Na pasta backend, execute o servidor Node.js:
```bash
Copiar código
node server.js
```
O servidor estará rodando em http://localhost:3000.

2. **Rodando o Front-End (Angular):**

Na pasta frontend, execute o servidor Angular:
```
bash
Copiar código
ng serve
```
O front-end estará disponível em http://localhost:4200.