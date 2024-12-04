<template>
  <div class="container mt-4">
    <h1>Отслеживание грузов</h1>

    <div class="row g-3 align-items-end mb-3">
      <div class="col">
        <label for="searchName" class="form-label">Поиск по названию:</label>

        <div class="autocomplete">
          <input
            type="text"
            v-model="searchName"
            class="form-control"
            placeholder="Введите название"
            @input="filterNameSuggestions"
            @blur="hideSuggestions('name')" />

          <ul
            v-if="filteredNames.length && showSuggestions.name"
            class="suggestions">
            <li
              v-for="(name, index) in filteredNames"
              :key="index"
              @click="selectName(name)">
              {{ name }}
            </li>
          </ul>
        </div>
      </div>

      <div class="col">
        <label for="filterStatus" class="form-label">Фильтр по статусу:</label>

        <select id="filterStatus" v-model="filterStatus" class="form-select">
          <option value="">Все</option>

          <option value="Ожидает отправки">Ожидает отправки</option>

          <option value="В пути">В пути</option>

          <option value="Доставлен">Доставлен</option>
        </select>
      </div>

      <div class="col">
        <label for="filterOrigin" class="form-label">
          Фильтр по городу отправления:
        </label>

        <select id="filterOrigin" v-model="filterOrigin" class="form-select">
          <option value="">Все</option>

          <option v-for="city in cargoList" :key="city.id" :value="city.origin">
            {{ city.origin }}
          </option>
        </select>
      </div>

      <div class="col">
        <label for="filterDestination" class="form-label">
          Фильтр по городу получения:
        </label>

        <select
          id="filterDestination"
          v-model="filterDestination"
          class="form-select">
          <option value="">Все</option>

          <option
            v-for="city in cargoList"
            :key="city.id"
            :value="city.destination">
            {{ city.destination }}
          </option>
        </select>
      </div>
    </div>

    <table class="table table-bordered">
      <thead>
        <tr>
          <th>#</th>

          <th>Название</th>

          <th>Статус</th>

          <th>Откуда</th>

          <th>Куда</th>

          <th>Дата</th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="cargo in filteredCargoList" :key="cargo.id">
          <td>{{ cargo.id }}</td>

          <td>{{ cargo.name }}</td>

          <td>
            <select
              v-model="cargo.status"
              class="form-select"
              :class="statusClass(cargo.status)"
              @change="checkStatus(cargo)">
              <option value="Ожидает отправки">Ожидает отправки</option>

              <option value="В пути">В пути</option>

              <option value="Доставлен">Доставлен</option>
            </select>
          </td>

          <td>{{ cargo.origin }}</td>

          <td>{{ cargo.destination }}</td>

          <td>{{ cargo.departureDate }}</td>
        </tr>
      </tbody>
    </table>

    <h2>Добавить новый груз</h2>

    <form @submit.prevent="addNewCargo">
      <div>
        <label>Название:</label>

        <input
          v-model="newCargo.name"
          type="text"
          placeholder="Введите название"
          class="form-control" />
      </div>

      <!-- <div>
        <label>Откуда:</label>

        <select v-model="newCargo.origin" class="form-select">
          <option value="">Выберите город</option>

          <option
            v-for="city in citiesNameAndSubject"
            :key="city.name"
            :value="city.name">
            {{ city.name }}
          </option>
        </select>
      </div>

      <div>
        <label>Куда:</label>

        <select v-model="newCargo.destination" class="form-select">
          <option value="">Выберите город</option>

          <option
            v-for="city in citiesNameAndSubject"
            :key="city.name"
            :value="city.name">
            {{ city.name }}
          </option>
        </select>
      </div> -->

      <div class="row">
        <div class="col">
          <label>Откуда:</label>

          <div class="autocomplete">
            <input
              type="text"
              v-model="newCargo.origin"
              class="form-control"
              placeholder="Введите город"
              @input="filterCities('origin')"
              @blur="hideSuggestions('origin')" />

            <ul
              v-if="filteredCities.origin.length && showSuggestions.origin"
              class="suggestions">
              <li
                v-for="(city, index) in filteredCities.origin"
                :key="index"
                @click="selectCity(city.name, 'origin')">
                {{ city.name }}
              </li>
            </ul>
          </div>
        </div>

        <div class="col">
          <label>Куда:</label>

          <div class="autocomplete">
            <input
              type="text"
              v-model="newCargo.destination"
              class="form-control"
              placeholder="Введите город"
              @input="filterCities('destination')"
              @blur="hideSuggestions('destination')" />

            <ul
              v-if="
                filteredCities.destination.length && showSuggestions.destination
              "
              class="suggestions">
              <li
                v-for="(city, index) in filteredCities.destination"
                :key="index"
                @click="selectCity(city.name, 'destination')">
                {{ city.name }}
              </li>
            </ul>
          </div>
        </div>

        <div class="col">
          <label>Дата отправления:</label>

          <input
            v-model="newCargo.departureDate"
            type="date"
            class="form-control" />
        </div>
      </div>

      <button type="submit" class="btn btn-primary mt-3">Добавить груз</button>
    </form>
  </div>
