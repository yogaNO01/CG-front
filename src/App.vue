<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'
import BaseIcon from './components/BaseIcon.vue'
import {
  catalogFilters,
  categoryPosters,
  featureCards,
  navItems,
  news,
  serviceHighlights,
  stats,
} from './mock/home'

const routeHash = ref(window.location.hash || '#/')
const activeCategory = ref(null)
const activeSupplierCategory = ref('')
const activeDetailImage = ref(0)
const selectedSpec = ref('标准配置')
const purchaseQuantity = ref(1)
const storeSearchTerm = ref('')
const recommendationOffset = ref(0)
const activeCatalogSort = ref('综合排序')
const activeCatalogFilters = ref({})
const quoteDialogOpen = ref(false)
const quoteSubmitted = ref(false)
const quoteMessage = ref('')
const quoteContactName = ref('')
const quoteContactPhone = ref('')
const quoteSubmitting = ref(false)
const quoteError = ref('')
const categories = ref([])
const catalogProducts = ref([])
const supplierShowcases = ref([])
const selectedProduct = ref(null)
const selectedCompany = ref(null)
const apiError = ref('')

const displayLabels = {
  platform_verified: '平台认证', factory_audited: '已验厂', customization: '支持定制', fast_shipping: '快速发货',
  after_sales: '售后保障', platform_guarantee: '平台保障', source_factory: '源头工厂', source_supply: '源头直供',
  hot_sale: '热销优品', spot: '现货供应', custom: '支持定制', preorder: '预约供货',
  production_line: '生产线', workshop: '车间', engineering_project: '工程项目', selection_support: '选型支持',
  project_service: '项目服务', active: '存续', cancelled: '注销', revoked: '吊销', moved_out: '迁出',
  limited_liability_company: '有限责任公司', sole_proprietorship: '个人独资企业', partnership: '合伙企业', individual_business: '个体工商户',
  in_stock: '现货充足', low_stock: '库存紧张', out_of_stock: '暂时缺货',
}
const displayCode = (value, fallback = '—') => value ? (displayLabels[value] || value) : fallback
const displayCodes = (values, fallback = []) => {
  const labels = (Array.isArray(values) ? values : []).map((value) => displayCode(value, '')).filter(Boolean)
  return labels.length ? labels : fallback
}

const emptyProduct = {
  id: '', name: '商品加载中', price: '—', tag: '', badge: '', scene: 'factory', icon: 'factory', image: '/images/product-touch-panel.png',
  images: ['/images/product-touch-panel.png'], products: [], raw: {},
}
const emptySupplier = { id: '', name: '店铺加载中', shortName: '', mark: '', tone: 'blue', logo: '', category: '', location: '', products: [], raw: {} }

const api = async (path, { method = 'GET', body } = {}) => {
  const response = await fetch(`/api${path}`, {
    method,
    headers: body ? { 'Content-Type': 'application/json' } : undefined,
    body: body ? JSON.stringify(body) : undefined,
  })
  const payload = await response.json()
  if (!response.ok || payload.code !== 0) throw new Error(payload.message || '数据请求失败')
  return payload.data
}

const formatPrice = (sku) => {
  if (!sku?.displayPrice) return '面议'
  const amount = Number(sku.displayPrice)
  const value = Number.isInteger(amount) ? amount.toLocaleString('zh-CN') : amount.toLocaleString('zh-CN', { minimumFractionDigits: 2 })
  return `¥${value}${sku.priceFrom ? '起' : ''}`
}
const skuLabel = (sku) => (sku?.specValues || []).map((spec) => spec.specValue).filter(Boolean).join(' / ') || sku?.specValue || '标准配置'
const toProduct = (item, index = 0) => ({
  id: item.productId,
  companyId: item.companyId,
  name: item.productName,
  price: formatPrice(item.defaultSku),
  tag: (item.sellingPoints || []).join(' · ') || item.shortDescription || '',
  badge: item.marketingBadge === 'source_supply' ? '源头直供' : item.marketingBadge === 'hot_sale' ? '热销优品' : '优选商品',
  scene: ['factory', 'vehicle', 'material', 'parts', 'motor', 'solar'][index % 6],
  icon: ['factory', 'car', 'grid', 'gear', 'diamond', 'grid'][index % 6],
  image: item.mainImageUrl || '/images/product-touch-panel.png',
  images: item.imageUrls?.length ? item.imageUrls : [item.mainImageUrl || '/images/product-touch-panel.png'],
  location: item.originPlace || item.company?.shippingAddress?.displayName || '中国',
  supplier: item.company?.companyName || '',
  years: item.company?.statistics?.operatingYears ? `${item.company.statistics.operatingYears}年` : '',
  raw: item,
})
const toSupplier = (item = {}, index = 0) => {
  const companyName = item.companyName || '店铺加载中'
  return {
    id: item.companyId || '',
    name: companyName,
    shortName: item.shortName || companyName,
    mark: item.logoText || companyName.slice(0, 2),
    tone: item.themeColor || ['blue', 'green', 'orange', 'teal'][index % 4],
    logo: item.logoUrl || '',
    category: item.featuredProducts?.[0]?.categoryName || item.products?.[0]?.categoryName || '',
    location: item.shippingAddress?.displayName || '',
    products: (item.products || item.featuredProducts || []).map((product, productIndex) => toProduct(product, productIndex)),
    raw: item,
  }
}

const loadHome = async () => {
  try {
    apiError.value = ''
    const [categoryData, productData, companyData] = await Promise.all([
      api('/public/categories'), api('/public/products?page=1&pageSize=24'), api('/public/companies?page=1&pageSize=20'),
    ])
    categories.value = categoryData.map((root) => ({
      id: root.categoryId, name: root.categoryName, icon: root.icon || 'grid',
      groups: (root.children || []).map((group) => ({ title: group.categoryName, items: (group.children || []).map((child) => child.categoryName) })),
    }))
    catalogProducts.value = productData.items.map(toProduct)
    supplierShowcases.value = companyData.items.map(toSupplier)
    activeSupplierCategory.value = supplierShowcases.value[0]?.category || ''
  } catch (error) {
    apiError.value = error.message || '暂时无法加载商城数据'
  }
}

const loadRouteData = async () => {
  const [, query = ''] = routeHash.value.split('?')
  const params = new URLSearchParams(query)
  try {
    if (isProductDetailPage.value && params.get('id')) {
      const data = await api(`/public/products/${encodeURIComponent(params.get('id'))}`)
      selectedProduct.value = toProduct(data)
      selectedSpec.value = skuLabel(data.defaultSku)
      activeDetailImage.value = 0
    }
    if (isCompanyPage.value && params.get('id')) {
      const companyId = encodeURIComponent(params.get('id'))
      const [data, products] = await Promise.all([
        api(`/public/companies/${companyId}`), api(`/public/companies/${companyId}/products?page=1&pageSize=24`),
      ])
      selectedCompany.value = toSupplier({ ...data, products: products.items })
    }
  } catch (error) {
    apiError.value = error.message || '暂时无法加载详情数据'
  }
}

