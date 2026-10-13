# GUÍA TOTAL: ASTARA RETAIL (Copia y Pega)

## 🟢 COMMIT 1: Configuración

**1. `.env.development`** (En la raíz del proyecto)
```env
VITE_API_BASE_URL=http://localhost:3000
VITE_SHOWROOM_SECTIONS_ENDPOINT_PATH=/showroom-sections
VITE_VEHICLE_INVENTORY_ITEMS_ENDPOINT_PATH=/vehicle-inventory-items

VITE_EXTERNAL_API_BASE_URL=https://dummyjson.com
VITE_VEHICLES_ENDPOINT_PATH=/products/category/vehicle

VITE_LOGO_API_URL=https://img.logo.dev/astararetail.pe?token=pk_X-1ZO13GSgeOoUrIuJ6GMQ
```

**2. `README.md`** (En la raíz, pon tu código)
```markdown
# Astara Showroom Management
Frontend web application for Astara Retail Perú.
## Authorship
- **Author:** Piero Francisco Montes Chang
- **Student Code:** TU_CODIGO_AQUI
- **NRC:** 8168
- **GitHub Private Repository URL:** https://github.com/TU_USER/ea8168TU_CODIGO
```

**Terminal:**
```bash
git add .
git commit -m "chore(config): initialize project structure and variables"
git push -u origin main
```

---

## 🟢 COMMIT 2: Shared Infra y Base de Datos

**1. `server/db.json`**
```json
{
  "showroom-sections": [
    { "id": 1, "name": "SUV & Crossover Main Showroom", "vehicleCategory": "vehicle", "maxCapacity": 10 },
    { "id": 2, "name": "Sedan & Hatchback Exhibition Bay", "vehicleCategory": "vehicle", "maxCapacity": 8 },
    { "id": 3, "name": "Commercial & Pick-up Yard", "vehicleCategory": "vehicle", "maxCapacity": 6 }
  ],
  "vehicle-inventory-items": [
    { "id": 1, "sectionId": 1, "vehicleCategoryId": "vehicle", "vehicleId": 1, "vehicleName": "Essence Mascara LaMia", "quantity": 3, "registeredAt": "2026-10-12T08:30:00.000Z" }
  ]
}
```

**2. `src/shared/infrastructure/base-endpoint.js`**
```javascript
/**
 * @summary Base Endpoint
 * @author Piero Francisco Montes Chang
 */
export class BaseEndpoint {
    static getBaseUrl() { return import.meta.env.VITE_API_BASE_URL; }
}
```

**3. `src/shared/infrastructure/base-api.js`**
```javascript
/**
 * @summary Base API
 * @author Piero Francisco Montes Chang
 */
import axios from 'axios';
import { BaseEndpoint } from './base-endpoint.js';

export class BaseApi {
    constructor() { this.http = axios.create({ baseURL: BaseEndpoint.getBaseUrl() }); }
}
```

**Terminal:**
```bash
git add .
git commit -m "feat(shared): setup fake backend database and base infrastructure"
git push -u origin main
```

---

## 🟢 COMMIT 3: Showroom Domain & API

**1. `src/showroom/domain/model/showroom-section.entity.js`**
```javascript
/**
 * @summary Showroom Section Entity
 * @author Piero Francisco Montes Chang
 */
export class ShowroomSection {
    constructor({ id = null, name = '', vehicleCategory = '', maxCapacity = 0 } = {}) {
        this.id = id; this.name = name; this.vehicleCategory = vehicleCategory; this.maxCapacity = maxCapacity;
    }
}
```

**2. `src/showroom/domain/model/vehicle-inventory-item.entity.js`**
```javascript
/**
 * @summary Vehicle Inventory Item Entity
 * @author Piero Francisco Montes Chang
 */
export class VehicleInventoryItem {
    constructor({ id = null, sectionId = null, vehicleCategoryId = 'vehicle', vehicleId = null, vehicleName = '', quantity = 0, registeredAt = '' } = {}) {
        this.id = id; this.sectionId = sectionId; this.vehicleCategoryId = vehicleCategoryId; this.vehicleId = vehicleId; this.vehicleName = vehicleName; this.quantity = quantity; this.registeredAt = registeredAt;
    }
}
```

