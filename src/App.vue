<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref } from 'vue'
import { ElDropdown, ElDropdownItem, ElDropdownMenu } from 'element-plus'
import 'element-plus/es/components/dropdown/style/css'
import BaseIcon from './components/BaseIcon.vue'
const navItems = ['首页', '实力优品', '采购资讯', '发布采购需求', '商家入驻']
const navHrefs = { 首页: '#/', 实力优品: '#/catalog', 采购资讯: '#/news', 发布采购需求: '#/procurement', 商家入驻: '#/merchant-join' }
const serviceHighlights = [{ title: '认证供应商', desc: '真实企业资料核验', icon: 'shield' }, { title: '按需采购', desc: '支持询价与定制', icon: 'document' }, { title: '快速响应', desc: '采购线索人工跟进', icon: 'headset' }, { title: '采购保障', desc: '清晰的商品信息', icon: 'thumb' }]
const categoryPosters = [{ title: '工业采购', desc: '认证供应商，真实货源', image: '/images/industry-solution.png', href: '#/catalog' }]
const featureCards = [{ title: '源头工厂', desc: '优选认证供应商', action: '查看货源', image: '/images/factory-direct.png' }, { title: '行业解决方案', desc: '适配工业采购场景', action: '了解更多', image: '/images/smart-manufacturing.png' }]
const catalogFilters = [{ label: '供应方式', values: [{ label: '全部', value: '' }, { label: '现货', value: 'spot' }, { label: '支持定制', value: 'custom' }] }, { label: '认证服务', values: [{ label: '全部', value: '' }, { label: '平台认证', value: 'platform' }, { label: '已验厂', value: 'factory' }] }]
const homeSlides = [
  { eyebrow: '智能制造', title: '智能制造\n驱动产业升级', desc: '甄选优质工业设备，助力企业高质量发展', image: '/images/smart-manufacturing-bg.png', category: '机械设备' },
  { eyebrow: '工程建材', title: '工程物料\n一站式采购', desc: '从基础材料到施工配套，货源清晰、交付可控', image: '/images/market-building-banner.png', category: '建材家居' },
  { eyebrow: '电子仪表', title: '精准选型\n让生产更稳定', desc: '汇集专业仪表与工控设备，支持批量询价', image: '/images/market-instrument-banner.png', category: '电子仪表' },
]

const routeHash = ref(window.location.hash || '#/')
const activeCategory = ref(null)
const activeSupplierCategory = ref('')
const activeDetailImage = ref(0)
const selectedSpec = ref('标准配置')
const purchaseQuantity = ref(1)
const recommendationOffset = ref(0)
const featuredProductOffset = ref(0)
const featuredVisibleCount = ref(window.innerWidth <= 599 ? 2 : window.innerWidth <= 899 ? 3 : 4)
const featuredCarouselDirection = ref('next')
const featuredCarouselAnimating = ref(false)
let featuredCarouselTimer
const activeHomeSlide = ref(0)
let homeCarouselTimer
const activeCatalogSort = ref('综合排序')
const activeCatalogFilters = ref({ '供应方式': '', '认证服务': '' })
const quoteDialogOpen = ref(false)
const quoteSubmitted = ref(false)
const quoteMessage = ref('')
const quoteContactName = ref('')
const quoteContactPhone = ref('')
const quoteSubmitting = ref(false)
const quoteError = ref('')
const activeDetailSection = ref('product-details')
const categories = ref([])
// 首页数据与实力优品查询结果分离，名录的筛选或空结果不能覆盖首页内容。
const homeProducts = ref([])
const homeSupplierShowcases = ref([])
const homeLatestNews = ref([])
const catalogProducts = ref([])
// 首页精选与实力优品使用独立的数据源，避免从名录页返回后覆盖首页当前分类。
const featuredProducts = ref([])
const activeFeaturedCategoryId = ref('')
const featuredProductsLoading = ref(false)
const supplierShowcases = ref([])
const selectedProduct = ref(null)
const selectedCompany = ref(null)
const apiError = ref('')
const homeStatistics = ref({})
const latestNews = ref([])
const selectedNews = ref(null)
const homepageNewsCoverByTitle = {
  '制造企业采购如何建立合格供应商清单': '/images/news/supplier-qualification.png',
  '工业品询价前需要确认的六项参数': '/images/news/rfq-technical-parameters.png',
  '供应链韧性：从单一货源到多源协同': '/images/news/supply-chain-resilience.png',
}
const homepageNewsCover = (item) => homepageNewsCoverByTitle[item.title] || item.coverImage || '/images/news/supplier-qualification.png'
const searchKeyword = ref('')
const searchType = ref('product')
const catalogLoading = ref(false)
const catalogTotal = ref(0)
const readPurchaseList = () => {
  try {
    const stored = JSON.parse(localStorage.getItem('qcy-purchase-list') || '[]')
    return Array.isArray(stored) ? stored : []
  } catch {
    return []
  }
}
const purchaseList = ref(readPurchaseList())
const selectedPurchaseKeys = ref([])
const purchaseListFeedback = ref('')
const quoteTarget = ref(null)
const staticPage = ref('')
const showBackToTop = ref(false)
const procurementOpen = ref(false)
const procurementSubmitting = ref(false)
const procurementMessage = ref('')
const procurementForm = ref({ categoryId: '', description: '', quantity: '', unit: '件', contactName: '', contactPhone: '' })
const procurementCategoryOpen = ref(false)

const displayLabels = {
  platform_verified: '平台认证', factory_audited: '已验厂', customization: '支持定制', fast_shipping: '快速发货',
  after_sales: '售后保障', platform_guarantee: '平台保障', source_factory: '源头工厂', source_supply: '源头直供',
  hot_sale: '热销优品', spot: '现货供应', custom: '支持定制', preorder: '预约供货',
  production_line: '生产线', workshop: '车间', engineering_project: '工程项目', selection_support: '选型支持',
  project_service: '项目服务', active: '存续', cancelled: '注销', revoked: '吊销', moved_out: '迁出',
  limited_liability_company: '有限责任公司', sole_proprietorship: '个人独资企业', partnership: '合伙企业', individual_business: '个体工商户',
  in_stock: '现货充足', low_stock: '库存紧张', out_of_stock: '暂时缺货',
}
const displayCode = (value, fallback = '—') => {
  if (!value) return fallback
  // 兼容历史种子数据：model_hz-pk-6050 这类内部键不直接暴露给用户。
  if (value.startsWith('model_')) return `型号：${value.slice('model_'.length).toUpperCase()}`
  return displayLabels[value] || value
}
const displayCodes = (values, fallback = []) => {
  const labels = (Array.isArray(values) ? values : []).map((value) => displayCode(value, '')).filter(Boolean)
  return labels.length ? labels : fallback
}
const formatNewsDate = (value) => value ? String(value).slice(0, 10).replaceAll('-', '.') : ''

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
const productTagParts = (tag) => {
  const parts = String(tag || '')
    .split(/[·•|｜]/)
    .map((part) => part.trim())
    .filter(Boolean)
  return parts.length > 1 ? [parts[0], parts.slice(1).join(' / ')] : parts
}
const catalogFilterProfile = (item, index = 0) => {
  const identity = String(item.productId || item.productName || index)
  const checksum = [...identity].reduce((total, character) => total + character.charCodeAt(0), 0)
  return {
    supplyMethod: checksum % 2 === 0 ? 'spot' : 'custom',
    verification: Math.floor(checksum / 2) % 2 === 0 ? 'platform_verified' : 'factory_audited',
  }
}
const toProduct = (item, index = 0) => {
  const filterProfile = catalogFilterProfile(item, index)
  const serviceGuarantees = (item.serviceGuarantees || []).filter((service) => !['platform_verified', 'platform_guarantee', 'factory_audited'].includes(service))
  const normalizedItem = {
    ...item,
    supplyMethod: [filterProfile.supplyMethod],
    serviceGuarantees: [...serviceGuarantees, filterProfile.verification],
  }
  return {
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
  raw: normalizedItem,
  }
}
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
    const data = await api('/public/home')
    const { categories: categoryData, featuredProducts: productData, featuredCompanies: companyData } = data
    categories.value = categoryData.map((root) => ({
      id: root.categoryId, name: root.categoryName, icon: root.icon || 'grid',
      groups: (root.children || []).map((group) => ({ id: group.categoryId, title: group.categoryName, items: (group.children || []).map((child) => ({ id: child.categoryId, name: child.categoryName })) })),
    }))
    homeProducts.value = productData.map(toProduct)
    homeSupplierShowcases.value = companyData.map(toSupplier)
    homeStatistics.value = data.statistics || {}
    homeLatestNews.value = data.latestNews || []
    activeSupplierCategory.value = homeSupplierShowcases.value[0]?.category || ''
    // 每次进入首页都从第一个一级分类开始，不继承实力优品页的 categoryId。
    await selectFeaturedCategory(categories.value[0]?.id)
  } catch (error) {
    apiError.value = error.message || '暂时无法加载商城数据'
  }
}

