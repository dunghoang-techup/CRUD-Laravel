<template>
    <div class="api-calling">
        <h1>CREATE PRODUCT</h1>
        <div class="error" v-if="errors.length">
            <span v-for="(err, index) in errors" :key="index">
                {{err}}
            </span>
            <hr>
        </div>
        <div class="form-group">
            <div class="input-group mb-3">
                <div class="input-group-prepend">
                    <span class="input-group-text" >Name</span>
                </div>
                <input type="text" v-model="product.name" class="form-control" placeholder="Product">
            </div>
            <div class="input-group mb-3">
                <div class="input-group-prepend">
                    <span class="input-group-text">Price</span>
                </div>
                <input type="number" class="form-control" v-model.number="product.price" placeholder="0">
            </div>
            <div class="button-create">
                <button class="btn btn-outline-primary" @click="createProduct">Create</button>
            </div>
        </div>
        <hr>
        <div class="list-products">
            <h2>LIST PRODUCT</h2>
            <div class="product-table">
                <table class="table table-bordered">
                    <thead class="thead-dark">
                    <tr>
                        <th>ID</th>
                        <th>Name</th>
                        <th>Price</th>
                        <th>Date Created</th>
                        <th>Actions</th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="(prod, index) in list_products" :key="prod.id">
                        <td>{{prod.id}}</td>
                        <td v-if="!prod.isEdit">
                            {{prod.name}}
                        </td>
                        <td v-else>
                            <input type="text" class="form-control" v-model="selectedProduct.name">
                        </td>
                        <td v-if="!prod.isEdit">
                            {{prod.price}}
                        </td>
                        <td v-else>
                            <input type="text" class="form-control" v-model.number="selectedProduct.price">
                        </td>
                        <td>{{prod.created_at}}</td>
                        <td v-if="!prod.isEdit">
                            <button class="btn btn-success" @click="selectProduct(prod)">Edit</button>
                        </td>
                        <td v-else>
                            <button class="btn btn-primary" @click="updateProduct(index)">Save</button>
                            <button class="btn btn-danger" @click="prod.isEdit = false">Cancel</button>
                        </td>
                    </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                product: {
                    name: '',
                    price: ''
                },
                selectedProduct: {},
                errors: [],
                list_products: []
            }
        },
        created() {
            this.getListProducts();
        },
        methods: {
            createProduct() {
                this.errors= [];
                axios.post('/products', {name: this.product.name, price: this.product.price})
                    .then(response => {
                        this.list_products.push({...response.data.product, isEdit: false});
                        alert('Created!');
                    })
                    .catch(err => this.errors= err.response.data.errors.name)
            },
            getListProducts() {
                axios.get('/products')
                .then(res => {
                    this.list_products = res.data;
                    this.list_products.forEach(item => {
                        Vue.set(item, 'isEdit', false);
                    })
                })
                .catch(err => {
                    console.log(err.response.data.errors.name);
                })
            },
            selectProduct(product) {
                this.selectedProduct = { ...product };
                product.isEdit = true;
            },
            updateProduct(productIndex) {
                axios.put('/products/'+ this.selectedProduct.id, {name: this.selectedProduct.name, price: this.selectedProduct.price})
                    .then(res => {
                        this.list_products[productIndex].name = this.selectedProduct.name;
                        this.list_products[productIndex].price = this.selectedProduct.price;
                        this.list_products[productIndex].isEdit = false;
                        console.log(res.data);
                    })
                    .catch(err => this.errors = err.response.data.errors.name)
            }
        }
    }
</script>

<style lang="scss" scoped>
.error {
    span {
        color: red;
    }
}
.form-group {
    width: fit-content;
}

</style>
