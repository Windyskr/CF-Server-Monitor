<template>
  <div id="editModal" class="modal-overlay" :class="{ active: show }">
    <div class="modal-dialog edit-server-modal">
      <div class="modal-header">
        <div class="modal-title">{{ currentServerName }}</div>
        <button class="modal-close" @click="$emit('close')">✕</button>
      </div>
      <input type="hidden" v-model="editForm.id">

      <div class="edit-body">
        <section class="edit-section">
          <div class="edit-section-title">{{ trans.editSectionBasic || 'Basics' }}</div>
          <div class="edit-grid">
            <div class="form-group span-3">
              <label class="form-label">{{ trans.hostnameLabel }} <span class="required">*</span></label>
              <input type="text" name="edit_name" autocomplete="off" v-model="editForm.name" class="form-input" placeholder="e.g. My Server">
            </div>
            <div class="form-group">
              <label class="form-label">{{ trans.groupName }}</label>
              <input type="text" name="edit_server_group" autocomplete="off" v-model="editForm.server_group" class="form-input" placeholder="e.g. US VPS">
            </div>
            <div class="form-group span-2">
              <label class="form-label">{{ trans.region }} CN/US/GB</label>
              <input type="text" name="edit_region" autocomplete="off" v-model.trim="editForm.region" class="form-input" :placeholder="trans.regionPlaceholder">
            </div>
            <div class="form-group span-3">
              <label class="form-label">{{ trans.tags }}</label>
              <input type="text" name="edit_tags" autocomplete="off" v-model="editForm.tags" class="form-input" :placeholder="trans.tagsPlaceholder">
            </div>
            <div class="form-group span-3">
              <label class="form-label">{{ trans.note }}</label>
              <textarea name="edit_note" autocomplete="off" v-model="editForm.note" class="form-textarea edit-note" rows="1" :placeholder="trans.notePlaceholder"></textarea>
            </div>
          </div>
        </section>

        <section class="edit-section">
          <div class="edit-section-title">{{ trans.billing }}</div>
          <div class="edit-grid">
            <div class="form-group">
              <label class="form-label">{{ trans.currency }}</label>
              <select v-model="editForm.currency" class="form-select">
                <option v-for="item in currencySelectOptions" :key="item.symbol" :value="item.symbol">{{ currencyLabel(item) }}</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">{{ trans.price }}</label>
              <input type="text" name="edit_price" autocomplete="off" inputmode="decimal" v-model="editForm.price" class="form-input" placeholder="40.00" @blur="normalizePriceInput">
            </div>
            <div class="form-group">
              <label class="form-label">{{ trans.billingCycle }}</label>
              <select v-model="editForm.billing_cycle" class="form-select">
                <option v-for="item in billingCycleOptions" :key="item.value" :value="item.value">{{ cycleLabel(item) }}</option>
              </select>
            </div>
            <div class="form-group span-2">
              <label class="form-label">{{ trans.expirationDate }}</label>
              <input type="date" name="edit_expire_date" autocomplete="off" v-model="editForm.expire_date" class="form-input" @click="openDatePicker">
            </div>
            <div class="form-group">
              <label class="form-label">{{ trans.autoRenewal }}</label>
              <label class="checkbox-item edit-check">
                <input type="checkbox" v-model="editForm.auto_renewal">
                <b>{{ trans.enabled }}</b>
              </label>
            </div>
          </div>
        </section>

        <section class="edit-section">
          <div class="edit-section-title">{{ trans.editSectionTraffic || 'Traffic' }}</div>
          <div class="edit-grid">
            <div class="form-group span-2">
              <label class="form-label">{{ trans.trafficLimit }} (GB)</label>
              <input type="number" name="edit_traffic_limit" autocomplete="off" v-model="editForm.traffic_limit" class="form-input" :placeholder="trans.noLimit || '∞'" min="0" step="1">
            </div>
            <div class="form-group span-2">
              <label class="form-label">{{ trans.trafficCalcType }}</label>
              <select v-model="editForm.traffic_calc_type" class="form-select">
                <option v-for="item in trafficCalcOptions" :key="item.value" :value="item.value">{{ item.label }}</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">
                {{ trans.trafficResetDay }}
                <HelpTooltip :text="trans.trafficResetDayTip" />
              </label>
              <select name="edit_reset_day" v-model="editForm.reset_day" class="form-select">
                <option :value="0">0</option>
                <option v-for="day in 31" :key="day" :value="day">{{ day }}</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">
                {{ trans.trafficResetTime }}
                <HelpTooltip :text="trans.trafficResetTimeTip" />
              </label>
              <input type="time" name="edit_reset_time" autocomplete="off" v-model="editForm.reset_time" class="form-input">
            </div>
            <div class="form-group span-2">
              <label class="form-label">{{ trans.networkInterface }}</label>
              <input type="text" name="edit_interface" autocomplete="off" v-model.trim="editForm.interface" class="form-input" :placeholder="trans.networkInterfacePlaceholder">
            </div>
            <div class="form-group span-2">
              <label class="form-label">
                {{ trans.rxCorrection }} (GB)
                <HelpTooltip :text="trans.correctionHint" />
              </label>
              <input type="number" name="edit_rx_correction" autocomplete="off" v-model="editForm.rx_correction" class="form-input" placeholder="0" min="0" step="0.1">
            </div>
            <div class="form-group span-2">
              <label class="form-label">
                {{ trans.txCorrection }} (GB)
                <HelpTooltip :text="trans.correctionHint" />
              </label>
              <input type="number" name="edit_tx_correction" autocomplete="off" v-model="editForm.tx_correction" class="form-input" placeholder="0" min="0" step="0.1">
            </div>
          </div>

          <div class="edit-subtitle">
            {{ trans.interfaceTrafficTitle || 'Per-interface traffic' }}
            <HelpTooltip :text="trans.interfaceTrafficHint" />
          </div>
          <template v-if="!interfaceJsonInvalid">
            <div v-if="interfaceRows.length" class="iface-table">
              <div class="iface-row iface-head">
                <span>{{ trans.interfaceName || 'Interface' }}</span>
                <span>{{ trans.interfaceAlias || 'Alias' }}</span>
                <span>{{ trans.trafficLimit }} (GB)</span>
                <span>{{ trans.trafficCalcType }}</span>
                <span></span>
              </div>
              <div v-for="(row, index) in interfaceRows" :key="row.key" class="iface-row">
                <input v-model.trim="row.name" type="text" autocomplete="off" maxlength="64" placeholder="eth0" :aria-label="trans.interfaceName" :class="['form-input', { 'input-invalid': duplicateInterfaceNames.has(row.name) }]">
                <input v-model.trim="row.alias" type="text" autocomplete="off" maxlength="64" :placeholder="trans.interfaceAliasPlaceholder" :aria-label="trans.interfaceAlias" class="form-input">
                <input v-model="row.limit" type="number" min="0" step="1" :placeholder="trans.noLimit || '∞'" :aria-label="`${trans.trafficLimit} (GB)`" class="form-input">
                <select v-model="row.calc" :aria-label="trans.trafficCalcType" class="form-select">
                  <option v-for="item in trafficCalcOptions" :key="item.value" :value="item.value">{{ item.label }}</option>
                </select>
                <button type="button" class="iface-remove" :title="trans.delete" :aria-label="trans.delete" @click="removeInterfaceRow(index)">✕</button>
              </div>
            </div>
            <p v-else class="iface-empty">{{ trans.interfaceTrafficEmpty }}</p>
            <button type="button" class="btn btn-sm iface-add" @click="addInterfaceRow">+ {{ trans.addInterface || 'Add interface' }}</button>
          </template>
          <div v-else class="edit-grid">
            <p class="iface-empty span-6">{{ trans.interfaceJsonInvalid }}</p>
            <div class="form-group span-3">
              <label class="form-label">{{ trans.interfaceAliases }}</label>
              <textarea v-model.trim="editForm.interface_aliases" class="form-textarea" rows="3" placeholder='{"eth0":"公网","eth1":"内网"}'></textarea>
            </div>
            <div class="form-group span-3">
              <label class="form-label">{{ trans.interfaceTrafficSettings }}</label>
              <textarea v-model.trim="editForm.interface_traffic_settings" class="form-textarea" rows="3" placeholder='{"eth0":{"traffic_limit":1000,"traffic_calc_type":"total"}}'></textarea>
            </div>
          </div>
        </section>

        <section class="edit-section">
          <div class="edit-section-title">{{ trans.editSectionAgent || 'Agent' }}</div>
          <div class="edit-grid">
            <div class="form-group">
              <label class="form-label">
                {{ trans.collectInterval }}
                <HelpTooltip :text="trans.collectIntervalHint" />
              </label>
              <select v-model="editForm.collect_interval" class="form-select">
                <option v-for="second in [0, 1, 2, 5, 10]" :key="second" :value="second">{{ second }}</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">{{ trans.reportInterval }}</label>
              <select v-model="editForm.report_interval" class="form-select">
                <option v-for="second in [30, 60, 120, 180]" :key="second" :value="second">{{ second }}</option>
              </select>
            </div>
            <div class="form-group span-2">
              <label class="form-label">
                {{ trans.connectionMode }}
                <HelpTooltip v-if="!isWssReportEnabled" :text="trans.wssReportDisabledConnectionHint" />
              </label>
              <select v-model="editForm.connection_mode" class="form-select" :disabled="!isWssReportEnabled">
                <option value="auto">{{ trans.connectionModeAuto }}</option>
                <option value="http">{{ trans.connectionModeHttp }}</option>
              </select>
            </div>
            <div v-if="isWssReportEnabled && editForm.connection_mode === 'auto'" class="form-group">
              <label class="form-label">{{ trans.wssReportInterval }}</label>
              <select v-model="editForm.wss_report_interval" class="form-select">
                <option v-for="second in 5" :key="second" :value="second">{{ second }}</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">
                {{ trans.pingMode }}
                <HelpTooltip :text="trans.pingModeIcmpRootHint" />
              </label>
              <select v-model="editForm.ping_mode" class="form-select">
                <option value="tcp">TCP</option>
                <option value="icmp">ICMP (root)</option>
              </select>
            </div>
          </div>
        </section>

        <details class="edit-section edit-collapsible" :open="pingNodesOpen" @toggle="pingNodesOpen = $event.target.open">
          <summary class="edit-section-title">
            {{ trans.editSectionPingNodes || 'Probe nodes' }}
            <span class="edit-section-note">{{ trans.pingNodesInheritHint }}</span>
            <span class="edit-section-caret" aria-hidden="true">▾</span>
          </summary>
          <div class="edit-grid">
            <div v-for="node in pingNodeFields" :key="node.field" class="form-group span-3">
              <label class="form-label">
                {{ node.label }}
                <HelpTooltip :text="trans.pingNodeTip" />
              </label>
              <input type="text" :name="`edit_${node.field}`" autocomplete="off" v-model.trim="editForm[node.field]" :class="['form-input', { 'input-invalid': pingNodeErrors[node.field] }]" :placeholder="node.placeholder">
              <p v-if="pingNodeErrors[node.field]" class="text-red text-sm mt-1">{{ pingNodeErrors[node.field] }}</p>
            </div>
          </div>
        </details>

        <section class="edit-section">
          <div class="edit-section-title">{{ trans.editSectionOptions || 'Options' }}</div>
          <div class="edit-options">
            <label class="checkbox-item edit-check">
              <input type="checkbox" :checked="editForm.auto_update" @change="handleAutoUpdateChange">
              <b>{{ trans.autoUpdate }}</b>
            </label>
            <label class="checkbox-item edit-check">
              <input type="checkbox" v-model="editForm.is_hidden">
              <b>{{ trans.hideFromPublic }}</b>
            </label>
            <label v-if="hasNodeNotificationOptions" class="checkbox-item edit-check">
              <input type="checkbox" v-model="editForm.offline_notify_disabled">
              <b>{{ trans.disableOfflineNotify }}</b>
            </label>
          </div>
        </section>
      </div>

      <div class="modal-footer flex-justify-between">
        <button @click="$emit('save')" class="btn btn-primary" :disabled="hasPingNodeErrors">{{ trans.save }}</button>
        <button @click="$emit('close')" class="btn">{{ trans.cancel }}</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue'
