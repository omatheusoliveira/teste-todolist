<template>
  <div>
    <v-card>
      <v-toolbar flat>
        <v-toolbar-title class="pl-4"> Cadastrar Tarefa </v-toolbar-title>
        <v-spacer></v-spacer>
      </v-toolbar>
      <v-divider></v-divider>
      <v-card-text>
        <v-row class="px-9 py-7">
          <v-card style="width: 100%; heigth: calc(80vh - 100px)">
            <v-container>
              <v-row>
                <v-col cols="12" sm="12" md="12">
                  <v-select
                    :items="users"
                    label="Atribuir a usuários"
                    v-model="task.user_id"
                    item-text="name"
                    item-value="id"
                    dense
                  ></v-select>
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12" sm="12" md="12">
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
              <v-btn class="ma-2" color="error"> Cancelar </v-btn>
              <v-btn class="ma-2" color="primary" @click="addTask()">
                Salvar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-row>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "CadastrarTask",

  data: () => ({
    loading: false,

    task: {
      descricao: "",
      status: 0,
    },

    status: [{value: 0, field: 'Ativo'}],

    users: [],

  }),

  async mounted() {
    await this.carregarUser();
  },

  methods: {
    async carregarUser() {
      await axios
        .get(`https://6415ac7e351c4aed490c5c4f.mockapi.io/api/v1/users`)
        .then((res) => {
          this.users = res.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },

    addTask() {
      axios
        .post(`https://6415ac7e351c4aed490c5c4f.mockapi.io/api/v1/tasks`, this.task)
        .then(() => {
          this.$swal({
            title: "Sucesso!!!",
            text: "Tarefa criada com sucesso.",
            icon: "success",
            confirmButtonText: "Ok",
            confirmButtonColor: "#4BB543",
            allowOutsideClick: true,
          });
          this.task = {
            description: "",
            status: null,
          };
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>
<style scoped>
.alignmentDialog {
  display: flex;
  justify-content: flex-end;
}
.alinhamento {
  padding-bottom: 0 !important;
  padding-top: 25px !important;
}
</style>
