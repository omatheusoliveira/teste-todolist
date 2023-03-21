<template class="content">
  <div>
    <!-- Inicio Principal -->
    <v-card>
      <v-toolbar flat>
        <v-icon> fas fa-list </v-icon>
        <v-toolbar-title class="pl-4"> Lista de Tarefas </v-toolbar-title>
        <v-spacer></v-spacer>
      </v-toolbar>
      <v-divider></v-divider>
      <v-row class="d-flex justify-end mt-5 mr-5">
        <v-col cols="3">
          <v-select
            dense
            outlined
            label="Filtro"
            v-model="filtro"
            :items="statusFiltro"
            item-text="field"
            item-value="value"
            clearable
            @change="filtrarGrid"
          ></v-select>
        </v-col>
      </v-row>
      <v-card-text>
        <v-row>
          <v-col class="px-6 py-6">
            <v-data-table
              dense
              :headers="listTask.cabecalho"
              :items="listTask.items"
              :items-per-page="listTask.porPagina"
              :loading="listTask.carregando"
              :sort-by.sync="sortBy"
              :sort-desc.sync="sortDesc"
              height="73vh"
              hide-default-footer
              fixed-header
              class="elevation-1"
            >
              <template v-slot:[`item.operacoes`]="{ item }">
                <v-icon
                  size="16"
                  color="green"
                  class="operacoes"
                  @click="abrirEditarTask(item)"
                  >fas fa-pen</v-icon
                >
                <v-icon
                  size="16"
                  color="red"
                  class="operacoes"
                  @click="deleteTask(item)"
                  >fas fa-trash</v-icon
                >
              </template>

              <template v-slot:[`item.done`]="{ item }" > {{ item.done ==  0 ? 'Ativo' : 'Concluído'}}</template>

            </v-data-table>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>
    <!-- Fim Principal -->

    <!-- Modal Alteracao -->
    <v-row class="px-9 py-7" justify="end">
      <v-dialog v-model="dialogTask" max-width="600">
        <v-card>
          <v-container>
            <v-card-title class="text-h5" style="justify-content: center">
              Atualizar - Tarefa
            </v-card-title>
            <v-row class="pt-8 d-flex justify-center">
              <v-col cols="6" sm="8" md="5">
                <v-select
                  :items="statusTask"
                  label="Status da Tarefa"
                  v-model="task.done"
                  dense
                ></v-select>
              </v-col>
              <v-col cols="6" sm="8" md="5">
                <v-text-field
                  label="Usuário responsável"
                  v-model="task.name"
                  dense
                  readonly
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row class="d-flex justify-center">
              <v-col cols="10" sm="10" md="10">
                <v-textarea
                  name="input-7-1"
                  outlined
                  label="Descrição"
                  no-resize
                  clearable
                  auto-grow
                  v-model="task.descricao"
                ></v-textarea>
              </v-col>
            </v-row>
          </v-container>

          <v-card-actions class="alignmentDialog">
            <v-btn
              class="ma-2"
              color="error"
              @click="(dialogTask = false), carregarTask()"
            >
              Cancelar
            </v-btn>
            <v-btn class="ma-2" color="primary" @click="editarTask()">
              Salvar
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-row>
    <!-- Fim Modal Alteracao -->
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ListarTask",

  data: () => ({
    dialogTask: false,

    sortBy: "id",
    sortDesc: true,

    task: {
      descricao: "",
      done: null,
      name: "",
      id: "",
    },

    filtro: null,

    statusFiltro: [
      { value: 2, field: "Todos" },
      { value: 0, field: "Ativas" },
      { value: 1, field: "Concluídas" },
    ],

    statusTask: ["Ativa", "Concluída"],

    listTask: {
      cabecalho: [
        { text: "ID ", value: "id", sortable: true },
        { text: "Nome do responsável", value: "name", sortable: false },
        { text: "Descrição", value: "descricao", sortable: false },
        { text: "Status", value: "done", sortable: true },
        { text: "", value: "operacoes", sortable: false, align: "end" },
      ],
      items: [],
      porPagina: 10,
      carregando: false,
    },
  }),

  async mounted() {
    await this.carregarTask();
  },

  methods: {

    async filtrarGrid(){
      await this.carregarTask();
      if(this.filtro != 2){
        this.listTask.items = this.listTask.items.filter(x => x.done == this.filtro) 
      }
    },

    async carregarTask() {
      this.listTask.carregando = true;
      await axios
        .get(`https://6415ac7e351c4aed490c5c4f.mockapi.io/api/v1/tasks`)
        .then((res) => {
          console.log(res);
          this.listTask.items = res.data;

          this.listTask.porPagina = res.data.length;
          this.listTask.carregando = false;
        })
        .catch((error) => {
          console.log(error);
          this.listTask.carregando = false;
        });
    },

    abrirEditarTask(item) {
      this.dialogTask = true;
      this.task = item;
    },

    editarTask() {
      axios
        .put(
          `https://6415ac7e351c4aed490c5c4f.mockapi.io/api/v1/tasks/${this.task.id}`,
          this.task
        )
        .then(() => {
          this.$swal({
            title: "Sucesso!!!",
            text: "Tarefa editada com sucesso.",
            icon: "success",
            confirmButtonText: "Ok",
            confirmButtonColor: "#4BB543",
            allowOutsideClick: true,
          });
          this.dialogTask = false;
          this.carregarTask();
        })
        .catch((error) => {
          console.log(error);
        });
    },

    deleteTask(item) {
      this.$swal({
        title: "Confirmação",
        html: `Deseja realmente excluir a Tarefa?`,
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "Sim",
        cancelButtonText: "Não",
        confirmButtonColor: "#4BB543",
        cancelButtonColor: "#d33",
        allowOutsideClick: false,
      }).then(async (result) => {
        if (result.isConfirmed) {
          await axios
            .delete(
              `https://6415ac7e351c4aed490c5c4f.mockapi.io/api/v1/tasks/${item.id}`
            )
            .then(() => {
              this.$swal({
                title: "Sucesso!!!",
                text: "Tarefa excluída com sucesso.",
                icon: "success",
                confirmButtonText: "Ok",
                confirmButtonColor: "#4BB543",
                allowOutsideClick: true,
              });
              this.carregarTask();
            });
        }
      });
    },
  },
};
</script>

<style scoped>
.content {
  overflow-y: hidden;
}

.alignmentDialog {
  display: flex;
  justify-content: flex-end;
}
.operacoes {
  cursor: pointer;
  justify-content: flex-end;
  padding-right: 8px;
}
.operacoes::after {
  background-color: transparent !important;
}
.v-icon.v-icon::after {
  display: none;
}
.v-dialog {
  overflow-y: unset;
}
.v-text-field__details {
  display: none;
}
</style>