import HelpTooltip from '../../../components/HelpTooltip.vue'
import { PING_NODE_FIELDS, validatePingNode } from '../../../utils/pingNode.js'
import { currentLang } from '../../../utils/i18n.js'
import { BILLING_CYCLES, CURRENCY_OPTIONS, normalizePrice, renewExpireDateIfNeeded } from '../../../utils/server.js'

const editForm = defineModel('editForm', { type: Object, required: true })

const props = defineProps({
  trans: { type: Object, required: true },
  show: { type: Boolean, default: false },
  currentServerName: { type: String, default: '' },
  settings: { type: Object, required: true }
})

const pingNodeErrorMessage = computed(() => (
  props.trans.invalidPingNodeFormat || 'Use domain, IPv4, or host:port. Port must be 1-65535.'
))

const pingNodeErrors = computed(() => Object.fromEntries(
  PING_NODE_FIELDS.map(field => [
    field,
    validatePingNode(editForm.value[field]).valid ? '' : pingNodeErrorMessage.value
  ])
))

const hasPingNodeErrors = computed(() => Object.values(pingNodeErrors.value).some(Boolean))

const PING_NODE_DEFAULT_PLACEHOLDERS = {
  custom_ct: 'gd-ct-dualstack.ip.zstaticcdn.com',
  custom_cu: 'gd-cu-dualstack.ip.zstaticcdn.com',
  custom_cm: 'gd-cm-dualstack.ip.zstaticcdn.com',
  custom_bd: 'ip.zstaticcdn.com'
}
const PING_NODE_LABEL_KEYS = { custom_ct: 'customCt', custom_cu: 'customCu', custom_cm: 'customCm', custom_bd: 'customBd' }

