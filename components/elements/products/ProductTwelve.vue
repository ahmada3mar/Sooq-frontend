<template>
    <div class="product product-2 text-center">
        <figure class="product-media">
            <span class="product-label label-circle label-new" v-if="product.new">New</span>
            <span class="product-label label-circle label-sale" v-if="product.sale_price">Sale</span>
            <span class="product-label label-circle label-top" v-if="product.top">Top</span>
            <span
                class="product-label label-circle label-out"
                v-if="product.stock === 0"
            >Out Of Stock</span>

            <nuxt-link :to="'/product/default/'+ product.slug">
                <div class="product-card" 
                v-lazy:background="`${baseUrl}${product.pictures[0].url}`"></div>
                <div class="product-card product-image-hover" 
                v-lazy:background="`${baseUrl}${product.sm_pictures[1].url}`"
                v-if="product.pictures[1]"
                ></div>
            </nuxt-link>
            <div class="product-action-vertical" v-if="product.stock !== 0">
                <nuxt-link
                    to="/shop/wishlist"
                    class="btn-product-icon btn-wishlist btn-expandable added-to-wishlist"
                    v-if="isInWishlist(product)"
                    key="inWishlist"
                >
                    <span>go to wishlist</span>
                </nuxt-link>
                <a
                    href="javascript:;"
                    class="btn-product-icon btn-wishlist btn-expandable"
                    @click.prevent="addToWishlist({product: product})"
                    v-else
                    key="notInWishlist"
                >
                    <span>add to wishlist</span>
                </a>
                <button
                    class="btn-product-icon btn-quickview"
                    title="Quick view"
                    @click.prevent="quickView({product: product})"
                >
                    <span>quick view</span>
                </button>
            </div>
            <div class="product-action product-action-transparent" v-if="product.stock !== 0">
                <nuxt-link
                    :to="'/product/default/' + product.slug"
                    class="btn-product btn-cart btn-select"
                    v-if="product.variants.length > 0"
                >
                    <span>select options</span>
                </nuxt-link>
                <button
                    class="btn-product btn-cart"
                    @click.prevent="addToCart( {product: product} )"
                    v-else
                >
                    <span>add to cart</span>
                </button>
            </div>
        </figure>

        <div class="product-body">
            <div class="product-cat">
                <span v-for="(cat, index) of product.category" :key="index">
                    <nuxt-link
                        :to="{path: '/shop/sidebar/list', query: {category: cat.slug}}"
                    >{{ cat.name }}</nuxt-link>
                    {{ index < product.category.length - 1 ? ',' : '' }}
                </span>
            </div>
            <h3 class="product-title">
                <nuxt-link :to="'/product/default/'+ product.slug">{{ product.name }}</nuxt-link>
            </h3>

            <div class="product-price" v-if="product.stock==0" key="outPrice">
                <span class="out-price">${{ product.price.toFixed(2) }}</span>
            </div>

            <template v-else>
                <div class="product-price" v-if="minPrice == maxPrice">${{ minPrice.toFixed(2) }}</div>
                <template v-else>
                    <div class="product-price" v-if="product.variants.length == 0">
                        <span class="new-price">${{ minPrice.toFixed(2) }}</span>
                        <span class="old-price">${{ maxPrice.toFixed(2) }}</span>
                    </div>
                    <div
                        class="product-price"
                        v-else
                    >${{minPrice.toFixed(2)}}&ndash;${{maxPrice.toFixed(2)}}</div>
                </template>
            </template>

            <div class="ratings-container">
                <div class="ratings">
                    <div class="ratings-val" :style="{width: product.ratings * 20 + '%'}"></div>
                    <span class="tooltip-text">{{ product.ratings.toFixed(2) }}</span>
                </div>
                <span class="ratings-text">( {{ product.review }} Reviews )</span>
            </div>

            <div class="product-nav product-nav-dots" v-if="product.variants.length > 0">
                <div class="row no-gutters">
                    <a
                        href="javascript:;"
                        :style="{'background-color': item.color}"
                        v-for="(item, index) in product.variants"
                        :key="index"
                    >
                        <span class="sr-only">Color name</span>
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { mapGetters, mapActions } from 'vuex';
import { baseUrl } from '~/repositories/repository';
export default {
    props: {
        product: Object
    },
    data: function() {
        return {
            baseUrl: baseUrl,
            maxPrice: 0,
            minPrice: 99999
        };
    },
    computed: {
        ...mapGetters('cart', ['canAddToCart']),
        ...mapGetters('wishlist', ['isInWishlist']),
        ...mapGetters('compare', ['isInCompare'])
    },

    created: function() {
        let min = this.minPrice;
        let max = this.maxPrice;
        this.product.variants.map(item => {
            if (min > item.price) min = item.price;
            if (max < item.price) max = item.price;
        }, []);

        if (this.product.variants.length == 0) {
            min = this.product.sale_price
                ? this.product.sale_price
                : this.product.price;
            max = this.product.price;
        }

        this.minPrice = min;
        this.maxPrice = max;
    },
    methods: {
        ...mapActions('cart', ['addToCart']),
        ...mapActions('wishlist', ['addToWishlist']),
        ...mapActions('compare', ['addToCompare']),
        quickView: function() {
            this.$modal.show(
                () => import('~/components/elements/modals/QuickViewModalTwo'),
                {
                    product: this.product
                },
                { width: '1030', height: 'auto', adaptive: true }
            );
        }
    }
};
</script>