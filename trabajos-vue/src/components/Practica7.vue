<template>
  <div>
    <form v-if="!isAuthenticated" @submit.prevent="onSubmit">
      <input required v-model="email" placeholder="Email" />
      <input required v-model="password" type="password" placeholder="Password" />
      <button type="submit">Access</button>
    </form>

    <div v-if="isAuthenticated">
      <p v-if="message">{{ message }} {{ email }}</p>
      <button @click="logout">Logout</button>

      <div class="content">
        <div class="movies-container" v-if="movies.length">
          <h2>Películas:</h2>
          <div class="container">
            <div class="tarjeta" 
                 v-for="movie in movies" 
                 :key="movie.id"
                 @click="fetchMovieDetails(movie.id)">
              <img
                v-if="movie.poster_path"
                :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
                :alt="movie.title"
                class="movie-poster"
              />
              <h3 class="titulo-pelicula">{{ movie.title }}</h3>
              <p class="clasificacion">Clasificación: {{ movie.vote_average }}</p>
            </div>
          </div>
        </div>
        
        <div class="detalles-peliculas" v-if="selectedMovie">
          <h2>Detalles de la Película:</h2>
          <h3>{{ selectedMovie.title }}</h3>
          <p>{{ selectedMovie.overview }}</p>
          <p><strong>Fecha de lanzamiento:</strong> {{ selectedMovie.release_date }}</p>
          <p><strong>Clasificación:</strong> {{ selectedMovie.vote_average }}</p>
          <img :src="`https://image.tmdb.org/t/p/w500${selectedMovie.poster_path}`" :alt="selectedMovie.title" />

          <div class="calificar-container">
            <h3>Calificar la Película</h3>
            <input type="number" v-model="userRating" min="1" max="10" placeholder="Ingresa una calificación (1-10)" />
            <button @click="calificarPelicula">Añadir Calificación</button>
            <button @click="borrarCalificacion">Eliminar Calificación</button>
          </div>
          <p v-if="ratedMovie">Tu calificación: {{ ratedMovie }}</p>
        </div>
      </div>
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
      movies: [],
      selectedMovie: null,
      sessionId: null,
      userRating: null,
      ratedMovie: null,
    };
  },

  created() {
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
      this.isAuthenticated = true;
      this.message = "Sesión recuperada";
      this.fetchMovies();
    }
  },

  methods: {
    async onSubmit() {
      try {
        const requestTokenResponse = await fetch('https://api.themoviedb.org/3/authentication/token/new?api_key=4e72051e3bc2c615ed21d74e9a55ac50');
        const requestTokenData = await requestTokenResponse.json();
        const requestToken = requestTokenData.request_token;

        const myHeaders = new Headers();
        myHeaders.append("Content-Type", "application/json");

        const raw = JSON.stringify({
          "username": this.email,
          "password": this.password,
          "request_token": requestToken
        });

        const requestOptions = {
          method: "POST",
          headers: myHeaders,
          body: raw,
        };

        const loginResponse = await fetch(`https://api.themoviedb.org/3/authentication/token/validate_with_login?api_key=4e72051e3bc2c615ed21d74e9a55ac50`, requestOptions);
        const loginData = await loginResponse.json();

        if (loginResponse.ok && loginData.success) {
          this.isAuthenticated = true;
          this.message = `Bienvenido, ${this.email}`;
          localStorage.setItem('user', JSON.stringify(this.email));

          const sessionResponse = await fetch(`https://api.themoviedb.org/3/authentication/session/new?api_key=4e72051e3bc2c615ed21d74e9a55ac50`, {
            method: 'POST',
            headers: myHeaders,
            body: JSON.stringify({ request_token: requestToken })
          });
          const sessionData = await sessionResponse.json();
          this.sessionId = sessionData.session_id;

          this.fetchMovies();
        } else {
          alert('Error en la autenticación: ' + (loginData.status_message || 'Credenciales inválidas'));
          this.message = '';
        }
      } catch (error) {
        console.error(error);
      }
    },

    async fetchMovies() {
      try {
        const response = await fetch('https://api.themoviedb.org/3/movie/popular?api_key=4e72051e3bc2c615ed21d74e9a55ac50');
        const data = await response.json();
        this.movies = data.results;
      } catch (error) {
        console.error('Error al obtener las películas:', error);
      }
    },

    async fetchMovieDetails(movieId) {
      try {
        const response = await fetch(`https://api.themoviedb.org/3/movie/${movieId}?api_key=4e72051e3bc2c615ed21d74e9a55ac50`);
        const data = await response.json();
        this.selectedMovie = data;

        await this.fetchUserRating(movieId);
      } catch (error) {
        console.error('Error al obtener los detalles de la película:', error);
      }
    },

    async fetchUserRating(movieId) {
      try {
        const response = await fetch(`https://api.themoviedb.org/3/movie/${movieId}/account_states?api_key=4e72051e3bc2c615ed21d74e9a55ac50&session_id=${this.sessionId}`);
        const data = await response.json();

        if (data.rated && typeof data.rated === 'object') {
          this.ratedMovie = data.rated.value;
        } else {
          this.ratedMovie = null;
        }
      } catch (error) {
        console.error('Error al obtener la calificación del usuario:', error);
      }
    },

    async calificarPelicula() {
      try {
        if (!this.userRating || this.userRating < 1 || this.userRating > 10) {
          alert('Por favor, ingresa una calificación válida (1-10).');
          return;
        }

        const response = await fetch(`https://api.themoviedb.org/3/movie/${this.selectedMovie.id}/rating?api_key=4e72051e3bc2c615ed21d74e9a55ac50&session_id=${this.sessionId}`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ value: this.userRating })
        });

        if (response.ok) {
          this.ratedMovie = this.userRating;
          alert('Calificación añadida con éxito.');
        } else {
          alert('No se pudo añadir la calificación.');
        }
      } catch (error) {
        console.error('Error al calificar la película:', error);
      }
    },

    async borrarCalificacion() {
      try {
        const response = await fetch(`https://api.themoviedb.org/3/movie/${this.selectedMovie.id}/rating?api_key=4e72051e3bc2c615ed21d74e9a55ac50&session_id=${this.sessionId}`, {
          method: 'DELETE',
          headers: {
            'Content-Type': 'application/json'
          }
        });

        if (response.ok) {
          this.ratedMovie = null;
          alert('Calificación eliminada con éxito.');
        } else {
          alert('No se pudo eliminar la calificación.');
        }
      } catch (error) {
        console.error('Error al eliminar la calificación:', error);
      }
    },

    logout() {
      this.isAuthenticated = false;
      this.email = "";
      this.password = "";
      this.movies = [];
      this.message = '';
      this.sessionId = null;
      localStorage.removeItem('user');
    }
  }
};
</script>



<style>
.content {
  display: flex;
  align-items: flex-start; 
  gap: 20px; 
}

.movies-container {
  flex: 2; 
}

.container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}


.tarjeta {
  border: 1px solid #ccc;
  padding: 10px;
  width: 200px;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.tarjeta:hover {
  transform: scale(1.05);
}

.movie-poster {
  width: 100%;
  height: auto;
}

.titulo-pelicula {
  font-size: 16px;
  font-weight: bold;
}

.clasificacion {
  font-size: 14px;
  color: #555;
}

.detalles-peliculas{
  flex: 3; /* Menos ancho */
  border: 1px solid #ccc;
  padding: 20px;
  border-radius: 5px;
  background-color: #000000;
}

.detalles-peliculas img {
  max-width: 100%;
  height: auto;
  margin-top: 10px;
}

.detalles-peliculas h3 {
  margin-top: 0;
}

</style>
