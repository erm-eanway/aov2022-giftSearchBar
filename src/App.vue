<script setup lang="ts">
import { reactive, watch } from 'vue'
import spinner from './components/spinner.vue'

// expected product format from database
type Product = {
  id: number
  title: string
  price: number
}

const state = reactive({
  // create a search term reference string
  // because it will change as the user searches
  searchTerm: '',
  // loading will change while typing and getting data
  loading: false,
  // products will change based on search term
  products: [] as Product[],
})

const debounce = (func: Function, timeout: number) => {
  let timer: number
  return (...args) => {
    clearTimeout(timer)
    timer = setTimeout(() => {
      func.apply(this, args)
    }, timeout)
  }
}

const fetchProducts = async (term: string) => {
  console.log(term)
  const res = await fetch(`https://dummyjson.com/products/search?q=${term}`)
  const json = await res.json()
  return json.products as Product[]
}

const findProducts = async term => {
  // no action for empty search
  // need to have empty string check inside debounced function
  // otherwise going from one to no characters (e.g., backspacing)
  // would skip over the empty string and return the results for
  // the last, non-empty string
  if (term) {
    state.products = await fetchProducts(term)

    // alert needs to be inside debounced function
    // otherwise it will appear before products are updated
    // however, loading spinner still appears when alert is up
    if (!state.products.length) {
      alert('No products found. Please try another search term.')
    }
  }

  // unloading needs to be inside the debounced function
  // otherwise it would immediately unload
  state.loading = false
}

// need to wrap the watched function in debounce
// to avoid calling the database before typing is complete
const findProductsDB = debounce(findProducts, 500)

// watch the search term for changes, when it does,
// find the products for that new term
watch(
  // need an empty function to watch a reactive property
  () => state.searchTerm,
  newTerm => {
    // reset the intermediate product array so old products don't appear
    state.products = []

    // start loading spinner immediately
    // must be outside debounced function
    state.loading = true
    findProductsDB(newTerm)
  }
)
</script>

<template>
  <div class="w-full h-full flex flex-col gap-5 justify-center items-center">
    <h1 class="text-4xl font-bold">Gift Search Bar</h1>
    <input type="text" class="p-2 border-2 border-gray-dark" v-model="state.searchTerm" placeholder="Start typing..." />
    <!--If loading-->
    <spinner v-if="state.loading" />
    <!--Otherwise if non-empty search term AND no results-->
    <p v-else-if="(state.searchTerm != '') & !state.products.length">No results found.</p>
    <!--Otherwise empty search term OR results-->
    <ul v-else class="list-disc">
      <li v-for="product in state.products" :key="product.id">{{ product.title }} - ${{ product.price }}</li>
    </ul>
  </div>
</template>
