<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'
import BaseIcon from './components/BaseIcon.vue'
import {
  catalogFilters,
  catalogProducts,
  categoryPosters,
  categories,
  featureCards,
  navItems,
  news,
  products,
  serviceHighlights,
  supplierShowcaseCategories,
  supplierShowcases,
  stats,
} from './mock/home'

const routeHash = ref(window.location.hash || '#/')
const activeCategory = ref(null)
const activeSupplierCategory = ref(supplierShowcaseCategories[0])
const activeDetailImage = ref(0)
const selectedSpec = ref('标准配置')
const purchaseQuantity = ref(1)

const updateRoute = () => {
  routeHash.value = window.location.hash || '#/'
}

onMounted(() => {
  window.addEventListener('hashchange', updateRoute)
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

const categoryNames = computed(() => categories.map((item) => item.name))

const detailProduct = computed(() => {
  const [, query = ''] = routeHash.value.split('?')
  const productName = new URLSearchParams(query).get('name')
  return catalogProducts.find((product) => product.name === productName) || catalogProducts[0]
})

const detailGallery = computed(() => [
  detailProduct.value.image,
  '/images/smart-manufacturing.png',
  '/images/market-equipment-banner.png',
  '/images/industry-solution.png'
])

const detailRelatedProducts = computed(() => catalogProducts
  .filter((product) => product.name !== detailProduct.value.name)
  .slice(0, 4))

const detailSupplier = computed(() => {
  const matchedSupplier = supplierShowcases.find((supplier) => supplier.products.some((product) => product.name === detailProduct.value.name))
  const productIndex = Math.max(0, catalogProducts.findIndex((product) => product.name === detailProduct.value.name))
  return matchedSupplier || supplierShowcases[productIndex % supplierShowcases.length]
})

const detailSupplierLocation = computed(() => ['江苏苏州', '河北石家庄', '河南郑州', '福建龙岩', '上海'][supplierShowcases.indexOf(detailSupplier.value) % 5])

const productDetailHref = (product) => `#/product?name=${encodeURIComponent(product.name)}`
const openProduct = (product) => {
  window.location.hash = productDetailHref(product)
}
const decreaseQuantity = () => {
  purchaseQuantity.value = Math.max(1, purchaseQuantity.value - 1)
}
const increaseQuantity = () => {
  purchaseQuantity.value += 1
}

const hotCategoryLinks = computed(() => {
  const mechanical = categories.find((item) => item.name === '机械设备')
  const detailed = mechanical?.groups?.flatMap((group) => group.items) || []
  return [...detailed.slice(0, 5), ...categoryNames.value.slice(1, 4)]
})

const recommendTabs = computed(() => categoryNames.value.slice(0, 6))
const visibleSupplierShowcases = computed(() => {
  const matches = supplierShowcases.filter((supplier) => supplier.category === activeSupplierCategory.value)
  const defaultSuppliers = supplierShowcases.filter((supplier) => supplier.category === '机械设备')
  const supplements = defaultSuppliers.filter((supplier) => !matches.some((match) => match.name === supplier.name))
  return [...matches, ...supplements].slice(0, 4)
})

const premiumCatalogProducts = computed(() => catalogProducts.slice(0, 6))
const factoryCatalogProducts = computed(() => catalogProducts.slice(2, 8))
const recommendedProducts = computed(() => catalogProducts.slice(0, 12).map((product, index) => ({
  ...product,
  serviceTags: [
    ['安心购', '在线交易', '48小时发货'],
    ['安心购', '支持定制', '现货速发'],
    ['认证供应商', '在线交易', '少货必赔'],
    ['实力工厂', '批量优惠', '极速报价']
  ][index % 4],
  location: ['湖北武汉', '河北石家庄', '广东广州', '山东济南', '江苏苏州', '上海'][index % 6],
  supplier: ['擎采云认证供应商', '源头实力工厂', '工业设备优选店', '品质保障企业'][index % 4],
  years: ['1年', '3年', '5年', '8年'][index % 4],
  video: index % 3 !== 1
})))
const marketSections = computed(() => [
  {
    title: '机械设备',
    desc: '工控设备、搬运起重、包装设备',
    image: '/images/market-equipment-banner.png',
    bannerClass: 'equipment-bg',
    tags: ['水处理设备', '通用设备', '过滤设备', '食品加工机械'],
    products: catalogProducts.slice(0, 3)
  },
  {
    title: '建材家居',
    desc: '基建材料、功能材料、灯饰照明',
    image: '/images/market-building-banner.png',
    bannerClass: 'full-bg',
    tags: ['基建材料', '功能材料', '电工电料', '灯饰照明'],
    products: catalogProducts.slice(3, 6)
  },
  {
    title: '化工能源',
    desc: '涂料油漆、水处理化学品、工程塑料',
    image: '/images/market-chemical-banner.png',
    bannerClass: 'full-bg',
    tags: ['水处理化学品', '涂料油漆', '工程塑料', '橡塑制品'],
    products: catalogProducts.slice(6, 9)
  },
  {
    title: '电子仪表',
    desc: '集成电路、专业仪表、工控终端',
    image: '/images/market-instrument-banner.png',
    bannerClass: 'full-bg',
    tags: ['专业仪表', '检测仪器', '集成电路', '工控终端'],
    products: catalogProducts.slice(1, 4)
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
        <a v-for="(item, index) in navItems" :key="item" :class="{ active: index === 0 && !isCatalogPage && !isProductDetailPage }" href="#/">
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

  <main v-if="!isCatalogPage && !isProductDetailPage" class="page">
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
            <a href="#">更多 &gt;</a>
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
              <button class="promo-more">了解更多<BaseIcon name="arrowRight" /></button>
              <button class="slide-next" aria-label="下一张"><BaseIcon name="arrowRight" /></button>
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
                  <button>{{ card.action }}<BaseIcon name="arrowRight" /></button>
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
            <a href="#">更多推荐-&gt;</a>
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
                <button type="button" @click.stop>立即询价</button>
              </div>
            </article>
            <button class="round-arrow right" aria-label="下一组"><BaseIcon name="arrowRight" /></button>
          </div>
        </section>

        <aside class="right-column">
          <button class="demand-card">
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
          <a href="#">查看全部<BaseIcon name="arrowRight" /></a>
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
              <div class="supplier-mark" :class="`supplier-mark-${supplier.tone}`">{{ supplier.mark }}</div>
              <div>
                <h3>{{ supplier.name }}</h3>
                <span>{{ supplier.shortName }} · 已认证供应商</span>
              </div>
              <button type="button">进店<BaseIcon name="arrowRight" /></button>
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
          <a href="#">查看更多<BaseIcon name="arrowRight" /></a>
        </div>
        <div class="premium-strip">
          <article v-for="product in premiumCatalogProducts" :key="`premium-${product.name}`" class="premium-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
            <img :src="product.image" :alt="product.name" />
            <h3>{{ product.name }}</h3>
            <div><strong>{{ product.price }}</strong><button type="button" @click.stop>立即询价</button></div>
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
          <button>发布采购需求<BaseIcon name="arrowRight" /></button>
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
          <a href="#">换一批<BaseIcon name="arrowRight" /></a>
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
          <a v-for="value in filter.values" :key="value" href="#">{{ value }}</a>
        </div>
      </div>

      <div class="catalog-toolbar">
        <div class="sorts">
          <button class="active">综合排序</button>
          <button>人气排序</button>
          <button>价格</button>
          <button>交付速度</button>
        </div>
        <div class="chips">
          <span>所有地区</span>
          <span>实力供应商</span>
          <span>已验厂企业</span>
          <span>安心购</span>
        </div>
      </div>

      <div class="catalog-grid">
        <article v-for="product in catalogProducts" :key="`${product.name}-${product.price}`" class="catalog-card" role="link" tabindex="0" @click="openProduct(product)" @keydown.enter="openProduct(product)">
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
            <button type="button" @click.stop>立即询价</button>
          </div>
          <p>擎采云认证供应商 · 支持批量报价</p>
        </article>
      </div>
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
          <div class="detail-tags"><span>实力工厂</span><span>品质保障</span></div>
          <h1>{{ detailProduct.name }}</h1>
          <p class="detail-subtitle">{{ detailProduct.tag }} · 支持工程采购、批量报价与按需定制</p>
          <div class="detail-price-panel">
            <span>参考报价</span>
            <strong>{{ detailProduct.price }}</strong>
            <em>具体价格以询价结果为准</em>
          </div>
          <dl class="detail-facts">
            <div><dt>服务保障</dt><dd><span>平台认证</span><span>支持定制</span><span>快速发货</span></dd></div>
            <div><dt>发货地</dt><dd>江苏 · 苏州　预计 48 小时内发货</dd></div>
            <div><dt>规格选择</dt><dd class="detail-specs"><button v-for="spec in ['标准配置', '工程加强款', '按图定制']" :key="spec" type="button" :class="{ active: selectedSpec === spec }" @click="selectedSpec = spec">{{ spec }}</button></dd></div>
            <div><dt>采购数量</dt><dd class="detail-quantity"><button type="button" @click="decreaseQuantity">−</button><input :value="purchaseQuantity" aria-label="采购数量" readonly /><button type="button" @click="increaseQuantity">＋</button><span>件起订</span></dd></div>
          </dl>
          <div class="detail-actions">
            <button type="button" class="detail-inquiry"><BaseIcon name="headset" />立即询价</button>
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
              <div><span>产品类型</span><strong>工业自动化设备</strong></div>
              <div><span>供货方式</span><strong>现货与定制</strong></div>
              <div><span>适用场景</span><strong>产线、车间、工程项目</strong></div>
              <div><span>质量服务</span><strong>售后技术支持</strong></div>
              <div><span>交付周期</span><strong>以实际沟通为准</strong></div>
            </div>
          </article>
          <article id="purchase-notes" class="detail-info-card detail-copy-card">
            <h2>商品说明</h2>
            <p>该商品适用于工业采购与项目配套场景。支持根据安装空间、接口需求及交付周期进行配置，批量采购可获取专项报价。</p>
            <div class="detail-feature-list"><span><BaseIcon name="shield" />平台认证供应商</span><span><BaseIcon name="document" />提供选型支持</span><span><BaseIcon name="headset" />专属采购对接</span></div>
          </article>
        </div>
      </section>
        </div>

        <div class="detail-right-rail">
          <aside class="detail-supplier">
            <div class="detail-supplier-head"><span>认证供应商</span><BaseIcon name="shield" /></div>
            <div class="detail-supplier-name"><i :class="`supplier-mark-${detailSupplier.tone}`">{{ detailSupplier.mark }}</i><div><strong>{{ detailSupplier.name }}</strong><span>{{ detailSupplier.shortName }} · 已验厂企业</span></div></div>
            <p>主营：{{ detailSupplier.products.map((product) => product.name).join('、') }}</p>
            <div class="detail-supplier-tags"><span>{{ detailSupplier.category }}</span><span>支持定制</span><span>源头供货</span></div>
            <div class="detail-supplier-stats"><span><b>5 年</b>经营年限</span><span><b>98%</b>响应率</span><span><b>{{ detailSupplierLocation }}</b>发货地</span></div>
            <div class="detail-supplier-actions"><button type="button">联系商家</button><button type="button">进入店铺</button></div>
          </aside>

          <aside class="detail-related">
            <div class="detail-side-title"><h2>同店推荐</h2><a href="#/">查看更多</a></div>
            <a v-for="product in detailRelatedProducts" :key="product.name" :href="productDetailHref(product)" class="detail-related-item">
              <img :src="product.image" :alt="product.name" />
              <div><strong>{{ product.name }}</strong><span>{{ product.tag }}</span><b>{{ product.price }}</b></div>
            </a>
          </aside>
        </div>
      </div>
    </section>
  </main>

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
