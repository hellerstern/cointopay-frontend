<template>
  <div class="card">
    <h5 class="title">Cointopay Shopping Carts</h5>
    <hr class="border-gray-300 dark:border-gray-600"/>
    <div class="body">
      <div class="flex flex-wrap justify-center gap-1 md:gap-2">
        <div class="w-28 h-28 p-2 shadow rounded-md flex flex-col items-center justify-center bg-gray-50 dark:bg-gray-700"
             v-tippy="{content: cart.name}"
             :key="index" v-for="(cart, index) of cartsList">
             <a class="text-sm dark:text-gray-100 break-all" :href="cart.url" target="_blank">
          <img class="h-8 mb-2 mx-auto object-contain" :src="cart.icon" alt=""/></a>
          <a class="text-sm dark:text-gray-100 break-all" :href="cart.url" target="_blank">{{ cart.name }}</a>
        </div>
      </div>
    </div>
  </div>
  <div class="card">
    <h5 class="title">Configure Shopify Settings</h5>
    <hr class="border-gray-300 dark:border-gray-600"/>
    <div class="body">
      <div class="flex items-center mb-4">
        <button @click="isShopify = !isShopify" v-tippy="'Configure Shopify Settings'">
          <img alt="Shopify" class="cursor-pointer" src="@/assets/images/shopify-brands.svg" style="width: 60px"/>
        </button>
        <a class="ml-4"
           href="https://cointopay.freshdesk.com/support/solutions/articles/13000078242-shopify-integration-guide"
           target="_blank">Shopify integration guide</a>
      </div>
      <div :class="{'hidden': !isShopify}">
        <form @submit.prevent="submit()">
          <div class="mb-2 grid grid-cols-3 gap-1 md:gap-4">
            <label class="col-span-3 md:col-span-1 mb-1">
              <a href="https://www.shopify.com/" target="_blank">Shopify</a>
              <span class="ml-3">{{ translate('account.shopping_cart.shopify_api') }}</span>
            </label>
            <div class="col-span-3 md:col-span-2">
              <input class="form-input w-full" type="text" v-model="form.shopifyApiKey"/>
              <div class="text-blue-500 text-sm">{{ showError(errorBag, 'shopifyApiKey') }}</div>
            </div>
          </div>

          <div class="mb-2 grid grid-cols-3 gap-1 md:gap-4">
            <label class="col-span-3 md:col-span-1 mb-1">
              {{ translate('account.shopping_cart.shopify_password') }}
            </label>
            <div class="col-span-3 md:col-span-2">
              <input class="form-input w-full" type="text" v-model="form.shopifyPassword"/>
              <div class="text-blue-500 text-sm">{{ showError(errorBag, 'shopifyPassword') }}</div>
            </div>
          </div>

          <div class="mb-2 grid grid-cols-3 gap-1 md:gap-4">
            <label class="col-span-3 md:col-span-1 mb-1">
              {{ translate('account.shopping_cart.shopify_url') }}&nbsp;
            </label>
            <div class="col-span-3 md:col-span-2">
              <input class="form-input w-full" type="text" v-model="form.shopifyUrl"/>
              <div class="text-blue-500 text-sm">{{ showError(errorBag, 'shopifyUrl') }}</div>
            </div>
          </div>

          <div class="mb-2 grid grid-cols-3 gap-1 md:gap-4">
            <label class="col-span-3 md:col-span-1 mb-1" for="script">
              {{ translate('account.shopping_cart.shopify_script') }}</label>
            <div class="col-span-3 md:col-span-2">
              <p @click="generateScript()" class="mb-0 cursor-pointer">Generate Script</p>
              <textarea class="form-input w-full" id="script" rows="3" v-model="script"></textarea>
              <p class="help text-xs">Paste under: Settings - Checkout - Additional content and scripts</p>
            </div>
          </div>

          <div class="mt-4 text-right">
            <j-button :disabled="isProcessing">
              <div v-if="isProcessing" class="mr-3">
                <icon :name="'spinner'" classes="w-4 h-4 text-white"></icon>
              </div>
              <span>{{ translate('account.button_update') }}</span>
            </j-button>
          </div>

        </form>
      </div>
    </div>
  </div>
