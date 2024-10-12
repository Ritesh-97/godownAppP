<template>
  <div class="layout">
    <header class="header">
      <div>
        <img src="/src/logo.svg" alt="Godown's Logo" style="width: 100px; height: 100px;">
      </div>
      <h1>Godown's Inventories Management</h1>
      <button v-if="isLoggedIn" @click="logout">Logout</button>
            <button @click="showAllItems">Show All Items</button>

      <!-- <input type="text" v-model="searchQuery" placeholder="Search items" @input="filterItems"> -->
      <div class="advanced-search" v-if="isLoggedIn">
     
      <input type="text" v-model="searchQuery" placeholder="Search by name or category">
      <input type="number" v-model="searchQuantity" placeholder="Search by quantity">
      <input type="number" v-model="searchPrice" placeholder="Search by price">
      <input type="text" v-model="searchStatus" placeholder="Search by status">
      <input type="text" v-model="searchBrand" placeholder="Search by brand">
      <input type="text" v-model="searchCategory" placeholder="Search by category">
      <button @click="filterItems">Search</button>
    </div>
    </header>

    

    <div class="content-wrapper">
      <aside class="left-aside" v-if="isLoggedIn">
        <h2>Locations</h2>
        <ul class="tree">
          <li v-for="(node, nodeIndex) in tree" :key="nodeIndex">
            <div @click="toggleNode(node)" class="tree-node">
              {{ node.name }}
              <span>{{ node.collapsed ? '+' : '-' }}</span>
            </div>
            <ul v-if="!node.collapsed" class="children">
              <li v-for="(child, childIndex) in node.children" :key="childIndex">
                <div @click="toggleNode(child)" class="tree-child">
                  {{ child.name }}
                  <span>{{ child.collapsed ? '+' : '-' }}</span>
                </div>
                <ul v-if="!child.collapsed" class="sub-children">
                  <li v-for="(subchild, subchildIndex) in child.children" :key="subchildIndex">
                    <div @click="toggleNode(subchild)" class="tree-subchild">
                      {{ subchild.name }}
                      <span>{{ subchild.collapsed ? '+' : '-' }}</span>
                    </div>
                  </li>
                </ul>
              </li>
            </ul>
          </li>
        </ul>
      </aside>

      <main class="main-content">
  <div v-if="isLoggedIn">
    <h2><u>Items_In_Inventory  </u></h2>
    <div v-if="filteredItems.length > 0">
      <h3>Selected Items</h3>
      <ul>
        <li v-for="item in filteredItems" :key="item.item_id">
          <div class="item">
            <img :src="item.image_url" alt="Item Image">
            <div class="item-details">
              <h4>{{ item.name }}</h4>
              <p>Quantity: {{ item.quantity }}</p>
              <p>Price: {{ item.price }}</p>
              <p>Status: {{ item.status }}</p>
              <p>Brand: {{ item.brand }} </p>
              <p>Category: {{ item.category }}</p>
              <p>Attributes : {{ item.attributes }}</p>
            </div>
          </div>
        </li>
      </ul>
      <!-- Add a button to trigger AI prompt generation -->
      <button @click="generatePrompt">Generate AI Prompt</button>
      <!-- Display the generated AI prompt -->
      <div v-if="generatedPrompt">
        <h3>Generated Prompt</h3>
        <p>{{ generatedPrompt }}</p>
      </div>
    </div>
  </div>
  <LoginComponent v-else />
