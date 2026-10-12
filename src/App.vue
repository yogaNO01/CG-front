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
const recommendationPage = ref(1)
const recommendationProducts = ref([])
const recommendationLoading = ref(false)
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
const newsCover = (item) => item?.coverImage || ''
const searchKeyword = ref('')
const searchType = ref('product')
const catalogLoading = ref(false)
const catalogTotal = ref(0)
const defaultCatalogPageSize = 20
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
const activeMerchantGuide = ref('preparation')
const merchantGuides = [
  {
    id: 'preparation', group: '卖家指南', label: '入驻前准备', title: '商家入驻前准备', updatedAt: '2026-09-29',
    intro: '准备充分的企业资料与商品信息，能让后续资料核验和店铺配置更顺畅。以下内容用于帮助您提前梳理信息，不代表审核承诺。',
    sections: [
      { title: '适用商家', paragraphs: ['擎采云主要服务工业品、工程物料、设备配套及企业服务领域的供货商家。生产工厂、品牌方、经销商均可根据实际经营情况准备相关资料。'] },
      { title: '建议提前准备的资料', list: ['企业主体证明与有效的经营信息', '主营产品、型号、应用场景与服务区域说明', '产品实拍图、工厂或仓储环境图等可佐证供货能力的材料', '涉及行业准入时，按产品属性准备相应的许可或授权文件'] },
      { title: '信息填写原则', paragraphs: ['企业名称、产品参数、图片与资质应保持一致、真实且可核验。请避免使用与实际经营无关的品牌、认证或宣传描述。', '如同一主体存在多个经营地点或多个业务品牌，建议先明确本次展示的主体、主营方向和可供范围，避免采购方产生理解偏差。'] },
      { title: '资料有效性与更新', paragraphs: ['请留意营业期限、行业许可、品牌授权等材料的有效期。企业名称、地址、经营范围或关键资质发生变化时，应及时更新说明，避免展示信息与实际情况不一致。'] },
      { title: '暂不适合展示的内容', list: ['无法证明来源或无法稳定供货的产品', '需具备特定经营许可但暂未准备相应材料的产品', '带有夸大承诺、绝对化表述或无法核实的荣誉宣传', '与工业采购无关、仅用于引流或重复发布的信息'] },
      { title: '首次整理资料的小建议', paragraphs: ['可以先从采购方最常问的问题开始整理：产品做什么、有哪些规格、能否定制、最低订量是多少、通常多久交货、由谁提供售后。将这些信息形成基础资料，后续维护商品会更高效。', '涉及多个产品线时，建议按类目分别建立资料清单，不必一次展示全部产品，但每一条已展示的信息应能及时响应和核实。'] },
    ],
  },
  {
    id: 'quality', group: '卖家指南', label: '商品发布注意事项', title: '商品信息发布注意事项', updatedAt: '2026-09-29',
    intro: '商品页面是采购方判断是否进一步沟通的重要依据。清楚、可比较的信息比泛泛的宣传更有效。',
    sections: [
      { title: '商品标题怎么写', paragraphs: ['建议使用“产品名称 + 核心规格/型号 + 关键用途”的方式，让采购方能够快速判断是否匹配。标题不宜堆叠无关关键词或夸张用语。', '例如可围绕材质、压力等级、尺寸、功率、适配设备等真实特征补充标题，而不是使用“全网最低”“万能”“顶级”等无法验证的描述。'] },
      { title: '商品内容应包含', list: ['可选型号、关键参数与计量单位', '适用场景、起订要求、供货周期或定制说明', '清晰且与实物一致的产品图片', '对采购决策有影响的材质、工艺、安装或售后信息'] },
      { title: '图片与参数的对应关系', paragraphs: ['主图应能清楚识别商品主体；细节图可补充接口、铭牌、包装、工艺或使用状态。若展示图为示意图、组合图或非标准配置，应在说明中明确标注。', '规格参数应使用统一单位，并与询价、交付时实际可供的版本保持一致。型号较多时，可先列出常用规格并说明可进一步选型。'] },
      { title: '价格与库存的展示建议', paragraphs: ['当价格会随规格、数量、原材料或地区变化时，可使用“按实际询价为准”的说明，并写清影响价格的关键条件。不要用明显不适用于多数订单的价格吸引点击。', '库存、现货和交期信息应根据真实库存及生产安排维护；无法立即确认时，应明确回复采购方核实周期。'] },
      { title: '服务能力如何写得可信', paragraphs: ['可以具体说明是否支持定制、打样、选型、安装指导、项目配套、检测或售后响应，而不是泛泛写“服务完善”。每一项能力都应有对应人员、流程或资源可以落地。'] },
      { title: '避免出现的问题', paragraphs: ['不要发布无法提供、参数不明或与实际不符的商品；不要在商品信息中填写与交易无关的联系方式、外部导流内容或未经授权的品牌信息。', '同一商品不应通过微调标题、图片或价格反复发布。相近型号可以在同一商品内说明差异，方便采购方比较和询价。'] },
    ],
  },
  {
    id: 'qualification', group: '卖家指南', label: '资质与授权材料', title: '资质与授权材料准备说明', updatedAt: '2026-09-29',
    intro: '不同产品需要的证明材料并不完全相同。将企业主体、产品合规与品牌授权材料按实际经营范围归类，有助于后续核验与展示。',
    sections: [
      { title: '企业主体材料', paragraphs: ['企业主体材料应能反映当前有效的经营信息。若企业名称、统一社会信用代码、地址或经营范围发生变化，请优先以最新材料为准。'] },
      { title: '产品相关材料', paragraphs: ['对于有行业准入、备案、许可、检测或执行标准要求的产品，应根据产品属性准备相应材料。材料应与拟展示产品的名称、型号、适用范围或生产主体具有合理关联。'] },
      { title: '品牌与渠道授权', paragraphs: ['经销、代理或展示品牌商品时，建议提前确认授权主体、授权产品、区域与有效期。授权链路存在多级关系时，应保证必要的上下游关系可以说明。'] },
      { title: '材料使用边界', list: ['只展示仍在有效期内且与当前产品相符的材料', '不要修改、拼接或遮盖影响判断的关键信息', '不要将单一产品或单次检测结果扩大用于全部产品', '涉及商业秘密的材料，可在沟通中按实际需要提供说明'] },
    ],
  },
  {
    id: 'verification', group: '卖家指南', label: '资料核验说明', title: '企业资料核验说明', updatedAt: '2026-09-29',
    intro: '平台会根据商家提交的信息进行基础核对。资料完整且彼此一致，有助于减少补充沟通。',
    sections: [
      { title: '核验关注哪些内容', paragraphs: ['重点关注企业主体是否真实有效、经营范围与主营产品是否合理匹配，以及展示的认证、授权或资质是否仍在有效期内。', '对于具有行业准入要求的产品，平台可能关注相应许可、备案、检测报告或授权链路是否完整。'] },
      { title: '提交材料的清晰度', paragraphs: ['请提交完整、清晰且能够辨识关键信息的材料。图片应避免过度遮挡、裁切或压缩；如材料包含多页，请保证必要页面齐全。'] },
      { title: '资料不一致时怎么办', paragraphs: ['如企业名称、地址、品牌授权或产品类别发生变化，请以当前有效资料为准重新整理；必要时，平台会提示您补充说明。', '若展示主体与生产主体、品牌方不一致，应如实说明各方关系，并按实际情况准备可验证的授权或合作材料。'] },
      { title: '认证展示说明', paragraphs: ['平台认证信息仅基于已核验的材料展示，不代表对商品品质、交付能力或合作结果作出保证。', '认证或资质展示不应被用于超出材料范围的宣传，例如将单项检测、局部授权解释为全部产品或全部区域的保证。'] },
      { title: '补充材料的常见情形', list: ['产品类别与营业范围或已有资料关联不清晰', '展示品牌与企业主体之间缺少有效授权说明', '许可、检测、备案等材料临近失效或页面不完整', '产品图片、型号、标签与提交的资料无法相互对应'] },
    ],
  },
  {
    id: 'operation', group: '卖家指南', label: '店铺信息维护', title: '店铺信息维护与采购沟通建议', updatedAt: '2026-09-29',
    intro: '持续维护企业与商品信息，能让采购方获得更稳定的判断依据。以下建议侧重于真实展示与高效沟通。',
    sections: [
      { title: '哪些信息应定期检查', list: ['企业简介、主营类目和服务区域是否仍与实际经营相符', '在售商品的型号、库存状态、起订要求和交期说明', '资质、授权、检测或认证材料是否临近有效期', '联系人、售后方式和交付能力说明是否仍然有效'] },
      { title: '收到询盘后如何回应', paragraphs: ['建议优先确认采购方的型号、技术参数、数量、用途和交期，再提供与需求相匹配的供货建议。对于无法确认的内容，应明确说明需要进一步核实，而不是做出不确定承诺。'] },
      { title: '报价与交期说明', paragraphs: ['报价应说明适用的规格、数量范围、税费或运费口径，以及报价有效期。交期受原材料、排产或运输影响时，可说明当前预计安排和可能需要确认的条件。'] },
      { title: '发生变更时的处理', paragraphs: ['产品停产、规格调整、供货周期变化或服务范围缩减时，应尽快更新页面信息。对已经沟通中的采购需求，建议主动说明变化原因和可替代方案。'] },
      { title: '售后与争议处理建议', paragraphs: ['采购前应尽量确认产品版本、验收标准、交付方式和责任边界。发生质量、数量或交付争议时，建议保留沟通记录、订单信息、物流凭据、检验或现场照片，并基于事实协商处理。', '对于需要技术确认的情况，应由了解产品的人员参与回复，避免客服口径与实际能力不一致。'] },
      { title: '长期维护的节奏', paragraphs: ['建议定期检查重点商品，而不是等到出现询盘争议后再集中修改。可以按月查看产品参数、图片、交期与资质有效期；在原材料、排产或业务范围变化时同步更新。'] },
    ],
  },
  {
    id: 'quotation', group: '卖家指南', label: '报价与交期说明', title: '报价与交期沟通说明', updatedAt: '2026-09-29',
    intro: '工业采购中的报价通常与规格、数量、交付方式和服务范围有关。清楚说明报价条件，有助于双方更快确认是否继续推进。',
    sections: [
      { title: '报价前建议确认的信息', list: ['产品型号、关键参数、图纸或样品要求', '采购数量、批次安排和是否需要打样', '收货地区、运输方式、包装或送货要求', '是否含税、是否需要安装、调试或技术支持'] },
      { title: '报价内容应说明什么', paragraphs: ['报价应尽量对应具体规格、数量范围和交付条件，并说明价格是否含税、运费或包装费用。对于阶梯价格、定制费用或模具费用，也应说明适用条件。'] },
      { title: '交期如何表达', paragraphs: ['交期应结合当前库存、采购周期、排产和物流安排判断。若只能提供预计交期，应明确这是预估时间，并说明影响交付的前置条件。'] },
      { title: '价格或交期变化时', paragraphs: ['当原材料、汇率、排产、运输或客户规格变化影响报价时，应及时告知采购方变化原因、有效期限与可选方案，避免沿用已经不适用的承诺。'] },
    ],
  },
  {
    id: 'delivery', group: '卖家指南', label: '履约与售后说明', title: '履约与售后服务说明', updatedAt: '2026-09-29',
    intro: '在订单确认前沟通清楚交付、验收和售后边界，可以减少后续争议，也能提升采购方对服务能力的判断。',
    sections: [
      { title: '交付前确认', paragraphs: ['建议双方确认产品版本、数量、包装要求、收货信息、交付时间和验收方式。对于非标定制、工程配套或分批供货，应将关键节点和变更条件说明清楚。'] },
      { title: '发货与签收', paragraphs: ['发货前应核对型号、数量、随货资料和包装状态；发货后可保留物流、装箱、签收等记录。出现运输损坏、少件或错发时，应及时核实并与采购方沟通处理方式。'] },
      { title: '质量与售后响应', paragraphs: ['收到质量反馈时，先确认批次、使用条件、问题现象和可佐证信息。对于需要技术判断的问题，应由具备产品知识的人员参与处理，并明确回复时点和下一步安排。'] },
      { title: '服务范围的边界', paragraphs: ['安装、调试、现场服务、备件或退换货等服务内容，应根据实际合同、产品属性和双方约定执行。不要将可选服务描述为默认包含的服务。'] },
    ],
  },
  {
    id: 'conduct', group: '卖家指南', label: '经营行为与风险提示', title: '商家经营行为与风险提示', updatedAt: '2026-09-29',
    intro: '真实、清晰、有边界的信息展示是采购沟通的基础。本页提示常见的风险场景，帮助商家减少因信息不准确造成的误解。',
    sections: [
      { title: '真实经营与如实展示', paragraphs: ['商家应基于实际可供产品、产能、授权与服务能力展示信息。不应冒用他人主体、品牌、资质、图片或项目案例，也不应将合作意向描述为已经获得的资格。'] },
      { title: '不宜使用的宣传表述', list: ['“全行业最低价”“永久现货”“任何场景都适用”等无法验证的绝对化承诺', '无法证明来源的排名、荣誉、专利、认证或客户案例', '以明显偏离实际条件的价格、交期或配置吸引采购方咨询', '将样品、试制版本或局部能力描述为标准量产供货能力'] },
      { title: '品牌与知识产权', paragraphs: ['展示他人品牌、商标、专利产品或技术资料时，应确保拥有合法来源和必要授权。收到权利人异议或发现页面存在误用时，应先核实并及时处理相关内容。'] },
      { title: '沟通边界', paragraphs: ['与采购方沟通时，应围绕产品、规格、报价、交期和服务条件提供清晰答复。不要引导采购方绕开必要的核验与确认流程，也不要承诺超出企业实际能力的方案。'] },
      { title: '发现风险后的处理', paragraphs: ['若发现商品参数写错、资质过期、图片来源不清或供货状态发生变化，建议先暂停相关展示并完成核实。对于已在沟通中的采购方，应主动说明影响范围和后续安排。'] },
    ],
  },
  {
    id: 'faq', group: '常见问题', label: '商家常见问题', title: '商家入驻常见问题', updatedAt: '2026-09-29',
    intro: '以下回答聚焦商家准备和信息展示。实际合作安排以平台后续沟通和相关规则为准。',
    sections: [
      { title: '没有完整商品资料，可以先了解吗？', paragraphs: ['可以先整理主营方向和可供能力。平台后续会根据类目说明需要补充的商品信息与资料。', '但在正式展示前，至少应能够清楚说明产品范围、主要规格和基础供货条件，避免采购方无法判断适配性。'] },
      { title: '贸易商可以展示品牌商品吗？', paragraphs: ['可以，但需要保证货源真实，并在需要时提供与品牌、产品相匹配的有效授权或进货证明。', '如授权存在产品、区域或期限限制，应在展示和沟通中按照实际范围说明。'] },
      { title: '为什么商品信息需要写得这么细？', paragraphs: ['工业采购往往涉及型号、尺寸、材质、交期等条件。信息越清晰，采购方越容易判断是否与需求相符，也能减少无效询盘。', '准确的参数还能帮助商家在沟通初期筛除不适配需求，把时间留给真正可成交的项目。'] },
      { title: '资料更新后是否需要重新核验？', paragraphs: ['涉及企业主体、资质有效期、品牌授权或主营方向等关键变化时，建议及时更新并按要求补充核验材料。'] },
      { title: '可以承诺最低价或最快交付吗？', paragraphs: ['不建议使用无法稳定兑现的绝对承诺。价格和交期通常与规格、数量、材料、排产及服务区域有关，应根据实际条件进行说明。'] },
      { title: '发现页面信息有误怎么办？', paragraphs: ['应尽快更正企业或商品信息。若错误内容涉及资质、品牌、产品参数或供货能力，建议先暂停相关表述，核实后再恢复展示。'] },
      { title: '采购方要求提供检测或认证材料，必须马上给吗？', paragraphs: ['应根据产品属性、合作阶段和资料范围进行确认。可以先说明现有材料的适用范围与有效期；涉及保密、项目专用或需要进一步核实的材料，应与采购方明确沟通方式。'] },
      { title: '商品型号更新后，旧型号还可以保留吗？', paragraphs: ['如果旧型号仍可供货，可明确标注版本、库存或交期；若已停产或被替代，应在页面中清晰说明，必要时提供替代型号及差异说明。'] },
      { title: '采购方询问的需求不在主营范围内，应该怎么回复？', paragraphs: ['建议如实说明当前无法覆盖的部分；如确有可靠的配套方案，也应明确自身负责的产品与服务边界，避免将第三方能力误认为自身承诺。'] },
    ],
  },
  {
    id: 'product-faq', group: '常见问题', label: '商品信息常见问题', title: '商品信息常见问题', updatedAt: '2026-09-29',
    intro: '围绕型号、图片、参数和供货状态，整理商家在维护商品信息时经常遇到的问题。',
    sections: [
      { title: '一个商品可以写多个型号吗？', paragraphs: ['可以。型号之间仅存在规格差异时，可在同一商品中说明可选型号与对应参数；如果产品用途、材质、交付方式差异较大，建议分开展示，方便采购方比较。'] },
      { title: '没有实拍图可以使用效果图吗？', paragraphs: ['可以作为辅助图片，但应明确为示意或效果图，不能让采购方误以为是实际交付产品。主图优先使用能真实反映产品外观和配置的图片。'] },
      { title: '参数暂时无法确认怎么办？', paragraphs: ['不建议填写猜测或通用参数。可以先展示已确认的参数，并说明其余信息需要根据型号或工况进一步选型确认。'] },
      { title: '商品停售后需要删除吗？', paragraphs: ['如不再供货，应及时下架或标明停售；如有替代型号，可在说明中提供对应关系，但不要继续按旧型号承诺现货或原交期。'] },
    ],
  },
  {
    id: 'service-faq', group: '常见问题', label: '履约服务常见问题', title: '履约服务常见问题', updatedAt: '2026-09-29',
    intro: '以下内容帮助商家在报价、交付与售后阶段建立更一致的沟通口径。',
    sections: [
      { title: '采购方要求样品，应该怎么约定？', paragraphs: ['应说明样品是否收费、规格版本、交付时间、运费承担和样品确认方式。样品仅用于测试时，也应明确其与批量供货版本可能存在的差异。'] },
      { title: '非标产品的交期如何确认？', paragraphs: ['应在确认图纸、技术要求、材料、数量和预付款等关键条件后安排。对于尚未完成技术确认的项目，可先提供估算周期并注明前提。'] },
      { title: '遇到售后问题可以由销售直接答复吗？', paragraphs: ['基础进度可以由销售同步，但涉及技术责任、质量判断或处理方案时，建议由技术、质量或售后人员共同确认，保证答复可执行。'] },
      { title: '采购方临时变更需求怎么办？', paragraphs: ['先确认变更会对规格、价格、排产和交期产生哪些影响，再书面同步新的方案。未确认前，不宜按原订单继续承诺不受影响。'] },
    ],
  },
]

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
// 后端日期字段有时以 ISO 时间字符串返回；工商档案仅展示日期部分，避免暴露时区时间。
const formatBusinessDate = (value, fallback = '—') => {
  if (!value) return fallback
  const date = String(value).match(/^\d{4}-\d{2}-\d{2}/)?.[0]
  return date || String(value)
}
// 法定代表人属于个人信息，仅保留首个字符，其余字符统一脱敏。
const maskLegalRepresentative = (value) => {
  const name = Array.from(String(value || '').trim())
  return name.length ? `${name[0]}${'*'.repeat(Math.max(0, name.length - 1))}` : '—'
}

