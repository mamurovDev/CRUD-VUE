<template>
  <div id="app">
    <p class="text-red-500">{{ error }}</p>
    <form
      class="flex flex-col w-full items-center"
      @submit.prevent="
        !isUpdating
          ? request('POST')
          : request('PUT', JSON.stringify(formValues), formValues.id)
      "
    >
      <h2 class="text-xl">Create a new product...</h2>
      <label for="name" class="mt-3 mb-1 flex items-center justify-between">
        Name:
        <input
          required
          id="name"
          type="text"
          placeholder="Type a name..."
          v-model.trim.lazy="formValues.name_uz"
          class="border rounded px-3 py-1 ml-2"
        />
      </label>
      <label
        for="productType"
        class="mt-3 mb-1 flex items-center justify-between"
      >
        Product Type:
        <select
          id="productType"
          required
          class="rounded border px-3 py-1 ml-2"
          v-model="formValues.product_type_id"
        >
          <option disabled selected>Select product type</option>
          <option v-for="prod in productTypes" :key="prod.id" :value="prod.id">
            {{ prod.name_uz }}
          </option>
        </select>
      </label>
      <label for="cost" class="mt-3 mb-1 flex items-center justify-between">
        Cost:
        <input
          required
          id="cost"
          type="text"
          placeholder="Enter cost"
          v-model.trim.lazy.number="formValues.cost"
          class="border rounded px-3 py-1 ml-2 flex items-center justify-between"
        />
      </label>
      <label for="address" class="mt-3 mb-1 flex items-center justify-between">
        Address:
        <input
          required
          id="address"
          type="text"
          placeholder="Enter an address..."
          v-model.trim.lazy="formValues.address"
          class="border rounded px-3 py-1 ml-2 flex items-center justify-between"
        />
      </label>
      <button
        v-if="!isUpdating"
        type="submit"
        class="mt-4 bg-green-500 px-3 py-1.5 rounded-xl text-white"
      >
        Submit
      </button>
      <button
        v-if="isUpdating"
        class="mt-4 bg-green-500 px-3 py-1.5 rounded-xl text-white"
      >
        Uptade
      </button>
    </form>

    <div v-for="product in products" :key="product.id" class="card">
      <div class="flex justify-between">
        <strong class="text-xl">Name: {{ product.name_uz }}</strong>
        <div>Product Type ID: {{ product.product_type_id }}</div>
      </div>
      <p>ID: {{ product.id }}</p>
      <p>Cost: {{ product.cost }}</p>
      <address>Address: {{ product.address }}</address>
      <div class="flex items-center justify-between">
        <time datetime="">Created Date: {{ product.created_date }}</time>
        <div class="w-40 flex justify-around">
          <button
            class="p-2 rounded-md bg-orange-400 text-white"
            @click="
              formValues = product;
              isUpdating = true;
            "
          >
            Edit
          </button>
          <button class="danger" @click="request('DELETE', null, product.id)">
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
const api = axios.create({
  baseURL: "http://94.158.54.194:9092",
});
export default {
  name: "App",
  components: {},
  data() {
    return {
      isUpdating: false,
      error: "",
      products: [],
      productTypes: [],
      formValues: {
        name_uz: "",
        product_type_id: null,
        cost: null,
        address: "",
        created_date: Date.now(),
      },
    };
  },
  methods: {
    emptyForm() {
      this.formValues = {
        name_uz: "",
        product_type_id: null,
        cost: null,
        address: "",
      };
    },
    async request(method, body, id) {
      try {
        let res;
        switch (method) {
          case "GET":
            res = await api.get("/api/product/");
            this.products = res.data;
            break;
          case "POST":
            res = await api.post("/api/product/", this.formValues);
            this.products.push(this.formValues);
            this.emptyForm();
            setTimeout(() => {
              this.request("GET", null, null);
            }, 0);
            break;
          case "DELETE":
            res = await api.delete(`/api/product/${id}`);
            this.products = this.products.filter(
              (product) => product.id !== id
            );
            break;
          case "PUT":
            res = await api.put(`/api/product`, body, {
              headers: {
                "Content-Type": "application/json",
                accept: "application/json",
              },
            });
            this.emptyForm();
            break;
          default:
            this.error = "Invalid request method: " + method;
            console.error("Invalid request method:", method);
        }
      } catch (error) {
        this.error = "Something went wrong with the request";
        console.error("Something went wrong with the request: ", error);
      }
    },
    async getProductTypes() {
      try {
        const res = await api.get("/api/product/get-product-types");
        this.productTypes = res.data;
      } catch (error) {
        this.error = "Error fetching product types:";
        console.error("Error fetching product types: ", error);
      }
    },
  },

  mounted() {
    this.request("GET");
    this.getProductTypes();
  },
};
</script>

<style>
@import url(./style.css);
body {
  display: flex;
  align-items: center;
  flex-direction: column;
  background: #d0cba7;
}
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 60%;
}

form input,
select {
  width: 60%;
}
.card {
  width: 100%;
  border: 3px solid rgb(76, 37, 37);
  border-radius: 15px;
  padding: 20px;
  margin-top: 20px;
}

.danger {
  background-color: rgb(184, 3, 3);
  color: white;
  border: none;
  border-radius: 5px;
  padding: 7px;
  cursor: pointer;
}

@media (min-width: 300px) and (max-width: 700px) {
  #app {
    width: 90%;
  }
}
</style>