**3. `src/showroom/infrastructure/showroom-section.assembler.js`**
```javascript
/**
 * @summary Showroom Section Assembler
 * @author Piero Francisco Montes Chang
 */
import { ShowroomSection } from "../domain/model/showroom-section.entity.js";
export class ShowroomSectionAssembler {
    static toEntityFromResource(resource) { return new ShowroomSection({ ...resource }); }
    static toEntitiesFromResponse(response) {
        if (response.status !== 200) return [];
        const res = response.data instanceof Array ? response.data : (response.data['showroom-sections'] ?? []);
        return res.map(r => this.toEntityFromResource(r));
    }
}
```

**4. `src/showroom/infrastructure/vehicle-inventory-item.assembler.js`**
```javascript
/**
 * @summary Vehicle Inventory Item Assembler
 * @author Piero Francisco Montes Chang
 */
import { VehicleInventoryItem } from "../domain/model/vehicle-inventory-item.entity.js";
export class VehicleInventoryItemAssembler {
    static toEntityFromResource(resource) { return new VehicleInventoryItem({ ...resource }); }
    static toEntitiesFromResponse(response) {
        if (response.status !== 200) return [];
        const res = response.data instanceof Array ? response.data : (response.data['vehicle-inventory-items'] ?? []);
        return res.map(r => this.toEntityFromResource(r));
    }
}
```

**5. `src/showroom/infrastructure/showroom.api.js`**
```javascript
/**
 * @summary Showroom Local API Integration
 * @author Piero Francisco Montes Chang
 */
import { BaseApi } from "../../shared/infrastructure/base-api.js";
export class ShowroomApi extends BaseApi {
    getSections() { return this.http.get(import.meta.env.VITE_SHOWROOM_SECTIONS_ENDPOINT_PATH); }
    getItems() { return this.http.get(import.meta.env.VITE_VEHICLE_INVENTORY_ITEMS_ENDPOINT_PATH); }
    createItem(item) { return this.http.post(import.meta.env.VITE_VEHICLE_INVENTORY_ITEMS_ENDPOINT_PATH, item); }
}
```

**Terminal:**
```bash
git add .
git commit -m "feat(showroom): implement domain entities, assemblers and local api service"
git push -u origin main
```

---

## 🟢 COMMIT 4: Showroom Store

**1. `src/showroom/application/showroom.store.js`**
```javascript
/**
 * @summary Showroom Store
 * @author Piero Francisco Montes Chang
 */
import { defineStore } from "pinia";
import { ref } from "vue";
import { ShowroomApi } from "../infrastructure/showroom.api.js";
import { ShowroomSectionAssembler } from "../infrastructure/showroom-section.assembler.js";
import { VehicleInventoryItemAssembler } from "../infrastructure/vehicle-inventory-item.assembler.js";

const api = new ShowroomApi();
export const useShowroomStore = defineStore('showroom', () => {
    const sections = ref([]);
    const items = ref([]);
    
    const fetchSections = async () => {
        const response = await api.getSections();
        sections.value = ShowroomSectionAssembler.toEntitiesFromResponse(response);
    };
    const fetchItems = async () => {
        const response = await api.getItems();
        items.value = VehicleInventoryItemAssembler.toEntitiesFromResponse(response);
    };
    const addItem = async (item) => {
        const response = await api.createItem(item);
        items.value.push(VehicleInventoryItemAssembler.toEntityFromResource(response.data));
    };

    const getItemsBySectionId = (sectionId) => items.value.filter(i => Number(i.sectionId) === Number(sectionId));
    const getTotalVehiclesBySectionId = (sectionId) => getItemsBySectionId(sectionId).reduce((total, i) => total + Number(i.quantity), 0);
    const getAvailableCapacity = (section) => Number(section.maxCapacity) - getTotalVehiclesBySectionId(section.id);

    return { sections, items, fetchSections, fetchItems, addItem, getItemsBySectionId, getTotalVehiclesBySectionId, getAvailableCapacity };
});
export default useShowroomStore;
```

