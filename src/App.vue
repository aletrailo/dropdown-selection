<template>
  <div>
    <h1>Vehicle Selection</h1>
    <p>Select a year, make, and model to view vehicle details.</p>

    <label for="year">Year:</label>
    <select v-model="selectedYear" @change="fetchMakes" :disabled="isLoading" aria-label="Select Year">
      <option value="" disabled hidden>Select Year</option>
      <option v-for="year in years" :key="year.year" :value="year.year">{{ year.year }}</option>
    </select>

    <label for="make">Make:</label>
    <select v-model="selectedMake" @change="fetchModels" :disabled="!selectedYear || isLoading" aria-label="Select Make">
      <option value="" disabled hidden>Select Make</option>
      <option v-if="!selectedYear" disabled>Please select a year first</option>
      <option v-for="make in makes" :key="make.make" :value="make.make">{{ make.make }}</option>
    </select>

    <label for="model">Model:</label>
    <select v-model="selectedModel" :disabled="!selectedMake || isLoading" aria-label="Select Model">
      <option value="" disabled hidden>Select Model</option>
      <option v-if="!selectedMake" disabled>Please select a make first</option>
      <option v-for="model in models" :key="model.vehicle_id" :value="model">{{ model.model }}</option>
    </select>

    <span v-if="isLoading" id="spinner"></span>

    <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>

    <div v-if="selectedModel" class="vehicle-details">
      <h2>Vehicle Details</h2>
      <p><strong>Type:</strong> {{ selectedModel.type }}</p>
      <p><strong>Vehicle Type:</strong> {{ selectedModel.vehicle_type }}</p>
      <p><strong>Body Type:</strong> {{ selectedModel.body_type }}</p>
      <p><strong>Original Model:</strong> {{ selectedModel.original_model }}</p>
    </div>
    <button @click="resetSelection" :disabled="isLoading" class="reset-button">Clear Selection</button>
  </div>
</template>

<script>
const proxyUrl = 'https://api.allorigins.win/raw?url=';
const baseUrl = 'https://rateengine.ship.cars/v2/vehicles/';
const token = '5cbe12fb62f4941267d623499a2a4fd5948fd3ef';

export default {
  data() {
    return {
      years: [],
      makes: [],
      models: [],
      selectedYear: '',
      selectedMake: '',
      selectedModel: '',
      isLoading: false,
      errorMessage: ''
    };
  },
  mounted() {
    this.fetchYears();
  },
  methods: {
    async fetchData(endpoint, params) {
      const url = proxyUrl + encodeURIComponent(`${baseUrl}${endpoint}/?${new URLSearchParams(params)}`);
      this.isLoading = true;
      this.errorMessage = '';
      try {
        const response = await fetch(url, {
          headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        });
        if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);
        return await response.json();
      } catch (error) {
        console.error(`Error fetching ${endpoint}:`, error);
        this.errorMessage = `Failed to load data from ${endpoint}. Please check your internet connection or try again later.`;
        return [];
      } finally {
        this.isLoading = false;
      }
    },
    async fetchYears() {
      const params = { format: 'json', token };
      this.years = await this.fetchData('years', params);
    },
    async fetchMakes() {
      if (!this.selectedYear) return;
      this.selectedMake = '';
      this.selectedModel = '';
      this.models = [];

      const params = { format: 'json', token, year: this.selectedYear };
      this.makes = await this.fetchData('makes', params);
    },
    async fetchModels() {
      if (!this.selectedYear || !this.selectedMake) return;
      
      this.selectedModel = '';
      this.models = [];

      const params = {
        format: 'json',
        token,
        year: this.selectedYear,
        make: this.selectedMake
      };
      this.models = await this.fetchData('models', params);
    },
    resetSelection() {
      this.selectedYear = '';
      this.selectedMake = '';
      this.selectedModel = '';
      this.makes = [];
      this.models = [];
      this.errorMessage = '';
    }
  }
};
</script>

<style>
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f8f9fa;
  color: #212529;
  margin: 0;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}
</style>

<style scoped>
div {
  background-color: #ffffff;
  border-radius: 0.5rem;
  box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
  padding: 2rem;
  max-width: 600px;
  width: 100%;
  text-align: center;
}

h1 {
  font-size: 1.75rem;
  margin-bottom: 1rem;
  color: #007bff;
}

p {
  font-size: 1rem;
  margin-bottom: 1.5rem;
  color: #6c757d;
}

label {
  font-size: 0.875rem;
  font-weight: 500;
  display: block;
  margin-top: 1rem;
  margin-bottom: 0.5rem;
  text-align: left;
}

select {
  margin-bottom: 1rem;
  padding: 0.5rem;
  border-radius: 0.25rem;
  border: 1px solid #ced4da;
  width: 100%;
  box-sizing: border-box;
  transition: border-color 0.2s;
}

select:focus {
  border-color: #007bff;
  outline: none;
  box-shadow: 0 0 0 0.25rem rgba(0, 123, 255, 0.25);
}

.reset-button {
  background-color: #007bff;
  color: #ffffff;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  cursor: pointer;
  margin-top: 1rem;
}

.reset-button:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}

.reset-button:hover:not(:disabled) {
  background-color: #0056b3;
}

.error-message {
  color: #dc3545;
  font-weight: bold;
  margin-top: 1rem;
}

.vehicle-details {
  text-align: left;
  margin-top: 1.5rem;
  background-color: #f1f1f1;
  padding: 1rem;
  border-radius: 0.5rem;
}

#spinner {
    width: 100px;
    height: 100px;
    position: fixed;
    z-index: 1;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    border-radius: 50%;
    border: 6px solid rgb(183, 213, 252);
    box-sizing: border-box;
}

#spinner::before {
    content: "";
    display: block;
    width: 100px;
    height: 100px;
    position: absolute;
    top: -6px;
    left: -6px;
    z-index: 2;
    border-radius: 50%;
    border: 6px solid rgb(40, 85, 143);
    border-color: transparent transparent transparent rgb(40, 85, 143);
    box-sizing: border-box;
    animation: spin 1.5s ease-out infinite;
}

@keyframes spin {
    0% {
        transform: rotate(0deg);
    }

    20% {
        transform: rotate(180deg);
    }

    80% {
        trasform: rotate(270deg);
    }

    100% {
        transform: rotate(360deg);
    }
}
</style>
