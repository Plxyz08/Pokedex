<template>
  <div>
    <header class="header">
      <a href="#" class="home-link">
        <div class="logo-container">
          <h1 class="logo">Pokedex <img src="/src/assets/pokeball.png" alt="Pokeball" class="pokeball" />
          </h1>
        </div>
      </a>
      <div class="search-filter">
        <input v-model="busqueda" placeholder="Buscar por nombre" @input="filtrarPokemon" class="search-input" />
        <select v-model="filtro" @change="filtrarPorTipo" class="filter-select">
          <option value="">Todos los tipos</option>
          <option v-for="tipo in tiposFiltro" :key="tipo">{{ tipo }}</option>
        </select>
      </div>
    </header>
    <div class="pokemon-container">
      <div class="pokemon-card" v-for="pokemon in pokemonsFiltrados" :key="pokemon.id" @click="mostrarDetalles(pokemon)">
        <div class="pokemon-info">
          <p
            style="font-family: 'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif; color: rgb(255, 0, 255); font-size: 30px; font-weight:bold;">
            # {{ pokemon.numero }}</p>
          <p style="color: black;">{{ pokemon.nombre }}</p>
          <img :src="pokemon.imagen" alt="">
          <div class="tipos">
            <div v-for="(tipo, index) in pokemon.tipos" :key="index" :class="['tipo', tipo]">
              <p>{{ tipo }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <button @click="mostrarmas()" class="cargar-mas">Cargar más</button>

    <div v-if="mostrarModal" class="modal">
      <div class="modal-content modal-large">
        <button @click="cerrarModal" type="button" class="botonCerrar" data-bs-dismiss="modal" aria-label="Close">❌</button>
        <div class="pokemon-details">
          <div class="pokemon-info-left">
            <p class="numero-pokemon">#{{ pokemonSeleccionado.numero }}</p>
            <h2>{{ pokemonSeleccionado.nombre }}</h2>
            <div class="pokemon-image">
              <img :src="pokemonSeleccionado.imagen" alt="">
            </div>
            <p class="tipo-pokemon">Tipo:</p>
            <div v-for="(tipo, index) in pokemonSeleccionado.tipos" :key="index" :class="['tipo', tipo]">
              <p>{{ tipo }}</p>
            </div>
            <p class="altura-peso">Altura: {{ pokemonSeleccionado.altura / 10 }}m     -     Peso: {{ pokemonSeleccionado.peso / 10
            }}Kg</p>
          </div>
          <div class="pokemon-info-right">
            <div class="estadisticas">
              <p>Estadísticas:</p>
              <div v-for="(stat, index) in pokemonSeleccionado.estadisticas" :key="index">
                <p>{{ stat.nombre }}: {{ stat.valor }}</p>
                <div class="barra">
                  <div class="relleno" :style="{ width: (stat.valor * 0.3) + '%' }"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <button @click="cerrarModal">Cerrar</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref, onMounted, computed } from "vue";

export default {
  setup() {
    const tiposFiltro = [
      "Fire",
      "Grass",
      "Electric",
      "Water",
      "Ground",
      "Rock",
      "Fairy",
      "Poison",
      "Bug",
      "Dragon",
      "Psychic",
      "Flying",
      "Fighting",
    ];

    const pokemons = ref([]);
    const mostrarModal = ref(false);
    const pokemonSeleccionado = ref({});
    const pagina = ref(1);
    const offset = ref(0);

    onMounted(async () => {
      cargarMasTarjetas();
    });

    const mostrarmas = () => {
      offset.value += 50;
      cargarMasTarjetas();
    };

    async function cargarMasTarjetas() {
      try {
        let response = await axios.get(`https://pokeapi.co/api/v2/pokemon?limit=50&offset=${offset.value}`);
        const results = response.data.results;

        for (const result of results) {
          const pokemonData = await axios.get(result.url);
          const data = pokemonData.data;
          const estadisticas = data.stats.map(stat => ({
            nombre: stat.stat.name,
            valor: stat.base_stat,
          }));
          const pokemon = {
            id: data.id,
            imagen: data.sprites.other['official-artwork'].front_default,
            numero: data.id,
            nombre: data.name,
            tipos: data.types.map(type => type.type.name),
            altura: data.height,
            peso: data.weight,
            estadisticas: estadisticas,
          };
          pokemons.value.push(pokemon);
        }

        pagina.value++;
      } catch (error) {
        console.error(error);
      }
    }

    const mostrarDetalles = (pokemon) => {
      pokemonSeleccionado.value = pokemon;
      mostrarModal.value = true;
    };

    const cerrarModal = () => {
      mostrarModal.value = false;
    };

    const busqueda = ref("");
    const filtro = ref("");

    const pokemonsFiltrados = computed(() => {
      let filtered = pokemons.value;

      if (busqueda.value.trim() !== "") {
        filtered = filtered.filter(pokemon =>
          pokemon.nombre.toLowerCase().includes(busqueda.value.trim().toLowerCase())
        );
      }

      if (filtro.value) {
        filtered = filtered.filter(pokemon =>
          pokemon.tipos.includes(filtro.value.toLowerCase())
        );
      }

      return filtered;
    });

    return {
      tiposFiltro,
      pokemons,
      mostrarModal,
      pokemonSeleccionado,
      busqueda,
      filtro,
      mostrarmas,
      cargarMasTarjetas,
      mostrarDetalles,
      cerrarModal,
      pokemonsFiltrados,
    };
  },
};
</script>


