<template>
    <div v-show="list.length">
        <div class="list-control">
            <div class="list-control-filter">
                <span>品牌：</span>
                <span class="list-control-filter-item"
                      v-for="item in brands"
                      :class="{ on: filterBrand.find(brand => brand === item) }"
                      @click="handleFilterBrand(item)">{{ item }}</span>
            </div>
            <div class="list-control-filter">
                <span>颜色：</span>
                <span class="list-control-filter-item"
                      v-for="item in colors"
                      :class="{ on: filterColor.find(color => item === color) }"
                      @click="handleFilterColor(item)">{{ item }}</span>
            </div>
            <div class="list-control-order">
                <span>排序：</span>
                <span class="list-control-order-item"
                      :class="{ on: !order }"
                      @click="handleOrderDefault">默认</span>
                <span class="list-control-order-item"
                      :class="{ on: order === 'sales' }"
                      @click="handleOrderSales">销量
                    <template v-if="order === 'sales'">↓</template>
                </span>
                <span class="list-control-order-item"
                      :class="{ on: order.indexOf('cost') > -1 }"
                      @click="handleOrderCost">价格
                    <template v-if="order === 'cost-desc'">↑</template>
                    <template v-if="order === 'cost-asc'">↓</template>
                </span>
            </div>
        </div>
        <Product v-for="item in filteredAndOrderedList" :info="item" :key="item.id"></Product>
        <div class="product-not-found"
             v-show="!filteredAndOrderedList.length">暂无相关商品
        </div>
    </div>
</template>

<script>
    import Product from '../components/product.vue';

    export default {
        components: {Product},
        data() {
            return {
                /* 排序依据，可选值为
                 sales(销量)
                 cost-desc (价格降序)
                 cost-asc (价格升序)*/
                order: '',
                /*filterBrand: '',
                filterColor: '',*/
                filterBrand: [],
                filterColor: [],
                currentList: []
            }
        },
        computed: {
            list() {
                return this.$store.state.productList;
            },
            brands() {
                return this.$store.getters.brands;
            },
            colors() {
                return this.$store.getters.colors;
            },
            filteredAndOrderedList() {
                // 复制原始数据
                let list = [...this.list];
                // 按品牌过滤
                if (this.filterBrand.length) {
                    // list = list.filter(item => item.brand === this.filterBrand);
                    let brand_list = [];
                    this.filterBrand.find(brand => {
                        list.map(item => {
                            brand === item.brand && brand_list.push(item);
                        });
                    });
                    list = [...brand_list];
                }
                // 按颜色过滤
                if (this.filterColor.length) {
                    // list = list.filter(item => item.color === this.filterColor);
                    let color_list = [];
                    this.filterColor.find(color => {
                        list.map(item => {
                            color === item.color && color_list.push(item);
                        });
                    });
                    list = [...color_list];
                }
                // 排序
                if (this.order) {
                    switch (this.order) {
                        case 'sales':
                            list = list.sort((a, b) => b.sales - a.sales);
                            break;
                        case 'cost-desc':
                            list = list.sort((a, b) => b.cost - a.cost);
                            break;
                        case 'cost-asc':
                            list = list.sort((a, b) => a.cost - b.cost);
                            break;
                    }
                }
                return list;
            }
        },
        methods: {
            handleOrderDefault() {
                this.order = '';
            },
            handleOrderSales() {
                this.order = 'sales';
            },
            handleOrderCost() {
                this.order = this.order === 'cost-desc' ? 'cost-asc' : 'cost-desc';
            },
            handleFilterBrand(brand) {
                // this.filterBrand = this.filterBrand === item ? '' : item;
                let filter_brand = this.filterBrand.find(item => item === brand);
                if (filter_brand) {
                    let index = this.filterBrand.findIndex(item => item === brand);
                    this.filterBrand.splice(index, 1);
                } else {
                    this.filterBrand.push(brand);
                }
            },
            handleFilterColor(color) {
                // this.filterColor = this.filterColor === item ? '' : item;
                let filter_colors = this.filterColor.find(item => item === color)
                if (filter_colors) {
                    let index = this.filterColor.findIndex(item => item === color);
                    this.filterColor.splice(index, 1);
                } else {
                    this.filterColor.push(color);
                }
            }
        },
        mounted() {
            // 初始化时， 通过Vuex的action请求数据
            this.$store.dispatch('getProductList');
        }
    }
</script>

<style scoped>

    .list-control {
        background: #fff;
        border-radius: 6px;
        margin: 16px;
        padding: 16px;
        box-shadow: 0 1px 1px rgba(0, 0, 0, .2);
    }

    .list-control-filter {
        margin-bottom: 16px;
    }

    .list-control-filter-item,
    .list-control-order-item {
        cursor: pointer;
        display: inline-block;
        border: 1px solid #e9eaec;
        border-radius: 4px;
        margin-right: 6px;
        padding: 2px 6px;
    }

    .list-control-filter-item.on,
    .list-control-order-item.on {
        background: #f2352e;
        border: 1px solid #f2352e;
        color: #fff;
    }

    .product-not-found {
        text-align: center;
        padding: 32px;
    }
</style>