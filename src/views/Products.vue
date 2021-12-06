<template>
  <Loading :active="this.isLoading"></Loading>
  <div class="text-end">
    <button type="button" class="btn btn-primary mt-3" @click="openModal(true)">新增產品</button>
  </div>
  <table class="table mt-4">
    <thead>
      <tr>
        <th width="120">分類</th>
        <th>產品名稱</th>
        <th width="120">原價</th>
        <th width="120">售價</th>
        <th width="100">是否啟用</th>
        <th width="200">編輯</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(item,key) in products" :key="key">
        <td>{{ item.category }}</td>
        <td>{{ item.title }}</td>
        <td class="text-right">{{ item.origin_price }}</td>
        <td class="text-right">{{ item.price }}</td>
        <td>
          <span class="text-success" v-if="item.is_enabled">啟用</span>
          <span class="text-muted" v-else>未啟用</span>
        </td>
        <td>
          <div class="btn-group">
            <button class="btn btn-outline-primary btn-sm" @click="openModal(false, item)">編輯</button>
            <button class="btn btn-outline-danger btn-sm" @click="openDelProductModal(item)">刪除</button>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
  <ProductModal ref="productModal" :product="tempProduct" @update-product="updateProduct"></ProductModal>
  <DelModal ref="delProductModal" :item="tempProduct" @del-item="delProduct"></DelModal>
</template>

<script>
import ProductModal from '../components/ProductModal.vue'
import DelModal from '../components/DelModal.vue'

export default {
  data () {
    return {
      products: [],
      pagination: {},
      tempProduct: {},
      isNew: false,
      isLoading: false
    }
  },
  components: {
    ProductModal,
    DelModal
  },
  methods: {
    getProducts () {
      const api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/products`
      this.isLoading = true
      this.$http.get(api).then((res) => {
        this.isLoading = false
        if (res.data.success) {
          this.products = res.data.products
          this.pagination = res.data.pagination
        }
      })
    },
    openModal (isNew, item) {
      if (isNew) {
        this.tempProduct = {}
      } else {
        this.tempProduct = { ...item }
      }
      this.isNew = isNew
      const productComponent = this.$refs.productModal
      productComponent.showModal()
    },
    updateProduct (item) {
      this.tempProduct = item
      let api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/product`
      let httpMethod = 'post'
      const productComponent = this.$refs.productModal
      if (!this.isNew) {
        api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/product/${item.id}`
        httpMethod = 'put'
      }
      this.isLoading = true
      this.$http[httpMethod](api, { data: this.tempProduct }).then((res) => {
        this.isLoading = false
        productComponent.hideModal()
        this.getProducts()
      })
    },
    openDelProductModal (item) {
      this.tempProduct = { ...item }
      const delComponent = this.$refs.delProductModal
      delComponent.showModal()
    },
    delProduct () {
      const api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/product/${this.tempProduct.id}`
      this.isLoading = true
      this.$http.delete(api).then((res) => {
        this.isLoading = false
        const delComponent = this.$refs.delProductModal
        delComponent.hideModal()
        this.getProducts()
      })
    }
  },
  mounted () {
    this.getProducts()
  }
}
</script>
