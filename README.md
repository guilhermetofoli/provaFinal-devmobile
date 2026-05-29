# 📦 Guia do Back-end: MockAPI.io

Este repositório documenta a configuração completa do **MockAPI.io** utilizado como camada de banco de dados e API RESTful para o aplicativo móvel de controle de almoxarifado. Ele detalha a estrutura de requisições, paginação, filtros, ordenação e respostas customizadas para apoiar o desenvolvimento ágil do projeto.

---

## 🚀 Guia de Início Rápido (Quick Start Guide)

O MockAPI permite criar endpoints, gerar dados fictícios (*mockados*) e realizar operações completas de CRUD de forma ágil para o desenvolvimento em React Native.

### 1. Criando o Projeto
Cada projeto no MockAPI ganha uma **URL Base única** compartilhada por todos os seus recursos e endpoints.
* Acesse o painel e crie um novo projeto (ex: `Almoxarifado App`).

### 2. Recursos Relacionados e Endpoints Gerados
Quando definimos uma relação entre recursos (por exemplo, onde um `User` possui várias `tasks`), o MockAPI gera automaticamente rotas diretas e rotas aninhadas:

| Método | URL | Payload (Corpo) | Resposta | Descrição |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/users` | Vazio | `User[]` | Lista todos os usuários |
| **GET** | `/users/:userId` | Vazio | `User` | Detalhes de um usuário específico |
| **POST** | `/users` | `User` | `User` | Cria um novo usuário |
| **PUT/PATCH** | `/users/:userId` | `User` | `User` | Atualiza dados do usuário |
| **DELETE** | `/users/:userId` | Vazio | `User` | Remove um usuário |
| **GET** | `/users/:userId/tasks` ou `/tasks` | Vazio | `Task[]` | Lista as tarefas de um usuário específico |
| **GET** | `/users/:userId/tasks/:taskId` | Vazio | `Task` | Detalhes de uma tarefa específica do usuário |
| **POST** | `/users/:userId/tasks` | `Task` | `Task` | Cria uma nova tarefa vinculada ao usuário |
| **PUT/PATCH** | `/users/:userId/tasks/:taskId`| `Task` | `Task` | Atualiza a tarefa do usuário |
| **DELETE** | `/users/:userId/tasks/:taskId`| Vazio | `Task` | Remove a tarefa do usuário |

### 3. Geração Automática de Dados (Faker.js)
Para testar o aplicativo com dados realistas sem precisar cadastrar um por um:
1. No esquema do recurso, associe funções do **Faker.js** para cada campo (ex: `lorem.sentences`, `datatype.number`).
2. Na tela inicial do projeto, passe o mouse sobre o retângulo cinza abaixo do nome do recurso, digite a quantidade de registros desejada e clique para gerar os dados de teste.
3. Clique no botão **Data** ao lado do recurso para inspecionar ou alterar os registros manualmente via modal.

---

## 🛠️ Respostas Customizadas (Custom Responses)

Por padrão, o MockAPI retorna diretamente o item individual ou a coleção de dados do banco de dados (representado pela variável interna `$mockData`). No entanto, para simular cenários reais do mercado (como paginação complexa ou envelopamento de objetos), podemos customizar a estrutura de retorno de qualquer endpoint.

### Como Configurar:
1. Abra o modal do Recurso (**Resource modal**) no painel do MockAPI.
2. Role até a seção **Endpoints**.
3. Altere o campo de resposta de um método (que por padrão vem preenchido com `$mockData`) pelo JSON customizado que desejar.

#### Exemplo de Estrutura Customizada:
Se você deseja que o seu endpoint retorne o total de itens no banco e um ID de requisição junto com a lista, configure o campo de resposta assim:

```json
{
  "status": "success",
  "count": "$count",
  "requestId": "$datatype.uuid",
  "items": "$mockData"
} 

```

* "status": "success": Dado estático (será igual em todas as requisições).
* "$count": Variável interna do MockAPI que injeta o número total de registros no banco.
* "$mockData": Injeta a coleção de dados original ou o registro consultado.
* "$datatype.uuid": Invoca um método do Faker.js em tempo de execução, gerando um UUID novo a cada chamada de API.

#Métodos Úteis do Faker.js para o Schema

Ao estruturar os campos dos recursos (como Insumos ou Movimentações), o painel do MockAPI disponibiliza uma série de métodos do Faker.js. Abaixo estão listados os principais módulos e exemplos aplicáveis a sistemas de gerenciamento:

---

* Módulo address: address.city, address.streetAddress, address.zipCode (Mapear fornecedores ou locais de entrega).
* Módulo commerce: commerce.productName, commerce.productDescription, commerce.price (Identificação de insumos médicos).
* Módulo datatype: datatype.number, datatype.boolean, datatype.uuid (Controle de quantidades, status e hashes).
* Módulo date: date.recent, date.future (Datas de entrada, baixa ou validade de materiais).
* Módulo name / internet: name.fullName, internet.email (Dados de enfermeiros, técnicos e administradores).

---

# Recursos e Parâmetros Avançados da URL
O MockAPI oferece suporte nativo a operações complexas de banco de dados diretamente via parâmetros de busca na URL (Query Parameters). Os cabeçalhos de requisição esperam obrigatoriamente a chave 'content-type': 'application/json'.

## 1. Filtragem e Busca Geral (Filtering)

Permite realizar buscas por strings em todos os campos ou segmentar por valores exatos.

| Parâmetro            | Tipo         | Exemplo         | Descrição                                                           |
|----------------------|--------------|-----------------|---------------------------------------------------------------------|
| `search`             | String       | `search=hello` | Busca o termo "hello" em todos os campos do recurso                |
| `filter`             | String       | `filter=hello` | Atalho com comportamento idêntico ao `search`                      |
| `[nome_do_campo]`    | String/Bool  | `completed=false` | Retorna itens com correspondência exata no campo especificado |

```javascript
// Exemplo: Buscar tarefas que correspondam à string "hello"
const url = new URL('https://PROJECT_TOKEN.mockapi.io/users/1/tasks');
url.searchParams.append('title', 'hello');

