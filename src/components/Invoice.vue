<script>
  import { ref } from "vue";
  import { onValue, update, push 
  } from '@firebase/database';

  import itemsRef from "../firebase";

  export default {
    setup() {
      const invoice_name = ref("");
      const form_invoice_name = ref(null);
      const search_value = ref("");
      const sub_total = ref(0);
      const tax = ref(0);
      const editing = ref(false);
      const key = ref(null);
      const row_data = ref([
        {
          id: 1,
          name: '',
          price: null,
          amount: null
        },
        {
          id: 2,
          name: '',
          price: null,
          amount: null
        }
      ]);
      const headers = [
        { text: "Item Name", value: "name", sortable: true },
        { text: "Item Price", value: "price", sortable: true },
        { text: "Amount", value: "amount", sortable: true },
      ];
      const items = ref([]);
      const headerItemClassNameFunction = (header, index) => {
        return "alpha-primary text-primary font-weight-bold";
      };
      return {
        headers,
        items,
        headerItemClassNameFunction,
        search_value,
        invoice_name,
        form_invoice_name,
        row_data,
        sub_total,
        tax,
        editing,
        key
      };
    },
    mounted() {
      this.getInvoices()
    },
    methods: {
      async resetForm() {
        this.row_data = [
          {
            id: this.items.length + 1,
            name: '',
            price: null,
            amount: null
          },
          {
            id:  this.items.length + 2,
            name: '',
            price: null,
            amount: null
          }
        ]
        this.sub_total = 0
        this.tax = 0
        this.form_invoice_name = null
      },
      formatData(datas) {
        for (let key in datas) {
          // console.log(datas, key, datas[key], '-------')
          const array = datas[key];
          array.key = key
          // const data = {}
          // this.invoice_name = datas[key][0]
          // this.items.push(datas[key][1])
          this.items.push(array)
        }
      },
      async getInvoices() {
        onValue(itemsRef, (snapshot) => {
          const data = snapshot.val();
          this.formatData(data);
        })

      },
      async addNewItem() {
        if (this.form_invoice_name !== null){
          const items_arr = {
            invoice_name: this.form_invoice_name,
            items: []
          };
          this.row_data.map(item => {
            if (item.name !== '' && item.price !== null && item.amount !== null) {
              this.sub_total = this.sub_total + (item.price * item.amount);
              items_arr.items.push(item)
            }
          })

          if (this.editing) {
            const newPostKey = this.key;
            const updates = {};
            updates[newPostKey] = items_arr;
            update(itemsRef, updates);
          } else {
            const newPostKey = push(itemsRef, 'items').key;
            const updates = {};
            updates[newPostKey] = items_arr;
            update(itemsRef, updates);
          }
        }
      },
      addNewItemRow() {
        const my_object = {
          name: '',
          price: 0,
          amount: 0
        }
        this.row_data.push(my_object);
      },
      deleteRow(index){    
        this.row_data.splice(index,1);             
      },  
      editInvoice(item) {
        this.editing = true;
        this.key = item.key;
        this.form_invoice_name = item.invoice_name;
        this.row_data = item.items;
      },  
      log (value) {
        console.log(value);
      }
    },
  }
</script>

