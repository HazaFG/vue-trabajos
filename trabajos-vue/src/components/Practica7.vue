<template>
  <div>
    <form v-if="!isAuthenticated" @submit.prevent="onSubmit">
      <input required v-model="email" placeholder="Email"/>
      <input required v-model="password" type="password" placeholder="Password"/>
      <button type="submit">Access</button>
    </form>

    <div v-if="isAuthenticated">

      <p v-if="message">{{ message }} {{ email }}</p>
      <button @click="logout">Logout</button>

      <table v-if="!isEditing && !isVisible">
        <thead>
          <tr>
            <th>Email</th>
            <th>Password</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(user, index) in users" :key="user.email">
            <td>{{ user.email }}</td>
            <td>{{ user.password }}</td>
            <td>
              <button @click="startEdit(index)">Edit</button>
              <button @click="remove(index)">Remove</button>
            </td>
          </tr>
        </tbody>
      </table>

      <div v-if="!isEditing && !isVisible">
        <button @click="isVisible=true">Add new user</button>
      </div>

      <form v-if="isVisible" @submit.prevent="addUser">
        <input required v-model="add_email" placeholder="New email"/>
        <input required v-model="add_password" type="password" placeholder="New password"/>
        <button type="submit">Add</button>
        <button @click="cancel" type="button">Cancel</button>
      </form>

      <form v-if="isEditing" @submit.prevent="updateUser">
        <input required v-model="edit_email" placeholder="Edit email"/>
        <input required v-model="edit_password" type="password" placeholder="Edit password"/>
        <button type="submit">Update</button>
        <button @click="cancelEdit" type="button">Cancel</button>
      </form>
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      email: '',
      password: '',
      message: '',
      isAuthenticated: false,
      users: [],
      isVisible: false,
      add_email: '',
      add_password: '',
      isEditing: false,
      editIndex: null,
      edit_email: '',
      edit_password: '',
    };
  },

  created() {
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
      this.isAuthenticated = true;
      this.users = JSON.parse(storedUser);
      this.message = "SESION GUARDADA!!!";
    }
  },

  methods: {
    async onSubmit() {
      try {
        const response = await fetch('users.json');
        const users = await response.json();

        const validUser = users.some(user =>
          user.email === this.email && user.password === this.password
        );

        if (validUser) {
          this.message = "Bienvenido";
          this.isAuthenticated = true;
          this.users = users;
          localStorage.setItem('user', JSON.stringify(users));
        } else {
          alert("No existe el usuario");
          this.message = '';
        }
      } catch (error) {
        console.error(error);
      }
    },

    logout() {
      localStorage.clear();
      this.isAuthenticated = false;
      this.email = '';
      this.password = '';
    },

    cancel() {
      this.isVisible = false;
      this.add_email = "";
      this.add_password = "";
    },

    startEdit(index) {
      this.editIndex = index;
      this.edit_email = this.users[index].email;
      this.edit_password = this.users[index].password;
      this.isEditing = true;
    },

    updateUser() {
      this.users[this.editIndex].email = this.edit_email;
      this.users[this.editIndex].password = this.edit_password;
      localStorage.setItem('user', JSON.stringify(this.users));
      this.isEditing = false;
      this.edit_email = '';
      this.edit_password = '';
      this.message = 'Usuario actualizado correctamente';
    },

    cancelEdit() {
      this.isEditing = false;
      this.edit_email = '';
      this.edit_password = '';
    },

    remove(index) {
      this.users.splice(index, 1);
      localStorage.setItem('user', JSON.stringify(this.users));
      this.message = 'Usuario eliminado correctamente';
    },

    addUser() {
      const newUser = {
        email: this.add_email,
        password: this.add_password
      };

      this.users.push(newUser);
      this.add_email = '';
      this.add_password = ''; 
      this.isVisible = false; 
      this.message = 'Usuario agregado correctamente'; 

      localStorage.setItem('user', JSON.stringify(this.users));
    }
  }
};
</script>

<style>
table {
  background-color: rgb(5, 0, 1);
}

th {
  background-color: rgb(234, 78, 0);
}
</style>