</main>

    </div>

    <footer class="footer">
      <div class="footer-content">
        <p>Godown's Inventories Management &copy; 2024</p>
        <p class="footer-links">
          <a href="#" @click.prevent="openModal('privacy')">Privacy Policy</a> | 
          <a href="#" @click.prevent="openModal('terms')">Terms of Service</a> | 
          <a href="#" @click.prevent="openModal('contact')">Contact Us</a>
        </p>
      </div>
    </footer>

    <!-- Contact Modal -->
    <div v-if="isModalOpen && modalType === 'contact'" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <span class="close-button" @click="closeModal">&times;</span>
        <h2>Contact Information</h2>
        <p>Email: info@godowns.com</p>
        <p>Phone: +123 456 7890</p>
        <p>Address: 123 Warehouse Lane, Industrial City</p>
      </div>
    </div>

    <!-- Privacy Policy Modal -->
    <div v-if="isModalOpen && modalType === 'privacy'" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <span class="close-button" @click="closeModal">&times;</span>
        <h2>Privacy Policy</h2>
        <p>Your privacy is important to us. We collect and use your personal information to provide services and improve our offerings. Please review our full policy for more details.</p>
      </div>
    </div>

    <!-- Terms of Service Modal -->
    <div v-if="isModalOpen && modalType === 'terms'" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <span class="close-button" @click="closeModal">&times;</span>
        <h2>Terms of Service</h2>
        <p>By using our services, you agree to comply with our terms. Please read them carefully to understand your rights and responsibilities.</p>
      </div>
    </div>

  </div>
</template>

<script>
import axios from 'axios';
import itemsData from '/src/items.json';
import LoginComponent from '/src/ItemDetails.vue'; // Import the login component

export default {
  name: 'VppLayout',
  components: {
    LoginComponent
  },
  data() {
  return {
    nodes: [],
    tree: [],
    items: itemsData, // items.json data
    selectedItems: [],
    isLoggedIn: false,
    searchQuery: '', // Search by name or category
    searchQuantity: null, // Search by quantity
    searchPrice: null, // Search by price
    searchStatus: '', // Search by status
    searchBrand: '', // Search by brand
    searchCategory: '', // Search by category
    generatedPrompt: '', // Store generated prompt here
    isModalOpen: false,
    modalType: '',
    isGeneratingAIcontent: false,
    
  };
},

  computed: {
  filteredItems() {
    return this.selectedItems.filter((item) => {
      const matchesNameOrCategory = item.name.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
                                    item.category.toLowerCase().includes(this.searchQuery.toLowerCase());

      const matchesQuantity = this.searchQuantity === null || item.quantity >= this.searchQuantity;
      const matchesPrice = this.searchPrice === null || item.price <= this.searchPrice;
      const matchesStatus = this.searchStatus === '' || item.status.toLowerCase().includes(this.searchStatus.toLowerCase());
      const matchesBrand = this.searchBrand === '' || item.brand.toLowerCase().includes(this.searchBrand.toLowerCase());
      const matchesCategory = this.searchCategory === '' || item.category.toLowerCase().includes(this.searchCategory.toLowerCase());

      // Return true only if all conditions are met
      return matchesNameOrCategory &&
             matchesQuantity &&
             matchesPrice &&
             matchesStatus &&
             matchesBrand &&
             matchesCategory 
    });
  }
},

  created() {
    fetch('src/godowns.json')
      .then((response) => response.json())
      .then((data) => {
        this.nodes = data;
        this.buildTree();
      });

    // Check if the user is logged in (e.g., using a local storage token)
    this.isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
  },
  methods: {

    openModal(type) {
      this.modalType = type;
      this.isModalOpen = true;
    },
    closeModal() {
      this.isModalOpen = false;
      this.modalType = ''; // Reset modal type
    },



    toggleNode(node) {
      node.collapsed = !node.collapsed;
      if (node.id) {
        this.selectedItems = this.getItemsByGodown(node.id);
      }
    },
    buildTree() {
        const nodeMap = new Map();
        this.nodes.forEach((node) => {
          node.children = [];
          node.collapsed = true;
          nodeMap.set(node.id, node);
        });
        this.nodes.forEach((node) => {
          if (node.parent_godown !== null) {
            const parent = nodeMap.get(node.parent_godown);
            if (parent) {
              parent.children.push(node);
            }
          }
        });
        this.tree = this.nodes.filter((node) => node.parent_godown === null);
    },
    getItemsByGodown(godownId) {
      return this.items.filter((item) => item.godown_id === godownId);
    },
showAllItems() {
      this.selectedItems = this.items; // Show all items by resetting selectedItems to all items
    },
    
    
    generatePrompt() {
      const itemNames = this.selectedItems.map(item => item.name).join(', ');

      // Call the AI API
      axios.post('https://api.openai.com/v1/completions', {
        model: 'text-davinci-003',
        prompt: `Generate a prompt based on the following items: ${itemNames}`,
        max_tokens: 100
      }, {
        headers: {
          Authorization: `Bearer sk-1ToulcG4rJqJ3Na4y4BdEVjb05UL3dNIvvV_t7PF-WT3BlbkFJ89GatC-Y34ssbMEOPsX8F6aAc-HIfRDh3ImeWi0K8A`
        }
      }).then(response => {
        this.generatedPrompt = response.data.choices[0].text;
      }).catch(error => {
        console.error('Error generating prompt:', error);
      });
    },
    logout() {
      localStorage.removeItem('isLoggedIn');
      this.isLoggedIn = false;
    }
  }
};
</script>


