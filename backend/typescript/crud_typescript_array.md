# CRUD Completo com TypeScript Utilizando Arrays

Este guia descreve como criar um CRUD básico utilizando TypeScript e arrays para armazenar dados temporariamente.

## Configurar o Ambiente

1. Certifique-se de que o TypeScript está instalado:
    ```bash
    npm install -g typescript
    ```
2. Inicialize um novo projeto:
    ```bash
    mkdir meu-crud-typescript
    cd meu-crud-typescript
    npm init -y
    npm install typescript --save-dev
    npx tsc --init
    ```

## Estrutura do Projeto

Crie um arquivo chamado `crud.ts` onde todo o código será implementado.

## Definir a Interface dos Dados

No arquivo `crud.ts`, defina uma interface para o tipo de dados:

```typescript
interface Item {
    id: number;
    nome: string;
}
```

## Criar o Array para Armazenar Dados

Crie um array que servirá como base de dados temporária:

```typescript
const itens: Item[] = [];
```

## Implementar as Operações do CRUD

### 1. Operação de Criação (Create)

Adicione novos itens ao array:

```typescript
function adicionarItem(nome: string): void {
    const novoItem: Item = {
        id: itens.length + 1,
        nome
    };
    itens.push(novoItem);
    console.log("Item adicionado:", novoItem);
}
```

### 2. Operação de Leitura (Read)

Liste todos os itens do array:

```typescript
function listarItens(): void {
    console.log("Lista de itens:", itens);
}
```

### 3. Operação de Atualização (Update)

Atualize o nome de um item pelo ID:

```typescript
function atualizarItem(id: number, novoNome: string): void {
    const item = itens.find(item => item.id === id);
    if (item) {
        item.nome = novoNome;
        console.log("Item atualizado:", item);
    } else {
        console.log("Item não encontrado.");
    }
}
```

### 4. Operação de Remoção (Delete)

Remova um item do array pelo ID:

```typescript
function removerItem(id: number): void {
    const index = itens.findIndex(item => item.id === id);
    if (index !== -1) {
        const [removido] = itens.splice(index, 1);
        console.log("Item removido:", removido);
    } else {
        console.log("Item não encontrado.");
    }
}
```

## Testar as Funções

Adicione chamadas às funções para testar cada operação:

```typescript
adicionarItem("Item 1");
adicionarItem("Item 2");
listarItens();

atualizarItem(1, "Item Atualizado");
listarItens();

removerItem(2);
listarItens();
```

## Compilar e Executar o Código

1. Compile o código TypeScript para JavaScript:
    ```bash
    npx tsc crud.ts
    ```
2. Execute o arquivo JavaScript gerado:
    ```bash
    node crud.js
    ```

## Código Completo

```typescript
interface Item {
    id: number;
    nome: string;
}

const itens: Item[] = [];

function adicionarItem(nome: string): void {
    const novoItem: Item = {
        id: itens.length + 1,
        nome
    };
    itens.push(novoItem);
    console.log("Item adicionado:", novoItem);
}

function listarItens(): void {
    console.log("Lista de itens:", itens);
}

function atualizarItem(id: number, novoNome: string): void {
    const item = itens.find(item => item.id === id);
    if (item) {
        item.nome = novoNome;
        console.log("Item atualizado:", item);
    } else {
        console.log("Item não encontrado.");
    }
}

function removerItem(id: number): void {
    const index = itens.findIndex(item => item.id === id);
    if (index !== -1) {
        const [removido] = itens.splice(index, 1);
        console.log("Item removido:", removido);
    } else {
        console.log("Item não encontrado.");
    }
}

// Testando o CRUD
adicionarItem("Item 1");
adicionarItem("Item 2");
listarItens();

atualizarItem(1, "Item Atualizado");
listarItens();

removerItem(2);
listarItens();
```

---