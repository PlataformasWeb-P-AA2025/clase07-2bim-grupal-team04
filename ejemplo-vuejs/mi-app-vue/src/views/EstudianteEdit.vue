<template>
  <div class="estudiante-edit-container">
    <h2>Editar Estudiante y Teléfonos</h2>
    <p v-if="loading">Cargando datos del estudiante...</p>
    <p v-if="error" class="error-message">{{ error }}</p>

    <form @submit.prevent="saveEstudiante" v-else-if="estudiante">
      <fieldset class="form-section">
        <legend>Datos del Estudiante</legend>
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
          <input
            type="email"
            id="correo"
            v-model="estudiante.correo"
            required
          />
        </div>
      </fieldset>

      <hr />
      <fieldset class="form-section">
        <legend>Números Telefónicos</legend>
        <div
          v-for="(numero, index) in numerosTelefonicos"
          :key="numero.url || 'new-' + index"
          class="telefono-item-edit"
        >
          <input v-model="numero.telefono" placeholder="Número" required />
          <select v-model="numero.tipo" required>
            <option disabled value="">Seleccione un tipo</option>
            <option value="público">Público</option>
            <option value="semiprivado">Semiprivado</option>
            <option value="particular">Particular</option>
          </select>
          <button
            type="button"
            @click="removeTelefono(index, numero)"
            class="remove-telefono-button"
          >
            Eliminar
          </button>
        </div>

        <button type="button" @click="addTelefono" class="add-telefono-button">
          Agregar Nuevo Teléfono
        </button>
        <p v-if="!numerosTelefonicos.length" class="no-telefonos-message">
          No hay números telefónicos registrados. Agrega uno.
        </p>
      </fieldset>

      <div class="form-actions">
        <button type="submit" class="save-button">
          Guardar Todos los Cambios
        </button>
        <button type="button" @click="cancelEdit" class="cancel-button">
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
  props: ["estudianteUrl"],
  data() {
    return {
      estudiante: null,
      numerosTelefonicos: [],
      originalEstudiante: null,
      originalNumerosTelefonicos: [],
      loading: true,
      error: null,
    };
  },
  async created() {
    const decodedUrl = decodeURIComponent(this.estudianteUrl);
    await this.fetchEstudianteAndPhones(decodedUrl);
  },
  methods: {
    async fetchEstudianteAndPhones(url) {
      try {
        this.loading = true;
        this.error = null;

        const estudianteResponse = await api.get(url);
        this.estudiante = estudianteResponse.data;
        this.originalEstudiante = { ...estudianteResponse.data };

        const telefonosResponse = await api.get("numerosts/");
        this.numerosTelefonicos = telefonosResponse.data.results
          .filter((numero) => numero.estudiante === url)
          .map((numero) => ({ ...numero }));

        this.originalNumerosTelefonicos = this.numerosTelefonicos.map((n) => ({
          ...n,
        }));

        console.log("Estudiante cargado:", this.estudiante);
        console.log("Números telefónicos cargados:", this.numerosTelefonicos);
      } catch (err) {
        console.error(
          "Error al cargar estudiante o teléfonos:",
          err.response || err
        );
        this.error = "No se pudieron cargar los datos para editar.";
        this.estudiante = null;
      } finally {
        this.loading = false;
      }
    },

    addTelefono() {
      this.numerosTelefonicos.push({
        telefono: "",
        tipo: "", // El valor inicial será vacío, forzando la selección
        estudiante: this.estudiante.url,
        isNew: true,
      });
    },

    removeTelefono(indexToRemove, numero) {
      if (
        confirm(
          `¿Estás seguro de que quieres eliminar el número ${
            numero.telefono || "vacío"
          }?`
        )
      ) {
        this.numerosTelefonicos.splice(indexToRemove, 1);
        console.log("Número eliminado localmente:", numero);
      }
    },

    async saveEstudiante() {
      try {
        this.error = null;
        let promises = [];

        if (
          JSON.stringify(this.estudiante) !==
          JSON.stringify(this.originalEstudiante)
        ) {
          promises.push(api.put(this.estudiante.url, this.estudiante));
          console.log("Actualizando datos del estudiante...");
        }

        for (const originalNum of this.originalNumerosTelefonicos) {
          const found = this.numerosTelefonicos.find(
            (currentNum) => currentNum.url === originalNum.url
          );
          if (!found) {
            promises.push(api.delete(originalNum.url));
            console.log("Eliminando número telefónico:", originalNum.telefono);
          }
        }

        for (const currentNum of this.numerosTelefonicos) {
          if (currentNum.isNew) {
            const payload = {
              telefono: currentNum.telefono,
              tipo: currentNum.tipo,
              estudiante: this.estudiante.url,
            };
            promises.push(api.post("numerosts/", payload));
            console.log("Creando nuevo número:", currentNum.telefono);
          } else {
            const originalData = this.originalNumerosTelefonicos.find(
              (n) => n.url === currentNum.url
            );

            if (
              originalData &&
              (originalData.telefono !== currentNum.telefono ||
                originalData.tipo !== currentNum.tipo)
            ) {
              const payload = {
                telefono: currentNum.telefono,
                tipo: currentNum.tipo,
                estudiante: this.estudiante.url,
              };
              promises.push(api.put(currentNum.url, payload));
              console.log(
                "Actualizando número telefónico:",
                currentNum.telefono
              );
            }
          }
        }

        await Promise.all(promises);

        console.log("Todos los cambios guardados exitosamente.");
        this.$router.push({
          name: "EstudianteDetail",
          params: { estudianteUrl: encodeURIComponent(this.estudiante.url) },
        });
      } catch (err) {
        console.error("Error al guardar cambios:", err.response || err);
        this.error =
          "No se pudieron guardar los cambios. Revisa los datos y la consola para más detalles.";
        if (err.response && err.response.data) {
          this.error += ` (${JSON.stringify(err.response.data)})`;
        }
      }
    },

    cancelEdit() {
      this.estudiante = { ...this.originalEstudiante };
      this.$router.push({
        name: "EstudianteDetail",
        params: { estudianteUrl: encodeURIComponent(this.estudiante.url) },
      });
    },
  },
};
</script>