let featuredCategoryRequestId = 0
const selectFeaturedCategory = async (categoryId) => {
  if (!categoryId) {
    activeFeaturedCategoryId.value = ''
    featuredProducts.value = []
    featuredProductsLoading.value = false
    return
  }
  const requestId = ++featuredCategoryRequestId
  activeFeaturedCategoryId.value = categoryId
  featuredProductOffset.value = 0
  featuredCarouselAnimating.value = false
  featuredProducts.value = []
  featuredProductsLoading.value = true
  try {
    const data = await api(`/public/products?${new URLSearchParams({ categoryId, page: '1', pageSize: '12' })}`)
    // 快速切换标签时，只接收最后一次请求的结果。
    if (requestId === featuredCategoryRequestId) featuredProducts.value = (data.items || []).map(toProduct)
  } catch (error) {
    if (requestId === featuredCategoryRequestId) {
      featuredProducts.value = []
      apiError.value = error.message || '精选商品加载失败'
    }
  } finally {
    if (requestId === featuredCategoryRequestId) featuredProductsLoading.value = false
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
    if (isCatalogPage.value) {
      catalogLoading.value = true
      searchKeyword.value = params.get('keyword') || ''
      searchType.value = params.get('type') || 'product'
      activeCatalogFilters.value = {
        '供应方式': params.get('supplyMethod') || '',
        '认证服务': params.get('verification') || '',
      }
      activeCatalogSort.value = ({ popularity: '人气排序', price_asc: '价格', delivery_speed: '交付速度' })[params.get('sort')] || '综合排序'
      const endpoint = searchType.value === 'company' ? '/public/companies' : '/public/products'
      // `type` 仅用于前端决定展示商品还是供应商；公开 API 使用不同的
      // endpoint，并且会拒绝未声明的查询参数。不能将它原样转发，否则请求
      // 失败后会保留首页的旧数据，造成“搜索总是全部结果”的假象。
      const apiParams = new URLSearchParams(params)
      apiParams.delete('type')
      const data = await api(`${endpoint}?${apiParams.toString()}`)
      if (searchType.value === 'company') supplierShowcases.value = (data.items || []).map(toSupplier)
      else catalogProducts.value = (data.items || []).map(toProduct)
      catalogTotal.value = data.total || 0
    }
    if (isNewsPage.value) {
      if (params.get('id')) selectedNews.value = await api(`/public/news/${encodeURIComponent(params.get('id'))}`)
      else {
        selectedNews.value = null
        latestNews.value = (await api('/public/news?page=1&pageSize=20')).items || []
      }
    }
  } catch (error) {
    if (isCatalogPage.value) {
      // 搜索请求异常时不能继续展示首页缓存的数据，以免误导为匹配结果。
      if (searchType.value === 'company') supplierShowcases.value = []
      else catalogProducts.value = []
      catalogTotal.value = 0
    }
    apiError.value = error.message || '暂时无法加载详情数据'
  } finally {
    catalogLoading.value = false
  }
}

const updateRoute = () => {
  routeHash.value = window.location.hash || '#/'
  // Hash 路由切换后始终从新页面顶部开始，避免保留上一页的滚动位置。
  window.scrollTo({ top: 0, left: 0, behavior: 'auto' })
  // 回到首页时始终恢复第一个分类的精选内容，不保留此前名录页或标签页的筛选状态。
  if (routeHash.value === '#/' || routeHash.value === '') void selectFeaturedCategory(categories.value[0]?.id)
  void loadRouteData()
}
const updateBackToTopVisibility = () => {
  showBackToTop.value = window.scrollY > 320
}
const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: window.matchMedia('(prefers-reduced-motion: reduce)').matches ? 'auto' : 'smooth',
  })
}

onMounted(() => {
  window.addEventListener('hashchange', updateRoute)
  window.addEventListener('resize', updateFeaturedVisibleCount)
  window.addEventListener('scroll', updateBackToTopVisibility, { passive: true })
  document.addEventListener('click', closeProcurementCategoryOnOutsideClick)
  updateBackToTopVisibility()
  startFeaturedProductCarousel()
  startHomeCarousel()
  void loadHome().then(loadRouteData)
})

onUnmounted(() => {
  window.removeEventListener('hashchange', updateRoute)
  window.removeEventListener('resize', updateFeaturedVisibleCount)
  window.removeEventListener('scroll', updateBackToTopVisibility)
  document.removeEventListener('click', closeProcurementCategoryOnOutsideClick)
  window.clearInterval(featuredCarouselTimer)
  window.clearInterval(homeCarouselTimer)
})

const currentCategory = computed(() => {
  if (!routeHash.value.startsWith('#/catalog')) return ''
  const [, query = ''] = routeHash.value.split('?')
  const categoryId = new URLSearchParams(query).get('categoryId')
  return categories.value.find((item) => item.id === categoryId)?.name || '实力优品'
})

const isCatalogPage = computed(() => routeHash.value.startsWith('#/catalog'))
const isProductDetailPage = computed(() => routeHash.value.startsWith('#/product'))
const isCompanyPage = computed(() => routeHash.value.startsWith('#/company'))
const isNewsPage = computed(() => routeHash.value.startsWith('#/news'))
const isStaticPage = computed(() => routeHash.value.startsWith('#/static'))
const isPurchaseListPage = computed(() => routeHash.value.startsWith('#/purchase-list'))
const isProcurementPage = computed(() => routeHash.value.startsWith('#/procurement'))
const isMerchantJoinPage = computed(() => routeHash.value.startsWith('#/merchant-join'))
const isNavActive = (item) => {
  if (item === '首页') return routeHash.value === '#/' || routeHash.value === ''
  if (item === '实力优品') return isCatalogPage.value || isProductDetailPage.value
  if (item === '采购资讯') return isNewsPage.value
  if (item === '发布采购需求') return isProcurementPage.value
  if (item === '商家入驻') return isMerchantJoinPage.value
  return false
}

const categoryNames = computed(() => categories.value.map((item) => item.name))
const selectedProcurementCategory = computed(() => categories.value.find((category) => category.id === procurementForm.value.categoryId) || null)
const supplierShowcaseCategories = computed(() => [...new Set(homeSupplierShowcases.value.map((supplier) => supplier.category).filter(Boolean))])
const featuredCarouselProducts = computed(() => {
  const total = featuredProducts.value.length
  if (!total) return []
  const count = Math.min(featuredVisibleCount.value, total)
  const start = featuredCarouselDirection.value === 'previous'
    ? featuredProductOffset.value - 1
    : featuredProductOffset.value
  const length = total > count ? count + 1 : count
  return Array.from({ length }, (_, index) => featuredProducts.value[(start + index + total) % total])
})
const featuredTrackStyle = computed(() => {
  const count = featuredVisibleCount.value
  const gap = count === 2 ? 10 : 14
  const step = `calc(-${100 / count}% - ${gap / count}px)`
  const movesLeft = featuredCarouselDirection.value === 'next' && featuredCarouselAnimating.value
  const restsAfterPrevious = featuredCarouselDirection.value === 'previous' && !featuredCarouselAnimating.value
  return {
    gridAutoColumns: `calc(${100 / count}% - ${((count - 1) * gap) / count}px)`,
    transform: movesLeft || restsAfterPrevious ? `translateX(${step})` : 'translateX(0)',
  }
})

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
const detailDescriptionHtml = computed(() => detailProduct.value.raw?.description || '')
const detailOverview = computed(() => detailProduct.value.raw?.shortDescription || `${detailProduct.value.name}面向 ${detailScenarios.value.join('、')} 场景提供稳定供货，支持采购选型与批量询价。`)
const detailSellingPoints = computed(() => {
  const sellingPoints = detailProduct.value.raw?.sellingPoints || []
  return sellingPoints.length ? sellingPoints : [detailProduct.value.tag, ...detailFeatureTags.value].filter(Boolean)
})
const detailTechnicalParameters = computed(() => detailProduct.value.raw?.technicalParameters || [])
const formatTechnicalParameter = (parameter) => {
  const value = Array.isArray(parameter?.value) ? parameter.value.join('、') : parameter?.value
  return value ? `${value}${parameter?.unit || ''}` : '—'
}
const scrollToDetailSection = (sectionId) => {
  activeDetailSection.value = sectionId
  document.getElementById(sectionId)?.scrollIntoView({
    behavior: window.matchMedia('(prefers-reduced-motion: reduce)').matches ? 'auto' : 'smooth',
    block: 'start',
  })
}
const newsArticleHtml = computed(() => (selectedNews.value?.content || '').replace(/<h1[^>]*>[\s\S]*?<\/h1>/i, ''))
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
const quoteProductLabel = computed(() => quoteTarget.value?.name || (isCompanyPage.value ? `${currentStoreSupplier.value.name} 企业服务咨询` : detailProduct.value.name))
const companyTab = computed(() => {
  if (!isCompanyPage.value) return 'products'
  const [, query = ''] = routeHash.value.split('?')
  const tab = new URLSearchParams(query).get('tab')
  return ['home', 'products', 'archive'].includes(tab) ? tab : 'products'
})
const storeTabs = [
  { label: '供应商品', value: 'products' },
  { label: '企业档案', value: 'archive' },
  { label: '留言问价', value: 'quote' }
]

