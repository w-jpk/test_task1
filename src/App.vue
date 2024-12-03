<template>
  <div class="container mt-4">
    <h1>Отслеживание грузов</h1>

    <div class="mb-3">
      <label for="filterStatus">Фильтр по статусу:</label>

      <select id="filterStatus" v-model="filterStatus" class="form-select">
        <option value="">Все</option>

        <option value="Ожидает отправки">Ожидает отправки</option>

        <option value="В пути">В пути</option>

        <option value="Доставлен">Доставлен</option>
      </select>
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

        <input v-model="newCargo.name" type="text" class="form-control" />
      </div>

      <div>
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
      </div>

      <div>
        <label>Дата отправления:</label>

        <input
          v-model="newCargo.departureDate"
          type="date"
          class="form-control" />
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
      cities,
      newCargo: {
        name: "",
        origin: "",
        destination: "",
        departureDate: "",
        status: "Ожидает отправки",
      },
      filterStatus: "",
    };
  },
  computed: {
    filteredCargoList() {
      if (this.filterStatus === "") {
        return this.cargoList;
      } else {
        return this.cargoList.filter(
          (cargo) => cargo.status === this.filterStatus
        );
      }
    },
    citiesNameAndSubject() {
      return this.cities.map((city) => ({
        name: city.name + ", " + city.subject,
      }));
    },
  },
  methods: {
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
  mounted() {},
};
</script>

<style>
.container {
  max-width: 800px;
}
</style>
