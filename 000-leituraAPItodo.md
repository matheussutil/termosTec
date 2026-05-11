# Comentários dos códigos do Core.ts e da API feitos em sala
### CORE.TS
```typescript
//CORE.TS
const jsonFilePath = __dirname + '/data.temp.json';//variavel que fornece o caminho 
const list: string[] = await loadFromFile();//recebe uma lista do loadfile

//função assincrona 
async function loadFromFile() {
  try {//metodo
    const file = Bun.file(jsonFilePath);//usa funçaõ do bun para referenciar
    const content = await file.text();//le o arquivo
    return JSON.parse(content) as string[];//converte o JSON em um array de string
  } catch (error: any) {
    if (error.code === 'ENOENT')//caso tenha erro
      return [];//retorna lista vazia
    throw error;//lança o erro
  }
}

//função de salvar o arquivo
async function saveToFile() {
  try {
    await Bun.write(jsonFilePath, JSON.stringify(list));//função do Bun, por meio do filepath monta uma lista em JSON do que tem no arquivo
  } catch (error: any) {
   throw new Error("Erro ao salvar os dados no arquivo: " + error.message);//mensagem que aparece caso aconteça um erro
  }
}

//função de adicionar itens
async function addItem(item: string) {
  list.push(item);//adiciona um item na lista
  await saveToFile();//salva as mudanças
}

//função para mostrar os itens
async function getItems() {
  return list;//retorna a ultima lista salva
}

//função que altera os valores de um item
async function updateItem(index: number, newItem: string) {
  if (index < 0 || index >= list.length)//verifica se o index do item é valido
    throw new Error("Index fora dos limites");//mensagem caso não seja
  list[index] = newItem;//muda o valor do item selecionada, conforme index
  await saveToFile();//salva mudanças
}

//função de remover itens
async function removeItem(index: number) {
  if (index < 0 || index >= list.length)//verifica se o index do item é valido
    throw new Error("Index fora dos limites");//mensagem caso não seja
  list.splice(index, 1);//apaga uma item da lista conforme index
  await saveToFile();//salva
}


export default { addItem, getItems, updateItem, removeItem };//possibilita o uso das funções em outros lugares
```
---
### API
```typescript
//API
import todo from "./core.ts";

const server = Bun.serve({
  port: 3000,

  routes: {
    "/": new Response(Bun.file("./public/index.html")),

    "/api/todo": {
      GET: async () => {
        const items = await todo.getItems()
        return Response.json(items)
      },

      POST: async (req) => {
        const data = await req.json() as any;
        const item = data.item || null;
        if (!item)
          return Response.json('Por favor, forneça um item para adicionar.', { status: 400 });
        await todo.addItem(item);
        return Response.json(data);
      },
    },

    "/api/todo/:index": {
      PUT: async (req) => {
        const index = parseInt(req.params.index);
        if (isNaN(index))
          return Response.json('Índice inválido. um número inteiro é esperado.', { status: 400 });
        const data = await req.json() as any;
        const newItem = data.newItem || null;
        if (!newItem)
          return Response.json('Por favor, forneça um novo item para atualizar.', { status: 400 });
        try {
          await todo.updateItem(index, newItem);
          return Response.json(`Item no índice ${index} atualizado para "${newItem}".`);
        } catch (error: any) {
          return Response.json(error.message, { status: 400 });
        }
      },

      DELETE: async (req) => {
        const index = parseInt(req.params.index);
        if (isNaN(index))
          return Response.json('Índice inválido.', { status: 400 });
        try {
          await todo.removeItem(index);
          return Response.json(`Item no índice ${index} removido com sucesso.`);
        } catch (error: any) {
          return Response.json(error.message, { status: 400 });
        }
      },
    },

    // EXEMPLO BÁSICO

    "/api/exemplo": {
      GET: () => {
        return new Response(`Esse é o exemplo: ${Date.now()}`)
      },

      POST: async (req) => {
        const data = await req.json() as any;
        data.recebidoEm = new Date().toLocaleDateString("pt-BR");
        return Response.json(data);
      },
    },

    "/api/exemplo/:id": {
      PUT: async (req, params) => {
        const { id } = req.params;
        const data = await req.json() as any;
        data.id = id;
        data.recebidoEm = new Date().toLocaleDateString("pt-BR");
        return Response.json(data);
      },

      PATCH: async (req, params) => {
        const { id } = req.params;
        const data = await req.json() as any;
        data.chavesAtualizadas = Object.keys(data);
        data.id = id;
        data.atualizadoEm = new Date().toLocaleDateString("pt-BR");
        return Response.json(data);
      },

      DELETE: (req, params) => {
        const { id } = req.params;
        return new Response(`Recurso com id ${id} deletado`, { status: 200 });
      }
    }
    // FIM DO EXEMPLO BÁSICO
  },

  async fetch(req) {
    return new Response(`Not Found`, { status: 404 });
  },
});

console.log(`Server running at http://localhost:${server.port}`);
```