const productDetailHref = (product) => `#/product?id=${encodeURIComponent(product.id)}`
const companyDetailHref = (supplier, tab = 'products') => `#/company?id=${encodeURIComponent(supplier.id)}&tab=${tab}`
const openProduct = (product) => {
  window.location.hash = productDetailHref(product)
}
const openCompany = (supplier, tab = 'products') => {
  window.location.hash = companyDetailHref(supplier, tab)
}
const openCatalog = (category = '机械设备') => {
  const categoryId = categories.value.find((item) => item.name === category)?.id || ''
  window.location.hash = `#/catalog?categoryId=${encodeURIComponent(categoryId)}`
}
const openProcurement = () => {
  window.location.hash = '#/procurement'
}
const categoryHref = (id) => `#/catalog?categoryId=${encodeURIComponent(id)}`
const findCategoryByName = (name) => categories.value
  .flatMap((root) => [root, ...root.groups, ...root.groups.flatMap((group) => group.items)])
  .find((category) => category.name === name || category.title === name)
const marketSectionHref = (section) => categoryHref(findCategoryByName(section.category)?.id || '')
const marketTagHref = (section, tag) => {
  const category = findCategoryByName(tag.target)
  if (category?.id) return categoryHref(category.id)

  // 展示文案与后台类目名称不完全一致时，保留所属一级类目并使用关键词检索，
  // 避免旧的 `category` 参数被忽略后跳转成无筛选的实力优品页。
  const parentCategory = findCategoryByName(section.category)
  return `#/catalog?${new URLSearchParams({
    ...(parentCategory?.id ? { categoryId: parentCategory.id } : {}),
    keyword: tag.label,
    page: '1',
    pageSize: '20',
  })}`
}
const featuredCatalogHref = computed(() => categoryHref(activeFeaturedCategoryId.value || categories.value[0]?.id || ''))
const performSearch = () => { window.location.hash = `#/catalog?${new URLSearchParams({ type: searchType.value, keyword: searchKeyword.value.trim(), page: '1', pageSize: '20' })}` }
const searchTypeLabel = computed(() => searchType.value === 'company' ? '店铺' : '商品')
const catalogSearchTypeLabel = computed(() => searchType.value === 'company' ? '供应商' : '货源')
const selectSearchType = (type) => {
  if (searchType.value === type) return
  searchType.value = type
  searchKeyword.value = ''
  if (!isCatalogPage.value) return
  window.location.hash = `#/catalog?${new URLSearchParams({ type, page: '1', pageSize: '20' })}`
}
const applyCatalogFilter = (key, value) => { const params = new URLSearchParams(routeHash.value.split('?')[1] || ''); value ? params.set(key, value) : params.delete(key); params.set('page', '1'); window.location.hash = `#/catalog?${params}` }
const openQuoteFor = (product) => { openQuoteDialog(product) }
const purchaseItemKey = (item) => `${item.productId}-${item.skuId || 'default'}`
const persistPurchaseList = () => localStorage.setItem('qcy-purchase-list', JSON.stringify(purchaseList.value))
const selectedPurchaseItems = computed(() => purchaseList.value.filter((item) => selectedPurchaseKeys.value.includes(purchaseItemKey(item))))
const purchaseItemCount = computed(() => selectedPurchaseItems.value.reduce((total, item) => total + Math.max(1, Number(item.quantity) || 1), 0))
const allPurchaseItemsSelected = computed(() => purchaseList.value.length > 0 && selectedPurchaseKeys.value.length === purchaseList.value.length)
const addToPurchaseList = () => {
  const item = { productId: detailProduct.value.id, skuId: selectedSku.value?.skuId, name: detailProduct.value.name, image: detailProduct.value.image, spec: selectedSpec.value, company: detailSupplier.value.name, price: formatPrice(selectedSku.value), quantity: purchaseQuantity.value }
  const index = purchaseList.value.findIndex((entry) => entry.productId === item.productId && entry.skuId === item.skuId)
  if (index >= 0) purchaseList.value[index] = item
  else purchaseList.value.push(item)
  persistPurchaseList()
  purchaseListFeedback.value = index >= 0 ? '已更新采购清单中的商品数量' : '已加入采购清单'
  window.setTimeout(() => { purchaseListFeedback.value = '' }, 2400)
}
const removePurchaseItem = (index) => {
  const [removed] = purchaseList.value.splice(index, 1)
  selectedPurchaseKeys.value = selectedPurchaseKeys.value.filter((key) => key !== purchaseItemKey(removed))
  persistPurchaseList()
}
const updatePurchaseQuantity = (index, amount) => {
  const item = purchaseList.value[index]
  item.quantity = Math.max(1, (Number(item.quantity) || 1) + amount)
  persistPurchaseList()
}
const normalizePurchaseQuantity = (index) => {
  const item = purchaseList.value[index]
  item.quantity = Math.max(1, Number(item.quantity) || 1)
  persistPurchaseList()
}
const toggleAllPurchaseItems = () => {
  selectedPurchaseKeys.value = allPurchaseItemsSelected.value ? [] : purchaseList.value.map(purchaseItemKey)
}
const clearSelectedPurchaseItems = () => {
  purchaseList.value = purchaseList.value.filter((item) => !selectedPurchaseKeys.value.includes(purchaseItemKey(item)))
  selectedPurchaseKeys.value = []
  persistPurchaseList()
}
const createDemandFromList = () => {
  if (!selectedPurchaseItems.value.length) {
    purchaseListFeedback.value = '请先选择要询价的商品'
    window.setTimeout(() => { purchaseListFeedback.value = '' }, 2400)
    return
  }
  procurementMessage.value = ''
  procurementForm.value = {
    ...procurementForm.value,
    description: selectedPurchaseItems.value.map((item) => `${item.name}｜${item.spec || '标准配置'}｜${item.company}｜数量 ${item.quantity}`).join('\n'),
    quantity: String(purchaseItemCount.value),
    unit: '项',
  }
  window.location.hash = '#/procurement?from=list'
}
const selectProcurementCategory = (categoryId) => {
  procurementForm.value.categoryId = categoryId
  procurementCategoryOpen.value = false
}
const closeProcurementCategoryOnOutsideClick = (event) => {
  if (!event.target.closest('.procurement-category-select')) procurementCategoryOpen.value = false
}
const submitProcurement = async () => { if (!procurementForm.value.categoryId) { procurementMessage.value = '请选择采购类目'; return }; if (!/^1[3-9]\d{9}$/.test(procurementForm.value.contactPhone)) { procurementMessage.value = '请输入有效的中国大陆手机号'; return }; procurementSubmitting.value = true; try { await api('/public/procurement-demands', { method: 'POST', body: procurementForm.value }); procurementMessage.value = '采购需求已提交，我们将人工跟进。采购清单已保留，方便您后续查看。' } catch (error) { procurementMessage.value = error.message || '提交失败' } finally { procurementSubmitting.value = false } }
const cycleRecommendations = () => {
  if (homeProducts.value.length) recommendationOffset.value = (recommendationOffset.value + 4) % homeProducts.value.length
}
const updateFeaturedVisibleCount = () => {
  featuredVisibleCount.value = window.innerWidth <= 599 ? 2 : window.innerWidth <= 899 ? 3 : 4
  featuredProductOffset.value %= Math.max(1, featuredProducts.value.length)
}
const canSlideFeaturedProducts = () => featuredProducts.value.length > featuredVisibleCount.value && !featuredCarouselAnimating.value
const showPreviousFeaturedProduct = async () => {
  if (!canSlideFeaturedProducts()) return
  featuredCarouselDirection.value = 'previous'
  await nextTick()
  featuredCarouselAnimating.value = true
}
const showNextFeaturedProduct = () => {
  if (!canSlideFeaturedProducts()) return
  featuredCarouselDirection.value = 'next'
  featuredCarouselAnimating.value = true
}
const finishFeaturedProductSlide = (event) => {
  if (event.propertyName !== 'transform' || !featuredCarouselAnimating.value) return
  const total = featuredProducts.value.length
  featuredProductOffset.value = featuredCarouselDirection.value === 'next'
    ? (featuredProductOffset.value + 1) % total
    : (featuredProductOffset.value - 1 + total) % total
  featuredCarouselDirection.value = 'next'
  featuredCarouselAnimating.value = false
}
const startFeaturedProductCarousel = () => {
  featuredCarouselTimer = window.setInterval(showNextFeaturedProduct, 5000)
}
const showHomeSlide = (index) => {
  activeHomeSlide.value = (index + homeSlides.length) % homeSlides.length
}
const showNextHomeSlide = () => showHomeSlide(activeHomeSlide.value + 1)
const showPreviousHomeSlide = () => showHomeSlide(activeHomeSlide.value - 1)
const pauseHomeCarousel = () => window.clearInterval(homeCarouselTimer)
const startHomeCarousel = () => {
  window.clearInterval(homeCarouselTimer)
  if (homeSlides.length < 2 || window.matchMedia('(prefers-reduced-motion: reduce)').matches) return
  homeCarouselTimer = window.setInterval(showNextHomeSlide, 5200)
}
const selectHomeSlide = (index) => {
  showHomeSlide(index)
  startHomeCarousel()
}
const selectCatalogFilter = (label, value) => {
  activeCatalogFilters.value = { ...activeCatalogFilters.value, [label]: value }
}
const catalogFilterTokens = (value) => {
  if (value == null) return []
  if (Array.isArray(value)) return value.flatMap(catalogFilterTokens)
  if (typeof value === 'object') return Object.entries(value).flatMap(([key, item]) => item === true ? [key] : catalogFilterTokens(item))
  return [String(value).trim().toLowerCase()]
}
const includesCatalogFilterToken = (tokens, candidates) => candidates.some((candidate) => tokens.some((token) => token === candidate || token.includes(candidate)))
const filteredCatalogProducts = computed(() => {
  const supplyMethod = activeCatalogFilters.value['供应方式']
  const verification = activeCatalogFilters.value['认证服务']
  return catalogProducts.value.filter((product) => {
    const raw = product.raw || {}
    const explicitSupplyTokens = catalogFilterTokens([raw.supplyMethod, raw.supplyMethods, raw.supplyType])
    const supplyTokens = explicitSupplyTokens.length ? explicitSupplyTokens : catalogFilterTokens([raw.sellingPoints, raw.shortDescription, product.tag])
    const explicitServiceTokens = catalogFilterTokens([raw.serviceGuarantees, raw.serviceGuarantee, raw.certifications, raw.company?.verification, raw.verification])
    const serviceTokens = explicitServiceTokens.length ? explicitServiceTokens : []
    const matchesSupply = !supplyMethod
      || (supplyMethod === 'spot' && includesCatalogFilterToken(supplyTokens, ['spot', 'in_stock', '现货', '现货供应']))
      || (supplyMethod === 'custom' && includesCatalogFilterToken(supplyTokens, ['custom', 'customization', '支持定制', '按需定制', '定制']))
    const matchesVerification = !verification
      || (verification === 'platform' && includesCatalogFilterToken(serviceTokens, ['platform_verified', 'platform_guarantee', 'subject_verified', '平台认证', '平台保障', '主体资质']))
      || (verification === 'factory' && includesCatalogFilterToken(serviceTokens, ['factory_audited', 'factory', '已验厂', '验厂']))
    return matchesSupply && matchesVerification
  })
})
const sortedCatalogProducts = computed(() => {
  const products = filteredCatalogProducts.value
  if (activeCatalogSort.value === '价格') return [...products].sort((a, b) => Number(a.raw?.defaultSku?.displayPrice || Infinity) - Number(b.raw?.defaultSku?.displayPrice || Infinity))
  if (activeCatalogSort.value === '人气排序') return [...products].sort((a, b) => (b.raw?.salesCount || 0) - (a.raw?.salesCount || 0))
  if (activeCatalogSort.value === '交付速度') return [...products].sort((a, b) => Number(a.raw?.shipWithinHours || Infinity) - Number(b.raw?.shipWithinHours || Infinity))
  return products
})
const decreaseQuantity = () => {
  purchaseQuantity.value = Math.max(Number(selectedSku.value?.minOrderQuantity || detailProduct.value.raw?.minOrderQuantity || 1), purchaseQuantity.value - 1)
}
const increaseQuantity = () => {
  purchaseQuantity.value += 1
}
const openQuoteDialog = (product = null) => {
  quoteTarget.value = product
  quoteSubmitted.value = false
  quoteError.value = ''
  quoteDialogOpen.value = true
}
const closeQuoteDialog = () => {
  quoteDialogOpen.value = false
}
const submitQuote = async () => {
  if (!/^1[3-9]\d{9}$/.test(quoteContactPhone.value)) { quoteError.value = '请输入有效的中国大陆手机号'; return }
  quoteSubmitting.value = true
  quoteError.value = ''
  try {
    const isCompanyInquiry = isCompanyPage.value
    await api('/inquiries', {
      method: 'POST',
      body: {
        companyId: quoteTarget.value?.companyId || currentStoreSupplier.value.id,
        ...(isCompanyInquiry && !quoteTarget.value ? {} : { productId: quoteTarget.value?.id || detailProduct.value.id, skuId: quoteTarget.value?.raw?.defaultSku?.skuId || selectedSku.value?.skuId }),
        requirement: quoteMessage.value,
        contactName: quoteContactName.value,
        contactPhone: quoteContactPhone.value,
        ...(isCompanyInquiry && !quoteTarget.value ? {} : { expectedSpec: selectedSpec.value, quantity: String(purchaseQuantity.value), unit: detailProduct.value.raw?.unit }),
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
  const detailed = mechanical?.groups?.flatMap((group) => group.items.map((item) => item.name)) || []
  return [...detailed.slice(0, 5), ...categoryNames.value.slice(1, 4)]
})

const recommendTabs = computed(() => categoryNames.value.slice(0, 6))
const visibleSupplierShowcases = computed(() => {
  const matches = homeSupplierShowcases.value.filter((supplier) => supplier.category === activeSupplierCategory.value)
  const defaultSuppliers = homeSupplierShowcases.value.filter((supplier) => supplier.category === supplierShowcaseCategories.value[0])
  const supplements = defaultSuppliers.filter((supplier) => !matches.some((match) => match.name === supplier.name))
  return [...matches, ...supplements]
})

const premiumCatalogProducts = computed(() => homeProducts.value.slice(0, 6))
const factoryCatalogProducts = computed(() => homeProducts.value.slice(2, 8))
const recommendedProducts = computed(() => Array.from({ length: Math.min(12, homeProducts.value.length) }, (_, index) => homeProducts.value[(recommendationOffset.value + index) % homeProducts.value.length]).map((product) => ({
  ...product,
  serviceTags: product.raw?.serviceGuarantees || ['platform_verified'],
  video: Boolean(product.raw?.videoUrls?.length)
})))
const marketSections = computed(() => [
  {
    title: '机械设备',
    category: '机械设备',
    desc: '工控设备、搬运起重、包装设备',
    image: '/images/market-equipment-banner.png',
    bannerClass: 'equipment-bg',
    tags: [
      { label: '水处理设备', target: '污水处理设备' },
      { label: '通用设备', target: '通用设备' },
      { label: '过滤设备', target: '过滤设备' },
      { label: '食品加工机械', target: '食品餐饮设备' },
    ],
    products: homeProducts.value.slice(0, 3)
  },
  {
    title: '建材家居',
    category: '建材家居',
    desc: '基建材料、功能材料、灯饰照明',
    image: '/images/market-building-banner.png',
    bannerClass: 'full-bg',
    tags: [
      { label: '基建材料', target: '基建材料' },
      { label: '功能材料', target: '功能材料' },
      { label: '电工电料', target: '电工电料' },
      { label: '灯饰照明', target: '灯饰照明' },
    ],
    products: homeProducts.value.slice(3, 6)
  },
  {
    title: '化工能源',
    category: '化工能源',
    desc: '涂料油漆、水处理化学品、工程塑料',
    image: '/images/market-chemical-banner.png',
    bannerClass: 'full-bg',
    tags: [
      { label: '水处理化学品', target: '水处理化学品' },
      { label: '涂料油漆', target: '涂料/油漆' },
      { label: '工程塑料', target: '工程塑料' },
      { label: '橡塑制品', target: '橡塑制品' },
    ],
    products: homeProducts.value.slice(6, 9)
  },
  {
    title: '电子仪表',
    category: '电子仪表',
    desc: '集成电路、专业仪表、工控终端',
    image: '/images/market-instrument-banner.png',
    bannerClass: 'full-bg',
    tags: [
      { label: '专业仪表', target: '专业仪器仪表' },
      { label: '检测仪器', target: '检测仪器' },
      { label: '集成电路', target: '集成电路' },
      { label: '工控终端', target: '工控终端' },
    ],
    products: homeProducts.value.slice(1, 4)
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
        <a v-for="item in navItems" :key="item" :class="{ active: isNavActive(item) }" :href="navHrefs[item]">
          {{ item }}
        </a>
      </nav>

      <div class="header-actions">
        <a class="cart-btn" href="#/purchase-list"><BaseIcon name="cart" />我的采购<span>{{ purchaseList.length }}</span></a>
        <button class="login-btn" @click="apiError = '登录功能暂未开放'">登录</button>
        <button class="register-btn" @click="apiError = '注册功能暂未开放'">免费注册</button>
      </div>
    </div>
  </header>

  <div v-if="purchaseListFeedback" class="purchase-toast" role="status">
    <BaseIcon name="shield" />{{ purchaseListFeedback }}
    <a href="#/purchase-list">查看清单</a>
  </div>

  <section v-if="isProductDetailPage || isCompanyPage" class="store-banner">
    <div class="store-banner-inner">
      <div class="store-identity">
        <div class="store-emblem" :class="[{ 'has-logo': Boolean(currentStoreSupplier.logo) }, `supplier-mark-${currentStoreSupplier.tone}`]">
          <img v-if="currentStoreSupplier.logo" :src="currentStoreSupplier.logo" :alt="`${currentStoreSupplier.name} Logo`" />
          <span v-else>{{ currentStoreSupplier.name.slice(0, 1) }}</span>
        </div>
          <div class="store-profile">
            <div class="store-name-line">
            <a :href="companyDetailHref(currentStoreSupplier)">{{ currentStoreSupplier.name }}</a>
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

      <div class="company-profile-actions store-banner-actions">
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
      </nav>
    </div>
  </section>

  <main v-if="!isCatalogPage && !isProductDetailPage && !isCompanyPage && !isNewsPage && !isStaticPage && !isPurchaseListPage && !isProcurementPage && !isMerchantJoinPage" class="page">
    <section class="hero">
      <div class="hero-bg"></div>
      <div class="hero-inner">
        <div class="hero-copy">
          <h1>
            连接优质供需<br />
            让企业采购<span>更简单</span>
          </h1>
          <p>海量工业资源 · 真实可靠的供应商 · 专业高效的采购服务</p>

          <form class="search-box" @submit.prevent="performSearch">
            <div class="search-box-surface">
              <el-dropdown class="search-type-wrap" trigger="click" placement="bottom-start" :teleported="true" popper-class="search-type-dropdown" @command="selectSearchType">
                <button type="button" class="search-type-trigger" aria-haspopup="listbox">
                  <span>{{ searchTypeLabel }}</span><span class="search-type-chevron" aria-hidden="true"></span>
                </button>
                <template #dropdown>
                  <el-dropdown-menu aria-label="搜索类型">
                    <el-dropdown-item command="product" :class="{ active: searchType === 'product' }">商品搜索</el-dropdown-item>
                    <el-dropdown-item command="company" :class="{ active: searchType === 'company' }">店铺搜索</el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
              <input v-model.trim="searchKeyword" placeholder="请输入产品名称、品牌、型号或供应商" />
              <button type="submit" class="search-submit"><BaseIcon name="search" />搜索</button>
            </div>
          </form>

          <div class="hot-line">
            <strong>热门搜索：</strong>
            <a
              v-for="item in hotCategoryLinks"
              :key="item"
              :href="categoryHref(categories.flatMap((category) => category.groups).flatMap((group) => group.items).find((child) => child.name === item)?.id || '')"
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
              :key="item.id"
              class="category-entry"
              @mouseenter="activeCategory = item"
            >
              <a :href="categoryHref(item.id)">
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
              <a :href="categoryHref(activeCategory.id)">进入专区<BaseIcon name="arrowRight" /></a>
            </div>
            <div class="flyout-grid">
              <div v-for="group in activeCategory.groups" :key="group.title" class="flyout-group">
                <h4>{{ group.title }}</h4>
                <div>
                  <a
                    v-for="child in group.items"
                    :key="child.id"
                    :href="categoryHref(child.id)"
                  >
                    {{ child.name }}
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
            <article class="manufacturing-card" aria-roledescription="carousel" aria-label="工业采购精选" @mouseenter="pauseHomeCarousel" @mouseleave="startHomeCarousel" @focusin="pauseHomeCarousel" @focusout="startHomeCarousel">
              <div
                v-for="(slide, index) in homeSlides"
                :key="slide.title"
                class="home-slide"
                :class="{ active: index === activeHomeSlide }"
                :style="{ backgroundImage: `url(${slide.image})` }"
                :aria-hidden="index !== activeHomeSlide"
              ></div>
              <div class="promo-text">
                <small>{{ homeSlides[activeHomeSlide].eyebrow }}</small>
                <h2>{{ homeSlides[activeHomeSlide].title }}</h2>
                <p>{{ homeSlides[activeHomeSlide].desc }}</p>
              </div>
              <button class="promo-more" @click="openCatalog(homeSlides[activeHomeSlide].category)">了解更多<BaseIcon name="arrowRight" /></button>
              <button class="slide-next slide-previous" aria-label="查看上一张精选" @click="showPreviousHomeSlide(); startHomeCarousel()"><BaseIcon name="arrowRight" /></button>
              <button class="slide-next" aria-label="查看下一张精选" @click="showNextHomeSlide(); startHomeCarousel()"><BaseIcon name="arrowRight" /></button>
              <div class="dots" role="tablist" aria-label="轮播图切换">
                <button v-for="(slide, index) in homeSlides" :key="slide.title" type="button" :class="{ active: index === activeHomeSlide }" :aria-label="`切换至${slide.eyebrow}`" :aria-selected="index === activeHomeSlide" role="tab" @click="selectHomeSlide(index)"></button>
              </div>
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
                v-for="category in categories.slice(0, 6)"
                :key="category.id"
                :class="{ active: category.id === activeFeaturedCategoryId }"
                href="#"
                @click.prevent="selectFeaturedCategory(category.id)"
              >
                {{ category.name }}
              </a>
            </div>
            <a :href="featuredCatalogHref">查看更多<BaseIcon name="arrowRight" /></a>
          </div>

          <div class="product-row">
            <template v-if="featuredProducts.length">
              <button class="round-arrow left" aria-label="上一组" :disabled="featuredProducts.length <= featuredVisibleCount || featuredCarouselAnimating" @click="showPreviousFeaturedProduct"><BaseIcon name="arrowRight" /></button>
              <div class="product-carousel-viewport">
                <div class="product-card-track" :class="{ moving: featuredCarouselAnimating }" :style="featuredTrackStyle" @transitionend="finishFeaturedProductSlide">
                  <article v-for="(product, index) in featuredCarouselProducts" :key="`${product.id}-${index}`" class="product-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
                    <div class="product-image" :class="`scene-${product.scene}`">
                      <img :src="product.image" :alt="product.name" @error="$event.target.style.display = 'none'" />
                      <BaseIcon :name="product.icon" />
                    </div>
                    <strong>{{ product.name }}</strong>
                    <div class="product-spec" :class="{ single: productTagParts(product.tag).length < 2 }">
                      <span v-for="part in productTagParts(product.tag)" :key="part">{{ part }}</span>
                    </div>
                    <div class="product-foot">
                      <span class="price">{{ product.price }}</span>
                      <button type="button" @click.stop="openQuoteFor(product)">立即询价</button>
                    </div>
                  </article>
                </div>
              </div>
              <button class="round-arrow right" aria-label="下一组" :disabled="featuredProducts.length <= featuredVisibleCount || featuredCarouselAnimating" @click="showNextFeaturedProduct"><BaseIcon name="arrowRight" /></button>
            </template>
            <div v-else class="featured-empty-state" :class="{ loading: featuredProductsLoading }" :aria-live="featuredProductsLoading ? 'polite' : 'off'">
              <span class="featured-empty-icon"><BaseIcon :name="featuredProductsLoading ? 'search' : 'box'" /></span>
              <div>
                <strong>{{ featuredProductsLoading ? '正在加载精选商品' : '该分类暂未上架精选商品' }}</strong>
                <p>{{ featuredProductsLoading ? '请稍候，正在为您整理商品。' : '您可以切换其他分类，或前往实力优品浏览更多货源。' }}</p>
              </div>
              <a v-if="!featuredProductsLoading" :href="featuredCatalogHref">查看实力优品<BaseIcon name="arrowRight" /></a>
            </div>
          </div>
        </section>

        <aside class="right-column">
          <button class="demand-card" type="button" @click="openProcurement">
            <strong>发布采购需求</strong>
            <span>让优质供应商主动联系您</span>
            <b><BaseIcon name="document" /></b>
          </button>

          <section class="stats-card">
            <div class="panel-title"><h3>平台数据</h3><a href="#/catalog">查看商品<BaseIcon name="arrowRight" /></a></div>
            <div class="stats-grid">
              <div class="stat-item"><i><BaseIcon name="box" /></i><div><strong>200+</strong><span>优质商品</span></div></div>
              <div class="stat-item"><i><BaseIcon name="shield" /></i><div><strong>100+</strong><span>认证供应商</span></div></div>
              <div class="stat-item"><i><BaseIcon name="grid" /></i><div><strong>200+</strong><span>行业解决方案</span></div></div>
              <div class="stat-item"><i><BaseIcon name="headset" /></i><div><strong>7x24小时</strong><span>专业客服支持</span></div></div>
            </div>
          </section>

          <section class="news-card">
            <div class="panel-title"><h3>采购资讯</h3><a href="#/news">查看更多<BaseIcon name="arrowRight" /></a></div>
            <a v-for="item in homeLatestNews.slice(0, 3)" :key="item.id" class="news-item" :href="`#/news?id=${item.id}`">
              <img class="news-thumb" :src="homepageNewsCover(item)" :alt="item.title" />
              <div>
                <strong>{{ item.title }}</strong>
                <span>{{ item.summary }}</span>
              </div>
              <time>{{ item.publishedAt?.slice(5, 10) }}</time>
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
                <h3><a :href="companyDetailHref(supplier)">{{ supplier.name }}</a></h3>
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
            <div><strong>{{ product.price }}</strong><button type="button" @click.stop="openQuoteFor(product)">立即询价</button></div>
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
          <button type="button" @click="openProcurement">发布采购需求<BaseIcon name="arrowRight" /></button>
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
                <h3><a :href="marketSectionHref(section)">{{ section.title }}<BaseIcon name="arrowRight" /></a></h3>
                <p>{{ section.desc }}</p>
                <div class="market-tags">
                  <a v-for="tag in section.tags" :key="`${section.title}-${tag.label}`" :href="marketTagHref(section, tag)">
                    {{ tag.label }}
                  </a>
                </div>
              </div>
              <img v-if="section.products[0]" :src="section.products[0].image" :alt="section.title" />
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
          <button type="button" class="recommend-refresh" @click="cycleRecommendations">换一批<BaseIcon name="refresh" /></button>
        </div>
        <div class="recommend-grid">
          <article v-for="product in recommendedProducts" :key="`home-rec-${product.name}-${product.price}`" class="recommend-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
            <a class="recommend-image" :href="productDetailHref(product)" @click.stop>
              <img :src="product.image" :alt="product.name" />
              <span v-if="product.video" class="play-mark"></span>
            </a>
            <h3>{{ product.name }}</h3>
            <div class="recommend-tags">
              <span v-for="tag in product.serviceTags" :key="`${product.name}-${tag}`">{{ displayCode(tag) }}</span>
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

  <main v-else-if="isNewsPage" class="news-page">
    <section v-if="!selectedNews" class="news-page-hero">
      <div class="news-page-hero-inner">
        <div>
          <span class="news-page-kicker"><BaseIcon name="document" />采购资讯</span>
          <h1>让每次采购都有据可依</h1>
          <p>面向制造企业与采购团队，沉淀供应商管理、询价协同、交付验收等实操经验。</p>
        </div>
        <div class="news-page-hero-note"><b>{{ latestNews.length }}</b><span>篇近期采购参考</span></div>
      </div>
    </section>
    <section class="news-wrap">
      <div class="catalog-crumb"><a href="#/">首页</a><span>/</span><a href="#/news">采购资讯</a><span v-if="selectedNews">/</span><strong>{{ selectedNews?.title || '采购资讯' }}</strong></div>
      <article v-if="selectedNews" class="news-article">
        <header class="news-article-head">
          <p>采购资讯</p>
          <h1>{{ selectedNews.title }}</h1>
          <div><time>{{ formatNewsDate(selectedNews.publishedAt) }}</time><span>{{ selectedNews.viewCount || 0 }} 阅读</span></div>
        </header>
        <img v-if="selectedNews.coverImage" class="news-article-cover" :src="selectedNews.coverImage" :alt="selectedNews.title" />
        <div class="news-article-body" v-html="newsArticleHtml"></div>
        <footer><a href="#/news"><BaseIcon name="arrowRight" />返回资讯列表</a></footer>
      </article>
      <template v-else-if="latestNews.length">
        <article class="news-lead">
          <img :src="latestNews[0].coverImage || '/images/industry-solution.png'" :alt="latestNews[0].title" />
          <div>
            <span>最新发布</span>
            <h2>{{ latestNews[0].title }}</h2>
            <p>{{ latestNews[0].summary }}</p>
            <footer><time>{{ formatNewsDate(latestNews[0].publishedAt) }}</time><a :href="`#/news?id=${latestNews[0].id}`">阅读文章<BaseIcon name="arrowRight" /></a></footer>
          </div>
        </article>
        <section class="news-list-section">
          <header><div><span>全部资讯</span><h2>采购知识与行业观察</h2></div><p>按发布时间排序</p></header>
          <div class="news-list">
            <a v-for="(item, index) in latestNews.slice(1)" :key="item.id" class="news-list-item" :href="`#/news?id=${item.id}`">
              <time>{{ formatNewsDate(item.publishedAt) }}</time>
              <img :src="item.coverImage || '/images/industry-solution.png'" :alt="item.title" />
              <div><span>采购参考 {{ String(index + 2).padStart(2, '0') }}</span><h3>{{ item.title }}</h3><p>{{ item.summary }}</p></div>
              <BaseIcon name="arrowRight" />
            </a>
          </div>
        </section>
      </template>
      <div v-else class="empty-state"><span class="empty-state-icon"><BaseIcon name="document" /></span><h2>暂无采购资讯</h2><p>新的行业动态和采购指南将在这里发布。</p><a href="#/">返回首页</a></div>
    </section>
  </main>
  <main v-else-if="isPurchaseListPage" class="catalog-page purchase-page">
    <section class="purchase-hero">
      <div class="purchase-shell">
        <div><span><BaseIcon name="document" />采购工作台</span><h1>把要买的，放在一起处理</h1><p>勾选商品后生成一条采购需求，平台会协助匹配合适的供应商。</p></div>
        <div class="purchase-hero-count"><strong>{{ purchaseList.length }}</strong><span>种待采购商品</span></div>
      </div>
    </section>
    <section class="purchase-shell purchase-main">
      <div class="catalog-crumb"><a href="#/">首页</a><span>/</span><strong>我的采购清单</strong></div>
      <div v-if="!purchaseList.length" class="empty-state"><span class="empty-state-icon"><BaseIcon name="cart" /></span><h2>采购清单还是空的</h2><p>将需要的商品加入清单，方便统一询价和采购。</p><a href="#/catalog">去挑选商品</a></div>
      <template v-else>
        <div class="purchase-board">
          <div class="purchase-board-head">
            <label class="purchase-check"><input type="checkbox" :checked="allPurchaseItemsSelected" @change="toggleAllPurchaseItems" /><span>全选</span></label>
            <p>已选 <b>{{ selectedPurchaseItems.length }}</b> 种商品，共 <b>{{ purchaseItemCount }}</b> 件</p>
            <button type="button" @click="clearSelectedPurchaseItems" :disabled="!selectedPurchaseItems.length">删除所选</button>
          </div>
          <article v-for="(item, index) in purchaseList" :key="purchaseItemKey(item)" class="purchase-row" :class="{ selected: selectedPurchaseKeys.includes(purchaseItemKey(item)) }">
            <label class="purchase-check"><input v-model="selectedPurchaseKeys" type="checkbox" :value="purchaseItemKey(item)" :aria-label="`选择 ${item.name}`" /><span></span></label>
            <a :href="`#/product?id=${encodeURIComponent(item.productId)}`" class="purchase-image"><img :src="item.image" :alt="item.name" /></a>
            <div class="purchase-product"><a :href="`#/product?id=${encodeURIComponent(item.productId)}`">{{ item.name }}</a><p>{{ item.spec || '标准配置' }}</p><span>{{ item.company }}</span></div>
            <div class="purchase-price"><span>参考价格</span><strong>{{ item.price }}</strong></div>
            <div class="purchase-quantity"><span>采购数量</span><div><button type="button" aria-label="减少数量" @click="updatePurchaseQuantity(index, -1)">−</button><input v-model.number="item.quantity" type="number" min="1" aria-label="采购数量" @change="normalizePurchaseQuantity(index)" /><button type="button" aria-label="增加数量" @click="updatePurchaseQuantity(index, 1)">＋</button></div></div>
            <button type="button" class="purchase-remove" @click="removePurchaseItem(index)">移除</button>
          </article>
        </div>
        <aside class="purchase-summary">
          <div><span>本次采购</span><strong>{{ selectedPurchaseItems.length }} 种商品</strong><p>合计 {{ purchaseItemCount }} 件，提交后由采购顾问协助跟进。</p></div>
          <button type="button" @click="createDemandFromList">生成采购需求<BaseIcon name="arrowRight" /></button>
        </aside>
      </template>
    </section>
  </main>
  <main v-else-if="isProcurementPage" class="catalog-page procurement-page">
    <section class="procurement-hero"><div class="purchase-shell"><span><BaseIcon name="headset" />人工协同采购</span><h1>说清需求，余下交给我们</h1><p>提交后，平台将根据类目和需求说明协助您匹配供应商。</p></div></section>
    <section class="purchase-shell procurement-main">
      <div class="catalog-crumb"><a href="#/">首页</a><span>/</span><strong>发布采购需求</strong></div>
      <div class="procurement-layout">
        <form class="procurement-form" @submit.prevent="submitProcurement">
          <header><span>填写采购信息</span><h2>让供应商快速读懂您的需求</h2><p v-if="routeHash.includes('from=list')">已从采购清单带入商品信息，请补充采购类目与联系人。</p></header>
          <div class="procurement-fields">
            <div class="procurement-category-field">
              <span>采购类目</span>
              <div class="procurement-category-select" @click.stop>
                <button type="button" class="procurement-category-trigger" :class="{ placeholder: !selectedProcurementCategory }" :aria-expanded="procurementCategoryOpen" aria-haspopup="listbox" @click="procurementCategoryOpen = !procurementCategoryOpen" @keydown.esc="procurementCategoryOpen = false">
                  <span>{{ selectedProcurementCategory?.name || '请选择采购类目' }}</span><BaseIcon name="arrowRight" />
                </button>
                <div v-if="procurementCategoryOpen" class="procurement-category-menu" role="listbox" aria-label="采购类目">
                  <button v-for="category in categories" :key="category.id" type="button" role="option" :aria-selected="procurementForm.categoryId === category.id" :class="{ active: procurementForm.categoryId === category.id }" @click="selectProcurementCategory(category.id)">{{ category.name }}<span v-if="procurementForm.categoryId === category.id">已选</span></button>
                </div>
              </div>
            </div>
            <label>需求说明<textarea v-model.trim="procurementForm.description" required maxlength="1000" placeholder="请说明采购产品、规格、质量要求、期望交期等"></textarea><small>{{ procurementForm.description.length }}/1000</small></label>
            <div class="quote-form-row"><label>采购数量<input v-model="procurementForm.quantity" type="number" min="0.001" step="0.001" required /></label><label>单位<input v-model.trim="procurementForm.unit" required placeholder="如：件、台、批" /></label></div>
            <div class="quote-form-row"><label>联系人<input v-model.trim="procurementForm.contactName" required maxlength="80" placeholder="请输入您的称呼" /></label><label>联系电话<input v-model.trim="procurementForm.contactPhone" required type="tel" maxlength="32" placeholder="便于采购顾问联系您" /></label></div>
          </div>
          <p v-if="procurementMessage" class="procurement-message" :class="{ success: procurementMessage.startsWith('采购需求已提交') }">{{ procurementMessage }}</p>
          <footer><a href="#/purchase-list">返回采购清单</a><button type="submit" :disabled="procurementSubmitting">{{ procurementSubmitting ? '正在提交…' : '提交采购需求' }}<BaseIcon name="arrowRight" /></button></footer>
        </form>
        <aside class="procurement-aside"><div class="procurement-aside-mark"><BaseIcon name="shield" /></div><h2>提交后会发生什么？</h2><ol><li><b>1</b><div><strong>平台确认需求</strong><span>核对类目、规格与数量</span></div></li><li><b>2</b><div><strong>匹配合适供应商</strong><span>优先筛选认证及产能稳定的商家</span></div></li><li><b>3</b><div><strong>采购顾问联系您</strong><span>协助询价和后续采购协同</span></div></li></ol><p>请留下可联系的手机号，便于我们快速跟进。</p></aside>
      </div>
    </section>
  </main>
  <main v-else-if="isMerchantJoinPage" class="merchant-join-page" aria-label="商家入驻"></main>
  <main v-else-if="isStaticPage" class="catalog-page"><section class="catalog-wrap"><div class="catalog-crumb"><a href="#/">首页</a><span>/</span><strong>{{ new URLSearchParams(routeHash.split('?')[1] || '').get('title') || '平台说明' }}</strong></div><article class="detail-info-card"><h1>{{ new URLSearchParams(routeHash.split('?')[1] || '').get('title') || '平台说明' }}</h1><p>本页面为擎采云平台公开说明。平台当前提供商品浏览、店铺查询、询价、采购线索提交与本地采购清单功能；暂不提供在线支付、物流追踪、会员账户同步或自动报价服务。</p></article></section></main>
  <main v-else-if="isCatalogPage" class="catalog-page">
    <section class="catalog-hero">
      <div class="catalog-inner">
        <div class="catalog-title">
          <span>实力优品</span>
          <h1>{{ currentCategory }}采购专区</h1>
          <p>聚合源头工厂、认证供应商与可定制工业设备，支持快速询价和批量采购。</p>
        </div>
        <form class="catalog-search" @submit.prevent="performSearch">
          <el-dropdown class="catalog-search-type" trigger="click" placement="bottom-start" :teleported="true" popper-class="search-type-dropdown catalog-search-type-dropdown" @command="selectSearchType">
            <button type="button" aria-haspopup="listbox">{{ catalogSearchTypeLabel }}<BaseIcon name="arrowRight" /></button>
            <template #dropdown>
              <el-dropdown-menu aria-label="搜索类型">
                <el-dropdown-item command="product" :class="{ active: searchType === 'product' }">搜索货源</el-dropdown-item>
                <el-dropdown-item command="company" :class="{ active: searchType === 'company' }">搜索供应商</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          <input v-model.trim="searchKeyword" :placeholder="searchType === 'company' ? '搜索供应商名称、主营类目' : `搜索${currentCategory}产品、型号、供应商`" />
          <button type="submit" class="catalog-search-submit" :disabled="catalogLoading"><BaseIcon name="search" /> {{ catalogLoading ? '搜索中' : '搜索' }}</button>
        </form>
      </div>
    </section>

    <section class="catalog-wrap">
      <div class="catalog-crumb">
        <a href="#/">首页</a>
        <span>/</span>
        <strong>{{ currentCategory }}</strong>
        <em>部分商品支持来图定制，具体以供应商报价为准</em>
      </div>

      <div v-if="searchType === 'product'" class="catalog-filter-panel">
        <div v-for="filter in catalogFilters" :key="filter.label" class="filter-row">
          <strong>{{ filter.label }}</strong>
          <button v-for="value in filter.values" :key="value.value" type="button" :class="{ active: activeCatalogFilters[filter.label] === value.value }" @click="activeCatalogFilters[filter.label] = value.value; applyCatalogFilter(filter.label === '供应方式' ? 'supplyMethod' : 'verification', value.value)">{{ value.label }}</button>
        </div>
      </div>

      <div v-if="searchType === 'product'" class="catalog-toolbar">
        <div class="sorts">
          <button v-for="sort in [{label:'综合排序',value:'comprehensive'},{label:'人气排序',value:'popularity'},{label:'价格',value:'price_asc'},{label:'交付速度',value:'delivery_speed'}]" :key="sort.value" type="button" :class="{ active: activeCatalogSort === sort.label }" @click="activeCatalogSort = sort.label; applyCatalogFilter('sort', sort.value)">{{ sort.label }}</button>
        </div>
        <div class="chips">
          <span>所有地区</span>
          <span>实力供应商</span>
          <span>已验厂企业</span>
          <span>安心购</span>
        </div>
      </div>

      <div v-if="catalogLoading" class="empty-state empty-state-loading" role="status">
        <span class="empty-state-icon"><BaseIcon name="search" /></span>
        <h2>正在查找商品</h2>
        <p>请稍候，正在整理匹配结果。</p>
      </div>
      <div v-else-if="searchType === 'company' && !supplierShowcases.length" class="empty-state">
        <span class="empty-state-icon"><BaseIcon name="factory" /></span>
        <h2>没有找到匹配供应商</h2>
        <p>试试更换供应商名称、主营类目或关键词。</p>
      </div>
      <div v-else-if="searchType === 'company'" class="catalog-company-results">
        <header><p>共找到 <b>{{ catalogTotal }}</b> 家供应商<span v-if="searchKeyword">，与“{{ searchKeyword }}”相关</span></p></header>
        <div class="catalog-company-grid">
          <article v-for="supplier in supplierShowcases" :key="supplier.id" class="catalog-company-card" role="link" tabindex="0" @click="openCompany(supplier)" @keydown.enter="openCompany(supplier)">
            <div class="catalog-company-mark" :class="`supplier-mark-${supplier.tone}`"><img v-if="supplier.logo" :src="supplier.logo" :alt="`${supplier.name} Logo`" /><span v-else>{{ supplier.mark }}</span></div>
            <div><h2>{{ supplier.name }}</h2><p>{{ supplier.category || '工业采购供应商' }}</p><span>{{ supplier.location || '中国' }}</span></div>
            <BaseIcon name="arrowRight" />
          </article>
        </div>
      </div>
      <div v-else-if="!sortedCatalogProducts.length" class="empty-state">
        <span class="empty-state-icon"><BaseIcon name="search" /></span>
        <h2>没有找到匹配商品</h2>
        <p>试试更换关键词、类目或筛选条件。</p>
      </div>
      <div v-else class="catalog-grid">
        <article v-for="product in sortedCatalogProducts" :key="`${product.name}-${product.price}`" class="catalog-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
          <div class="catalog-card-image">
            <img :src="product.image" :alt="product.name" />
          </div>
          <h3>{{ product.name }}</h3>
          <div class="catalog-tags">
            <span>{{ product.badge }}</span>
          </div>
          <div class="catalog-spec" :class="{ single: productTagParts(product.tag).length < 2 }">
            <span v-for="part in productTagParts(product.tag)" :key="part">{{ part }}</span>
          </div>
          <div class="catalog-card-foot">
            <strong>{{ product.price }}</strong>
            <button type="button" @click.stop="openQuoteFor(product)">立即询价</button>
          </div>
          <p>{{ product.raw?.serviceGuarantees?.includes('factory_audited') ? '已验厂企业' : '平台认证供应商' }} · 支持批量报价</p>
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
            <div><dt>采购数量</dt><dd class="detail-quantity"><button type="button" @click="decreaseQuantity">−</button><input v-model.number="purchaseQuantity" :min="selectedSku?.minOrderQuantity || detailProduct.raw?.minOrderQuantity || 1" type="number" aria-label="采购数量" /><button type="button" @click="increaseQuantity">＋</button><span>{{ selectedSku?.minOrderQuantity || detailProduct.raw?.minOrderQuantity || 1 }} {{ detailProduct.raw?.unit || '件' }}起订</span></dd></div>
          </dl>
          <div class="detail-actions">
            <button type="button" class="detail-inquiry" @click="openQuoteDialog"><BaseIcon name="headset" />立即询价</button>
            <button type="button" class="detail-list" @click="addToPurchaseList"><BaseIcon name="document" />加入采购清单</button>
          </div>
        </section>

        </section>

      <section class="detail-content-grid">
        <div class="detail-description">
          <nav class="detail-anchor-nav" aria-label="商品详情导航">
            <button type="button" :class="{ active: activeDetailSection === 'product-details' }" @click="scrollToDetailSection('product-details')">商品详情</button>
            <button type="button" :class="{ active: activeDetailSection === 'product-parameters' }" @click="scrollToDetailSection('product-parameters')">产品参数</button>
            <button type="button" :class="{ active: activeDetailSection === 'purchase-notes' }" @click="scrollToDetailSection('purchase-notes')">采购说明</button>
          </nav>
          <article id="product-details" class="detail-info-card detail-overview-card">
            <h2>商品详情</h2>
            <div class="detail-rich-content">
              <div v-if="detailDescriptionHtml" v-html="detailDescriptionHtml"></div>
              <p v-else>{{ detailOverview }}</p>
            </div>
            <div class="detail-overview-grid">
              <div><span>商品品牌</span><strong>{{ detailProduct.raw?.brand || detailSupplier.shortName || '—' }}</strong></div>
              <div><span>产地</span><strong>{{ detailProduct.location || '—' }}</strong></div>
              <div><span>起订量</span><strong>{{ detailProduct.raw?.minOrderQuantity || 1 }} {{ detailProduct.raw?.unit || '件' }}</strong></div>
              <div><span>定制服务</span><strong>{{ detailProduct.raw?.customizationEnabled ? '支持按需定制' : '请联系供应商确认' }}</strong></div>
            </div>
            <ul class="detail-selling-points"><li v-for="point in detailSellingPoints" :key="point">{{ point }}</li></ul>
          </article>
          <article id="product-parameters" class="detail-info-card">
            <h2>产品参数</h2>
            <div class="detail-parameter-grid">
              <div><span>产品名称</span><strong>{{ detailProduct.name }}</strong></div>
              <div><span>商品型号</span><strong>{{ detailProduct.raw?.model || '—' }}</strong></div>
              <div><span>供货方式</span><strong>{{ detailSupplyMethods.join('、') }}</strong></div>
              <div><span>适用场景</span><strong>{{ detailScenarios.join('、') }}</strong></div>
              <div><span>质量服务</span><strong>{{ detailProduct.raw?.afterSalesService || '—' }}</strong></div>
              <div><span>交付周期</span><strong>{{ detailProduct.raw?.shipWithinHours ? `${detailProduct.raw.shipWithinHours} 小时内发货` : '以实际沟通为准' }}</strong></div>
              <div v-for="parameter in detailTechnicalParameters" :key="parameter.parameterId || parameter.name"><span>{{ parameter.name }}</span><strong>{{ formatTechnicalParameter(parameter) }}</strong></div>
            </div>
          </article>
          <article id="purchase-notes" class="detail-info-card detail-copy-card">
            <h2>采购说明</h2>
            <div class="detail-rich-content">
              <p>下单前请与供应商确认所选型号、规格、数量、交期及收货地址；页面报价仅作采购参考，最终以双方确认的报价单为准。</p>
              <ul>
                <li>标准配置以当前页面展示为准，特殊工况或非标需求可通过“留言问价”说明。</li>
                <li>批量采购、定制服务及运费请提前沟通，供应商将根据实际需求提供交付方案。</li>
                <li>收货后请及时核验商品外观、型号与数量，如有问题请保留凭证并联系供应商处理。</li>
              </ul>
            </div>
            <div class="detail-feature-list"><span v-for="tag in detailFeatureTags" :key="tag"><BaseIcon name="shield" />{{ tag }}</span></div>
          </article>
        </div>
      </section>
        </div>

        <div class="detail-right-rail">
          <aside class="detail-supplier">
            <div class="detail-supplier-head"><span>认证供应商</span><BaseIcon name="shield" /></div>
            <div class="detail-supplier-name"><i :class="[{ 'has-logo': Boolean(detailSupplier.logo) }, `supplier-mark-${detailSupplier.tone}`]"><img v-if="detailSupplier.logo" :src="detailSupplier.logo" :alt="`${detailSupplier.name} Logo`" /><span v-else>{{ detailSupplier.name.slice(0, 1) }}</span></i><div><a :href="companyDetailHref(detailSupplier)">{{ detailSupplier.name }}</a><span>{{ detailSupplier.shortName }} · 已验厂企业</span></div></div>
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
          <a href="#/procurement">发布采购需求</a>
          <a href="#/static?title=采购流程">采购流程</a>
          <a href="#/static?title=采购指南">采购指南</a>
          <a href="#/static?title=常见问题">常见问题</a>
        </div>
        <div>
          <h3>卖家服务</h3>
          <a href="#/static?title=商家入驻">商家入驻</a>
          <a href="#/static?title=入驻流程">入驻流程</a>
          <a href="#/static?title=商家服务">商家服务</a>
          <a href="#/static?title=商家中心">商家中心（暂未开放）</a>
        </div>
        <div>
          <h3>平台支持</h3>
          <a href="#/static?title=平台规则">平台规则</a>
          <a href="#/static?title=服务协议">服务协议</a>
          <a href="#/static?title=隐私政策">隐私政策</a>
          <a href="#/static?title=帮助中心">帮助中心</a>
        </div>
        <div>
          <h3>关于我们</h3>
          <a href="#/static?title=平台介绍">平台介绍</a>
          <a href="#/static?title=联系我们">联系我们</a>
          <a href="#/static?title=商务合作">商务合作</a>
          <a href="#/static?title=意见反馈">意见反馈</a>
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
      <button v-show="showBackToTop" type="button" class="back-top" aria-label="返回页面顶部" @click="scrollToTop">
        <BaseIcon name="arrowRight" />
        <span>返回顶部</span>
      </button>
    </div>
  </footer>
</template>