**Terminal:**
```bash
git add .
git commit -m "feat(showroom): add pinia state management store with capacity logic"
git push -u origin main
```

---

## 🟢 COMMIT 5: Subdominio Vehicles (DummyJSON)

**1. `src/vehicles/domain/model/vehicle.entity.js`**
```javascript
/**
 * @summary Vehicle Entity
 * @author Piero Francisco Montes Chang
 */
export class Vehicle {
    constructor({ id = null, title = '' } = {}) { this.id = id; this.title = title; }
}
```

**2. `src/vehicles/infrastructure/vehicle.assembler.js`**
```javascript
/**
 * @summary Vehicle Assembler
 * @author Piero Francisco Montes Chang
 */
import { Vehicle } from "../domain/model/vehicle.entity.js";
export class VehicleAssembler {
    static toEntityFromResource(resource) { return new Vehicle({ ...resource }); }
    static toEntitiesFromResponse(response) {
        if (response.status !== 200) return [];
        const res = response.data.products || [];
        return res.map(r => this.toEntityFromResource(r));
    }
}
```

**3. `src/vehicles/infrastructure/vehicles.api.js`**
```javascript
/**
 * @summary Vehicles External API
 * @author Piero Francisco Montes Chang
 */
import axios from "axios";
const http = axios.create({ baseURL: import.meta.env.VITE_EXTERNAL_API_BASE_URL });
export class VehiclesApi { getVehicles() { return http.get(import.meta.env.VITE_VEHICLES_ENDPOINT_PATH); } }
```

**4. `src/vehicles/application/vehicles.store.js`**
```javascript
/**
 * @summary Vehicles Store
 * @author Piero Francisco Montes Chang
 */
import { defineStore } from "pinia";
import { ref } from "vue";
import { VehiclesApi } from "../infrastructure/vehicles.api.js";
import { VehicleAssembler } from "../infrastructure/vehicle.assembler.js";

const api = new VehiclesApi();
export const useVehiclesStore = defineStore('vehicles', () => {
    const vehicles = ref([]);
    const fetchVehicles = async () => {
        const response = await api.getVehicles();
        vehicles.value = VehicleAssembler.toEntitiesFromResponse(response);
    };
    return { vehicles, fetchVehicles };
});
export default useVehiclesStore;
```

**Terminal:**
```bash
git add .
git commit -m "feat(vehicles): implement dummyjson external api integration and store"
git push -u origin main
```

---

## 🟢 COMMIT 6: El Layout Completo, Rutas e i18n

**1. `src/locales/en.json`**
```json
{
  "toolbar": { "title": "Astara Showroom Management", "home": "Home", "new-item": "New Vehicle Inventory Item" },
  "home": { "title": "Premium Vehicles & Showroom Management Delivered to Your Dealership.", "section-title": "My Showroom Collections", "no-items": "No Vehicles", "total-vehicles": "Total Vehicles", "available-capacity": "Available Capacity" },
  "form": { "title": "New Vehicle Inventory Item", "subtitle": "Add Vehicle Inventory Items to Your Collection", "section-label": "Showroom Section", "vehicle-label": "Vehicle", "quantity-label": "Quantity", "create": "Create", "cancel": "Cancel", "errors": { "required": "Please fill all fields.", "capacity": "Cannot register: quantity exceeds available capacity in this section." } },
  "not-found": { "title": "Page Not Found", "btn": "Return Home" }
}
```