fetch(url, {
  method: 'GET',
  headers: { 'content-type': 'application/json' },
})
.then(res => res.ok ? res.json() : Promise.reject('Erro HTTP'))
.then(tasks => console.log(tasks));
```

## 2. Paginação (Pagination)

Evita sobrecarregar a interface móvel limitando o volume de dados trafegados por requisição.

| Parâmetro     | Tipo   | Exemplo      | Descrição                                                |
|----------------|--------|---------------|------------------------------------------------------------|
| `page` / `p`   | Number | `page=1`      | Especifica o número da página desejada                    |
| `limit` / `l`  | Number | `limit=10`    | Quantidade máxima de itens retornados por página          |


```javascript
// Exemplo: Buscar a página 1 trazendo apenas 10 registros não completados
const url = new URL('https://PROJECT_TOKEN.mockapi.io/users/1/tasks');
url.searchParams.append('completed', false);
url.searchParams.append('page', 1);
url.searchParams.append('limit', 10);

fetch(url, {
  method: 'GET',
  headers: { 'content-type': 'application/json' },
})
.then(res => res.json())
.then(paginatedTasks => console.log(paginatedTasks));
```

## 3. Ordenação (Sorting)

Organiza a exibição sequencial dos dados conforme o atributo escolhido.

| Parâmetro                              | Tipo   | Exemplo             | Descrição                                                                 |
|----------------------------------------|--------|----------------------|-----------------------------------------------------------------------------|
| `sortBy` / `sortby` / `orderBy` / `orderby` | String | `sortBy=title` | Define qual campo guiará o critério de ordenação                          |
| `order`                                | String | `order=desc`        | Define o sentido da ordenação: `asc` (crescente/padrão) ou `desc` (decrescente) |


```javascript
// Exemplo: Ordenar lista de tarefas de forma decrescente pelo título
const url = new URL('https://PROJECT_TOKEN.mockapi.io/users/1/tasks');
url.searchParams.append('sortBy', 'title');
url.searchParams.append('order', 'desc');

fetch(url, {
  method: 'GET',
  headers: { 'content-type': 'application/json' },
})
.then(res => res.json())
.then(sortedTasks => console.log(sortedTasks));
```

#Requisições Assíncronas e Ciclo de Vida (React Native)

Para alimentar a interface móvel do almoxarifado de saúde, as chamadas à API utilizam a sintaxe assíncrona async/await mapeada dentro do ciclo de vida dos componentes através dos React Hooks:

useState: Cria e altera estados locais para armazenar os dados de resposta e atualizar os indicadores visuais na tela (como telas de carregamento).

useEffect: Dispara a rotina de busca de dados de forma automática no momento em que a tela é carregada (montada na árvore de componentes), utilizando uma matriz de dependências vazia [] para garantir que a função seja executada apenas uma única vez após a renderização inicial.R

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator, StyleSheet } from 'react-native';

const API_URL = 'https://PROJECT_TOKEN.mockapi.io/users/1/tasks';

export default function ListaTarefas() {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(true);

  // Execução assíncrona com tratamento de erros
  const fetchTasks = async () => {
    try {
      const response = await fetch(API_URL, {
        method: 'GET',
        headers: { 'content-type': 'application/json' }
      });

      if (!response.ok) throw new Error(`Erro HTTP! Status: ${response.status}`);

      const data = await response.json();
      setTasks(data);
    } catch (error) {
      console.error("Erro ao buscar dados do MockAPI:", error);
    } finally {
      setLoading(false);
    }
  };

  // Gancho de ciclo de vida (useEffect) executado na montagem do componente
  useEffect(() => {
    fetchTasks();
  }, []); // Array de dependências vazio garante execução única

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" color="#FF6B00" />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={tasks}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.card}>
            <Text style={styles.title}>{item.title}</Text>
            <Text>Status: {item.completed ? '✅ Concluída' : '⏳ Pendente'}</Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#f5f5f5' },
  center: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  card: { padding: 16, backgroundColor: '#fff', marginBottom: 8, borderRadius: 8, elevation: 2 },
  title: { fontSize: 16, fontWeight: 'bold', marginBottom: 4 }
});
```