</template>

<script>
import { cities } from "@/russian-cities";
export default {
  data() {
    return {
      cargoList: [
        {
          id: "CARGO001",
          name: "Строительные материалы",
          status: "В пути",
          origin: "Москва",
          destination: "Казань",
          departureDate: "2024-11-24",
        },
        {
          id: "CARGO002",
          name: "Хрупкий груз",
          status: "Ожидает отправки",
          origin: "Санкт-Петербург",
          destination: "Екатеринбург",
          departureDate: "2024-11-26",
        },
      ],
      newCargo: {
        name: "",
        origin: "",
        destination: "",
        departureDate: "",
        status: "Ожидает отправки",
      },
      filteredCities: {
        origin: [],
        destination: [],
      },
      filteredNames: [],
      showSuggestions: {
        origin: false,
        destination: false,
        name: false,
      },
      filterStatus: "",
      filterOrigin: "",
      filterDestination: "",
      searchName: "",
      cities,
    };
  },
  computed: {
    filteredCargoList() {
      return this.cargoList.filter((cargo) => {
        const matchesStatus =
          this.filterStatus === "" || cargo.status === this.filterStatus;

        const matchesOrigin =
          this.filterOrigin === "" || cargo.origin === this.filterOrigin;

        const matchesDestination =
          this.filterDestination === "" ||
          cargo.destination === this.filterDestination;

        const matchesName =
          this.searchName === "" ||
          cargo.name.toLowerCase().includes(this.searchName.toLowerCase());

        return (
          matchesStatus && matchesOrigin && matchesDestination && matchesName
        );
      });
    },

    citiesNameAndSubject() {
      return this.cities.map((city) => ({
        name: city.name + ", " + city.subject,
      }));
    },
  },
  methods: {
    filterCities(type) {
      const inputValue = this.newCargo[type].toLowerCase();
      if (inputValue.length === 0) {
        this.filteredCities[type] = [];

        this.showSuggestions[type] = false;

        return;
      }

      this.filteredCities[type] = this.citiesNameAndSubject.filter((city) =>
        city.name.toLowerCase().includes(inputValue)
      );

      this.showSuggestions[type] = true;
    },

    selectCity(cityName, type) {
      this.newCargo[type] = cityName;

      this.showSuggestions[type] = false;
    },

    filterNameSuggestions() {
      const inputValue = this.searchName.toLowerCase();

      if (!inputValue) {
        this.filteredNames = [];

        this.showSuggestions.name = false;

        return;
      }

      this.filteredNames = this.cargoList
        .map((cargo) => cargo.name)
        .filter((name) => name.toLowerCase().includes(inputValue));

      this.showSuggestions.name = true;
    },

    selectName(name) {
      this.searchName = name;

      this.showSuggestions.name = false;

      this.filterCargoByName();
    },

    hideSuggestions(type) {
      setTimeout(() => {
        this.showSuggestions[type] = false;
      }, 200);
    },

    filterCargoByName() {
      this.filteredCargoList = this.cargoList.filter((cargo) =>
        cargo.name.toLowerCase().includes(this.searchName.toLowerCase())
      );
    },

    addNewCargo() {
      if (
        this.newCargo.name === "" ||
        this.newCargo.origin === "" ||
        this.newCargo.destination === "" ||
        this.newCargo.departureDate === ""
      ) {
        alert("Все поля должны быть заполнены!");
        return;
      }
      const id = "CARGO" + String(this.cargoList.length + 1).padStart(3, "0");

      this.cargoList.push({ ...this.newCargo, id });

      this.newCargo = {
        name: "",
        origin: "",
        destination: "",
        departureDate: "",
        status: "Ожидает отправки",
      };
    },

    checkStatus(cargo) {
      const departureDate = new Date(cargo.departureDate);

      const currentDate = new Date();

      if (cargo.status === "Доставлен" && departureDate > currentDate) {
        alert(
          "Груз не может быть отмечен как 'Доставлен', если дата отправления находится в будущем."
        );
        cargo.status = "В пути";
      }
    },

    statusClass(status) {
      return {
        "bg-warning text-dark": status === "Ожидает отправки",
        "bg-primary text-white": status === "В пути",
        "bg-success text-white": status === "Доставлен",
      };
    },
  },
};
</script>

<style>
.container {
  max-width: 800px;
}

.autocomplete {
  position: relative;
}

.suggestions {
  position: absolute;
  z-index: 1000;
  background: white;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 100%;
  max-height: 200px;
  overflow-y: auto;
  list-style: none;
  padding: 0;
  margin: 0;
}

.suggestions li {
  padding: 8px;
  cursor: pointer;
}

.suggestions li:hover {
  background-color: #f0f0f0;
}
</style>
