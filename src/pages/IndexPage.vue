<template>
  <q-page class="q-pa-md">
    <q-card>
      <q-card-section class="row justify-between">
        <div class="text-h6">Lista de Tarefas</div>
        <q-btn color="primary" label="Criar Tarefa" @click="openDialog()" />
      </q-card-section>

      <q-separator />

      <q-card-section>
        <q-table
          :rows="posts"
          :columns="columns"
          row-key="id"
          dense
          :pagination="{ rowsPerPage: 10 }"
          separator="cell"
        >
          <template v-slot:body-cell-actions="props">
            <q-td :props="props">
              <q-btn color="green" size="sm" icon="edit" flat dense @click="openDialog(props.row)" />
              <q-btn color="red" size="sm" icon="delete" flat dense @click="deletePost(props.row.id)" />
              <q-btn
                :color="props.row.completed ? 'blue' : 'grey'"
                size="sm"
                icon="check"
                flat dense
                @click="toggleCompleted(props.row)"
              />
            </q-td>
          </template>
        </q-table>
      </q-card-section>
    </q-card>

    <!-- Modal de Criar/Editar Tarefa -->
    <q-dialog v-model="dialogVisible">
      <q-card style="width: 400px">
        <q-card-section>
          <div class="text-h6">{{ isEditing ? "Editar Tarefa" : "Criar Tarefa" }}</div>
        </q-card-section>
        <q-card-section>
          <q-input v-model="form.name" label="Nome da Tarefa" dense filled />
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancelar" color="grey" v-close-popup />
          <q-btn flat label="Salvar" color="primary" @click="savePost" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import { ref, onMounted } from "vue";
import axios from "axios";

export default {
  setup() {
    const posts = ref([]);
    const dialogVisible = ref(false);
    const isEditing = ref(false);
    const form = ref({ id: null, name: "", completed: false });

    const columns = [
      { name: "id", label: "ID", align: "left", field: "id", sortable: true },
      { name: "name", label: "Nome", align: "left", field: "name", sortable: true },
      { name: "completed", label: "Concluído", align: "center", field: "completed", sortable: true },
      { name: "actions", label: "Ações", align: "left", field: "actions" }
    ];

    const fetchData = async () => {
      try {
        const response = await axios.get("http://localhost/tasks", {
          withCredentials: true
        });
        posts.value = response.data.map(task => ({
          ...task,
          completed: Boolean(task.completed)
        }));
      } catch (error) {
        console.error("Erro ao buscar dados:", error);
      }
    };

    const openDialog = (post = null) => {
      if (post) {
        form.value = { ...post, completed: Boolean(post.completed) };
        isEditing.value = true;
      } else {
        form.value = { id: null, name: "", completed: false };
        isEditing.value = false;
      }
      dialogVisible.value = true;
    };

    // const savePost = async () => {
    //  try {
    //     await axios.put(`http://localhost/tasks/update/${form.value.id}`, { withCredentials: true });

    //     fetchData();
    //     dialogVisible.value = false;
    //   } catch (error) {
    //     console.error("Erro ao salvar tarefa:", error);
    //   }
    // };

    const savePost = async () => {
  try {
    if (isEditing.value) {
      // Editando uma tarefa existente (PUT)
      await axios.put(`http://localhost/tasks/update/${form.value.id}`, null, {
        params: {
          name: form.value.name,
          completed: form.value.completed ? 1 : 0
        },
        withCredentials: true
      });
    } else {
      await axios.post("http://localhost/tasks/create", { 
  name: form.value.name 
}, {
  headers: { 
    "Content-Type": "application/json"
  },
  withCredentials: false 
});

    }

    fetchData(); 
    dialogVisible.value = false;
  } catch (error) {
    console.error("Erro ao salvar tarefa:", error.response ? error.response.data : error.message);
  }
};



    const deletePost = async (id) => {
      try {
        await axios.delete(`http://localhost/tasks/delete/${id}`, { withCredentials: true });
        fetchData();
      } catch (error) {
        console.error("Erro ao deletar tarefa:", error);
      }
    };

    const toggleCompleted = async (task) => {
  try {
    
    const updatedStatus = !task.completed;

    await axios.put(`http://localhost/tasks/toggle/${task.id}`, null, {
      params: {
        completed: updatedStatus, 
      },
      withCredentials: true
    });

    fetchData(); 
  } catch (error) {
    console.error("Erro ao alternar o status da tarefa:", error);
  }
};




    onMounted(fetchData);

    return {
      posts,
      columns,
      dialogVisible,
      form,
      isEditing,
      openDialog,
      savePost,
      deletePost,
      toggleCompleted
    };
  }
};
</script>