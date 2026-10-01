<template>
  <div class="container py-4">
    <div class="text-center mb-4">
      <h1 class="fw-bold text-primary">Produtos em estoque</h1>
      <p class="text-muted">Gerencie os produtos cadastrados no estoque</p>
    </div> <!-- Formulário -->
    <div class="card shadow-sm mb-4">
      <div class="card-header bg-primary text-white">
        <h5 class="mb-0">Cadastro de produto</h5>
      </div>
      <div class="card-body">
        <div class="row g-3">
          <div class="col-md-6"> <label class="form-label">Nome do Produto</label> <input type="text"
              v-model="inputProduto.name" class="form-control" placeholder="Digite o nome do produto"> </div>
          <div class="col-md-6"> <label class="form-label">Preço</label> <input type="text" v-model="inputProduto.price"
              class="form-control" placeholder="Digite o preço"> </div>
          <div class="col-md-6"> <label class="form-label">Categoria</label> <input type="text"
              v-model="inputProduto.category" class="form-control" placeholder="Digite a categoria"> </div>
          <div class="col-md-6"> <label class="form-label">Estoque</label> <input type="text"
              v-model="inputProduto.stock" class="form-control" placeholder="Quantidade em estoque"> </div>
          <div class="col-12"> <button :disabled="!Number(inputProduto.price) || !Number(inputProduto.stock)"
              @click="salvarProduto" class="btn btn-success px-4"> Salvar produto </button> </div>
        </div>
      </div>
    </div> <!-- Tabela -->
    <div class="card shadow-sm">
      <div class="card-header bg-dark text-white">
        <h5 class="mb-0">Produtos cadastrados</h5>
      </div>
      <div class="card-body p-0">
        <div class="table-responsive">
          <table class="table table-hover table-striped align-middle mb-0">
            <thead class="table-primary">
              <tr>
                <th>Nome</th>
                <th>Preço</th>
                <th>Categoria</th>
                <th>Estoque</th>
                <th class="text-center">Ações</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="produto in produtos" :key="produto.id">
                <td class="fw-semibold">{{ produto.nome }}</td>
                <td> R$ {{ produto.preco }} </td>
                <td> <span class="badge bg-secondary"> {{ produto.categoria }} </span> </td>
                <td> <span class="badge" :class="produto.estoque > 0 ? 'bg-success' : 'bg-danger'"> {{ produto.estoque
                    }} </span> </td>
                <td class="text-center"> <button @click="atribuirValoresParaEdicao(produto.id)"
                    class="btn btn-sm btn-warning me-2"> Editar </button> <button @click="apagarProduto(produto.id)"
                    class="btn btn-sm btn-danger"> Apagar </button> </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
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
    return;
  }

  const requestBody = {
    nome: inputProduto.value.name,
    preco: Number(inputProduto.value.price),
    categoria: inputProduto.value.category,
    estoque: Number(inputProduto.value.stock)
  }

  if (inputProduto.value.id != '' && inputProduto.value.id != undefined) {
    await axios.patch(`https://api-lpv.onrender.com/produtos/${inputProduto.value.id}`, requestBody)
  } else {
    await axios.post("https://api-lpv.onrender.com/produtos", requestBody)
  }

  inputProduto.value = {};
  obterProdutos();
  return;
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
    const resposta = await axios.delete(`https://api-lpv.onrender.com/produtos/${idProduto}`);
    if (resposta.data == '') {
      alert("Produto excluído com sucesso.");
    } else {
      alert(`Erro! ${resposta.error}`)
    }
  }
  obterProdutos();
  return;
}

</script>

<style scoped>
.container {
  max-width: 1100px;
}

.card {
  border: none;
  border-radius: 12px;
  overflow: hidden;
}

.card-header {
  padding: 15px 20px;
}

.form-control {
  border-radius: 8px;
}

.form-control:focus {
  box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.15);
}

.btn {
  border-radius: 7px;
}

.table th,
.table td {
  padding: 14px 16px;
}

.table thead th {
  font-weight: 600;
}

.badge {
  font-size: 0.85rem;
  padding: 6px 9px;
}
</style>