const pingNodeFields = computed(() => ['custom_ct', 'custom_cu', 'custom_cm', 'custom_bd', 'node_1', 'node_2', 'node_3', 'node_4'].map(field => {
  const nodeIndex = field.startsWith('node_') ? field.slice(5) : ''
  return {
    field,
    label: props.settings[`${field}_name`] || (nodeIndex ? `Node ${nodeIndex}` : props.trans[PING_NODE_LABEL_KEYS[field]]),
    placeholder: props.settings[field] || PING_NODE_DEFAULT_PLACEHOLDERS[field] || 'host[:port] / [IPv6]:port'
  }
}))

// The node overrides are folded away unless one is set or invalid.
const pingNodesOpen = ref(false)
const hasPingNodeOverrides = () => pingNodeFields.value.some(({ field }) => String(editForm.value[field] ?? '').trim() !== '')
watch(hasPingNodeErrors, (hasErrors) => {
  if (hasErrors) pingNodesOpen.value = true
})

const billingCycleOptions = BILLING_CYCLES
const currencyOptions = CURRENCY_OPTIONS
const currencySelectOptions = computed(() => {
  const currentCurrency = String(editForm.value.currency || '').trim()
  if (!currentCurrency || currencyOptions.some(item => item.symbol === currentCurrency)) {
    return currencyOptions
  }
  return [
    { symbol: currentCurrency, nameZh: currentCurrency, nameEn: currentCurrency },
    ...currencyOptions
  ]
})