<style scoped>
.header {
  position: fixed;
  top: 0;
  left: 0;
  width: 99%;
  background-color: #ff5555;
  color: #fff;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  z-index: 2;
  border-bottom-right-radius: 15px;
  border-bottom-left-radius: 15px;
}

.home-link {
  color: #fff;
  text-decoration: none;
}

.logo {
  font-size: 40px;
  margin-left: 30px;
}

.search-filter {
  display: flex;
  align-items: center;
}

.search-input {
  padding: 8px;
  margin-right: 10px;
  border-radius: 5px;
  border: none;
  font-size: 14px;
  background-color: aliceblue;
  color: #636363;
}

.filter-select {
  padding: 8px;
  border-radius: 5px;
  font-size: 14px;
  background-color: aliceblue;
  color: #636363;
  margin-right: 10px;
}

.filter-select:focus,
.search-input:focus {
  outline: none;
}

.cargar-mas {
  background-color: #23b6b6;
  color: #fff;
  border: none;
  padding: 8px 16px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.cargar-mas:hover {
  background-color: #1c8a8a;
}
.pokeball {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}

.pokemon-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: flex-start;
  margin-top: 150px;
}

.pokemon-card {
  width: 240px;
  border: 3px solid #a1a1a1;
  padding: 10px;
  margin: 10px;
  text-align: center;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: rgb(228, 228, 228);
  box-shadow: 6px 11px 31px -8px rgba(148, 148, 148, 1);
  margin: 2%;
  cursor: pointer;
  transition: background-color 0.3s;
}

.pokemon-card:hover {
  background-color: #67c2ec;
}

.pokemon-info {
  flex: 1;
}

img {
  max-width: 100%;
}

button {
  margin-top: 10px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
}

.modal-content {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5);
  color: #535353;
}

.pokemon-details img {
  max-width: 200px;
}

.tipo {
  margin: 0%;
  padding: 3px 20px;
  margin-bottom: 5px;
  border-radius: 10px;
  text-align: center;
  font-weight: bold;
  text-transform: capitalize;
  font-size: 12px;
  max-width: 50px;
}

.tipos {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  justify-content: center;
}



.tipo.water {
  background-color: hsla(240, 100%, 50%, 0.501);
}

.tipo.fire {
  background-color: rgba(255, 166, 0, 0.497);
}

.tipo.grass {
  background-color: rgba(0, 128, 0, 0.504);
}

.tipo.electric {
  background-color: rgba(255, 255, 0, 0.486);
}

.tipo.ice {
  background-color: rgba(0, 255, 255, 0.492);
}

.tipo.fighting {
  background-color: rgba(255, 0, 0, 0.504);
}

.tipo.poison {
  background-color: rgba(128, 0, 128, 0.475);
}

.tipo.ground {
  background-color: rgba(165, 42, 42, 0.505);
}

.tipo.flying {
  background-color: rgba(250, 221, 162, 0.46);
}

.tipo.psychic {
  background-color: rgba(238, 120, 140, 0.481);
}

.tipo.bug {
  background-color: rgba(122, 165, 42, 0.505);
}

.tipo.rock {
  background-color: rgba(128, 128, 128, 0.533);
}

.tipo.ghost {
  background-color: rgba(95, 13, 95, 0.477);
}

.tipo.dragon {
  background-color: rgba(71, 5, 5, 0.505);
}

.tipo.dark {
  background-color: rgba(0, 0, 0, 0.455);
}

.tipo.steel {
  background-color: rgba(85, 107, 47, 0.453);
}

.tipo.fairy {
  background-color: rgba(165, 42, 42, 0.505);
}

.tipo.normal {
  background-color: rgba(97, 97, 238, 0.456);
}



.estadisticas {
  margin-top: 10px;
}

.estadisticas p {
  font-weight: bold;
}

.barra {
  background-color: #ccc;
  height: 10px;
  border-radius: 5px;
  margin-top: 5px;
  position: relative;
}

.relleno {
  height: 100%;
  background-color: #69f0ae;
  border-radius: 5px;
}

.modal-large {
  width: 90%;
  margin-top: 130px;
}

.pokemon-details {
  display: flex;
  align-items: flex-start;
}

.pokemon-info-left {
  flex: 1;
  padding-right: 50px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pokemon-info-right {
  flex: 2;
  margin-left: 20px;
  margin-top: 50px;
}

.numero-pokemon {
  font-size: 50px;
  color: #03a9f4;
  margin-top: 5px;
  font-weight: bold;
}

.altura-peso {
  font-weight: bold;
}
.botonCerrar{
    margin-left: 90%;
  }
@media (max-width: 600px) {
  .header {
    flex-direction: column;
    align-items: center;
  }

  .search-filter {
    margin-top: 0;
  }
  .botonCerrar{
    margin-left: 80%;
  }
  .modal-large{
    margin-top: 170px;
    height: 80%;
  }
}
</style>
