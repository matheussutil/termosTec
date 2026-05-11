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
import todo from "./core.ts";//importa a logica do core.ts para esse arquivo

//cria um servidor Bun
const server = Bun.serve({
  port: 3000,//porta do server

  //lista de rotas da API
  routes: {
    "/": new Response(Bun.file("./public/index.html")),//arquivo inicial
  //apresenta as funcionalidades do TODO
    "/api/todo": {
      //metodo GET
      GET: async () => {
        const items = await todo.getItems()//pega a lista de itens por meio da função getItems
        return Response.json(items)//retorna o array no formato JSON
      },
      
      //Metodo POST
      POST: async (req) => {
        const data = await req.json() as any;//pega os dados da requisição
        const item = data.item || null;//transforma em string se possivel, se não deixa nulo
        if (!item)//valida se ha item
          return Response.json('Por favor, forneça um item para adicionar.', { status: 400 });//em caso de erro gera a resposta com os status
        await todo.addItem(item);//usa da função addItem para adicionar o item
        return Response.json(data);//retorna o que foi adicionado
      },
    },

    //rotas para o Index
    "/api/todo/:index": {
      //Metodo PUT
      PUT: async (req) => {
        const index = parseInt(req.params.index);//paramentro para numero
        if (isNaN(index))//verifica se index tem numero
          return Response.json('Índice inválido. um número inteiro é esperado.', { status: 400 });//caso não seja numero da uma resposta e seu status
        const data = await req.json() as any;//pega os dados da requisição
        const newItem = data.newItem || null; //transforma em string se possivel, se não deixa nulo
        if (!newItem)//valida se ha item
          return Response.json('Por favor, forneça um novo item para atualizar.', { status: 400 });//em caso de erro gera a resposta com os status
        try {
          await todo.updateItem(index, newItem);//usa da função updateItem e troca o item conforme o index
          return Response.json(`Item no índice ${index} atualizado para "${newItem}".`);//resposta de sucesso
        } catch (error: any) {//pega erros
          return Response.json(error.message, { status: 400 });//resposta em caso de erro
        }
      },

      //Metodo DELETE
      DELETE: async (req) => {
        const index = parseInt(req.params.index);//paramentro para numero
        if (isNaN(index))//verifica se index tem numero
          return Response.json('Índice inválido.', { status: 400 });//caso não seja numero da uma resposta e seu status
        try {
          await todo.removeItem(index);//usa a função removeItem
          return Response.json(`Item no índice ${index} removido com sucesso.`);//resposta de sucesso
        } catch (error: any) {//pega erros
          return Response.json(error.message, { status: 400 });//resposta em caso de erro
        }
      },
    },
  },

  //executa caso não seja acessada nenhuma rota
  async fetch(req) {
    return new Response(`Not Found`, { status: 404 });//resposta da função
  },
});

//avisa que o servidor esta funcionando
console.log(`Server running at http://localhost:${server.port}`);
```