const updateRoute = () => {
  routeHash.value = window.location.hash || '#/'
  void loadRouteData()
}

onMounted(() => {
  window.addEventListener('hashchange', updateRoute)
  void loadHome().then(loadRouteData)
})

onUnmounted(() => {
  window.removeEventListener('hashchange', updateRoute)
})

const currentCategory = computed(() => {
  if (!routeHash.value.startsWith('#/catalog')) return ''
  const [, query = ''] = routeHash.value.split('?category=')
  return decodeURIComponent(query || '机械设备')
})

const isCatalogPage = computed(() => routeHash.value.startsWith('#/catalog'))
const isProductDetailPage = computed(() => routeHash.value.startsWith('#/product'))
const isCompanyPage = computed(() => routeHash.value.startsWith('#/company'))

const categoryNames = computed(() => categories.value.map((item) => item.name))
const supplierShowcaseCategories = computed(() => [...new Set(supplierShowcases.value.map((supplier) => supplier.category).filter(Boolean))])
const products = computed(() => catalogProducts.value.slice(0, 6))

const detailProduct = computed(() => selectedProduct.value || catalogProducts.value[0] || emptyProduct)

const detailGallery = computed(() => detailProduct.value.images?.length ? detailProduct.value.images : [detailProduct.value.image])

const detailRelatedProducts = computed(() => (detailProduct.value.raw?.relatedProducts || catalogProducts.value.filter((product) => product.id !== detailProduct.value.id).map((product) => product.raw).slice(0, 4)).map(toProduct))

const detailSupplier = computed(() => toSupplier(detailProduct.value.raw?.company || supplierShowcases.value.find((supplier) => supplier.id === detailProduct.value.companyId)?.raw || {}, 0))

const detailSupplierLocation = computed(() => detailSupplier.value.location || detailProduct.value.location || '')
const detailSpecs = computed(() => detailProduct.value.raw?.skus?.map((sku) => ({ id: sku.skuId, label: skuLabel(sku) })) || [])
const selectedSku = computed(() => detailProduct.value.raw?.skus?.find((sku) => skuLabel(sku) === selectedSpec.value) || detailProduct.value.raw?.defaultSku || null)
const detailServiceTags = computed(() => displayCodes(detailProduct.value.raw?.serviceGuarantees, ['平台认证', '支持定制']))
const detailSupplyMethods = computed(() => displayCodes(detailProduct.value.raw?.supplyMethod, ['—']))
const detailScenarios = computed(() => displayCodes(detailProduct.value.raw?.applicationScenarios, ['—']))
const detailFeatureTags = computed(() => displayCodes(detailProduct.value.raw?.featureTags, ['选型支持']))
const detailSupplierTags = computed(() => {
  const category = detailSupplier.value.category || detailProduct.value.raw?.categoryName || '工业设备'
  const capabilities = displayCodes(detailSupplier.value.raw?.serviceCapabilities, ['认证供应商'])
  return [...new Set([category, ...capabilities])]
})
const companyDetails = computed(() => currentStoreSupplier.value.raw || {})
const companyStatistics = computed(() => companyDetails.value.statistics || {})
const companyBusinessInfo = computed(() => companyDetails.value.businessInfo || {})
const companyVerification = computed(() => companyDetails.value.verification || {})

const companySupplier = computed(() => selectedCompany.value || detailSupplier.value || emptySupplier)
const currentStoreSupplier = computed(() => isCompanyPage.value ? companySupplier.value : detailSupplier.value)
const supplierLocation = (supplier) => supplier.location || supplier.raw?.shippingAddress?.displayName || ''
const quoteProductLabel = computed(() => isCompanyPage.value ? `${currentStoreSupplier.value.name} 企业服务咨询` : detailProduct.value.name)
const companyTab = computed(() => {
  if (!isCompanyPage.value) return 'products'
  const [, query = ''] = routeHash.value.split('?')
  const tab = new URLSearchParams(query).get('tab')
  return ['home', 'products', 'archive'].includes(tab) ? tab : 'home'
})
const storeTabs = [
  { label: '供应商品', value: 'products' },
  { label: '企业档案', value: 'archive' },
  { label: '留言问价', value: 'quote' }
]

const productDetailHref = (product) => `#/product?id=${encodeURIComponent(product.id)}`
const companyDetailHref = (supplier, tab = 'home') => `#/company?id=${encodeURIComponent(supplier.id)}&tab=${tab}`
const openProduct = (product) => {
  window.location.hash = productDetailHref(product)
}
const openCompany = (supplier, tab = 'home') => {
  window.location.hash = companyDetailHref(supplier, tab)
}
const openCatalog = (category = '机械设备') => {
  window.location.hash = `#/catalog?category=${encodeURIComponent(category)}`
}
const cycleRecommendations = () => {
  if (catalogProducts.value.length) recommendationOffset.value = (recommendationOffset.value + 4) % catalogProducts.value.length
}
const selectCatalogFilter = (label, value) => {
  activeCatalogFilters.value = { ...activeCatalogFilters.value, [label]: value }
}
const sortedCatalogProducts = computed(() => {
  if (activeCatalogSort.value === '价格') return [...catalogProducts.value].sort((a, b) => Number(a.raw?.defaultSku?.displayPrice || Infinity) - Number(b.raw?.defaultSku?.displayPrice || Infinity))
  if (activeCatalogSort.value === '人气排序') return [...catalogProducts.value].sort((a, b) => (b.raw?.salesCount || 0) - (a.raw?.salesCount || 0))
  return catalogProducts.value
})
const decreaseQuantity = () => {
  purchaseQuantity.value = Math.max(1, purchaseQuantity.value - 1)
}
const increaseQuantity = () => {
  purchaseQuantity.value += 1
}
const searchStore = () => {
  storeSearchTerm.value = storeSearchTerm.value.trim()
}
const openQuoteDialog = () => {
  quoteSubmitted.value = false
  quoteError.value = ''
  quoteDialogOpen.value = true
}
const closeQuoteDialog = () => {
  quoteDialogOpen.value = false
}
const submitQuote = async () => {
  quoteSubmitting.value = true
  quoteError.value = ''
  try {
    const isCompanyInquiry = isCompanyPage.value
    await api('/inquiries', {
      method: 'POST',
      body: {
        companyId: currentStoreSupplier.value.id,
        ...(isCompanyInquiry ? {} : { productId: detailProduct.value.id, skuId: selectedSku.value?.skuId }),
        requirement: quoteMessage.value,
        contactName: quoteContactName.value,
        contactPhone: quoteContactPhone.value,
        ...(isCompanyInquiry ? {} : { expectedSpec: selectedSpec.value, quantity: String(purchaseQuantity.value), unit: detailProduct.value.raw?.unit }),
      },
    })
    quoteSubmitted.value = true
    quoteMessage.value = ''
    quoteContactName.value = ''
    quoteContactPhone.value = ''
  } catch (error) {
    quoteError.value = error.message || '提交失败，请稍后重试'
  } finally {
    quoteSubmitting.value = false
  }
}
const openStoreTab = (tab) => {
  if (tab === 'quote') {
    openQuoteDialog()
    return
  }
  window.location.hash = companyDetailHref(currentStoreSupplier.value, tab)
}