**2. `src/locales/es.json`**
```json
{
  "toolbar": { "title": "Gestión de Showroom Astara", "home": "Inicio", "new-item": "Nuevo Vehículo en Inventario" },
  "home": { "title": "Vehículos Premium y Gestión de Showroom en su Concesionario.", "section-title": "Mis Colecciones", "no-items": "Sin Vehículos", "total-vehicles": "Vehículos Totales", "available-capacity": "Capacidad Disponible" },
  "form": { "title": "Nuevo Vehículo", "subtitle": "Agregue un vehículo a su colección", "section-label": "Sección", "vehicle-label": "Vehículo", "quantity-label": "Cantidad", "create": "Crear", "cancel": "Cancelar", "errors": { "required": "Llene todos los campos.", "capacity": "Error: la cantidad excede la capacidad disponible." } },
  "not-found": { "title": "Página no encontrada", "btn": "Volver" }
}
```

**3. `src/i18n.js`**
```javascript
import { createI18n } from 'vue-i18n';
import en from './locales/en.json';
import es from './locales/es.json';

const i18n = createI18n({
    legacy: false,
    locale: 'en',
    fallbackLocale: 'en',
    messages: { en, es }
});
export default i18n;
```

**4. `src/showroom/presentation/showroom-routes.js`**
```javascript
import Home from "../../shared/presentation/views/home.vue";
import NewVehicleInventoryItem from "./views/new-vehicle-inventory-item.vue";

export const showroomRoutes = [
    { path: '/home', component: Home, meta: { title: 'Home' } },
    { path: '/vehicle/items/new', component: NewVehicleInventoryItem, meta: { title: 'New Item' } }
];
```

**5. `src/shared/presentation/views/page-not-found.vue`**
```vue
<script setup>
/**
 * @summary Page Not Found View
 * @author Piero Francisco Montes Chang
 */
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";
const { t } = useI18n(); const router = useRouter();
</script>
<template>
  <div class="flex flex-column align-items-center mt-5">
    <h1>{{ t('not-found.title') }}</h1>
    <pv-button :label="t('not-found.btn')" @click="router.push('/home')" />
  </div>
</template>
```

**6. `src/router.js`**
```javascript
import { createRouter, createWebHistory } from "vue-router";
import { showroomRoutes } from "./showroom/presentation/showroom-routes.js";
import PageNotFound from "./shared/presentation/views/page-not-found.vue";

const router = createRouter({
    history: createWebHistory(),
    routes: [
        { path: '/', redirect: '/home' },
        ...showroomRoutes,
        { path: '/:pathMatch(.*)*', component: PageNotFound, meta: { title: 'Page Not Found' } }
    ]
});
router.beforeEach((to, from, next) => {
    document.title = to.meta.title ? `Astara - ${to.meta.title}` : 'Astara'; next();
});
export default router;
```

**7. `src/app.vue` (El Layout Principal)**
```vue
<script setup>
/**
 * @summary Main Layout
 * @author Piero Francisco Montes Chang
 */
import { ref } from "vue";
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";

const drawer = ref(false);
const { t, locale } = useI18n();
const router = useRouter();

const toggleLanguage = (lang) => { locale.value = lang; };
const navigate = (path) => { router.push(path); drawer.value = false; };
</script>

<template>
  <div class="min-h-screen flex flex-column">
    <pv-toolbar class="bg-primary text-white shadow-2">
      <template #start>
        <pv-button icon="pi pi-bars" @click="drawer = true" class="mr-3 p-button-text text-white" />
        <img :src="'https://img.logo.dev/astararetail.pe?token=pk_X-1ZO13GSgeOoUrIuJ6GMQ'" alt="Logo" height="30" class="mr-2" />
        <span class="font-bold text-xl">{{ t('toolbar.title') }}</span>
      </template>
    </pv-toolbar>

    <pv-drawer v-model:visible="drawer" header="Menu">
      <div class="flex flex-column gap-2 mt-4 h-full">
        <pv-button :label="t('toolbar.home')" class="p-button-text text-left" @click="navigate('/home')" />
        <pv-button :label="t('toolbar.new-item')" class="p-button-text text-left" @click="navigate('/vehicle/items/new')" />
        
        <div class="mt-auto flex justify-content-center gap-2 pb-4">
          <pv-button label="EN" @click="toggleLanguage('en')" :class="locale === 'en' ? 'p-button-primary' : 'p-button-outlined'" />
          <pv-button label="ES" @click="toggleLanguage('es')" :class="locale === 'es' ? 'p-button-primary' : 'p-button-outlined'" />
        </div>
      </div>
    </pv-drawer>

    <main class="flex-grow-1">
      <router-view />
    </main>
  </div>
</template>
```