<template>
  <div class="my-3 mx-5">
    <div v-for="item in items" class="mt-10 mb-20">
      <!-- invoice name and search -->
      <div class="mb-5 space-y-5">
        <div class="flex flex-row justify-around">
          <h1 class="text-xl font-bold">
            {{ item.invoice_name }}
          </h1>
          <p class="text-lg font-bold cursor-pointer"
            @click="editInvoice(item)"
          >
            Edit
          </p>
        </div>
        <input type="search" 
          v-model="search_value"
          class="rounded-lg border p-2" 
          placeholder="Search"
        />
      </div>
      <!-- invoice name and search -->

      <EasyDataTable
        :headers="headers"
        :items="item.items"
        :header-item-class-name="headerItemClassNameFunction"
        :search-value="search_value"
        rows-per-page="10"
        buttons-pagination="true"
        theme-color="#005E97"
        show-index
        alternating
      >
        <template #loading>
          <img
            src="https://i.pinimg.com/originals/94/fd/2b/94fd2bf50097ade743220761f41693d5.gif"
            style="width: 100px; height: 80px"
          />
        </template>
      </EasyDataTable>
    </div>

    <!-- Add New Invoice -->
    <div class="mt-10 mb-5">
      <p class="font-bold text-lg mb-7">
        New Invoice
      </p>
      <div class="flex flex-row items-center space-x-3">
        <p class="font-bold">Invoice Name</p>
        <input type="text" required
          v-model="form_invoice_name" 
          class="rounded-lg border p-2"
          :class="[ form_invoice_name === null && 'required:border-red-500' ]" 
        />
      </div>

      <!-- Form Table -->
      <table class="w-full">
        <thead class="bg-white border-b">
          <tr>
            <th scope="col" class="text-sm font-medium text-gray-900 p-4 text-left">
              Item Name
            </th>
            <th scope="col" class="text-sm font-medium text-gray-900 p-4 text-left">
              0 of Item
            </th>
            <th scope="col" class="text-sm font-medium text-gray-900 p-4 text-left">
              Price
            </th>
            <th scope="col" class="text-sm font-medium text-gray-900 p-4 text-left w-32">
              Total
            </th>
            <th scope="col" class="text-sm font-medium text-gray-900 p-4 text-left w-28">
              Action
            </th>
          </tr>
        </thead>
        <tbody>
          <tr 
            v-for="(item, index) in row_data"
            class=""
          >
            <input
              type="hidden"
              v-model="item.id"              
            />
            <td class="p-4 whitespace-nowrap text-sm font-medium text-gray-900">
              <input type="text" 
                v-model="item.name"
                class="rounded-lg border p-2" 
              />
            </td>
            <td class="text-sm text-gray-900 font-light p-4 whitespace-nowrap">
              <input type="number" 
                v-model="item.price"
                class="rounded-lg border p-2" 
              />
            </td>
            <td class="text-sm text-gray-900 font-light p-4 whitespace-nowrap">
              <input type="number" 
                v-model="item.amount"
                class="rounded-lg border p-2" 
              />
            </td>
            <td class="text-sm text-gray-900 font-light p-4 whitespace-nowrap">
              {{ 
                Object.keys(item).length > 0 ? 
                item.price * item.amount 
                : null 
              }}
            </td>
            <td class="text-sm text-gray-900 font-light p-4 whitespace-nowrap cursor-pointer">
              <span @click="deleteRow(index)">
                X
              </span>
            </td>
          </tr>
        </tbody>
      </table>

      <div class="flex flex-row space-x-5">
        <div 
          class="w-28 border border-gray-900 rounded-lg text-black text-20 p-2 text-center cursor-pointer"
          @click="addNewItemRow"
        >
          Add Item
        </div>
        <div 
          class="w-28 border border-gray-900 rounded-lg text-black text-20 p-2 text-center cursor-pointer"
          @click="resetForm"
        >
          Reset
        </div>
      </div>
      
      <!-- sub total, tax and total -->
      <div class="w-full">
        <div class="w-1/3 m-5 space-y-5 margin-auto">
          <div class="flex flex-row">
            <div class="w-24">
              <p class="text-black text-16">Sub Total</p>
            </div>
            <div class="w-full">
              <p class="text-right">
                {{ this.sub_total }}
              </p>
            </div>
          </div>
          <div class="flex flex-row">
            <div class="w-24">
              <p class="text-black text-16">Tax</p>
            </div>
            <div class="w-full">
              <p class="text-right">
                {{ this.sub_total * 0.05 }}
              </p>
            </div>
          </div>
          <div class="flex flex-row">
            <div class="w-24">
              <p class="text-black text-16">Total</p>
            </div>
            <div class="w-full">
              <p class="text-right">
                {{ this.sub_total * 0.05 + this.sub_total }}
              </p>
            </div>
          </div>
        </div>
      </div>
      <!-- sub total, tax and total -->

      <div
        v-if="editing" 
        class="w-28 border border-gray-900 rounded-lg text-black text-20 p-2 text-center cursor-pointer"
        @click="addNewItem"
      >
        Update
      </div>
      <div
        v-else 
        class="w-28 border border-gray-900 rounded-lg text-black text-20 p-2 text-center cursor-pointer"
        @click="addNewItem"
      >
        Create
      </div>
    </div> 

  </div>
</template>

<style scoped>
  .margin-auto {
    margin-left: auto;
  }
</style>