<template>
  <h1>Produtos em estoque</h1>
  <div class="entradaDados">
    <input type="text" v-model="inputProduto.name" placeholder="Nome do Produto">
    <input type="text" v-model="inputProduto.price" placeholder="Preço">
    <input type="text" v-model="inputProduto.category" placeholder="Categoria">
    <input type="text" v-model="inputProduto.stock" placeholder="Estoque">
    <button :disabled="!Number(inputProduto.price) || !Number(inputProduto.stock)"
      @click="salvarProduto">Salvar</button>
  </div>
  <div class="tabelaProdutos">
    <ul>
      <li v-for="produto in produtos" :key="produto.id">
        {{ produto.id }}
        {{ produto.nome }}
        R$ {{ produto.preco }}
        <ul>
          <li>
            {{ produto.categoria }}
            {{ produto.estoque }}
          </li>
        </ul>
        <button @click="atribuirValoresParaEdicao(produto.id)">Editar</button>
        <button @click="apagarProduto(produto.id)">Apagar</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import Produto from "./components/produtos.vue";
import { onMounted, ref } from 'vue';
import axios from 'axios';

onMounted(() => {
  obterProdutos();
});

const inputProduto = ref([])

const produtos = ref([])

async function obterProdutos() {
  const resposta = await axios.get('https://api-lpv.onrender.com/produtos');
  produtos.value = resposta.data.data;
}

async function salvarProduto() {
  let missingFields = '';
  !inputProduto.value.name ? missingFields += 'nome ' : null;
  !inputProduto.value.price ? missingFields += 'preço ' : null;
  !inputProduto.value.category ? missingFields += 'categoria ' : null;
  !inputProduto.value.stock ? missingFields += 'estoque ' : null;
  if (missingFields != '') {
    alert(`Campos faltando! Preencha os seguintes campos para prosseguir: ${missingFields.trim()}`);
    return 0;
  }

  if (inputProduto.value.id != '' && inputProduto.value.id != undefined) {
    const resposta = await axios.patch(`https://api-lpv.onrender.com/produtos/${inputProduto.value.id}`, {
      nome: inputProduto.value.name,
      preco: Number(inputProduto.value.price),
      categoria: inputProduto.value.category,
      estoque: Number(inputProduto.value.stock)
    })
  } else {
    const resposta = await axios.post("https://api-lpv.onrender.com/produtos", {
      nome: inputProduto.value.name,
      preco: Number(inputProduto.value.price),
      categoria: inputProduto.value.category,
      estoque: Number(inputProduto.value.stock)
    })
  }

  inputProduto.value = {};
  obterProdutos();
  return 0;
}

async function buscarProdutoPorID(idProduto) {
  const resposta = await axios.get(`https://api-lpv.onrender.com/produtos/${idProduto}`);
  return resposta.data;
}

async function atribuirValoresParaEdicao(idProduto) {
  const busca = await buscarProdutoPorID(idProduto);
  inputProduto.value.id = busca.id;
  inputProduto.value.name = busca.nome;
  inputProduto.value.price = busca.preco;
  inputProduto.value.category = busca.categoria;
  inputProduto.value.stock = busca.estoque;

}

async function apagarProduto(idProduto) {
  if (confirm("Tem certeza que deseja excluir este produto?")) {
    buscarProdutoPorID(idProduto);
    const resposta = await axios.delete(`https://api-lpv.onrender.com/produtos/${idProduto}`);
    if (resposta.data == '') {
      alert("Produto excluído com sucesso.");
    } else {
      alert(`Erro! ${resposta.error}`)
    }
  }
  obterProdutos();
  return 0;
}

</script>

<style scoped>
.entradaDados {
  width: 95%;
  border: 10px double rgb(70, 35, 146);
  margin: 10px;
  display: flex;
}

.tabelaProdutos {
  height: 675px;
  width: 95%;
  border: 10px double rgb(70, 35, 146);
  margin: 10px;
  display: flex;
}
</style>