**8. `src/main.js` (Todo configurado para PrimeVue 4)**
```javascript
import { createApp } from 'vue'
import App from './app.vue'
import './style.css'

import router from './router.js'
import i18n from './i18n.js'
import { createPinia } from 'pinia'

import PrimeVue from 'primevue/config';
import Aura from '@primevue/themes/aura';
import 'primeicons/primeicons.css';
import 'primeflex/primeflex.css';

import Button from "primevue/button";
import Card from "primevue/card";
import Toolbar from "primevue/toolbar";
import Drawer from "primevue/drawer";
import Select from "primevue/select";
import InputNumber from "primevue/inputnumber";

const app = createApp(App);

app.use(createPinia());
app.use(router);
app.use(i18n);
app.use(PrimeVue, { theme: { preset: Aura, options: { prefix: 'p', darkModeSelector: 'system', cssLayer: false } }, ripple: true });

app.component('pv-button', Button);
app.component('pv-card', Card);
app.component('pv-toolbar', Toolbar);
app.component('pv-drawer', Drawer);
app.component('pv-select', Select);
app.component('pv-input-number', InputNumber);

app.mount('#app');
```

**Terminal:**
```bash
git add .
git commit -m "feat(shared): configure routing, layout, i18n and primevue themes"
git push -u origin main
```

---

## 🟢 COMMIT 7: Vista Home y Tarjetas

**1. `src/showroom/presentation/components/showroom-section-summary.vue`**
```vue
<script setup>
/**
 * @summary Summary Card Component
 * @author Piero Francisco Montes Chang
 */
import { computed } from "vue";
import { useI18n } from "vue-i18n";
import useShowroomStore from "../../application/showroom.store.js";

const props = defineProps({ section: { type: Object, required: true } });
const { t } = useI18n();
const store = useShowroomStore();

const childItems = computed(() => store.getItemsBySectionId(props.section.id));
const totalVehicles = computed(() => store.getTotalVehiclesBySectionId(props.section.id));
const availableCapacity = computed(() => store.getAvailableCapacity(props.section));
</script>

<template>
  <pv-card class="h-full shadow-2">
    <template #header>
      <div class="p-3 font-bold text-xl border-bottom-1 surface-border">{{ section.name }}</div>
    </template>
    <template #content>
      <div v-if="childItems.length === 0" class="text-500 italic">{{ t('home.no-items') }}</div>
      <div v-else class="flex flex-column gap-2">
        <div v-for="item in childItems" :key="item.id" class="flex justify-content-between border-bottom-1 surface-border pb-1">
          <span>{{ item.vehicleName }}</span><span class="font-bold">Qty: {{ item.quantity }}</span>
        </div>
      </div>
    </template>
    <template #footer>
      <div class="flex flex-column gap-1 pt-2 border-top-1 surface-border font-bold text-sm">
        <span>{{ t('home.total-vehicles') }}: {{ totalVehicles }}</span>
        <span>{{ t('home.available-capacity') }}: {{ availableCapacity }}</span>
      </div>
    </template>
  </pv-card>
</template>
```

**2. `src/shared/presentation/views/home.vue`**
```vue
<script setup>
/**
 * @summary Home View
 * @author Piero Francisco Montes Chang
 */
import { onMounted } from "vue";
import { useI18n } from "vue-i18n";
import useShowroomStore from "../../../showroom/application/showroom.store.js";
import ShowroomSectionSummary from "../../../showroom/presentation/components/showroom-section-summary.vue";

const { t } = useI18n();
const store = useShowroomStore();

onMounted(() => {
  store.fetchSections();
  store.fetchItems();
});
</script>

<template>
  <div class="p-4">
    <h1 class="text-3xl font-bold mb-4">{{ t('home.title') }}</h1>
    <section>
      <h2 class="text-2xl font-bold mb-3">{{ t('home.section-title') }}</h2>
      <div class="grid">
        <div v-for="section in store.sections" :key="section.id" class="col-12 md:col-4 p-2">
          <showroom-section-summary :section="section" />
        </div>
      </div>
    </section>
  </div>
</template>
```