const cycleLabel = (item) => currentLang.value === 'zh' ? item.labelZh : item.labelEn
const currencyLabel = (item) => currentLang.value === 'zh'
  ? `${item.symbol} ${item.nameZh}`
  : `${item.symbol} ${item.nameEn}`

const TRAFFIC_CALC_TYPES = ['total', 'ul', 'dl', 'max']
const trafficCalcOptions = computed(() => [
  { value: 'total', label: props.trans.trafficCalcTotal },
  { value: 'ul', label: props.trans.trafficCalcUl },
  { value: 'dl', label: props.trans.trafficCalcDl },
  { value: 'max', label: props.trans.trafficCalcMax }
])

// Per-interface aliases and traffic settings are stored as two JSON objects
// keyed by interface name; the table edits them as one row per interface.
const interfaceRows = ref([])
const interfaceJsonInvalid = ref(false)
let interfaceRowKey = 0

const parseJsonObject = (value) => {
  if (value === null || value === undefined || value === '') return {}
  const parsed = typeof value === 'string' ? JSON.parse(value) : value
  if (!parsed || typeof parsed !== 'object' || Array.isArray(parsed)) throw new Error('not an object')
  return parsed
}

const loadInterfaceRows = () => {
  try {
    const aliases = parseJsonObject(editForm.value.interface_aliases)
    const settings = parseJsonObject(editForm.value.interface_traffic_settings)
    const names = [...new Set([...Object.keys(aliases), ...Object.keys(settings)])]
    interfaceRows.value = names.map(name => ({
      key: ++interfaceRowKey,
      name,
      alias: String(aliases[name] ?? ''),
      limit: settings[name]?.traffic_limit ?? '',
      calc: TRAFFIC_CALC_TYPES.includes(settings[name]?.traffic_calc_type) ? settings[name].traffic_calc_type : 'total'
    }))
    interfaceJsonInvalid.value = false
  } catch (_) {
    interfaceRows.value = []
    interfaceJsonInvalid.value = true
  }
}

