<!-- src/components/FirestoreExample.vue -->


<template>
  <section class="section" style="background-color: #cccccc;  width: 100%; height: 100vh;"  >
    <div class="container">
      <h1 class="title has-text-centered">Welcome to LocaMat</h1>

      <!-- Barre de recherche -->
      <div class="field has-addons">
        <div class="control is-expanded">
          <input class="input" type="text" placeholder="Search for equipment to borrow..." v-model="search"/>
        </div>
        <div class="control">
          <button class="button is-info">
            <i class="fas fa-search"></i>
          </button>
        </div>
      </div>

      <div>
        <button @click="handleClick">Cliquez-moi</button>
      </div>

      <!-- Filtres -->
      <div class="columns">
        <div class="column is-one-quarter">
          <aside class="menu">
            <p class="menu-label">Type of equipment</p>
            <ul class="menu-list">
              <li>
                <label>
                  <input type="checkbox" v-model="filters.type" value="Phone" />
                  Phone
                </label>
              </li>
              <li>
                <label>
                  <input type="checkbox" v-model="filters.type" value="Computer" />
                  Computer
                </label>
              </li>
              <li>
                <label>
                  <input type="checkbox" v-model="filters.type" value="Tablet" />
                  Tablet
                </label>
              </li>
            </ul>

            <p class="menu-label">Brand</p>
            <ul class="menu-list">
              <li>
                <label>
                  <input type="checkbox" v-model="filters.brand" value="Apple" />
                  Apple
                </label>
              </li>
              <li>
                <label>
                  <input type="checkbox" v-model="filters.brand" value="Samsung" />
                  Samsung
                </label>
              </li>
              <li>
                <label>
                  <input type="checkbox" v-model="filters.brand" value="Asus" />
                  Asus
                </label>
              </li>
              <li>
                <label>
                  <input type="checkbox" v-model="filters.brand" value="Other brand " />
                  Other brand
                </label>
              </li>
            </ul>

            <p class="menu-label">Status</p>
            <ul class="menu-list">
              <li>
                <label>
                  <input type="checkbox" v-model="filters.status" value="Borrowed" />
                  Borrowed
                </label>
              </li>
              <li>
                <label>
                  <input type="checkbox" v-model="filters.status" value="Not borrowed" />
                  Not borrowed
                </label>
              </li>
            </ul>
          </aside>
        </div>

        <!-- Liste des équipements -->
        <div class="columns is-multiline">
          <div class="column is-one-quarter" v-for="item in filteredEquipment" :key="item.id">
            <div class="card" @click="openModal(item)">
              <!-- Image de l'équipement -->
              <div class="card-image">
                <figure class="image is-4by3">
                  <img :src="item.image" :alt="item.name" />
                </figure>
              </div>

              <!-- Contenu de la carte -->
              <div class="card-content">
                <p class="title is-6">{{ item.name }}</p>
                <p class="subtitle is-7" :class="{'has-text-success': item.status === 'Not borrowed', 'has-text-danger': item.status === 'Borrowed'}">
                  {{ item.status }}
                </p>

              </div>
            </div>
          </div>
        </div>


        <!-- Modal pour afficher les détails du téléphone -->
        <div v-if="isModalVisible" class="modal is-active">
          <div class="modal-background" @click="closeModal"></div>
          <div class="modal-content">
            <div class="box">
              <h2 class="title is-4">{{ selectedPhone.name }}</h2>
              <p><strong>Brand:</strong> {{ selectedPhone.brand }}</p>
              <p><strong>Status:</strong> {{ selectedPhone.status }}</p>
              <p><strong>Type:</strong> {{ selectedPhone.type }}</p>
              <figure class="image is-4by3">
                <img :src="selectedPhone.image" alt="Phone Image" />
              </figure>
              <p><strong>Ref : </strong> {{ selectedPhone.ref || "No description available." }}</p>
              <p><strong>Phone : </strong>  01 23 45 67 89 </p>
              <label class="label"> Period :</label>
              <div class="control">
                <input class="input" type="number" placeholder="Enter a number..." v-model.number="userNumber"/>
              </div>

              <button class="button is-link" @click="toggleBorrowStatus(selectedPhone)">
                {{ selectedPhone.status === 'Not borrowed' ? 'Borrow' : 'Return' }}
              </button>
            </div>
          </div>
        </div>

      </div>

    </div>
  </section>