<style scoped>
.estudiante-edit-container {
  max-width: 700px;
  margin: 50px auto;
  padding: 25px;
  border: 1px solid #ddd;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  background-color: #f9f9f9;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.form-section {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 25px;
  background-color: #ffffff;
}

.form-section legend {
  font-size: 1.2em;
  font-weight: bold;
  color: #555;
  padding: 0 10px;
  margin-left: -10px; /* Alinea con el borde del fieldset */
}

.form-group {
  margin-bottom: 18px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: bold;
  color: #444;
}

.form-group input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ccc;
  border-radius: 6px;
  box-sizing: border-box;
  font-size: 1em;
}

hr {
  border: none;
  border-top: 1px solid #eee;
  margin: 30px 0;
}

.telefono-item-edit {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-bottom: 15px;
  background-color: #f0f0f0;
  padding: 10px 15px;
  border-radius: 6px;
  border: 1px solid #e0e0e0;
}

.telefono-item-edit input,
.telefono-item-edit select {
  /* Añadido select aquí */
  flex: 1;
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.remove-telefono-button {
  background-color: #dc3545;
  color: white;
  padding: 8px 12px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9em;
  transition: background-color 0.3s ease;
}

.remove-telefono-button:hover {
  background-color: #c82333;
}

.add-telefono-button {
  display: block;
  width: fit-content;
  margin: 20px 0 0 auto;
  padding: 10px 18px;
  background-color: #17a2b8;
  color: white;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  font-size: 1em;
  transition: background-color 0.3s ease;
}

.add-telefono-button:hover {
  background-color: #138496;
}

.no-telefonos-message {
  text-align: center;
  color: #666;
  margin-top: 15px;
  font-style: italic;
}

.form-actions {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-top: 30px;
}

.save-button,
.cancel-button {
  padding: 12px 25px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1.1em;
  font-weight: bold;
  transition: background-color 0.3s ease;
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
  margin-bottom: 20px;
  padding: 10px;
  background-color: #ffebeb;
  border: 1px solid #ffccca;
  border-radius: 5px;
}
</style>
