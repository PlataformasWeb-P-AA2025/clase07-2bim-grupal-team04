<template>
  <div class="estudiante-detail-container">
    <h2>Detalle del Estudiante</h2>
    <p v-if="loading">Cargando detalles...</p>
    <p v-if="error" class="error-message">{{ error }}</p>
    <div v-else-if="estudiante">
      <h4>Números Telefónicos:</h4>
      <ul v-if="numerosTelefonicos.length">
        <li
          v-for="numero in numerosTelefonicos"
          :key="numero.url"
          class="telefono-item"
        >
          <div v-if="!numero.isEditing" class="telefono-display">
            {{ numero.telefono }} ({{ numero.tipo }})
            <div class="telefono-actions">
              <button
                @click="startEditTelefono(numero)"
                class="edit-telefono-button"
              >
                Editar
              </button>
              <button
                @click="confirmDeleteTelefono(numero)"
                class="delete-telefono-button"
              >
                Eliminar
              </button>
            </div>
          </div>
          <div v-else class="telefono-edit-form">
            <input v-model="numero.telefono" placeholder="Número" />
            <input
              v-model="numero.tipo"
              placeholder="Tipo (ej. Celular, Casa)"
            />
            <div class="telefono-actions">
              <button
                @click="saveTelefono(numero)"
                class="save-telefono-button"
              >
                Guardar
              </button>
              <button
                @click="cancelEditTelefono(numero)"
                class="cancel-telefono-button"
              >
                Cancelar
              </button>
            </div>
          </div>
        </li>
      </ul>
      <p v-else>No tiene números telefónicos registrados.</p>

      <h3>{{ estudiante.nombre }} {{ estudiante.apellido }}</h3>
      <p><strong>Cédula:</strong> {{ estudiante.cedula }}</p>
      <p><strong>Correo:</strong> {{ estudiante.correo }}</p>

      <router-link :to="{ name: 'EstudiantesList' }" class="back-button"
        >Volver al Listado</router-link
      >
    </div>
    <p v-else>Estudiante no encontrado.</p>
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudianteDetail",
  props: ["estudianteUrl"],
  data() {
    return {
      estudiante: null,
      numerosTelefonicos: [],
      loading: true,
      error: null,
      // Usaremos esto para almacenar una copia de los datos originales del teléfono al editar
      originalTelefono: null,
    };
  },
  async created() {
    const decodedUrl = decodeURIComponent(this.estudianteUrl);
    await this.fetchEstudianteDetail(decodedUrl);
    await this.fetchNumerosTelefonicos(decodedUrl);
  },
  methods: {
    async fetchEstudianteDetail(url) {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get(url);
        this.estudiante = response.data;
      } catch (err) {
        console.error(
          "Error al cargar detalle del estudiante:",
          err.response || err
        );
        this.error = "No se pudo cargar el detalle del estudiante.";
      } finally {
        this.loading = false;
      }
    },
    async fetchNumerosTelefonicos(estudianteApiUrl) {
      try {
        const response = await api.get("numerosts/"); // Asegúrate de que este endpoint devuelve todos los números
        // Filtramos los números telefónicos que pertenecen a este estudiante usando la URL completa
        // Añadimos 'isEditing' y 'originalData' para la gestión de estado de edición inline
        this.numerosTelefonicos = response.data.results
          .filter((numero) => numero.estudiante === estudianteApiUrl)
          .map((numero) => ({
            ...numero,
            isEditing: false,
            originalData: null,
          }));
      } catch (err) {
        console.error(
          "Error al cargar números telefónicos:",
          err.response || err
        );
        this.error = "No se pudieron cargar los números telefónicos.";
      }
    },

    // --- Métodos para Teléfonos ---
    startEditTelefono(numero) {
      // Guardar una copia de los datos originales del teléfono
      numero.originalData = { ...numero };
      numero.isEditing = true;
    },
    async saveTelefono(numero) {
      try {
        // Enviar una solicitud PUT/PATCH a la API de Django para actualizar el número
        // Asegúrate de que tu API de Django espera 'estudiante' como la URL del estudiante, no solo el ID
        const payload = {
          telefono: numero.telefono,
          tipo: numero.tipo,
          estudiante: numero.estudiante, // Asegurarse de enviar la URL completa del estudiante
        };
        await api.put(numero.url, payload); // Usar numero.url para PUT/PATCH
        numero.isEditing = false;
        numero.originalData = null; // Limpiar la copia original
        console.log("Número telefónico actualizado:", numero);
        // Si no se actualiza automáticamente, podrías volver a llamar a fetchNumerosTelefonicos()
      } catch (err) {
        console.error("Error al guardar teléfono:", err.response || err);
        alert("Error al guardar el número telefónico."); // Alerta simple de error
      }
    },
    cancelEditTelefono(numero) {
      // Revertir a los datos originales
      Object.assign(numero, numero.originalData);
      numero.isEditing = false;
      numero.originalData = null;
    },
    confirmDeleteTelefono(numero) {
      if (
        confirm(
          `¿Estás seguro de que quieres eliminar el número ${numero.telefono}?`
        )
      ) {
        this.deleteTelefono(numero.url);
      }
    },
    async deleteTelefono(telefonoUrl) {
      try {
        await api.delete(telefonoUrl); // Usar telefonoUrl para DELETE
        this.numerosTelefonicos = this.numerosTelefonicos.filter(
          (num) => num.url !== telefonoUrl
        );
        console.log("Número telefónico eliminado:", telefonoUrl);
      } catch (err) {
        console.error("Error al eliminar teléfono:", err.response || err);
        alert("Error al eliminar el número telefónico."); // Alerta simple de error
      }
    },
  },
};
</script>

<style scoped>
.estudiante-detail-container {
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  text-align: left;
}

h2,
h3,
h4 {
  text-align: center;
  color: #333;
  margin-bottom: 15px;
}

p {
  margin-bottom: 10px;
}

ul {
  list-style: none; /* Cambiado a none para mejor control del layout */
  padding-left: 0;
  margin-bottom: 20px;
}

.telefono-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px dashed #eee;
}

.telefono-item:last-child {
  border-bottom: none;
}

.telefono-display {
  flex-grow: 1;
}

.telefono-edit-form {
  display: flex;
  flex-grow: 1;
  gap: 10px;
  align-items: center;
}

.telefono-edit-form input {
  flex-grow: 1;
  padding: 5px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.telefono-actions {
  display: flex;
  gap: 8px;
}

.edit-telefono-button,
.delete-telefono-button,
.save-telefono-button,
.cancel-telefono-button {
  padding: 6px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.85em;
}

.edit-telefono-button {
  background-color: #ffc107;
  color: #333;
}

.edit-telefono-button:hover {
  background-color: #e0a800;
}

.delete-telefono-button {
  background-color: #dc3545;
  color: white;
}

.delete-telefono-button:hover {
  background-color: #c82333;
}

.save-telefono-button {
  background-color: #28a745;
  color: white;
}

.save-telefono-button:hover {
  background-color: #218838;
}

.cancel-telefono-button {
  background-color: #6c757d;
  color: white;
}

.cancel-telefono-button:hover {
  background-color: #5a6268;
}

.back-button {
  display: block;
  width: fit-content;
  margin: 20px auto 0;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  text-decoration: none;
  text-align: center;
  transition: background-color 0.3s ease;
}

.back-button:hover {
  background-color: #0056b3;
}

.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>