<style scoped>
/* General Layout */
.layout {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #f3f4f6;
}

/* Header */
.header {
  background-color: #4a4e69;
  color: #fff;
  text-align: center;
  padding: 1.5rem;
  font-size: 1.8rem;
  width: 85vw;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2), 0 8px 20px rgba(0, 0, 0, 0.1);
}

/* Content Wrapper */
.content-wrapper {
  display: flex;
  flex: 1;
  padding: 2rem;
}

/* Main Content */
.main-content {
  flex: 2;
  padding: 1.5rem;
  background-color: #fafafa;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2), 0 8px 20px rgba(0, 0, 0, 0.1);
  margin: 0 1.5rem;
  border-radius: 10px;
  color: #333;
  overflow-y: auto;
  max-height: 80vh;
}

/* Tree Styles */
.tree {
  list-style: none;
  padding-left: 0;
}

.tree-node, .tree-child {
  cursor: pointer;
  padding: 0.5rem;
  background-color: #e9ecef;
  margin: 0.5rem 0;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
}

.tree-subchild {
  padding-left: 1.5rem;
  padding-top: 0.3rem;
}

.children {
  list-style: none;
  padding-left: 1rem;
}

.sub-children {
  list-style: none;
  padding-left: 1rem;
}

.items {
  list-style: none;
  padding-left: 2rem;
}

.item {
  display: flex;
  margin: 0.5rem 0;
  border-radius: 5px;
  background-color: #f0f0f0;
  padding: 1rem;
}

.item img {
  width: 100px;
  height: 100px;
  object-fit: cover;
  margin-right: 1rem;
}

.item-details {
  flex: 1;
}

/* Aside Styles */
.left-aside {
  flex: 1;
  padding: 1.5rem;
  background-color: #eaeaea;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2), 0 8px 20px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  color: #333;
  overflow-y: auto;
  max-height: 80vh;
}

/* Footer */
.footer {
  background-color: #4a4e69;
  color: #fff;
  text-align: center;
  padding: 1.5rem;
  font-size: 1.2rem;
  margin-top: auto;
}

/* Footer Styles */
.footer {
  background-color: #4a4e69; /* Dark background */
  color: #fff; /* White text */
  text-align: center;
  padding: 2rem 1.5rem;
  font-size: 1.1rem;
  margin-top: auto; /* Pushes footer to the bottom */
  box-shadow: 0 -5px 10px rgba(0, 0, 0, 0.2), 0 -8px 20px rgba(0, 0, 0, 0.1); /* Slight shadow for separation */
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
}

.footer p {
  margin: 0.5rem 0;
}

.footer-links a {
  color: #c9c9ff;
  text-decoration: none;
  margin: 0 0.5rem;
  transition: color 0.3s ease;
}

.footer-links a:hover {
  color: #d4af37; /* Change color on hover */
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .footer {
    font-size: 1rem;
    padding: 1.5rem;
  }

  .footer-links a {
    display: block;
    margin: 0.5rem 0;
  }
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* Make sure the modal is on top of everything */
}

.modal-content {
  background-color: #fff;
  padding: 2rem;
  width: 80%;
  max-width: 500px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  text-align: center;
  position: relative;
  color: #333;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 20px;
  font-size: 1.5rem;
  cursor: pointer;
  color: #333;
}

h2 {
  margin-bottom: 1rem;
}

/* Responsive Modal */
@media (max-width: 600px) {
  .modal-content {
    width: 90%;
    padding: 1.5rem;
  }

  .close-button {
    top: 5px;
    right: 10px;
  }
}


</style>
