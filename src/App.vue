<template>
  <div class="tabla-container">
    <h2>Selecciona una categoría</h2>
    <button @click="cargarDatos('clanes')" class="btn">Ver Clanes</button>
    <button @click="cargarDatos('criaturas')" class="btn">Ver Criaturas</button>

    <div v-if="vista">
      <h3>{{ titulo }}</h3>
      <table>
        <thead>
          <tr>
            <th v-for="(col, index) in encabezado" :key="index">{{ col }}</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in datos" :key="index">
            <td v-for="clave in claves" :key="clave">{{ item[clave] }}</td>
            <td>
              <button @click="editar(item.id)" class="btn">Editar</button>
              <button @click="eliminar(item.id)" class="btn btn-danger">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>

      <form @submit.prevent="guardar" class="form-crud">
        <h4>{{ editando ? 'Editar' : 'Agregar' }} {{ titulo }}</h4>
        <input v-model="form.nombre" placeholder="Nombre" required />
        <input v-if="vista === 'clanes'" v-model="form.region" placeholder="Región" required />
        <input v-if="vista === 'clanes'" v-model="form.lider" placeholder="Líder" required />
        <input v-if="vista === 'criaturas'" v-model="form.tipo" placeholder="Tipo" required />
        <input v-if="vista === 'criaturas'" v-model="form.habitat" placeholder="Hábitat" required />
        <button type="submit" class="btn">{{ editando ? 'Actualizar' : 'Agregar' }}</button>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      vista: null,
      titulo: '',
      encabezado: [],
      claves: [],
      datos: [],
      form: {
        id: null,
        nombre: '',
        region: '',
        lider: '',
        tipo: '',
        habitat: ''
      },
      editando: false,
    };
  },
  methods: {
    // Cargar los datos según la categoría seleccionada
    async cargarDatos(tipo) {
      try {
        this.vista = tipo;
        if (tipo === 'clanes') {
          this.titulo = "Clanes";
          this.encabezado = ['ID', 'Nombre', 'Región', 'Líder'];
          this.claves = ['id', 'nombre', 'region', 'lider'];
        } else {
          this.titulo = "Criaturas";
          this.encabezado = ['ID', 'Nombre', 'Tipo', 'Hábitat'];
          this.claves = ['id', 'nombre', 'tipo', 'habitat'];
        }

        const res = await axios.get(`http://localhost:3000/api/${tipo}`);
        this.datos = res.data;
        this.resetForm();
      } catch (error) {
        console.error('Error al obtener datos:', error);
      }
    },

    // Resetear el formulario
    resetForm() {
      this.form = {
        id: null,
        nombre: '',
        region: '',
        lider: '',
        tipo: '',
        habitat: ''
      };
      this.editando = false;
    },

    // Editar un item
    editar(id) {
      const item = this.datos.find(item => item.id === id);
      if (item) {
        this.form = { ...item };
        this.editando = true;
      }
    },

    // Eliminar un item
    async eliminar(id) {
      try {
        await axios.delete(`http://localhost:3000/api/${this.vista}/${id}`);
        this.cargarDatos(this.vista); // Refrescar los datos después de eliminar
      } catch (error) {
        console.error('Error al eliminar:', error);
      }
    },

    // Guardar un item (crear o actualizar)
    async guardar() {
      try {
        const { nombre, region, lider, tipo, habitat, id } = this.form;
        if (this.editando) {
          // Actualizar
          await axios.put(`http://localhost:3000/api/${this.vista}/${id}`, { nombre, region, lider, tipo, habitat });
          this.editando = false;
        } else {
          // Crear
          await axios.post(`http://localhost:3000/api/${this.vista}`, { nombre, region, lider, tipo, habitat });
        }
        this.cargarDatos(this.vista); // Refrescar los datos después de guardar
        this.resetForm();
      } catch (error) {
        console.error('Error al guardar:', error);
      }
    }
  },

  // Cargar los datos por defecto cuando se monta el componente
  mounted() {
    this.cargarDatos('clanes'); // Puedes poner 'criaturas' si prefieres cargar esas por defecto
  }
};
</script>

<style scoped>
.tabla-container {
  text-align: center;
  margin: 20px;
}

.btn {
  margin: 10px;
  padding: 10px 15px;
  border: none;
  background: #007bff;
  color: white;
  cursor: pointer;
  border-radius: 5px;
}

.btn:hover {
  background: #0056b3;
}

.btn-danger {
  background: #dc3545;
}

.btn-danger:hover {
  background: #c82333;
}

table {
  width: 100%;
  margin-top: 20px;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background: #007bff;
  color: white;
}

form {
  margin-top: 20px;
}

input {
  padding: 10px;
  margin: 5px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

form h4 {
  margin-bottom: 10px;
}

form button {
  margin-top: 10px;
  padding: 10px 20px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
}

form button:hover {
  background: #218838;
}
</style>
