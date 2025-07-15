<template>
  <div class="estudiante-edit-container">
    <h2>Editar Estudiante</h2>
    <p v-if="loading">Cargando datos del estudiante...</p>
    <p v-if="error" class="error-message">{{ error }}</p>
    <form @submit.prevent="saveEstudiante" v-else-if="estudiante">
      <div class="form-group">
        <label for="nombre">Nombre:</label>
        <input type="text" id="nombre" v-model="estudiante.nombre" required />
      </div>
      <div class="form-group">
        <label for="apellido">Apellido:</label>
        <input
          type="text"
          id="apellido"
          v-model="estudiante.apellido"
          required
        />
      </div>
      <div class="form-group">
        <label for="cedula">Cédula:</label>
        <input type="text" id="cedula" v-model="estudiante.cedula" required />
      </div>
      <div class="form-group">
        <label for="correo">Correo:</label>
        <input type="email" id="correo" v-model="estudiante.correo" required />
      </div>

      <div class="form-actions">
        <button type="submit" class="save-button">Guardar Cambios</button>
        <button
          type="button"
          @click="cancelEdit"
          class="cancel-button"
        >
          Cancelar
        </button>
      </div>
    </form>
    <p v-else>Estudiante no encontrado para edición.</p>
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudianteEdit",
  props: ["estudianteUrl"], // Recibe la URL del estudiante a editar
  data() {
    return {
      estudiante: null,
      loading: true,
      error: null,
      originalEstudiante: null, // Para revertir cambios si se cancela
    };
  },
  async created() {
    const decodedUrl = decodeURIComponent(this.estudianteUrl);
    await this.fetchEstudiante(decodedUrl);
  },
  methods: {
    async fetchEstudiante(url) {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get(url);
        this.estudiante = response.data;
        // Guardar una copia original para el caso de "Cancelar"
        this.originalEstudiante = { ...response.data };
      } catch (err) {
        console.error("Error al cargar estudiante para edición:", err.response || err);
        this.error = "No se pudo cargar el estudiante para editar.";
        this.estudiante = null; // Para que no se muestre el formulario vacío
      } finally {
        this.loading = false;
      }
    },
    async saveEstudiante() {
      try {
        this.error = null;
        // Enviamos una solicitud PUT o PATCH. PUT reemplaza el recurso, PATCH actualiza parcialmente.
        // Para formularios de edición, PATCH es a menudo más robusto para enviar solo los campos que cambiaron.
        // Sin embargo, para una edición completa, PUT es válido. Asegúrate de que tu API de Django soporte PUT/PATCH.
        await api.put(this.estudiante.url, this.estudiante);
        console.log("Estudiante actualizado exitosamente:", this.estudiante);
        // Redirigir de vuelta a la lista o al detalle después de guardar
        this.$router.push({ name: 'EstudianteDetail', params: { estudianteUrl: encodeURIComponent(this.estudiante.url) } });
      } catch (err) {
        console.error("Error al guardar estudiante:", err.response || err);
        this.error = "No se pudieron guardar los cambios. Revisa los datos.";
        // Puedes agregar lógica para mostrar errores de validación específicos de la API
      }
    },
    cancelEdit() {
      // Revertir a los datos originales si se cancela la edición
      this.estudiante = { ...this.originalEstudiante };
      // Redirigir de vuelta al detalle o a la lista
      this.$router.push({ name: 'EstudianteDetail', params: { estudianteUrl: encodeURIComponent(this.estudiante.url) } });
    },
  },
};
</script>

<style scoped>
.estudiante-edit-container {
  max-width: 600px;
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

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box; /* Asegura que el padding no aumente el ancho total */
}

.form-actions {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 20px;
}

.save-button,
.cancel-button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
}

.save-button {
  background-color: #007bff;
  color: white;
}

.save-button:hover {
  background-color: #0056b3;
}

.cancel-button {
  background-color: #6c757d;
  color: white;
}

.cancel-button:hover {
  background-color: #5a6268;
}

.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>