const emptyProduct = {
  id: '', name: '商品加载中', price: '—', tag: '', badge: '', scene: 'factory', icon: 'factory', image: '',
  images: [], products: [], raw: {},
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
  image: item.mainImageUrl || '',
  images: item.imageUrls?.filter(Boolean)?.length ? item.imageUrls.filter(Boolean) : (item.mainImageUrl ? [item.mainImageUrl] : []),
  location: item.originPlace || item.company?.shippingAddress?.displayName || '中国',
  supplier: item.company?.companyName || '',
  years: item.company?.statistics?.operatingYears ? `${item.company.statistics.operatingYears}年` : '',
  raw: normalizedItem,
  }
}
// A previous data migration created many supplier logos from the exact same
// "mountain" SVG. Keep uploaded, merchant-provided logos untouched, but replace
// those known placeholders with compact identity marks that read as actual
// industrial company logos at card size.
const generatedLogoFiles = new Set([
  'changsha-andun.svg', 'changzhou-ruikong.svg', 'chengdu-xishu.svg', 'chongqing-hengan.svg', 'dalian-huaan.svg', 'dongguan-kerun.svg',
  'foshan-jingrui.svg', 'hangzhou-chengjing.svg', 'hebei-langdun.svg', 'hefei-anyuan.svg', 'jiaxing-guwei.svg', 'linyi-jinhu.svg',
  'nanjing-lanqing.svg', 'qingdao-haiyue.svg', 'shenzhen-zhilian.svg', 'suzhou-honglian.svg', 'suzhou-saihu.svg', 'taizhou-tuoli.svg',
  'tianjin-haichuan.svg', 'weifang-taigu.svg', 'wenzhou-chuangjie.svg', 'wuhan-gelian.svg', 'wuxi-qingyuan.svg', 'yixing-hengtong.svg',
  ...Array.from({ length: 24 }, (_, index) => `featured-merchant-${String(index + 1).padStart(2, '0')}.svg`),
])
const escapeSvgText = (value) => String(value || '').replace(/[&<>"']/g, (character) => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&apos;' })[character])
const logoProfile = (category, identity) => {
  const key = `${category || ''}${identity || ''}`
  const hash = [...key].reduce((total, character) => total * 31 + character.charCodeAt(0), 7) >>> 0
  if (/包装/.test(key)) return { colors: ['#ef7b26', '#8d2d1c'], mark: '<path d="M55 84 120 48l65 36v72l-65 36-65-36V84Zm65 0v72m-65-72 65 36 65-36" fill="none" stroke="#fff" stroke-width="13" stroke-linejoin="round"/><path d="M84 103h72" stroke="#ffd79f" stroke-width="10" stroke-linecap="round"/>' }
  if (/水|过滤|环保/.test(key)) return { colors: ['#12a3a6', '#075b86'], mark: '<path d="M120 45c-27 34-47 57-47 79a47 47 0 0 0 94 0c0-22-20-45-47-79Z" fill="none" stroke="#fff" stroke-width="13"/><path d="M66 163c21-14 36-14 54 0 18 14 33 14 54 0" fill="none" stroke="#a8f2e9" stroke-width="11" stroke-linecap="round"/>' }
  if (/电缆|线缆/.test(key)) return { colors: ['#3f51b5', '#1a235f'], mark: '<path d="M54 79c30 0 30 82 60 82s30-82 60-82" fill="none" stroke="#fff" stroke-width="15" stroke-linecap="round"/><path d="M54 121c30 0 30-58 60-58s30 58 60 58" fill="none" stroke="#9bd6ff" stroke-width="11" stroke-linecap="round"/><circle cx="54" cy="79" r="10" fill="#ffd566"/><circle cx="174" cy="79" r="10" fill="#ffd566"/>' }
  if (/工控|控制|自动化|电气/.test(key)) return { colors: ['#1269c7', '#17306a'], mark: '<rect x="62" y="62" width="116" height="116" rx="24" fill="none" stroke="#fff" stroke-width="12"/><circle cx="95" cy="100" r="10" fill="#a5efff"/><circle cx="145" cy="94" r="10" fill="#a5efff"/><circle cx="127" cy="145" r="10" fill="#a5efff"/><path d="M103 105l32-8m-30 10 18 28m20-30-12 29" stroke="#fff" stroke-width="9" stroke-linecap="round"/>' }
  if (/扳手|五金|工具/.test(key)) return { colors: ['#485d75', '#18293e'], mark: '<path d="M158 55a34 34 0 0 0-38 45l-57 57a20 20 0 1 0 28 28l57-57a34 34 0 0 0 45-38l-24 24-24-8-8-24 21-27Z" fill="none" stroke="#fff" stroke-width="13" stroke-linejoin="round"/><circle cx="77" cy="170" r="6" fill="#ffcc61"/>' }
  if (/矿|防爆|安全/.test(key)) return { colors: ['#d66a23', '#6e2819'], mark: '<path d="M120 48 178 70v45c0 35-24 61-58 77-34-16-58-42-58-77V70l58-22Z" fill="none" stroke="#fff" stroke-width="12"/><path d="m94 125 17 17 35-39" fill="none" stroke="#ffdd82" stroke-width="13" stroke-linecap="round" stroke-linejoin="round"/>' }
  if (/材料|建材|防水/.test(key)) return { colors: ['#bd4d46', '#68243d'], mark: '<path d="m58 85 62-33 62 33-62 33-62-33Zm0 38 62 33 62-33m-124 38 62 33 62-33" fill="none" stroke="#fff" stroke-width="13" stroke-linejoin="round"/>' }
  if (/化工|药剂/.test(key)) return { colors: ['#7a48ad', '#3b216f'], mark: '<path d="M94 54h52m-36 0v44l-39 64a18 18 0 0 0 16 27h66a18 18 0 0 0 16-27l-39-64V54" fill="none" stroke="#fff" stroke-width="12" stroke-linejoin="round"/><path d="M89 142h62" stroke="#e8b7ff" stroke-width="12" stroke-linecap="round"/>' }
  const palette = [['#0d8b8d', '#073b56'], ['#bf5a32', '#5b2543'], ['#4662bf', '#20265d'], ['#22815d', '#123c43']][hash % 4]
  return { colors: palette, mark: '<path d="M120 48 174 79v62l-54 31-54-31V79l54-31Z" fill="none" stroke="#fff" stroke-width="13"/><path d="M92 128h56M120 100v56" stroke="#b9f4ed" stroke-width="12" stroke-linecap="round"/>' }
}
// Hand-authored/AI-generated artwork is used where a supplier's industry gives
// us a clear visual cue. These are kept as normal project assets instead of
// asking every browser to synthesize the same template at runtime.
const generatedBrandAssets = {
  'suzhou-honglian.svg': '/api/uploads/seed-company-logos-ai-suzhou-honglian.png',
  'foshan-jingrui.svg': '/api/uploads/seed-company-logos-ai-foshan-jingrui.png',
  'wenzhou-chuangjie.svg': '/api/uploads/seed-company-logos-ai-wenzhou-chuangjie.png',
  'wuxi-qingyuan.svg': '/api/uploads/seed-company-logos-ai-wuxi-qingyuan.png',
  'hangzhou-chengjing.svg': '/api/uploads/seed-company-logos-ai-hangzhou-chengjing.png',
  'tianjin-haichuan.svg': '/api/uploads/seed-company-logos-ai-tianjin-haichuan.png',
  'yixing-hengtong.svg': '/api/uploads/seed-company-logos-ai-yixing-hengtong.png',
  'hefei-anyuan.svg': '/api/uploads/seed-company-logos-ai-hefei-anyuan.png',
  'chengdu-xishu.svg': '/api/uploads/seed-company-logos-ai-chengdu-xishu.png',
  'shenzhen-zhilian.svg': '/api/uploads/seed-company-logos-ai-shenzhen-zhilian.png',
  'changzhou-ruikong.svg': '/api/uploads/seed-company-logos-ai-changzhou-ruikong.png',
  'wuhan-gelian.svg': '/api/uploads/seed-company-logos-ai-wuhan-gelian.png',
}
const merchantLogo = (originalLogo, shortName, category) => {
  const sourceFile = String(originalLogo || '').split('/').pop().replace(/^seed-company-logos-/, '')
  if (generatedBrandAssets[sourceFile]) return generatedBrandAssets[sourceFile]
  if (!generatedLogoFiles.has(sourceFile)) return originalLogo || ''
  const identityIndex = [...generatedLogoFiles].indexOf(sourceFile)
  const hue = (identityIndex * 47 + 198) % 360
  const palette = `hsl(${hue} 68% 39%)`
  const wordmark = escapeSvgText(String(shortName || '').slice(0, 2))
  const label = escapeSvgText(String(shortName || '').slice(0, 4))
  const layouts = [
    `<rect width="240" height="240" rx="42" fill="${palette}"/><path d="M52 112h136" stroke="#fff" stroke-width="12"/><text x="120" y="98" fill="#fff" font-size="38" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text><text x="120" y="153" fill="#fff" font-size="17" font-family="PingFang SC" text-anchor="middle">${label}</text>`,
    `<circle cx="120" cy="120" r="86" fill="none" stroke="${palette}" stroke-width="12"/><circle cx="120" cy="120" r="63" fill="${palette}"/><text x="120" y="135" fill="#fff" font-size="46" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text><circle cx="120" cy="43" r="7" fill="${palette}"/><circle cx="120" cy="197" r="7" fill="${palette}"/>`,
    `<path d="M32 50h176v140H32z" fill="#15263d"/><path d="M32 158h176v32H32z" fill="${palette}"/><text x="120" y="125" fill="#fff" font-size="48" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text><text x="120" y="183" fill="#fff" font-size="17" font-family="PingFang SC" text-anchor="middle">${label}</text>`,
    `<path d="M42 51h88l65 65-65 65H42z" fill="${palette}"/><path d="m96 74 40 42-40 42" fill="none" stroke="#fff" stroke-width="13"/><text x="120" y="190" fill="${palette}" font-size="26" font-family="PingFang SC" font-weight="800" text-anchor="middle">${label}</text>`,
    `<rect x="31" y="34" width="178" height="172" rx="26" fill="#f5f8fa"/><path d="M59 72h122v13H59zm0 30h92v13H59zm0 30h122v13H59z" fill="${palette}"/><text x="120" y="181" fill="#1f344b" font-size="34" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text>`,
    `<path d="M120 37 201 84v94l-81 47-81-47V84l81-47Z" fill="${palette}"/><path d="M76 115h88" stroke="#fff" stroke-width="8" opacity=".55"/><text x="120" y="139" fill="#fff" font-size="44" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text>`,
    `<rect width="240" height="240" rx="32" fill="#fff"/><path d="M38 55h164v25H38zM38 102h104v25H38zM38 149h164v25H38z" fill="${palette}"/><text x="166" y="126" fill="#1e293b" font-size="30" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text>`,
    `<circle cx="120" cy="120" r="92" fill="#fff" stroke="${palette}" stroke-width="18"/><path d="M68 135c29-60 74-60 104 0" fill="none" stroke="${palette}" stroke-width="13"/><text x="120" y="176" fill="${palette}" font-size="28" font-family="PingFang SC" font-weight="800" text-anchor="middle">${label}</text>`,
    `<path d="M35 45h170v150H35z" fill="#f0f7f7"/><path d="m52 159 74-89 62 89" fill="none" stroke="${palette}" stroke-width="16"/><text x="120" y="197" fill="${palette}" font-size="24" font-family="PingFang SC" font-weight="800" text-anchor="middle">${label}</text>`,
    `<path d="M40 65h160v110H40z" fill="${palette}"/><path d="M40 65 93 40h107v110l-53 25H40z" fill="#fff" opacity=".22"/><text x="120" y="130" fill="#fff" font-size="45" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text><text x="120" y="160" fill="#fff" font-size="16" font-family="PingFang SC" text-anchor="middle">${label}</text>`,
    `<path d="M53 52h134v136H53z" fill="#152238"/><circle cx="120" cy="120" r="45" fill="${palette}"/><text x="120" y="135" fill="#fff" font-size="42" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text><path d="M53 205h134" stroke="${palette}" stroke-width="12"/>`,
    `<rect x="28" y="52" width="184" height="136" rx="68" fill="${palette}"/><text x="120" y="122" fill="#fff" font-size="45" font-family="Arial" font-weight="800" text-anchor="middle">${wordmark}</text><text x="120" y="153" fill="#fff" font-size="16" font-family="PingFang SC" text-anchor="middle">${label}</text>`,
  ]
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 240 240">${layouts[identityIndex % layouts.length]}</svg>`
  return `data:image/svg+xml;charset=UTF-8,${encodeURIComponent(svg)}`
}
const toSupplier = (item = {}, index = 0) => {
  const companyName = item.companyName || '店铺加载中'
  const category = item.featuredProducts?.[0]?.categoryName || item.products?.[0]?.categoryName || ''
  return {
    id: item.companyId || '',
    name: companyName,
    shortName: item.shortName || companyName,
    mark: item.logoText || companyName.slice(0, 2),
    tone: item.themeColor || ['blue', 'green', 'orange', 'teal'][index % 4],
    logo: merchantLogo(item.logoUrl, item.shortName || companyName, category),
    category,
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
    recommendationProducts.value = homeProducts.value
    recommendationPage.value = 1
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
      // 列表接口默认每页只返回 20 条；即使初始链接没有分页参数，也显式传递，
      // 让 URL 状态与实际请求保持一致。
      if (!apiParams.get('page')) apiParams.set('page', '1')
      if (!apiParams.get('pageSize')) apiParams.set('pageSize', String(defaultCatalogPageSize))
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
const catalogPageSize = computed(() => {
  const [, query = ''] = routeHash.value.split('?')
  const pageSize = Number(new URLSearchParams(query).get('pageSize'))
  return Number.isInteger(pageSize) && pageSize > 0 ? pageSize : defaultCatalogPageSize
})
const catalogPage = computed(() => {
  const [, query = ''] = routeHash.value.split('?')
  const page = Number(new URLSearchParams(query).get('page'))
  return Number.isInteger(page) && page > 0 ? page : 1
})
const catalogPageCount = computed(() => Math.max(1, Math.ceil(catalogTotal.value / catalogPageSize.value)))
const catalogPaginationItems = computed(() => {
  const count = catalogPageCount.value
  if (count <= 7) return Array.from({ length: count }, (_, index) => index + 1)

  const start = Math.max(2, catalogPage.value - 2)
  const end = Math.min(count - 1, catalogPage.value + 2)
  const items = [1]
  if (start > 2) items.push('ellipsis-left')
  items.push(...Array.from({ length: end - start + 1 }, (_, index) => start + index))
  if (end < count - 1) items.push('ellipsis-right')
  items.push(count)
  return items
})
const setCatalogPage = (page) => {
  const nextPage = Math.min(Math.max(1, page), catalogPageCount.value)
  if (nextPage === catalogPage.value) return
  const [, query = ''] = routeHash.value.split('?')
  const params = new URLSearchParams(query)
  params.set('page', String(nextPage))
  params.set('pageSize', String(catalogPageSize.value))
  window.location.hash = `#/catalog?${params.toString()}`
}
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
const activeMerchantGuideContent = computed(() => merchantGuides.find((guide) => guide.id === activeMerchantGuide.value) || merchantGuides[0])
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
const categoryHref = (id) => id ? `#/catalog?categoryId=${encodeURIComponent(id)}` : '#/catalog'
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
const cycleRecommendations = async () => {
  if (recommendationLoading.value) return

  recommendationLoading.value = true
  try {
    const nextPage = recommendationPage.value + 1
    const data = await api(`/public/products?${new URLSearchParams({ page: String(nextPage), pageSize: '12' })}`)
    const products = (data.items || []).map(toProduct)

    // 到达最后一页时，从第一批重新开始，保持“换一批”始终可用。
    if (products.length) {
      recommendationProducts.value = products
      recommendationPage.value = nextPage
    } else {
      const firstPageData = await api('/public/products?page=1&pageSize=12')
      recommendationProducts.value = (firstPageData.items || []).map(toProduct)
      recommendationPage.value = 1
    }
  } catch (error) {
    apiError.value = error.message || '推荐商品加载失败'
  } finally {
    recommendationLoading.value = false
  }
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
  // 每个分类只展示自身的商家；不能再用首个分类的商家补位，
  // 否则同一公司会错误地出现在每一个优选厂商标签下。
  return homeSupplierShowcases.value
    .filter((supplier) => supplier.category === activeSupplierCategory.value)
    .slice(0, 4)
})

const premiumCatalogProducts = computed(() => homeProducts.value.slice(0, 6))
const factoryCatalogProducts = computed(() => homeProducts.value.slice(2, 8))
const recommendedProducts = computed(() => recommendationProducts.value.map((product) => ({
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
            <a href="#/catalog">更多 &gt;</a>
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
              <img v-if="newsCover(item)" class="news-thumb" :src="newsCover(item)" :alt="item.title" />
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
          <a href="#/catalog">查看更多<BaseIcon name="arrowRight" /></a>
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
          <button type="button" class="recommend-refresh" :class="{ 'is-loading': recommendationLoading }" :disabled="recommendationLoading" :aria-busy="recommendationLoading" @click="cycleRecommendations">换一批<BaseIcon name="refresh" /></button>
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
        <img v-if="newsCover(selectedNews)" class="news-article-cover" :src="newsCover(selectedNews)" :alt="selectedNews.title" />
        <div class="news-article-body" v-html="newsArticleHtml"></div>
        <footer><a href="#/news"><BaseIcon name="arrowRight" />返回资讯列表</a></footer>
      </article>
      <template v-else-if="latestNews.length">
        <article class="news-lead">
          <img v-if="newsCover(latestNews[0])" :src="newsCover(latestNews[0])" :alt="latestNews[0].title" />
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
              <img v-if="newsCover(item)" :src="newsCover(item)" :alt="item.title" />
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
  <main v-else-if="isMerchantJoinPage" class="merchant-join-page" aria-label="商家指南">
    <section class="merchant-guide-shell">
      <aside class="merchant-guide-nav" aria-label="商家指南目录">
        <div v-for="group in ['卖家指南', '常见问题']" :key="group" class="merchant-guide-group">
          <h2>{{ group }}</h2>
          <button v-for="guide in merchantGuides.filter((item) => item.group === group)" :key="guide.id" type="button" :class="{ active: activeMerchantGuide === guide.id }" @click="activeMerchantGuide = guide.id">{{ guide.label }}<BaseIcon v-if="activeMerchantGuide === guide.id" name="arrowRight" /></button>
        </div>
      </aside>
      <article class="merchant-guide-article">
        <header><p>商家指南</p><h1>{{ activeMerchantGuideContent.title }}</h1><time>更新日期：{{ activeMerchantGuideContent.updatedAt }}</time></header>
        <p class="merchant-guide-intro">{{ activeMerchantGuideContent.intro }}</p>
        <section v-for="(section, index) in activeMerchantGuideContent.sections" :key="section.title" class="merchant-guide-section"><h2>{{ index + 1 }}. {{ section.title }}</h2><p v-for="paragraph in section.paragraphs || []" :key="paragraph">{{ paragraph }}</p><ul v-if="section.list"><li v-for="item in section.list" :key="item">{{ item }}</li></ul></section>
        <footer><BaseIcon name="shield" />本页内容用于帮助商家了解资料准备与信息展示要求，具体以平台规则和实际沟通为准。</footer>
      </article>
    </section>
  </main>
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
      <nav v-if="catalogTotal > catalogPageSize" class="catalog-pagination" aria-label="搜索结果分页">
        <p class="catalog-pagination-summary"><span>共找到</span><b>{{ catalogTotal }}</b><span>条匹配结果</span><i></i><em>第 {{ catalogPage }} / {{ catalogPageCount }} 页</em></p>
        <div class="catalog-pagination-controls">
          <button type="button" class="catalog-pagination-direction previous" :disabled="catalogPage === 1" aria-label="上一页" @click="setCatalogPage(catalogPage - 1)"><BaseIcon name="arrowRight" /></button>
          <template v-for="item in catalogPaginationItems" :key="item">
            <button v-if="typeof item === 'number'" type="button" :class="{ active: item === catalogPage }" :aria-current="item === catalogPage ? 'page' : undefined" :aria-label="`第 ${item} 页`" @click="setCatalogPage(item)">{{ item }}</button>
            <span v-else class="catalog-pagination-ellipsis">…</span>
          </template>
          <button type="button" class="catalog-pagination-direction next" :disabled="catalogPage === catalogPageCount" aria-label="下一页" @click="setCatalogPage(catalogPage + 1)"><BaseIcon name="arrowRight" /></button>
        </div>
      </nav>
    </section>
  </main>

  <main v-else-if="isCompanyPage" class="company-page">
    <section class="company-wrap company-archive-wrap">
      <div class="company-crumb">
        <a href="#/">首页</a><span>/</span><a href="#/catalog?type=company">实力厂商</a><span>/</span><strong>{{ currentStoreSupplier.name }}</strong>
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
                <tr><th>企业名称</th><td>{{ currentStoreSupplier.name }}</td><th>注册资本</th><td>*******</td></tr>
                <tr><th>注册地址</th><td>{{ companyBusinessInfo.registeredAddress || supplierLocation(currentStoreSupplier) || '—' }}</td><th>企业网址</th><td>{{ companyBusinessInfo.websiteUrl || '—' }}</td></tr>
                <tr><th>营业执照</th><td>{{ companyVerification.subjectVerified ? '平台已核验' : '待核验' }}</td><th>统一社会信用代码</th><td>{{ companyBusinessInfo.unifiedSocialCreditCodeMasked || '—' }}</td></tr>
                <tr><th>组织机构代码</th><td>{{ companyBusinessInfo.organizationCode || '—' }}</td><th>纳税人识别号</th><td>{{ companyBusinessInfo.taxpayerId || '—' }}</td></tr>
                <tr><th>工商注册号</th><td>{{ companyBusinessInfo.registrationNumber || '—' }}</td><th>法定代表人</th><td>{{ maskLegalRepresentative(companyBusinessInfo.legalRepresentative) }}</td></tr>
                <tr><th>经营状态</th><td>{{ displayCode(companyBusinessInfo.businessStatus) }}</td><th>成立时间</th><td>{{ formatBusinessDate(companyBusinessInfo.establishedAt) }}</td></tr>
                <tr><th>营业期限</th><td>{{ formatBusinessDate(companyBusinessInfo.businessTermEnd || companyBusinessInfo.businessTermStart, '长期') }}</td><th>核验日期</th><td>—</td></tr>
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
        <a href="#/">首页</a><span>/</span><a :href="categoryHref(detailProduct.raw?.categoryId)">{{ detailProduct.raw?.categoryName || '商品分类' }}</a><span>/</span><strong>{{ detailProduct.name }}</strong>
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
            <div><dt>采购数量</dt><dd class="detail-quantity"><div class="detail-quantity-stepper"><button type="button" aria-label="减少数量" @click="decreaseQuantity">−</button><input v-model.number="purchaseQuantity" :min="selectedSku?.minOrderQuantity || detailProduct.raw?.minOrderQuantity || 1" type="number" aria-label="采购数量" /><button type="button" aria-label="增加数量" @click="increaseQuantity">＋</button></div><span class="detail-quantity-hint">{{ selectedSku?.minOrderQuantity || detailProduct.raw?.minOrderQuantity || 1 }} {{ detailProduct.raw?.unit || '件' }}起订</span></dd></div>
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
          <a href="#/merchant-join">商家入驻</a>
          <a href="#/merchant-join">入驻流程</a>
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
