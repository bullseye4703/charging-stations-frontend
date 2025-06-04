<template>
  <div class="container py-5">
    <button class="btn btn-secondary mb-4" @click="$router.back()">← Back</button>

    <div v-if="station" class="card p-4 shadow">
      <h3 class="card-title mb-3">{{ station.name }}</h3>
      <ul class="list-group list-group-flush">
        <li class="list-group-item">
          <strong>Status:</strong> {{ station.status }}
        </li>
        <li class="list-group-item">
          <strong>Power Output:</strong> {{ station.powerOutput }} kW
        </li>
        <li class="list-group-item">
          <strong>Connector Type:</strong> {{ station.connectorType }}
        </li>
        <li class="list-group-item">
          <strong>Latitude:</strong> {{ station.location?.latitude }}
        </li>
        <li class="list-group-item">
          <strong>Longitude:</strong> {{ station.location?.longitude }}
        </li>
      </ul>
    </div>

    <div v-else class="alert alert-info">
      Loading station details...
    </div>
  </div>
</template>

<script>
import api from '../axios';

export default {
  name: 'StationDetail',
  data() {
    return {
      station: null
    };
  },
  async created() {
    const id = this.$route.params.id;
    const token = localStorage.getItem('token');
    try {
      const res = await api.get(`/api/stations/${id}`, {
        headers: { Authorization: token }
      });
      this.station = res.data;
    } catch (err) {
      console.error('Error fetching station:', err);
    }
  }
};
</script>