const hotCategoryLinks = computed(() => {
  const mechanical = categories.value.find((item) => item.name === '机械设备')
  const detailed = mechanical?.groups?.flatMap((group) => group.items) || []
  return [...detailed.slice(0, 5), ...categoryNames.value.slice(1, 4)]
})

const recommendTabs = computed(() => categoryNames.value.slice(0, 6))
const visibleSupplierShowcases = computed(() => {
  const matches = supplierShowcases.value.filter((supplier) => supplier.category === activeSupplierCategory.value)
  const defaultSuppliers = supplierShowcases.value.filter((supplier) => supplier.category === supplierShowcaseCategories.value[0])
  const supplements = defaultSuppliers.filter((supplier) => !matches.some((match) => match.name === supplier.name))
  return [...matches, ...supplements].slice(0, 4)
})

const premiumCatalogProducts = computed(() => catalogProducts.value.slice(0, 6))
const factoryCatalogProducts = computed(() => catalogProducts.value.slice(2, 8))
const recommendedProducts = computed(() => Array.from({ length: Math.min(12, catalogProducts.value.length) }, (_, index) => catalogProducts.value[(recommendationOffset.value + index) % catalogProducts.value.length]).map((product) => ({
  ...product,
  serviceTags: product.raw?.serviceGuarantees || ['platform_verified'],
  video: Boolean(product.raw?.videoUrls?.length)
})))
const marketSections = computed(() => [
  {
    title: '机械设备',
    desc: '工控设备、搬运起重、包装设备',
    image: '/images/market-equipment-banner.png',
    bannerClass: 'equipment-bg',
    tags: ['水处理设备', '通用设备', '过滤设备', '食品加工机械'],
    products: catalogProducts.value.slice(0, 3)
  },
  {
    title: '建材家居',
    desc: '基建材料、功能材料、灯饰照明',
    image: '/images/market-building-banner.png',
    bannerClass: 'full-bg',
    tags: ['基建材料', '功能材料', '电工电料', '灯饰照明'],
    products: catalogProducts.value.slice(3, 6)
  },
  {
    title: '化工能源',
    desc: '涂料油漆、水处理化学品、工程塑料',
    image: '/images/market-chemical-banner.png',
    bannerClass: 'full-bg',
    tags: ['水处理化学品', '涂料油漆', '工程塑料', '橡塑制品'],
    products: catalogProducts.value.slice(6, 9)
  },
  {
    title: '电子仪表',
    desc: '集成电路、专业仪表、工控终端',
    image: '/images/market-instrument-banner.png',
    bannerClass: 'full-bg',
    tags: ['专业仪表', '检测仪器', '集成电路', '工控终端'],
    products: catalogProducts.value.slice(1, 4)
  }
])
</script>