</template>

<script>
import JButton from '@/templates/Button'
import Icon from '@/components/Icon'
import { computed, inject, onBeforeMount, reactive, toRefs, watch } from 'vue'
import * as _ from 'lodash'
import { useStore } from 'vuex'
import validator from '@/validator'
import { UPDATE_ACCOUNT_INFO } from '@/store/keys'
import compositionUtils from '@/compositionUtils'

export default {
  name: 'ShoppingCart',
  components: {
    JButton,
    Icon
  },
  setup () {
    // Providers
    const translate = inject('translate')
    const toast = inject('toast')
    // Store
    const store = useStore()

    const {
      validate,
      showError
    } = validator()
    const { saveNotification } = compositionUtils()

    const state = reactive({
      isShopify: false,
      carts: [
        {
          name: 'Wordpress WooCommerce',
          url: 'https://github.com/Cointopay/Wordpress-WooCommerce',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/woocommerce.png'
        },
        {
          name: 'Opencart-2.x',
          url: 'https://github.com/Cointopay/Opencart-2.x',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/opencart.png'
        },
        {
          name: 'OSCommerce',
          url: 'https://github.com/Cointopay/OSCommerce',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/oscommerce.png'
        },
        {
          name: 'CS-Cart',
          url: 'https://github.com/Cointopay/CS-Cart',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/cscart.png'
        },
        {
          name: 'Joomla-Virtuemart',
          url: 'https://github.com/Cointopay/Joomla-Virtuemart',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/joomla.png'
        },
        {
          name: 'Magento1.9',
          url: 'https://github.com/Cointopay/Magento1.9',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/Magento.png'
        },
        {
          name: 'Magento',
          url: 'https://github.com/Cointopay/Magento',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/Magento.png'
        },
        {
          name: 'Opencart-3x',
          url: 'https://github.com/Cointopay/Opencart-3x',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/opencart.png'
        },
        {
          name: 'Prestashop',
          url: 'https://github.com/Cointopay/Prestashop',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/prestashop.png'
        },
        {
          name: 'Ecwid',
          url: 'https://github.com/Cointopay/ecwid',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/ecwid.png'
        },
        {
          name: 'Opencart2x-refresh',
          url: 'https://github.com/Cointopay/Opencart2x-refresh',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/opencart.png'
        },
        {
          name: 'WooCommerce-Marketplace',
          url: 'https://github.com/Cointopay/WooCommerce-Marketplace',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/woocommerce.png'
        },
        {
          name: 'BoxBilling',
          url: 'https://github.com/Cointopay/BoxBilling',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/boxbilling.jpg'
        },
        {
          name: 'TomatoCart',
          url: 'https://github.com/Cointopay/TomatoCart',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/tomatocart.png'
        },
        {
          name: 'Odoo',
          url: 'https://github.com/Cointopay/Odoo',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/odoo.png'
        },
        {
          name: 'Coinpress.club-API-docs-and-requests',
          url: 'https://github.com/Cointopay/Coinpress.club-API-docs-and-requests',
          icon: 'https://avatars1.githubusercontent.com/u/8329602?s=460&u=4ca1dac5506a266f751a27dd8c91bf607e6b4129&v=4'
        },
        {
          name: 'WordPress-EasyDigitalDownloads',
          url: 'https://github.com/Cointopay/WordPress-EasyDigitalDownloads',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/wordpressecommerce.jpg'
        },
        {
          name: 'WordPressEcommerce',
          url: 'https://github.com/Cointopay/WordPressEcommerce',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/wordpressecommerce.jpg'
        },
        {
          name: 'Blesta',
          url: 'https://github.com/Cointopay/Blesta',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/blesta.png'
        },
        {
          name: 'Ubercart',
          url: 'https://github.com/Cointopay/Ubercart',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/ubercart.png'
        },
        {
          name: 'Zencart',
          url: 'https://github.com/Cointopay/Zencart',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/zencart.png'
        },
        {
          name: 'DrupalCommerce',
          url: 'https://github.com/Cointopay/DrupalCommerce',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/drupalcommerce.png'
        },
        {
          name: 'X-Cart',
          url: 'https://github.com/Cointopay/X-Cart',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/xcart.png'
        },
        {
          name: 'Wordpress-MembershipPro',
          url: 'https://github.com/Cointopay/Wordpress-MembershipPro',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/membershippro.jpg'
        },
        {
          name: 'Arastta',
          url: 'https://github.com/Cointopay/Arastta',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/arastta.png'
        },
        {
          name: 'WHMCS',
          url: 'https://github.com/Cointopay/WHMCS',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/whmcs.jpg'
        },
        {
          name: 'Opencart-1.5.6',
          url: 'https://github.com/Cointopay/Opencart-1.5.6',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/opencart.png'
        },
        {
          name: 'Opencart-2.3',
          url: 'https://github.com/Cointopay/Opencart-2.3',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/opencart.png'
        },
        {
          name: 'Opencart-2.2',
          url: 'https://github.com/Cointopay/Opencart-2.2',
          icon: 'https://s3-eu-west-1.amazonaws.com/cointopay/pics/opencart.png'
        }
      ],
      script: '',
      form: {
        shopifyApiKey: '',
        shopifyPassword: '',
        shopifyUrl: '',
        ecwidShopID: 0
      },
      errorBag: {},
      rules: {
        shopifyUrl: ['url']
      },
      isProcessing: false
    })

    // Computed
    const accountInfo = computed(() => store.state.accountInfo)
    const cartsList = computed(() => {
      return _.sortBy(state.carts, ['name'])
    })

    // Watch
    watch(() => ({ ...state.form }),
      (newVal, oldVal) => {
        if (Object.keys(state.errorBag).length > 0) {
          state.errorBag = validate(newVal, state.rules)
        }
      }
    )

    // Methods
    const generateScript = () => {
      const response = state.form.shopifyUrl.match(
        /(http(s)?:\/\/.)?(www\.)?[-a-zA-Z0-9@:%._+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_+.~#?&//=]*)/g
      )
      if (response) {
        state.script = `{% if order.financial_status == 'paid' %}
        <p>This order {{ order.order_number }} has been set to status paid already by the merchant.</p>
        {% else %}
        <p><iframe id="iCOINTOPAY" src="https://cointopay.com/SHOPIFYv2?Continue&TransactionID={{ checkout.order_id }}&url=${state.form.shopifyUrl}" style="width: 100%; height: 565px; border: 0;"></iframe></p>
        {% endif %}`
      } else {
        toast.error('Please enter a valid shopify url')
      }
    }
    const submit = () => {
      state.errorBag = validate(state.form, state.rules)
      if (Object.keys(state.errorBag).length === 0) {
        if (accountInfo.value) {
          state.isProcessing = true
          const payload = {
            ...accountInfo.value,
            ShopifyAPIKey: (state.form.shopifyApiKey) ? state.form.shopifyApiKey : null,
            ShopifyPassword: (state.form.shopifyPassword) ? state.form.shopifyPassword : null,
            ShopifyURL: state.form.shopifyUrl ? state.form.shopifyUrl : 'https://your-url-input.com',
            ECWID: state.form.ecwidShopID ? state.form.ecwidShopID : 0
          }
          store.dispatch(UPDATE_ACCOUNT_INFO, payload).then(response => {
            state.isProcessing = false
            if (response.status === 200) {
              // Show success toast
              toast.success('Shopping carts settings updated successfully')
              saveNotification('Shopping carts settings updated successfully')
            }
          }).catch(error => {
            state.isProcessing = false
            toast.error(error.data.message)
            saveNotification(error.data.message)
          })
        }
      }
    }

    onBeforeMount(() => {
      if (accountInfo.value) {
        state.form = {
          shopifyApiKey: accountInfo.value.ShopifyAPIKey,
          shopifyPassword: accountInfo.value.ShopifyPassword,
          shopifyUrl: accountInfo.value.ShopifyURL,
          ecwidShopID: accountInfo.value.ECWID
        }
      }
    })

    return {
      translate,
      ...toRefs(state),
      showError,
      cartsList,
      generateScript,
      submit
    }
  }
}
</script>

<style scoped lang="scss">
a {
  @apply text-blue-500 hover:underline;
}
</style>