</template>

<script>
import {db} from "@/firebase";
import { useRouter } from 'vue-router';
import {  collection, getDocs , doc, updateDoc  } from "firebase/firestore";


export default {

  name: "MyButton",
  setup() {
    const router = useRouter();
    return { router };
  } ,
  methods: {
    handleClick() {
      this.router.push('/HelloWorld');
    },
    openModal(phone) {
      this.selectedPhone = phone;
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
      this.selectedPhone = {};
      this.inputNumber = null;
    },
    async toggleBorrowStatus(item) {
      if (this.inputNumber === null || this.inputNumber <= 0) {
        alert("Please enter a valid number.");
        return;
      }

      const itemRef = doc(db, "Phones", item.id); // Accéder au document de l'équipement

      await updateDoc(itemRef, {
        Status: item.status === 'Borrowed' ? true : false, // Mettre à jour le statut dans la base de données
        BorrowerId: item.id , Time: this.inputNumber // Mettre à jour l'ID de l'emprunteur
      });

      // Mettre à jour le statut localement pour réactiver la vue
      item.status = item.status === 'Borrowed' ? 'Not borrowed' : 'Borrowed';
    },
    async saveNumber(item) {
      if (this.inputNumber === null || isNaN(this.inputNumber)) {
        alert("Please enter a valid number.");
        return;
      }

      const itemRef = doc(db, "Phones", item.id);
      await updateDoc(itemRef,
          { Time: this.inputNumber });

      alert("Number saved successfully!");
    }
  },


  data() {
    return {
      equipment: [],
      search: "",
      filters: {
        type: [],
        brand: [],
        status: [],
      },
      isModalVisible: false, // Etat du modal
      selectedPhone: {}, // Informations du téléphone sélectionnépas
      inputNumber: null,
    };
  },

  async created() {
    try {
      const querySnapshot = await getDocs(collection(db, "Phones"));
      this.equipment = querySnapshot.docs.map((doc) => ({
        id: doc.id,
        brand: doc.data().Brand,
        name: doc.data().Name,
        status: doc.data().Status ? "Borrowed" : "Not borrowed",
        type: doc.data().Type,
        image: `/images/${doc.data().image}`,
        ref : doc.data().ref ,
        Time : doc.data().Time,
      }));
    } catch (error) {
      console.error("Erreur lors de la récupération des données Firebase :", error);
    }
  },

  computed: {
    filteredEquipment() {
      return this.equipment.filter((item) => {
        const matchesSearch =
            this.search === "" ||
            item.name.toLowerCase().includes(this.search.toLowerCase());
        const matchesType =
            this.filters.type.length === 0 ||
            this.filters.type.includes(item.type);
        const matchesBrand =
            this.filters.brand.length === 0 ||
            this.filters.brand.includes(item.brand);
        const matchesStatus =
            this.filters.status.length === 0 ||
            this.filters.status.includes(item.status);
        return matchesSearch && matchesType && matchesBrand && matchesStatus;
      });
    },
  },


};
</script>

<style scoped>
.menu {
  background-color: white;
  padding: 1rem;
  border-radius: 5px;
}
.box {
  display: flex;
  flex-direction: column; /* Organisation verticale des contenus */
  gap: 10px; /* Espacement entre les éléments */
}



button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  top: 10px;
  right: 10px;
}
.modal {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 600px;
  max-height: 90%;
  overflow-y: auto;
}



</style>
