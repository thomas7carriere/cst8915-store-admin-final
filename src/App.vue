<template>
  <div id="app">
    <header class="topbar">
      <div class="brand">
        <div class="logo-box">BEST BUY</div>
        <div>
          <h1>Store Admin</h1>
          <p>Manage inventory and review customer orders</p>
        </div>
      </div>
    </header>

    <main class="content">
      <div class="tabs">
        <button
          :class="{ active: activeTab === 'products' }"
          @click="activeTab = 'products'"
        >
          Products
        </button>
        <button
          :class="{ active: activeTab === 'orders' }"
          @click="activeTab = 'orders'"
        >
          Orders
        </button>
      </div>

      <div v-if="message" class="message">
        {{ message }}
      </div>

      <section v-if="activeTab === 'products'">
        <div class="section-header">
          <h2>Product Inventory</h2>
          <button @click="fetchProducts">Refresh Products</button>
        </div>

        <div v-if="loadingProducts">Loading products...</div>
        <div v-else class="card-grid">
          <div v-for="product in products" :key="product.id" class="card">
            <h3>{{ product.name }}</h3>
            <p><strong>Brand:</strong> {{ product.brand }}</p>
            <p><strong>Category:</strong> {{ product.category }}</p>
            <p><strong>Price:</strong> ${{ Number(product.price).toFixed(2) }}</p>
            <p><strong>Current Stock:</strong> {{ product.stock }}</p>

            <div class="stock-edit">
              <label :for="`stock-${product.id}`">New Stock</label>
              <input
                :id="`stock-${product.id}`"
                type="number"
                min="0"
                v-model.number="stockInputs[product.id]"
              />
              <button @click="updateStock(product.id)">Update Stock</button>
            </div>
          </div>
        </div>
      </section>

      <section v-if="activeTab === 'orders'">
        <div class="section-header">
          <h2>Orders</h2>
          <button @click="fetchOrders">Refresh Orders</button>
        </div>

        <div v-if="loadingOrders">Loading orders...</div>
        <div v-else class="orders-table-wrap">
          <table class="orders-table">
            <thead>
              <tr>
                <th>Customer</th>
                <th>Total</th>
                <th>Status</th>
                <th>Created</th>
                <th>Items</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(order, index) in orders" :key="index">
                <td>{{ order.customerName }}</td>
                <td>${{ Number(order.total).toFixed(2) }}</td>
                <td>
                  <span class="status" :class="order.status">
                    {{ order.status }}
                  </span>
                </td>
                <td>{{ formatDate(order.createdAt) }}</td>
                <td>
                  <ul>
                    <li v-for="(item, i) in order.items" :key="i">
                      {{ item.name }} x{{ item.quantity }}
                    </li>
                  </ul>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      activeTab: "products",
      products: [],
      orders: [],
      stockInputs: {},
      loadingProducts: false,
      loadingOrders: false,
      message: ""
    };
  },
  methods: {
    async fetchProducts() {
      this.loadingProducts = true;
      try {
        const response = await fetch(
          `${process.env.VUE_APP_PRODUCT_SERVICE_URL}/products`
        );

        if (!response.ok) {
          throw new Error("Failed to fetch products");
        }

        const data = await response.json();
        this.products = data;

        data.forEach((product) => {
          this.stockInputs[product.id] = product.stock;
        });
      } catch (err) {
        this.message = `Error: ${err.message}`;
      } finally {
        this.loadingProducts = false;
      }
    },

    async fetchOrders() {
      this.loadingOrders = true;
      try {
        const response = await fetch(
          `${process.env.VUE_APP_ORDER_SERVICE_URL}/orders`
        );

        if (!response.ok) {
          throw new Error("Failed to fetch orders");
        }

        const data = await response.json();
        this.orders = data;
      } catch (err) {
        this.message = `Error: ${err.message}`;
      } finally {
        this.loadingOrders = false;
      }
    },

    async updateStock(productId) {
      try {
        const response = await fetch(
          `${process.env.VUE_APP_PRODUCT_SERVICE_URL}/products/${productId}/stock`,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json"
            },
            body: JSON.stringify({
              stock: this.stockInputs[productId]
            })
          }
        );

        if (!response.ok) {
          throw new Error("Failed to update stock");
        }

        this.message = `Stock updated for ${productId}`;
        await this.fetchProducts();

        setTimeout(() => {
          this.message = "";
        }, 3000);
      } catch (err) {
        this.message = `Error: ${err.message}`;
      }
    },

    formatDate(value) {
      if (!value) return "";
      return new Date(value).toLocaleString();
    }
  },
  mounted() {
    this.fetchProducts();
    this.fetchOrders();
  }
};
</script>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, Helvetica, sans-serif;
  background: #f4f6f8;
  color: #111827;
}

.topbar {
  background: #003b64;
  color: white;
  padding: 20px 32px;
  border-bottom: 5px solid #ffe000;
}

.brand {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  gap: 18px;
  align-items: center;
}

.logo-box {
  background: #ffe000;
  color: #111;
  font-weight: 800;
  padding: 12px 16px;
  border-radius: 4px;
  transform: rotate(-8deg);
}

.brand h1 {
  margin: 0;
}

.brand p {
  margin: 4px 0 0;
}

.content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 28px 32px 48px;
}

.tabs {
  display: flex;
  gap: 12px;
  margin-bottom: 24px;
}

.tabs button {
  border: none;
  background: #dbe4ea;
  padding: 10px 18px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 700;
}

.tabs button.active {
  background: #003b64;
  color: white;
}

.message {
  background: #e7f6ea;
  border: 1px solid #b8e0c0;
  color: #14532d;
  padding: 12px 14px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 18px;
}

.section-header button {
  background: #ffe000;
  border: none;
  padding: 10px 14px;
  border-radius: 8px;
  font-weight: 700;
  cursor: pointer;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
}

.card {
  background: white;
  border-radius: 12px;
  padding: 18px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.08);
}

.stock-edit {
  margin-top: 14px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.stock-edit input {
  padding: 8px;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
}

.stock-edit button {
  background: #003b64;
  color: white;
  border: none;
  padding: 10px 12px;
  border-radius: 8px;
  cursor: pointer;
}

.orders-table-wrap {
  overflow-x: auto;
  background: white;
  border-radius: 12px;
  padding: 12px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.08);
}

.orders-table {
  width: 100%;
  border-collapse: collapse;
}

.orders-table th,
.orders-table td {
  padding: 12px;
  border-bottom: 1px solid #e5e7eb;
  text-align: left;
  vertical-align: top;
}

.status {
  display: inline-block;
  padding: 6px 10px;
  border-radius: 999px;
  font-size: 0.85rem;
  font-weight: 700;
  text-transform: capitalize;
}

.status.pending {
  background: #fef3c7;
  color: #92400e;
}

.status.completed {
  background: #dcfce7;
  color: #166534;
}

.status.rejected {
  background: #fee2e2;
  color: #991b1b;
}
</style>