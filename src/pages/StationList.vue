<template>
  <div class="container py-4">
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h2>Charging Stations</h2>
      <button class="btn btn-outline-danger" @click="logout">Logout</button>
    </div>

    <div class="row mb-4">
      <div class="col-md-3">
        <select class="form-select" v-model="filterStatus">
          <option value="">All Status</option>
          <option value="Active">Active</option>
          <option value="Inactive">Inactive</option>
        </select>
      </div>
      <div class="col-md-3">
        <input
          type="number"
          class="form-control"
          v-model.number="filterPower"
          placeholder="Min Power Output (kW)"
        />
      </div>
      <div class="col-md-3">
        <input
          type="text"
          class="form-control"
          v-model="filterConnector"
          placeholder="Connector Type"
        />
      </div>
      <div class="col-md-3">
        <button class="btn btn-secondary w-100" @click="fetchStations">
          Apply Filters
        </button>
      </div>
    </div>

    <!-- MAP -->
    <div id="map" style="height: 400px" class="mb-4"></div>

    <!-- STATION LIST -->
    <ul class="list-group mb-4">
      <li
        v-for="station in filteredStations"
        :key="station._id"
        class="list-group-item d-flex justify-content-between align-items-center"
      >
        <div>
          <strong>
            <router-link :to="`/stations/${station._id}`">{{
              station.name
            }}</router-link>
          </strong>
          - {{ station.status }} - {{ station.powerOutput }} kW -
          {{ station.connectorType }}
        </div>
        <div>
          <button
            class="btn btn-sm btn-info me-2"
            @click="editStation(station)"
          >
            Edit
          </button>
          <button
            class="btn btn-sm btn-danger"
            @click="deleteStation(station._id)"
          >
            Delete
          </button>
        </div>
      </li>
    </ul>

    <!-- ADD/EDIT FORM -->
    <div class="card p-4">
      <h4>{{ formData._id ? "Edit" : "Add" }} Station</h4>
      <div class="row g-2">
        <div class="col-md-6">
          <input
            v-model="formData.name"
            class="form-control"
            placeholder="Name"
          />
        </div>
        <div class="col-md-3">
          <input
            v-model.number="formData.location.latitude"
            class="form-control"
            placeholder="Latitude"
          />
        </div>
        <div class="col-md-3">
          <input
            v-model.number="formData.location.longitude"
            class="form-control"
            placeholder="Longitude"
          />
        </div>
        <div class="col-md-4">
          <select v-model="formData.status" class="form-select">
            <option disabled value="">Select Status</option>
            <option>Active</option>
            <option>Inactive</option>
          </select>
        </div>
        <div class="col-md-4">
          <input
            v-model.number="formData.powerOutput"
            class="form-control"
            placeholder="Power Output (kW)"
          />
        </div>
        <div class="col-md-4">
          <input
            v-model="formData.connectorType"
            class="form-control"
            placeholder="Connector Type"
          />
        </div>
        <div class="col-12 mt-3">
          <button class="btn btn-primary w-100" @click="submitForm">
            {{ formData._id ? "Update" : "Create" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import api from "../axios";
import L from "leaflet";
import "leaflet/dist/leaflet.css";

export default {
  name: "ChargerList",
  data() {
    return {
      stations: [],
      formData: {
        name: "",
        location: { latitude: null, longitude: null },
        status: "",
        powerOutput: null,
        connectorType: "",
      },
      filterStatus: "",
      filterPower: 0,
      filterConnector: "",
    };
  },
  computed: {
    filteredStations() {
      return this.stations.filter(
        (s) =>
          (!this.filterStatus || s.status === this.filterStatus) &&
          (!this.filterPower || s.powerOutput >= this.filterPower) &&
          (!this.filterConnector ||
            s.connectorType
              .toLowerCase()
              .includes(this.filterConnector.toLowerCase()))
      );
    },
  },
  methods: {
    async fetchStations() {
      const token = localStorage.getItem("token");
      const res = await api.get("/api/stations", {
        headers: { Authorization: token },
      });
      this.stations = res.data;
      this.renderMap();
    },
    async submitForm() {
      const token = localStorage.getItem("token");
      if (this.formData._id) {
        await api.put(`/api/stations/${this.formData._id}`, this.formData, {
          headers: { Authorization: token },
        });
      } else {
        await api.post("/api/stations", this.formData, {
          headers: { Authorization: token },
        });
      }
      this.clearForm();
      this.fetchStations();
    },
    async deleteStation(id) {
      const token = localStorage.getItem("token");
      await api.delete(`/api/stations/${id}`, {
        headers: { Authorization: token },
      });
      this.fetchStations();
    },
    editStation(station) {
      this.formData = { ...station };
    },
    clearForm() {
      this.formData = {
        name: "",
        location: { latitude: 0, longitude: 0 },
        status: "",
        powerOutput: 0,
        connectorType: "",
      };
    },
    logout() {
      localStorage.removeItem("token");
      this.$router.push("/login");
    },
    renderMap() {
      if (this.map) {
        this.map.remove(); // Remove previous instance
      }

      this.map = L.map("map").setView([20.5937, 78.9629], 5);

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "&copy; OpenStreetMap contributors",
      }).addTo(this.map);

      this.stations.forEach((station) => {
        const { latitude, longitude } = station.location || {};
        if (latitude && longitude) {
          L.marker([latitude, longitude])
            .addTo(this.map)
            .bindPopup(
              `<b>${station.name}</b><br>${station.status}<br>${station.connectorType}`
            );
        }
      });
    },
  },
  mounted() {
    this.fetchStations();
  },
};
</script>