**Terminal:**
```bash
git add .
git commit -m "feat(home): add home view and showroom summary card grid"
git push -u origin main
```

---

## 🟢 COMMIT 8: Vista de Formulario (Validaciones)

**1. `src/showroom/presentation/views/new-vehicle-inventory-item.vue`**
```vue
<script setup>
/**
 * @summary Creation Form View
 * @author Piero Francisco Montes Chang
 */
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import { useI18n } from "vue-i18n";
import useShowroomStore from "../../application/showroom.store.js";
import useVehiclesStore from "../../../vehicles/application/vehicles.store.js";
import { VehicleInventoryItem } from "../../domain/model/vehicle-inventory-item.entity.js";

const { t } = useI18n();
const router = useRouter();
const showroomStore = useShowroomStore();
const vehiclesStore = useVehiclesStore();

const item = ref(new VehicleInventoryItem());
const selectedSection = ref(null);
const selectedVehicle = ref(null);

onMounted(async () => {
  await showroomStore.fetchSections();
  await showroomStore.fetchItems();
  await vehiclesStore.fetchVehicles();
});

const onCancel = () => router.push('/home');

const onSave = async () => {
  if (!selectedSection.value || !selectedVehicle.value || item.value.quantity <= 0) {
    alert(t('form.errors.required'));
    return;
  }
  
  const available = showroomStore.getAvailableCapacity(selectedSection.value);
  if (item.value.quantity > available) {
    alert(t('form.errors.capacity'));
    return;
  }
  
  item.value.sectionId = selectedSection.value.id;
  item.value.vehicleCategoryId = 'vehicle';
  item.value.vehicleId = selectedVehicle.value.id;
  item.value.vehicleName = selectedVehicle.value.title;
  item.value.registeredAt = new Date().toISOString(); 
  
  await showroomStore.addItem(item.value);
  router.push('/home');
};
</script>

<template>
  <div class="p-4 flex justify-content-center">
    <pv-card class="w-full md:w-6 shadow-3">
      <template #title>{{ t('form.title') }}</template>
      <template #subtitle>{{ t('form.subtitle') }}</template>
      <template #content>
        <form @submit.prevent="onSave" class="flex flex-column gap-4 mt-3">
          
          <div class="flex flex-column gap-2">
            <label for="section" class="font-bold">{{ t('form.section-label') }}</label>
            <pv-select id="section" v-model="selectedSection" :options="showroomStore.sections" optionLabel="name" class="w-full" />
          </div>

          <div class="flex flex-column gap-2">
            <label for="vehicle" class="font-bold">{{ t('form.vehicle-label') }}</label>
            <pv-select id="vehicle" v-model="selectedVehicle" :options="vehiclesStore.vehicles" optionLabel="title" class="w-full" />
          </div>

          <div class="flex flex-column gap-2">
            <label for="quantity" class="font-bold">{{ t('form.quantity-label') }}</label>
            <pv-input-number id="quantity" v-model="item.quantity" :min="1" showButtons class="w-full" />
          </div>
          
          <div class="flex justify-content-end gap-2 mt-4 border-top-1 surface-border pt-3">
            <pv-button type="button" :label="t('form.cancel')" severity="secondary" @click="onCancel" />
            <pv-button type="submit" :label="t('form.create')" severity="primary" />
          </div>
        </form>
      </template>
    </pv-card>
  </div>
</template>
```

**Terminal:**
```bash
git add .
git commit -m "feat(showroom): implement creation form view with capacity business validations"
git push -u origin main
```