<template>
  <header class="site-header">
    <div class="header-inner">
        <a class="brand" href="#/">
          <img src="/images/logo-qingcaiyun.png" alt="擎采云" />
        </a>

      <nav class="nav">
        <a v-for="(item, index) in navItems" :key="item" :class="{ active: index === 0 && !isCatalogPage && !isProductDetailPage && !isCompanyPage }" href="#/">
          {{ item }}
        </a>
      </nav>

      <div class="header-actions">
        <button class="icon-btn" aria-label="搜索"><BaseIcon name="search" /></button>
        <button class="cart-btn"><BaseIcon name="cart" />我的采购<span>0</span></button>
        <button class="login-btn"><BaseIcon name="user" />登录</button>
        <button class="register-btn">免费注册</button>
      </div>
    </div>
  </header>

  <section v-if="isProductDetailPage || isCompanyPage" class="store-banner">
    <div class="store-banner-inner">
      <div class="store-identity">
        <div class="store-emblem" :class="[{ 'has-logo': Boolean(currentStoreSupplier.logo) }, `supplier-mark-${currentStoreSupplier.tone}`]">
          <img v-if="currentStoreSupplier.logo" :src="currentStoreSupplier.logo" :alt="`${currentStoreSupplier.name} Logo`" />
          <span v-else>{{ currentStoreSupplier.name.slice(0, 1) }}</span>
        </div>
          <div class="store-profile">
            <div class="store-name-line">
            <a :href="companyDetailHref(currentStoreSupplier, 'home')">{{ currentStoreSupplier.name }}</a>
            <span class="store-verified"><BaseIcon name="shield" />认证供应商</span>
          </div>
          <div class="store-meta">
            <span>{{ currentStoreSupplier.shortName }}</span>
            <span>已验厂企业</span>
            <span>经营 5 年</span>
            <span>{{ supplierLocation(currentStoreSupplier) }}发货</span>
          </div>
        </div>
      </div>

      <form v-if="isProductDetailPage" class="store-search" @submit.prevent="searchStore">
        <label for="store-search-input">店内搜索</label>
        <input id="store-search-input" v-model="storeSearchTerm" type="search" placeholder="搜索本店商品、型号或关键词" />
        <button type="submit"><BaseIcon name="search" />搜索本店</button>
      </form>

      <div v-else class="company-profile-actions store-banner-actions">
        <button type="button" @click="openQuoteDialog"><BaseIcon name="headset" />留言问价</button>
        <a :href="companyDetailHref(currentStoreSupplier, 'archive')">企业档案<BaseIcon name="arrowRight" /></a>
      </div>

      <nav class="store-nav" aria-label="店铺导航">
        <button
          v-for="tab in storeTabs"
          :key="tab.value"
          type="button"
          :class="{ active: tab.value === (isCompanyPage ? companyTab : 'products') }"
          @click="openStoreTab(tab.value)"
        >
          {{ tab.label }}
        </button>
        <span v-if="storeSearchTerm" class="store-search-state">店内检索：{{ storeSearchTerm }}</span>
      </nav>
    </div>
  </section>

  <main v-if="!isCatalogPage && !isProductDetailPage && !isCompanyPage" class="page">
    <section class="hero">
      <div class="hero-bg"></div>
      <div class="hero-inner">
        <div class="hero-copy">
          <h1>
            连接优质供需<br />
            让企业采购<span>更简单</span>
          </h1>
          <p>海量工业资源 · 真实可靠的供应商 · 专业高效的采购服务</p>

          <div class="search-box">
            <button class="search-type">商品<BaseIcon name="arrowRight" /></button>
            <input value="请输入产品名称、品牌、型号或供应商" readonly />
            <button class="camera" aria-label="图片搜索"><BaseIcon name="camera" /></button>
            <button class="search-submit"><BaseIcon name="search" />搜索</button>
          </div>

          <div class="hot-line">
            <strong>热门搜索：</strong>
            <a
              v-for="item in hotCategoryLinks"
              :key="item"
              :href="`#/catalog?category=${encodeURIComponent(item)}`"
            >
              {{ item }}
            </a>
            <a href="#/catalog?category=机械设备">更多 &gt;</a>
          </div>
        </div>

        <div class="industry-copy">
          <p>INDUSTRY<br />CONNECTS</p>
          <p class="en">INDUSTRY<br />CONNECTS<br />A BETTER FUTURE</p>
          <h2>产业互联</h2>
          <span>共建更高效的采购生态</span>
        </div>
      </div>
    </section>

    <section class="content-shell">
      <div class="highlights">
        <div v-for="item in serviceHighlights" :key="item.title" class="highlight-item">
          <div class="highlight-icon"><BaseIcon :name="item.icon" /></div>
          <div>
            <strong>{{ item.title }}</strong>
            <span>{{ item.desc }}</span>
          </div>
        </div>
      </div>

      <div class="layout-grid">
        <aside class="category-panel" @mouseleave="activeCategory = null">
          <div class="category-title"><BaseIcon name="menu" />全部商品分类</div>
          <div class="category-scroll">
            <div
              v-for="item in categories"
              :key="item.name"
              class="category-entry"
              @mouseenter="activeCategory = item"
            >
              <a :href="`#/catalog?category=${encodeURIComponent(item.name)}`">
                <span class="cat-icon"><BaseIcon :name="item.icon" /></span>
                <span class="cat-name">{{ item.name }}</span>
                <b><BaseIcon name="arrowRight" /></b>
              </a>
            </div>
          </div>
          <div
            v-if="activeCategory"
            class="category-flyout"
            @mouseenter="activeCategory = activeCategory"
            @mouseleave="activeCategory = null"
          >
            <div class="flyout-head">
              <strong>{{ activeCategory.name }}</strong>
              <a :href="`#/catalog?category=${encodeURIComponent(activeCategory.name)}`">进入专区<BaseIcon name="arrowRight" /></a>
            </div>
            <div class="flyout-grid">
              <div v-for="group in activeCategory.groups" :key="group.title" class="flyout-group">
                <h4>{{ group.title }}</h4>
                <div>
                  <a
                    v-for="child in group.items"
                    :key="child"
                    :href="`#/catalog?category=${encodeURIComponent(child)}`"
                  >
                    {{ child }}
                  </a>
                </div>
              </div>
            </div>
          </div>
          <div class="category-posters">
            <article
              v-for="(poster, index) in categoryPosters"
              :key="poster.title"
              class="poster-card"
              :class="{ active: index === 0 }"
              :style="{ backgroundImage: `linear-gradient(90deg, rgba(5, 30, 68, 0.72), rgba(5, 30, 68, 0.2)), url(${poster.image})` }"
            >
              <strong>{{ poster.title }}</strong>
              <span>{{ poster.desc }}</span>
            </article>
            <div class="poster-dots">
              <span v-for="(poster, index) in categoryPosters" :key="poster.title" :class="{ active: index === 0 }"></span>
            </div>
          </div>
        </aside>

        <section class="main-column">
          <div class="promo-row">
            <article class="manufacturing-card">
              <div class="promo-text">
                <small>SMART MANUFACTURING</small>
                <h2>智能制造<br />驱动产业升级</h2>
                <p>甄选优质工业设备 · 助力企业高质量发展</p>
              </div>
              <button class="promo-more" @click="openCatalog()">了解更多<BaseIcon name="arrowRight" /></button>
              <button class="slide-next" aria-label="查看机械设备" @click="openCatalog()"><BaseIcon name="arrowRight" /></button>
              <div class="dots"><span></span><span></span><span></span></div>
            </article>

            <div class="side-promos">
              <article
                v-for="card in featureCards"
                :key="card.title"
                class="source-card"
                :style="{ backgroundImage: `linear-gradient(90deg, rgba(238, 248, 255, 0.98) 0%, rgba(232, 245, 255, 0.92) 45%, rgba(216, 237, 255, 0.42) 72%, rgba(216, 237, 255, 0.08) 100%), url(${card.image})` }"
              >
                <div>
                  <h3>{{ card.title }}</h3>
                  <p>{{ card.desc }}</p>
                  <button @click="openCatalog(card.title.includes('工厂') ? '机械设备' : '行业解决方案')">{{ card.action }}<BaseIcon name="arrowRight" /></button>
                </div>
              </article>
            </div>
          </div>

          <div class="recommend-head">
            <h2>精选推荐</h2>
            <div class="tabs">
              <a
                v-for="(tab, index) in recommendTabs"
                :key="tab"
                :class="{ active: index === 0 }"
                :href="`#/catalog?category=${encodeURIComponent(tab)}`"
              >
                {{ tab }}
              </a>
            </div>
            <a href="#/catalog?category=机械设备">更多推荐-&gt;</a>
          </div>

          <div class="product-row">
            <button class="round-arrow left" aria-label="上一组"><BaseIcon name="arrowRight" /></button>
            <article v-for="product in products" :key="product.name" class="product-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
              <div class="product-image" :class="`scene-${product.scene}`">
                <img :src="product.image" :alt="product.name" @error="$event.target.style.display = 'none'" />
                <BaseIcon :name="product.icon" />
              </div>
              <strong>{{ product.name }}</strong>
              <small>{{ product.tag }}</small>
              <div class="product-foot">
                <span class="price">{{ product.price }}</span>
                <button type="button" @click.stop="openQuoteDialog">立即询价</button>
              </div>
            </article>
            <button class="round-arrow right" aria-label="下一组"><BaseIcon name="arrowRight" /></button>
          </div>
        </section>

        <aside class="right-column">
          <button class="demand-card" @click="openQuoteDialog">
            <strong>发布采购需求</strong>
            <span>让优质供应商主动联系您</span>
            <b><BaseIcon name="menu" /></b>
          </button>

          <section class="stats-card">
            <div class="panel-title"><h3>平台数据</h3><a href="#">查看更多<BaseIcon name="arrowRight" /></a></div>
            <div class="stats-grid">
              <div v-for="item in stats" :key="item.value" class="stat-item">
                <i><BaseIcon :name="item.icon" /></i>
                <div>
                  <strong>{{ item.value }}</strong>
                  <span>{{ item.label }}</span>
                </div>
              </div>
            </div>
          </section>

          <section class="news-card">
            <div class="panel-title"><h3>采购资讯</h3><a href="#">查看更多<BaseIcon name="arrowRight" /></a></div>
            <a v-for="item in news" :key="item.title" class="news-item" href="#">
              <div class="news-thumb"></div>
              <div>
                <strong>{{ item.title }}</strong>
                <span>{{ item.date }}</span>
              </div>
              <time>{{ item.date }}</time>
            </a>
          </section>
        </aside>
      </div>
    </section>

    <section class="home-floor-wrap">
      <section class="supplier-showcase home-floor">
        <div class="supplier-showcase-head">
          <div>
            <h2>优选厂商</h2>
            <p>实力工厂与行业优选</p>
          </div>
          <a href="#/catalog?category=机械设备">查看全部<BaseIcon name="arrowRight" /></a>
        </div>
        <div class="supplier-tabs" role="tablist" aria-label="优选厂商品类">
          <button
            v-for="category in supplierShowcaseCategories"
            :key="category"
            :class="{ active: activeSupplierCategory === category }"
            type="button"
            @click="activeSupplierCategory = category"
          >
            {{ category }}
          </button>
        </div>
        <div class="supplier-showcase-grid">
          <article v-for="supplier in visibleSupplierShowcases" :key="supplier.name" class="supplier-showcase-card">
            <div class="supplier-card-head">
              <div class="supplier-mark" :class="[{ 'has-logo': Boolean(supplier.logo) }, `supplier-mark-${supplier.tone}`]"><img v-if="supplier.logo" :src="supplier.logo" :alt="`${supplier.name} Logo`" /><span v-else>{{ supplier.name.slice(0, 1) }}</span></div>
              <div>
                <h3><a :href="companyDetailHref(supplier, 'home')">{{ supplier.name }}</a></h3>
                <span>{{ supplier.shortName }} · 已认证供应商</span>
              </div>
              <button type="button" @click="openCompany(supplier)">进店<BaseIcon name="arrowRight" /></button>
            </div>
            <div class="supplier-products">
              <a v-for="product in supplier.products" :key="product.name" :href="productDetailHref(product)">
                <span class="supplier-product-visual">
                  <img :src="product.image" :alt="product.name" />
                  <b>查看详情</b>
                </span>
                <span>{{ product.name }}</span>
                <strong>{{ product.price }}</strong>
              </a>
            </div>
          </article>
        </div>
      </section>

      <section class="catalog-section home-floor premium-floor">
        <div class="catalog-section-head">
          <div>
            <span class="floor-eyebrow">SELECTED GOODS</span>
            <h2>优选精品</h2>
            <p>严选认证供应商，适合快速询价与小批量试采</p>
          </div>
          <a href="#/catalog?category=机械设备">查看更多<BaseIcon name="arrowRight" /></a>
        </div>
        <div class="premium-strip">
          <article v-for="product in premiumCatalogProducts" :key="`premium-${product.name}`" class="premium-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
            <img :src="product.image" :alt="product.name" />
            <h3>{{ product.name }}</h3>
            <div><strong>{{ product.price }}</strong><button type="button" @click.stop="openQuoteDialog">立即询价</button></div>
          </article>
        </div>
      </section>

      <section class="factory-section home-floor">
        <div class="factory-banner">
          <small>FACTORY DIRECT SUPPLY</small>
          <h2>源头工厂直连</h2>
          <p>按需定制 · 批量报价 · 工厂直连 · 减少中间沟通成本</p>
          <div class="factory-points">
            <span><BaseIcon name="shield" />品质保障</span>
            <span><BaseIcon name="car" />快速交付</span>
            <span><BaseIcon name="document" />支持定制</span>
          </div>
          <button @click="openQuoteDialog">发布采购需求<BaseIcon name="arrowRight" /></button>
          <b>直连优质制造商<br />让采购更简单</b>
        </div>
        <div class="factory-grid">
          <article v-for="product in factoryCatalogProducts" :key="`factory-${product.name}`" class="factory-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
            <img :src="product.image" :alt="product.name" />
            <h3>{{ product.name }}</h3>
            <p>{{ product.tag }}</p>
            <strong>{{ product.price }}</strong>
          </article>
        </div>
      </section>

      <section class="catalog-section home-floor market-floor">
        <div class="catalog-section-head">
          <div>
            <span class="floor-eyebrow">CATEGORY MARKET</span>
            <h2>品类市场</h2>
            <p>按工业采购场景聚合常用类目，提高找货效率</p>
          </div>
        </div>
        <div class="market-grid">
          <article v-for="section in marketSections" :key="section.title" class="market-block" :class="section.bannerClass ? `market-block-${section.bannerClass}` : ''">
            <div class="market-banner" :style="{ '--market-bg': `url(${section.image})` }">
              <div class="market-banner-copy">
                <h3>{{ section.title }}<BaseIcon name="arrowRight" /></h3>
                <p>{{ section.desc }}</p>
                <div class="market-tags">
                  <a v-for="tag in section.tags" :key="`${section.title}-${tag}`" :href="`#/catalog?category=${encodeURIComponent(tag)}`">
                    {{ tag }}
                  </a>
                </div>
              </div>
              <img :src="section.products[0].image" :alt="section.title" />
            </div>
            <div class="market-products">
              <a v-for="product in section.products" :key="`${section.title}-${product.name}`" :href="productDetailHref(product)">
                <img :src="product.image" :alt="product.name" />
                <span>{{ product.name }}</span>
                <strong>{{ product.price }}</strong>
              </a>
            </div>
          </article>
        </div>
      </section>

      <section class="catalog-section home-floor recommend-floor">
        <div class="recommend-floor-head">
          <div>
            <h2>为您推荐</h2>
            <p>根据您的浏览，为您推荐热销工业商品</p>
          </div>
          <button type="button" class="recommend-refresh" @click="cycleRecommendations">换一批<BaseIcon name="arrowRight" /></button>
        </div>
        <div class="recommend-grid">
          <article v-for="product in recommendedProducts" :key="`home-rec-${product.name}-${product.price}`" class="recommend-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
            <a class="recommend-image" :href="productDetailHref(product)" @click.stop>
              <img :src="product.image" :alt="product.name" />
              <span v-if="product.video" class="play-mark"></span>
            </a>
            <h3>{{ product.name }}</h3>
            <div class="recommend-tags">
              <span v-for="tag in product.serviceTags" :key="`${product.name}-${tag}`">{{ tag }}</span>
            </div>
            <div class="recommend-price-row">
              <strong>{{ product.price }}</strong>
              <em><BaseIcon name="home" />{{ product.location }}</em>
            </div>
            <div class="recommend-meta">
              <span><BaseIcon name="diamond" />{{ product.years }}</span>
              <p><BaseIcon name="shield" />{{ product.supplier }}</p>
            </div>
          </article>
        </div>
      </section>
    </section>
  </main>

  <main v-else-if="isCatalogPage" class="catalog-page">
    <section class="catalog-hero">
      <div class="catalog-inner">
        <div class="catalog-title">
          <span>商品名录</span>
          <h1>{{ currentCategory }}采购专区</h1>
          <p>聚合源头工厂、认证供应商与可定制工业设备，支持快速询价和批量采购。</p>
        </div>
        <div class="catalog-search">
          <button>货源<BaseIcon name="arrowRight" /></button>
          <input :value="`搜索${currentCategory}产品、型号、供应商`" readonly />
          <button class="catalog-search-submit"><BaseIcon name="search" /> 搜索</button>
        </div>
      </div>
    </section>

    <section class="catalog-wrap">
      <div class="catalog-crumb">
        <a href="#/">首页</a>
        <span>/</span>
        <strong>{{ currentCategory }}</strong>
        <em>部分商品支持来图定制，具体以供应商报价为准</em>
      </div>

      <div class="catalog-filter-panel">
        <div v-for="filter in catalogFilters" :key="filter.label" class="filter-row">
          <strong>{{ filter.label }}</strong>
          <button v-for="value in filter.values" :key="value" type="button" :class="{ active: activeCatalogFilters[filter.label] === value }" @click="selectCatalogFilter(filter.label, value)">{{ value }}</button>
        </div>
      </div>

      <div class="catalog-toolbar">
        <div class="sorts">
          <button v-for="sort in ['综合排序', '人气排序', '价格', '交付速度']" :key="sort" type="button" :class="{ active: activeCatalogSort === sort }" @click="activeCatalogSort = sort">{{ sort }}</button>
        </div>
        <div class="chips">
          <span>所有地区</span>
          <span>实力供应商</span>
          <span>已验厂企业</span>
          <span>安心购</span>
        </div>
      </div>

      <div class="catalog-grid">
        <article v-for="product in sortedCatalogProducts" :key="`${product.name}-${product.price}`" class="catalog-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
          <div class="catalog-card-image">
            <img :src="product.image" :alt="product.name" />
          </div>
          <h3>{{ product.name }}</h3>
          <div class="catalog-tags">
            <span>{{ product.badge }}</span>
            <span>{{ product.tag }}</span>
          </div>
          <div class="catalog-card-foot">
            <strong>{{ product.price }}</strong>
            <button type="button" @click.stop="openQuoteDialog">立即询价</button>
          </div>
          <p>擎采云认证供应商 · 支持批量报价</p>
        </article>
      </div>
    </section>
  </main>

  <main v-else-if="isCompanyPage" class="company-page">
    <section class="company-wrap company-archive-wrap">
      <div class="company-crumb">
        <a href="#/">首页</a><span>/</span><a href="#/catalog?category=机械设备">实力厂商</a><span>/</span><strong>{{ currentStoreSupplier.name }}</strong>
      </div>

      <article class="company-archive-card">
        <template v-if="companyTab === 'home'">
          <section class="company-home-summary">
            <div><span class="company-home-kicker"><BaseIcon name="factory" />认证供应商</span><h1>{{ currentStoreSupplier.name }}</h1><p>{{ companyDetails.introduction || `专注于 ${currentStoreSupplier.category} 领域，为工业采购、工程项目和批量供货提供稳定货源与按需定制支持。` }}</p></div>
            <dl><div><dt>{{ companyStatistics.operatingYears || '—' }}{{ companyStatistics.operatingYears ? ' 年' : '' }}</dt><dd>经营年限</dd></div><div><dt>{{ companyStatistics.responseRate || '—' }}{{ companyStatistics.responseRate ? '%' : '' }}</dt><dd>响应率</dd></div><div><dt>{{ companyStatistics.onSaleProductCount ?? currentStoreSupplier.products.length }} 款</dt><dd>在售商品</dd></div></dl>
          </section>
          <section class="company-tab-section">
            <div class="company-tab-heading"><h2>店铺推荐</h2><span>源头供货 · 支持定制</span></div>
            <div class="company-tab-products">
              <a v-for="product in currentStoreSupplier.products" :key="product.name" :href="productDetailHref(product)"><img :src="product.image" :alt="product.name" /><div><strong>{{ product.name }}</strong><span>认证供应商 · 支持询价</span><b>{{ product.price }}</b></div></a>
            </div>
          </section>
        </template>

        <section v-else-if="companyTab === 'products'" class="company-tab-section company-all-products">
          <div class="company-tab-heading"><h1>供应商品</h1><span>{{ currentStoreSupplier.products.length }} 款在售商品</span></div>
          <div class="company-tab-products">
            <a v-for="product in currentStoreSupplier.products" :key="product.name" :href="productDetailHref(product)"><img :src="product.image" :alt="product.name" /><div><strong>{{ product.name }}</strong><span>认证供应商 · 支持询价</span><b>{{ product.price }}</b></div></a>
          </div>
        </section>

        <template v-else>
          <section class="company-archive-section">
            <div class="company-archive-heading"><h1>企业简介</h1><span>企业档案</span></div>
            <p>{{ companyDetails.introduction || `${currentStoreSupplier.name} 是擎采云认证供应商，专注于 ${currentStoreSupplier.category} 领域，主营 ${currentStoreSupplier.products.map((product) => product.name).join('、')}。` }}</p>
          </section>

          <section class="company-archive-section company-verification-section">
            <div class="company-archive-heading"><h2>认证信息</h2><span>平台核验</span></div>
            <div class="company-verification-list">
              <div><i><BaseIcon name="shield" /></i><strong>主体资质核查</strong><span>{{ companyVerification.subjectVerified ? '企业名称与经营主体已核验' : '主体资质待核验' }}</span></div>
              <div><i><BaseIcon name="document" /></i><strong>验厂信息</strong><span>{{ companyVerification.factoryAudited ? '已完成验厂核验' : '暂未完成验厂核验' }}</span></div>
            </div>
          </section>

          <section class="company-archive-section company-business-section">
            <div class="company-archive-heading"><h2>工商信息</h2><span>企业信息已核验</span></div>
            <table class="company-business-table">
              <tbody>
                <tr><th>企业名称</th><td>{{ currentStoreSupplier.name }}</td><th>注册资本</th><td>{{ companyBusinessInfo.registeredCapital ? `${companyBusinessInfo.registeredCapital} ${companyBusinessInfo.registeredCapitalCurrency || ''}` : '—' }}</td></tr>
                <tr><th>注册地址</th><td>{{ companyBusinessInfo.registeredAddress || supplierLocation(currentStoreSupplier) || '—' }}</td><th>企业网址</th><td>{{ companyBusinessInfo.websiteUrl || '—' }}</td></tr>
                <tr><th>营业执照</th><td>{{ companyVerification.subjectVerified ? '平台已核验' : '待核验' }}</td><th>统一社会信用代码</th><td>{{ companyBusinessInfo.unifiedSocialCreditCodeMasked || '—' }}</td></tr>
                <tr><th>组织机构代码</th><td>{{ companyBusinessInfo.organizationCode || '—' }}</td><th>纳税人识别号</th><td>{{ companyBusinessInfo.taxpayerId || '—' }}</td></tr>
                <tr><th>工商注册号</th><td>{{ companyBusinessInfo.registrationNumber || '—' }}</td><th>法定代表人</th><td>{{ companyBusinessInfo.legalRepresentative || '—' }}</td></tr>
                <tr><th>经营状态</th><td>{{ displayCode(companyBusinessInfo.businessStatus) }}</td><th>成立时间</th><td>{{ companyBusinessInfo.establishedAt || '—' }}</td></tr>
                <tr><th>营业期限</th><td>{{ companyBusinessInfo.businessTermEnd || companyBusinessInfo.businessTermStart || '长期' }}</td><th>核验日期</th><td>{{ companyVerification.verifiedAt || '—' }}</td></tr>
                <tr><th>企业类型</th><td>{{ displayCode(companyBusinessInfo.companyType) }}</td><th>所属行业</th><td>{{ currentStoreSupplier.category || '—' }}</td></tr>
                <tr><th>经营范围</th><td colspan="3">{{ companyBusinessInfo.businessScope || '—' }}</td></tr>
                <tr><th>登记机关</th><td colspan="3">{{ companyBusinessInfo.registrationAuthority || '—' }}</td></tr>
              </tbody>
            </table>
            <p class="company-business-note"><BaseIcon name="shield" />以上企业信息由平台认证资料整理展示</p>
          </section>
        </template>
      </article>
    </section>
  </main>

  <main v-else class="detail-page">
    <section class="detail-wrap">
      <div class="detail-crumb">
        <a href="#/">首页</a><span>/</span><a href="#/catalog?category=机械设备">机械设备</a><span>/</span><strong>{{ detailProduct.name }}</strong>
      </div>

      <div class="detail-summary-layout">
        <div class="detail-main-column">
        <section class="detail-summary">
        <div class="detail-gallery">
          <div class="detail-main-image">
            <img :src="detailGallery[activeDetailImage]" :alt="detailProduct.name" />
            <span>厂家实拍</span>
          </div>
          <div class="detail-thumbnails">
            <button v-for="(image, index) in detailGallery" :key="image" :class="{ active: index === activeDetailImage }" type="button" @click="activeDetailImage = index">
              <img :src="image" :alt="`${detailProduct.name}图${index + 1}`" />
            </button>
          </div>
        </div>

        <section class="detail-purchase">
            <div class="detail-tags"><span v-for="tag in detailServiceTags" :key="tag">{{ tag }}</span></div>
          <h1>{{ detailProduct.name }}</h1>
          <p class="detail-subtitle">{{ detailProduct.tag }} · 支持工程采购、批量报价与按需定制</p>
          <div class="detail-price-panel">
            <span>参考报价</span>
            <strong>{{ detailProduct.price }}</strong>
            <em>具体价格以询价结果为准</em>
          </div>
          <dl class="detail-facts">
            <div><dt>服务保障</dt><dd><span v-for="tag in detailServiceTags" :key="tag">{{ tag }}</span></dd></div>
            <div><dt>发货地</dt><dd>{{ detailProduct.location || '—' }}　{{ detailProduct.raw?.shipWithinHours ? `预计 ${detailProduct.raw.shipWithinHours} 小时内发货` : '交付时间以沟通为准' }}</dd></div>
            <div><dt>规格选择</dt><dd class="detail-specs"><button v-for="spec in detailSpecs" :key="spec.id" type="button" :class="{ active: selectedSpec === spec.label }" @click="selectedSpec = spec.label">{{ spec.label }}</button></dd></div>
            <div><dt>采购数量</dt><dd class="detail-quantity"><button type="button" @click="decreaseQuantity">−</button><input :value="purchaseQuantity" aria-label="采购数量" readonly /><button type="button" @click="increaseQuantity">＋</button><span>{{ detailProduct.raw?.minOrderQuantity || 1 }} {{ detailProduct.raw?.unit || '件' }}起订</span></dd></div>
          </dl>
          <div class="detail-actions">
            <button type="button" class="detail-inquiry" @click="openQuoteDialog"><BaseIcon name="headset" />立即询价</button>
            <button type="button" class="detail-list"><BaseIcon name="document" />加入采购清单</button>
          </div>
        </section>

        </section>

      <section class="detail-content-grid">
        <div class="detail-description">
          <nav class="detail-anchor-nav"><a class="active" href="#product-info">商品详情</a><a href="#product-info">产品参数</a><a href="#purchase-notes">采购说明</a></nav>
          <article id="product-info" class="detail-info-card">
            <h2>产品参数</h2>
            <div class="detail-parameter-grid">
              <div><span>产品名称</span><strong>{{ detailProduct.name }}</strong></div>
              <div><span>商品型号</span><strong>{{ detailProduct.raw?.model || '—' }}</strong></div>
              <div><span>供货方式</span><strong>{{ detailSupplyMethods.join('、') }}</strong></div>
              <div><span>适用场景</span><strong>{{ detailScenarios.join('、') }}</strong></div>
              <div><span>质量服务</span><strong>{{ detailProduct.raw?.afterSalesService || '—' }}</strong></div>
              <div><span>交付周期</span><strong>{{ detailProduct.raw?.shipWithinHours ? `${detailProduct.raw.shipWithinHours} 小时内发货` : '以实际沟通为准' }}</strong></div>
            </div>
          </article>
          <article id="purchase-notes" class="detail-info-card detail-copy-card">
            <h2>商品说明</h2>
            <p>{{ detailProduct.raw?.description || detailProduct.raw?.shortDescription || '暂无商品说明。' }}</p>
            <div class="detail-feature-list"><span v-for="tag in detailFeatureTags" :key="tag"><BaseIcon name="shield" />{{ tag }}</span></div>
          </article>
        </div>
      </section>
        </div>

        <div class="detail-right-rail">
          <aside class="detail-supplier">
            <div class="detail-supplier-head"><span>认证供应商</span><BaseIcon name="shield" /></div>
            <div class="detail-supplier-name"><i :class="[{ 'has-logo': Boolean(detailSupplier.logo) }, `supplier-mark-${detailSupplier.tone}`]"><img v-if="detailSupplier.logo" :src="detailSupplier.logo" :alt="`${detailSupplier.name} Logo`" /><span v-else>{{ detailSupplier.name.slice(0, 1) }}</span></i><div><a :href="companyDetailHref(detailSupplier, 'home')">{{ detailSupplier.name }}</a><span>{{ detailSupplier.shortName }} · 已验厂企业</span></div></div>
            <p>主营：{{ detailSupplier.products.map((product) => product.name).join('、') }}</p>
            <div class="detail-supplier-tags"><span v-for="tag in detailSupplierTags" :key="tag">{{ tag }}</span></div>
            <div class="detail-supplier-stats"><span><b>{{ detailSupplier.raw?.statistics?.operatingYears || '—' }}{{ detailSupplier.raw?.statistics?.operatingYears ? ' 年' : '' }}</b>经营年限</span><span><b>{{ detailSupplier.raw?.statistics?.responseRate || '—' }}</b>响应率</span><span><b>{{ detailSupplierLocation || '—' }}</b>发货地</span></div>
            <div class="detail-supplier-actions">
              <button type="button" @click="openQuoteDialog">留言问价</button>
              <a :href="companyDetailHref(detailSupplier, 'archive')">企业档案<BaseIcon name="arrowRight" /></a>
            </div>
          </aside>

          <aside class="detail-related">
            <div class="detail-side-title"><h2>同店推荐</h2><a :href="companyDetailHref(detailSupplier, 'products')">查看更多</a></div>
            <a v-for="product in detailRelatedProducts" :key="product.name" :href="productDetailHref(product)" class="detail-related-item">
              <img :src="product.image" :alt="product.name" />
              <div><strong>{{ product.name }}</strong><span>{{ product.tag }}</span><b>{{ product.price }}</b></div>
            </a>
          </aside>
        </div>
      </div>
    </section>
  </main>

  <div v-if="quoteDialogOpen" class="quote-dialog-backdrop" role="presentation" @click.self="closeQuoteDialog">
    <section class="quote-dialog" role="dialog" aria-modal="true" aria-labelledby="quote-dialog-title">
      <button class="quote-dialog-close" type="button" aria-label="关闭留言问价" @click="closeQuoteDialog">×</button>
      <template v-if="!quoteSubmitted">
        <div class="quote-dialog-heading">
          <span><BaseIcon name="headset" /></span>
          <div><h2 id="quote-dialog-title">留言问价</h2><p>留下采购需求，商家会尽快为您提供报价与选型建议。</p></div>
        </div>
        <form class="quote-form" @submit.prevent="submitQuote">
          <label>咨询商品<input :value="quoteProductLabel" readonly /></label>
          <label>采购需求<textarea v-model="quoteMessage" required maxlength="300" placeholder="例如：采购数量、型号规格、交付时间等"></textarea></label>
          <div class="quote-form-row">
            <label>联系人<input v-model.trim="quoteContactName" required maxlength="80" placeholder="请输入您的称呼" /></label>
            <label>联系电话<input v-model.trim="quoteContactPhone" required type="tel" maxlength="32" placeholder="便于商家联系您" /></label>
          </div>
          <p class="quote-form-note">提交后仅向 {{ currentStoreSupplier.name }} 发送本次询价信息。</p>
          <p v-if="quoteError" class="quote-form-note">{{ quoteError }}</p>
          <div class="quote-form-actions"><button type="button" @click="closeQuoteDialog">取消</button><button type="submit" :disabled="quoteSubmitting">{{ quoteSubmitting ? '提交中…' : '提交询价' }}</button></div>
        </form>
      </template>
      <div v-else class="quote-submitted">
        <span><BaseIcon name="shield" /></span>
        <h2 id="quote-dialog-title">留言已提交</h2>
        <p>商家将根据您的采购需求尽快与您联系。</p>
        <button type="button" @click="closeQuoteDialog">我知道了</button>
      </div>
    </section>
  </div>

  <footer class="site-footer">
    <div class="footer-main">
      <section class="footer-brand-block">
        <div class="footer-brand">
          <img src="/images/logo-qingcaiyun-light.png" alt="擎采云" />
        </div>
        <h2>汇聚优质工业资源　助力企业高效采购</h2>
        <div class="footer-stats">
          <div>
            <i><BaseIcon name="diamond" /></i>
            <strong>海量商品</strong>
            <span>品类齐全</span>
          </div>
          <div>
            <i><BaseIcon name="factory" /></i>
            <strong>优质商家</strong>
            <span>源头直供</span>
          </div>
          <div>
            <i><BaseIcon name="thumb" /></i>
            <strong>高效对接</strong>
            <span>快速响应</span>
          </div>
          <div>
            <i><BaseIcon name="shield" /></i>
            <strong>安全交易</strong>
            <span>平台保障</span>
          </div>
        </div>
      </section>

      <nav class="footer-links">
        <div>
          <h3>买家服务</h3>
          <a href="#">发布采购需求</a>
          <a href="#">采购流程</a>
          <a href="#">采购指南</a>
          <a href="#">常见问题</a>
        </div>
        <div>
          <h3>卖家服务</h3>
          <a href="#">商家入驻</a>
          <a href="#">入驻流程</a>
          <a href="#">商家服务</a>
          <a href="#">商家中心</a>
        </div>
        <div>
          <h3>平台支持</h3>
          <a href="#">平台规则</a>
          <a href="#">服务协议</a>
          <a href="#">隐私政策</a>
          <a href="#">帮助中心</a>
        </div>
        <div>
          <h3>关于我们</h3>
          <a href="#">平台介绍</a>
          <a href="#">联系我们</a>
          <a href="#">商务合作</a>
          <a href="#">意见反馈</a>
        </div>
      </nav>

    </div>

    <div class="footer-bottom">
      <div class="friend-links">
        <strong>友情链接：</strong>
        <a href="#">工业设备</a>
        <a href="#">建材家居</a>
        <a href="#">电子仪表</a>
        <a href="#">化工能源</a>
        <a href="#">汽配汽用</a>
        <a href="#">更多<BaseIcon name="arrowRight" /></a>
      </div>
      <p>© 2026 擎采云 工业采购平台 版权所有　|　ICP备案号：待提供　|　公安备案号：待提供</p>
      <a class="back-top" href="#/">
        <BaseIcon name="arrowRight" />
        <span>返回顶部</span>
      </a>
    </div>
  </footer>
</template>
