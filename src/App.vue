<script setup lang="ts">
import { reactive, watch } from 'vue'
import spinner from './components/spinner.vue'

type Product = {
  id: number
  title: string
  price: number
}

// create a search term reference string
// because it will change as the user searches
const state = reactive({
  searchTerm: '',
  loading: false,
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
  const res = await fetch(`https://dummyjson.com/products/search?q=${term}`)
  const json = await res.json()
  return json.products as Product[]
}

const findProducts = async term => {
  if (!term) {
    state.products = []
    return
  }

  state.loading = true
  state.products = await fetchProducts(term)
  state.loading = false
}

// need to wrap the watched function in debounce
const findProductsDB = debounce(findProducts, 300)

// watch the search term for changes, when it does,
// find the products for that new term
// with a 300 ms debounce
watch(
  // need an empty function to watch a reactive property
  () => state.searchTerm,
  newTerm => findProductsDB(newTerm)
)
</script>

<template>
  <div class="w-full h-full flex flex-col gap-5 justify-center items-center">
    <h1 class="text-4xl font-bold">Gift Search Bar</h1>
    <input type="text" class="p-2 border-2 border-gray-dark" v-model="state.searchTerm" placeholder="Start typing..." />
    <spinner v-show="state.loading" />
    <ul class="list-disc">
      <li v-for="product in state.products" :key="product.id">
        {{ product.title }}
      </li>
    </ul>
  </div>
</template>
