<template>
  <q-layout view="hHh lpR fFf">
    <!-- Left Drawer -->
    <q-drawer v-model="leftDrawerOpen" side="left" bordered class="bg-grey-1">
      <q-list>
        <q-item-label header>Navigation</q-item-label>
        <q-item clickable v-ripple>
          <q-item-section avatar>
            <q-icon name="home" />
          </q-item-section>
          <q-item-section>
            Home
          </q-item-section>
        </q-item>
        <q-item clickable v-ripple>
          <q-item-section avatar>
            <q-icon name="shopping_cart" />
          </q-item-section>
          <q-item-section>
            Cart
          </q-item-section>
        </q-item>
        <q-item clickable v-ripple>
          <q-item-section avatar>
            <q-icon name="info" />
          </q-item-section>
          <q-item-section>
            About
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <!-- Navbar -->
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-btn dense flat round icon="menu" @click="toggleLeftDrawer" />
        <q-toolbar-title>
          Store Zikri
        </q-toolbar-title>
        <q-input dense outlined v-model="search" placeholder="Search products" class="q-ml-md q-mr-md">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
        <q-btn dense flat round icon="shopping_cart" />
      </q-toolbar>
    </q-header>

    <!-- Main Content -->
    <q-page-container>
      <q-page padding class="q-pa-md">
        <!-- Carousel -->
        <q-carousel
          v-model="slide"
          arrows
          infinite
          animated
          control-color="black"
          swipeable
          transition-prev="slide-right"
          transition-next="slide-left"
          height="630px"
          class="bg-dark text-white shadow-2 rounded-borders"
        >
          <q-carousel-slide v-for="(image, index) in carouselImages" :key="index" :name="index" :img-src="image">
          </q-carousel-slide>
        </q-carousel>

        <!-- Cards -->
        <div class="q-mt-xl row justify-center">
          <q-card v-for="(product, index) in filteredProducts" :key="product.name" class="my-card q-mb-xl q-pa-md col-xs-12 col-sm-6 col-md-4 col-lg-3" @click="showProductDetails(product)">
            <q-img :src="product.image" :alt="product.name" class="my-card-img">
              <q-badge color="red" floating>{{ product.discount }} Flash Sale</q-badge>
            </q-img>
            <q-card-section>
              <div class="text-h6 text-center">{{ product.name }}</div>
              <div class="text-subtitle1 text-center">{{ product.price }}</div>
            </q-card-section>
            <q-card-actions align="center">
              <q-btn flat label="Add to Cart" color="primary" @click.stop="addToCart(product)" />
              <q-btn flat icon="delete" color="red" @click.stop="deleteProduct(index)" />
            </q-card-actions>
          </q-card>
        </div>
      </q-page>
    </q-page-container>

    <!-- Product Modal -->
    <q-dialog v-model="productModalOpen">
      <q-card>
        <q-card-section>
          <div class="text-h6">{{ selectedProduct.name }}</div>
        </q-card-section>
        <q-img :src="selectedProduct.image" :alt="selectedProduct.name" class="my-card-img q-ma-md"></q-img>
        <q-card-section>
          <div>{{ selectedProduct.price }}</div>
          <div>{{ selectedProduct.discount }} Flash Sale</div>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Close" color="primary" @click="productModalOpen = false" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Snackbar -->
    <q-snackbar v-model="snackbar.open" :color="snackbar.color" :timeout="2000">
      {{ snackbar.message }}
    </q-snackbar>

    <!-- Footer -->
    <q-footer class="bg-blue-8 text-white">
      <q-toolbar>
        <q-toolbar-title>
          &copy; StoreZikri
        </q-toolbar-title>
        <div>
          <q-btn flat round icon="facebook" />
          <q-btn flat round icon="instagram" />
        </div>
      </q-toolbar>
    </q-footer>
  </q-layout>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    const leftDrawerOpen = ref(false);
    const slide = ref(0);
    const search = ref('');
    const carouselImages = [
      'https://i.pinimg.com/564x/f0/6a/62/f06a62f973a6fb6a7d594513b635d9f8.jpg',
      'https://i.pinimg.com/564x/ac/9b/c1/ac9bc1e7593290bbb6bfbd0ea4b189e9.jpg',
      'https://i.pinimg.com/736x/cb/0b/f8/cb0bf82afc60f4b8c9211c5f0df0c281.jpg',
      'https://i.pinimg.com/564x/9a/fd/f8/9afdf85ce320f90b6abc208fecdfe9cf.jpg'
    ];
    const products = ref([
      { name: 'Sepatu 1', price: 'Rp. 1.500.000K', image: 'https://i.pinimg.com/564x/9b/6e/86/9b6e86e3fd0a054989b04cb2bec98ab0.jpg', discount: '10%' },
      { name: 'Sepatu 2', price: 'Rp. 1.700.000K', image: 'https://i.pinimg.com/564x/e7/a4/99/e7a4997996af84b5e8b6194afbb1f143.jpg', discount: '15%' },
      { name: 'Sepatu 3', price: 'Rp. 1.850.000K', image: 'https://i.pinimg.com/564x/e9/3e/5a/e93e5aded795c9e60c95231fde2a4c9c.jpg', discount: '20%' },
      { name: 'Sepatu 4', price: 'Rp. 1.900.000K', image: 'https://i.pinimg.com/736x/5e/91/d0/5e91d01f03e3c89e5f51dd8ca9d2165c.jpg', discount: '25%' }
    ]);
    const productModalOpen = ref(false);
    const selectedProduct = ref({});
    const snackbar = ref({ open: false, message: '', color: '' });

    const toggleLeftDrawer = () => {
      leftDrawerOpen.value = !leftDrawerOpen.value;
    };

    const deleteProduct = (index) => {
      products.value.splice(index, 1);
      snackbar.value = { open: true, message: 'Product deleted', color: 'negative' };
    };

    const addToCart = (product) => {
      snackbar.value = { open: true, message: `${product.name} added to cart`, color: 'positive' };
    };

    const showProductDetails = (product) => {
      selectedProduct.value = product;
      productModalOpen.value = true;
    };

    const filteredProducts = computed(() => {
      return products.value.filter(product => product.name.toLowerCase().includes(search.value.toLowerCase()));
    });

    return {
      leftDrawerOpen,
      toggleLeftDrawer,
      slide,
      search,
      carouselImages,
      products,
      deleteProduct,
      addToCart,
      showProductDetails,
      productModalOpen,
      selectedProduct,
      snackbar,
      filteredProducts
    };
  }
};
</script>

<style>
.my-card {
  width: 100%;
  max-width: 500px;
  margin: 0 auto;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s;
}
.my-card:hover {
  transform: translateY(-10px);
}
.my-card-img {
  border-radius: 10px 10px 0 0;
  height: 200px;
}
.rounded-borders {
  border-radius: 10px;
}
.q-footer {
  border-top: 1px solid #003285;
}
</style>
