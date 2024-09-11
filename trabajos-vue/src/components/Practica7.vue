<template>
  <div>
    <h2>Arreglo de personas con lista</h2>
    
    <!-- Formulario para nombre y apellido -->
    <form @submit.prevent="agregarPersona">
      <input v-model="nombre" placeholder="Nombre" />
      <input v-model="apellido" placeholder="Apellido" />
      <button type="submit">Save data</button>
    </form>

    <!-- Lista de personas -->
    <ul>
      <li v-for="(persona, index) in personas" :key="index">
        {{ persona.nombre }} {{ persona.apellido }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      nombre: '',      
      apellido: '', 
      personas: []
    };
  },
  methods: {
    agregarPersona() {
      if (this.nombre.trim() !== '' && this.apellido.trim() !== '') {

        // Agregar la persona al arreglo
        this.personas.push({ nombre: this.nombre, apellido: this.apellido });
      }
    }
  }
};
</script>

<style scoped>

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin-bottom: 5px;
  padding: 10px;
}

</style>