const syncInterfaceJson = () => {
  if (interfaceJsonInvalid.value) return
  const aliases = {}
  const settings = {}
  for (const row of interfaceRows.value) {
    const name = String(row.name || '').trim()
    if (!name) continue
    const alias = String(row.alias || '').trim()
    if (alias) aliases[name] = alias
    const limit = row.limit === '' || row.limit === null || row.limit === undefined ? '' : Number(row.limit)
    const validLimit = limit !== '' && Number.isFinite(limit) && limit >= 0 ? limit : ''
    if (validLimit !== '' || row.calc !== 'total') {
      settings[name] = { traffic_limit: validLimit, traffic_calc_type: row.calc }
    }
  }
  editForm.value.interface_aliases = JSON.stringify(aliases)
  editForm.value.interface_traffic_settings = JSON.stringify(settings)
}

const duplicateInterfaceNames = computed(() => {
  const seen = new Set()
  const duplicates = new Set()
  for (const row of interfaceRows.value) {
    if (!row.name) continue
    if (seen.has(row.name)) duplicates.add(row.name)
    seen.add(row.name)
  }
  return duplicates
})

const addInterfaceRow = () => {
  interfaceRows.value.push({ key: ++interfaceRowKey, name: '', alias: '', limit: '', calc: 'total' })
}

const removeInterfaceRow = (index) => {
  interfaceRows.value.splice(index, 1)
}

watch(interfaceRows, syncInterfaceJson, { deep: true })

watch(
  () => [props.show, editForm.value.id],
  ([show]) => {
    if (!show) return
    loadInterfaceRows()
    pingNodesOpen.value = hasPingNodeOverrides() || hasPingNodeErrors.value
  },
  { immediate: true }
)

const normalizeTgNotifySetting = (value) => {
  if (value === true || value === 'true') return '5'
  if (value === false || value === 'false' || value === undefined || value === null || value === '') return '0'

  const minutes = Number(value)
  if (Number.isInteger(minutes) && (minutes === 0 || (minutes >= 2 && minutes <= 30))) {
    return String(minutes)
  }

  return '0'
}

const isOfflineNotifyEnabled = computed(() => normalizeTgNotifySetting(props.settings.tg_notify) !== '0')
const hasNodeNotificationOptions = computed(() => (
  !!props.settings.tg_bot_token &&
  isOfflineNotifyEnabled.value
))
const isWssReportEnabled = computed(() => props.settings.wss_report_enabled === true)

const normalizePriceInput = () => {
  editForm.value.price = normalizePrice(editForm.value.price)
}

const openDatePicker = (event) => {
  const input = event?.currentTarget
  if (typeof input?.showPicker !== 'function') return
  try {
    input.showPicker()
  } catch (_) {}
}

watch(
  () => [editForm.value.auto_renewal, editForm.value.billing_cycle, editForm.value.expire_date],
  () => {
    if (!editForm.value.auto_renewal) return
    const renewal = renewExpireDateIfNeeded(
      editForm.value.expire_date,
      editForm.value.billing_cycle,
      editForm.value.auto_renewal
    )
    if (renewal.renewed) {
      editForm.value.expire_date = renewal.expire_date
    }
  }
)

watch(
  isWssReportEnabled,
  (enabled) => {
    if (!enabled) {
      editForm.value.connection_mode = 'http'
    }
  },
  { immediate: true }
)

const emit = defineEmits(['save', 'close', 'toggle-auto-update'])

const handleAutoUpdateChange = (event) => {
  const nextValue = event.target.checked
  if (nextValue) {
    event.target.checked = false
  }
  emit('toggle-auto-update', nextValue)
}
</script>
