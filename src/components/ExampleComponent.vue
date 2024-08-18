<template>
  <q-page class="q-pa-md">
    <q-card class="q-pa-md">
      <q-card-section>
        <div class="text-h6">Dados da Empresa</div>
      </q-card-section>
      <q-form @submit.prevent="onSubmit">
        <q-input filled v-model="inputData" label="Insira o CNPJ"></q-input>

        <q-btn label="Enviar" type="submit" color="primary"></q-btn>
      </q-form>

      <q-card-section v-if="loading">
        <q-spinner-dots color="primary" />
        Carregando...
      </q-card-section>

      <q-card-section v-else>
        <q-list bordered>
          <q-item-label class="text-bold">CNPJ Raiz:</q-item-label>
          <q-item>{{ jsonData.cnpj_raiz }}</q-item>

          <q-item-label class="text-bold">Razão Social:</q-item-label>
          <q-item>{{ jsonData.razao_social }}</q-item>

          <q-item-label class="text-bold">Capital Social:</q-item-label>
          <q-item>{{ formatCurrency(jsonData.capital_social) }}</q-item>

          <q-item-label class="text-bold">Porte:</q-item-label>
          <q-item>{{ jsonData.porte?.descricao }}</q-item>

          <q-item-label class="text-bold">Natureza Jurídica:</q-item-label>
          <q-item>{{ jsonData.natureza_juridica?.descricao }}</q-item>

          <q-item-label class="text-bold">Data de Atualização:</q-item-label>
          <q-item>{{ formatDate(jsonData.atualizado_em) }}</q-item>

          <q-separator />

          <!-- Seção Sócios -->
          <div v-if="jsonData.socios && jsonData.socios.length">
            <q-item-label class="text-bold">Sócios:</q-item-label>
            <q-list bordered>
              <q-item v-for="(socio, index) in jsonData.socios" :key="index">
                <q-item-section>
                  <q-item-label class="text-bold">Nome:</q-item-label>
                  <q-item-label>{{ socio.nome }}</q-item-label>
                </q-item-section>
                <q-item-section>
                  <q-item-label class="text-bold">Qualificação:</q-item-label>
                  <q-item-label>{{
                    socio.qualificacao_socio?.descricao
                  }}</q-item-label>
                </q-item-section>
                <q-item-section>
                  <q-item-label class="text-bold">Entrada:</q-item-label>
                  <q-item-label>{{
                    formatDate(socio.data_entrada)
                  }}</q-item-label>
                </q-item-section>
                <!-- Adicionar outras informações conforme necessário -->
              </q-item>
            </q-list>
          </div>

          <q-separator />

          <!-- Seção Atividades Secundárias -->
          <div
            v-if="
              jsonData.estabelecimento &&
              jsonData.estabelecimento.atividades_secundarias &&
              jsonData.estabelecimento.atividades_secundarias.length
            "
          >
            <q-item-label class="text-bold"
              >Atividades Secundárias:</q-item-label
            >
            <q-list bordered>
              <q-item
                v-for="(atividade, index) in jsonData.estabelecimento
                  .atividades_secundarias"
                :key="index"
              >
                <q-item>{{ atividade.descricao }}</q-item>
              </q-item>
            </q-list>
          </div>

          <!-- Adicione outras seções conforme necessário -->
        </q-list>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
export default {
  name: 'ApiDataComponent',
  data() {
    return {
      inputData: '', // Adicionado para garantir que inputData esteja definido
      jsonData: {},
      loading: false,
    };
  },
  methods: {
    formatCurrency(value) {
      return new Intl.NumberFormat('pt-BR', {
        style: 'currency',
        currency: 'BRL',
      }).format(value);
    },
    formatDate(value) {
      return new Date(value).toLocaleDateString('pt-BR', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric',
      });
    },
    onSubmit() {
      // Verifica se o inputData está definido e não está vazio
      if (this.inputData) {
        const cnpj = this.inputData.replace(/\D/g, '');

        // Lógica para enviar os dados ao backend
        fetch(`https://publica.cnpj.ws/cnpj/${cnpj}`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
          },
        })
          .then((response) => response.json())
          .then((data) => {
            this.jsonData = data;
          })
          .catch((error) => {
            console.error('Erro:', error);
          })
          .finally(() => {
            this.loading = false;
          });
      }
    },
  },
};
</script>

<style scoped>
.text-bold {
  font-weight: bold;
}
</style>
