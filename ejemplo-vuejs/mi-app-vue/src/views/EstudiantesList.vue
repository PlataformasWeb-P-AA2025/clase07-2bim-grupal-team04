<template>
  <div class="estudiantes-list-container">
    <h2>Listado de Estudiantes</h2>
    <p v-if="loading">Cargando estudiantes...</p>
    <p v-if="error" class="error-message">{{ error }}</p>
    <ul v-else-if="estudiantes.length">
      <li
        v-for="estudiante in estudiantes"
        :key="estudiante.url"
        class="estudiante-item"
      >
        <router-link
          :to="{
            name: 'EstudianteDetail',
            params: { estudianteUrl: estudiante.url },
          }"
        >
          {{ estudiante.nombre }} {{ estudiante.apellido }} (Cédula:
          {{ estudiante.cedula }})
        </router-link>

        <div class="actions">
          <router-link
            :to="{
              name: 'EstudianteEdit', // Necesitarás definir esta ruta en router/index.js
              params: { estudianteUrl: encodeURIComponent(estudiante.url) },
            }"
            class="edit-button"
          >
            Editar
          </router-link>

          <button @click="confirmDelete(estudiante)" class="delete-button">
            Eliminar
          </button>
        </div>
      </li>
    </ul>
    <p v-else>No hay estudiantes registrados.</p>
    <router-link to="/estudiantes/nuevo" class="add-button"
      >Agregar Nuevo Estudiante</router-link
    >
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudiantesList",
  data() {
    return {
      estudiantes: [],
      loading: true,
      error: null,
    };
  },
  async created() {
    await this.fetchEstudiantes();
  },
  methods: {
    async fetchEstudiantes() {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get("estudiantes/");
        this.estudiantes = response.data.results || response.data;
        console.log("Estudiantes cargados:", this.estudiantes);
      } catch (err) {
        console.error("Error al cargar estudiantes:", err.response || err);
        this.error =
          "No se pudieron cargar los estudiantes. Asegúrate de estar logueado.";
      } finally {
        this.loading = false;
      }
    },
    confirmDelete(estudiante) {
      if (
        confirm(
          `¿Estás seguro de que quieres eliminar a ${estudiante.nombre} ${estudiante.apellido}?`
        )
      ) {
        this.deleteEstudiante(estudiante.url);
      }
    },
    async deleteEstudiante(estudianteUrl) {
      try {
        // La URL completa ya viene en estudiante.url, la usamos directamente
        await api.delete(estudianteUrl);
        console.log("Estudiante eliminado:", estudianteUrl);
        // Actualizar la lista después de la eliminación exitosa
        this.estudiantes = this.estudiantes.filter(
          (est) => est.url !== estudianteUrl
        );
      } catch (err) {
        console.error("Error al eliminar estudiante:", err.response || err);
        this.error = "No se pudo eliminar el estudiante. Inténtalo de nuevo.";
      }
    },
  },
};
</script>

<style scoped>
.estudiantes-list-container {
  max-width: 800px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

ul {
  list-style: none;
  padding: 0;
}

.estudiante-item {
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.estudiante-item:last-child {
  border-bottom: none;
}

.estudiante-item a {
  text-decoration: none;
  color: #007bff;
  font-weight: bold;
}

.estudiante-item a:hover {
  text-decoration: underline;
}

.estudiante-item .estudiante-link {
  text-decoration: none;
  color: #007bff;
  font-weight: bold;
  flex-grow: 1; /* Permite que el enlace ocupe el espacio restante */
}

.estudiante-item .estudiante-link:hover {
  text-decoration: underline;
}

.actions {
  display: flex;
  gap: 10px; /* Espacio entre los botones */
}

.edit-button,
.delete-button {
  padding: 8px 12px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9em;
  text-decoration: none; /* Para el router-link como botón */
  display: inline-block; /* Para que el router-link se comporte como un botón */
  text-align: center;
}

.edit-button {
  background-color: #ffc107; /* Amarillo para editar */
  color: #333;
}

.edit-button:hover {
  background-color: #e0a800;
}

.delete-button {
  background-color: #dc3545; /* Rojo para eliminar */
  color: white;
}

.delete-button:hover {
  background-color: #c82333;
}

.add-button {
  display: block;
  width: fit-content;
  margin: 20px auto 0;
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border-radius: 5px;
  text-decoration: none;
  text-align: center;
  transition: background-color 0.3s ease;
}

.add-button:hover {
  background-color: #218838;
}

.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>
