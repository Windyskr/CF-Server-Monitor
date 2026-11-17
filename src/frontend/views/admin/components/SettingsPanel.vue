<template>
  <div id="tab-settings" class="tab-content" :class="{ active: activeTab === 'settings' }">
    <div class="settings-layout">
      <nav class="settings-nav" :aria-label="trans.settings || 'Settings'">
        <button
          v-for="group in settingsGroups"
          :key="group.id"
          type="button"
          class="settings-nav-item"
          :class="{ active: activeGroup === group.id }"
          @click="selectGroup(group.id)"
        >
          <span class="settings-nav-icon">{{ group.icon }}</span>
          <span class="settings-nav-text">
            <span class="settings-nav-label">{{ group.label }}</span>
            <span class="settings-nav-desc">{{ group.desc }}</span>
          </span>
        </button>
      </nav>

      <div class="settings-body">
        <!-- ============ 通知渠道 ============ -->
        <section v-show="activeGroup === 'channel'" class="settings-group">
          <header class="settings-group-header">
            <h3>{{ st('channelTitle') }}</h3>
            <p>{{ st('channelDesc') }}</p>
          </header>

          <div class="settings-card">
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ st('channelType') }}</label>
                <select v-model="channelType" class="form-select">
                  <option v-for="option in channelOptions" :key="option.value" :value="option.value">{{ option.label }}</option>
                </select>
              </div>
            </div>

            <template v-if="channelType !== 'webhook'">
              <div class="form-row">
                <div class="form-group flex-1">
                  <label class="form-label">{{ currentChannel.tokenLabel }}</label>
                  <div class="password-input-wrapper">
                    <input type="text" name="tg_bot_token" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model.trim="settings.tg_bot_token" :class="['form-input', { 'secret-input-masked': !passwordVisible.tgBotToken }]" :placeholder="currentChannel.placeholder">
                    <button type="button" class="password-toggle" @click="$emit('toggle-password', 'tgBotToken')">
                      {{ passwordVisible.tgBotToken ? '🙈' : '👁️' }}
                    </button>
                  </div>
                  <p class="form-hint">{{ currentChannel.hint }}</p>
                  <p v-if="channelTokenMismatch" class="form-hint warning">⚠️ {{ st('channelMismatch').replace('{type}', detectedChannelLabel) }}</p>
                </div>

                <div v-if="currentChannel.chatId" class="form-group flex-1">
                  <label class="form-label">{{ currentChannel.chatIdLabel }}</label>
                  <div class="password-input-wrapper">
                    <input type="text" name="tg_chat_id" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model.trim="settings.tg_chat_id" :class="['form-input', { 'secret-input-masked': !passwordVisible.tgChatId }]" :placeholder="currentChannel.chatIdPlaceholder">
                    <button type="button" class="password-toggle" @click="$emit('toggle-password', 'tgChatId')">
                      {{ passwordVisible.tgChatId ? '🙈' : '👁️' }}
                    </button>
                  </div>
                  <p class="form-hint">{{ currentChannel.chatIdHint }}</p>
                </div>
              </div>
            </template>

            <template v-else>
              <div class="form-row">
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.webhookUrl || 'Webhook URL' }}</label>
                  <div class="password-input-wrapper">
                    <input type="text" name="notification_webhook_url" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.notification_webhook_url" :class="['form-input', { 'secret-input-masked': !passwordVisible.notificationWebhookUrl }]" placeholder="https://example.com/webhook">
                    <button type="button" class="password-toggle" @click="$emit('toggle-password', 'notificationWebhookUrl')">
                      {{ passwordVisible.notificationWebhookUrl ? '🙈' : '👁️' }}
                    </button>
                  </div>
                </div>

                <div class="form-group">
                  <label class="form-label">{{ trans.webhookMethod || 'Method' }}</label>
                  <select v-model="settings.notification_webhook_method" class="form-select">
                    <option value="POST">POST</option>
                    <option value="GET">GET</option>
                  </select>
                </div>

                <div v-if="settings.notification_webhook_method !== 'GET'" class="form-group">
                  <label class="form-label">{{ trans.webhookFormat || 'Body Format' }}</label>
                  <select v-model="settings.notification_webhook_format" class="form-select">
                    <option value="json">JSON</option>
                    <option value="form">x-www-form-urlencoded</option>
                    <option value="text">Text</option>
                  </select>
                </div>
              </div>

              <div class="form-row">
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.webhookHeaders || 'Headers' }}</label>
                  <textarea v-model="settings.notification_webhook_headers" class="form-textarea" rows="4" placeholder='{"Authorization":"Bearer your-token"}'></textarea>
                </div>

                <div class="form-group flex-1">
                  <label class="form-label">
                    {{ settings.notification_webhook_method === 'GET' ? (trans.webhookParams || 'Query Params') : (trans.webhookBody || 'Body') }}
                  </label>
                  <textarea v-model="settings.notification_webhook_body" class="form-textarea" rows="6" placeholder='{"title":"{{emoji}} {{event}}","content":"{{notification}}"}'></textarea>
                  <p class="form-hint">{{ st('webhookBodyHint') }}</p>
                </div>
              </div>
            </template>

            <div class="settings-actions">
              <button type="button" @click="$emit('send-test-notification')" class="btn btn-primary" :disabled="testNotificationLoading">{{ testNotificationLoading ? '⏳' : '📨' }} {{ trans.sendTestNotification }}</button>
              <span class="form-hint">{{ st('testHint') }}</span>
            </div>
          </div>

          <details class="settings-card settings-collapsible">
            <summary>{{ st('templateTitle') }} <span class="settings-collapsible-desc">{{ st('templateDesc') }}</span></summary>
            <div class="form-group">
              <textarea v-model="settings.notification_template" class="form-textarea" rows="5" placeholder="{{emoji}}【CF Server Monitor】{{event}}\n\n{{message}}\n\n{{time}}"></textarea>
              <p class="form-hint">{{ st('templateVars') }}</p>
            </div>
          </details>
        </section>

        <!-- ============ 告警与报告 ============ -->
        <section v-show="activeGroup === 'alerts'" class="settings-group">
          <header class="settings-group-header">
            <h3>{{ st('alertsTitle') }}</h3>
            <p>{{ st('alertsDesc') }}</p>
          </header>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('scheduleTitle') }}</h4>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.notificationTimezone || 'Notification Timezone' }}</label>
                <select v-model="selectedNotificationTimezone" class="form-select">
                  <option v-for="timezone in commonNotificationTimezones" :key="timezone" :value="timezone">{{ timezone }}</option>
                  <option :value="CUSTOM_NOTIFICATION_TIMEZONE_VALUE">{{ trans.custom || 'Custom' }}</option>
                </select>
                <input v-if="showCustomNotificationTimezone" type="text" v-model.trim="settings.notification_timezone" class="form-input mt-2" placeholder="Asia/Shanghai">
              </div>
              <div class="form-group flex-1">
                <label class="form-label">{{ st('dailyHour') }}</label>
                <select v-model="settings.expire_notification_time" class="form-select">
                  <option v-for="hour in expireNotificationHourOptions" :key="hour" :value="hour">{{ String(hour).padStart(2, '0') }}:00</option>
                </select>
              </div>
            </div>
            <p class="form-hint">{{ st('dailyHourHint') }}</p>
          </div>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('serverAlertsTitle') }}</h4>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ st('offlineLabel') }}</label>
                <select v-model="settings.tg_notify" class="form-select">
                  <option v-for="option in offlineNotifyOptions" :key="option.value" :value="option.value">{{ option.label }}</option>
                </select>
                <p class="form-hint">{{ st('offlineHint') }}</p>
              </div>

              <div class="form-group flex-1">
                <label class="form-label">{{ st('expireLabel') }}</label>
                <select v-model="selectedExpireReminder" class="form-select">
                  <option v-for="option in expireReminderOptions" :key="option.value" :value="option.value">{{ option.label }}</option>
                  <option :value="CUSTOM_EXPIRE_REMINDER_VALUE">{{ st('customDays') }}</option>
                </select>
                <div v-if="showCustomExpireReminder" class="input-with-unit mt-2">
                  <input type="number" v-model.number="settings.expire_reminder" @focus="markCustomExpireReminder" class="form-input" min="1" :max="EXPIRE_REMINDER_DAYS_MAX" step="1">
                  <span class="input-unit">{{ st('daysUnit') }}</span>
                </div>
                <p class="form-hint">{{ st('expireHint') }}</p>
              </div>
            </div>
          </div>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('trafficReportTitle') }}</h4>
            <div class="form-row">
              <div class="form-group flex-1 checkbox-item">
                <input type="checkbox" id="cfg_traffic_report_enabled" v-model="settings.traffic_report_enabled">
                <label for="cfg_traffic_report_enabled"><b>{{ st('trafficReportEnable') }}</b></label>
              </div>
            </div>
            <p class="form-hint">{{ st('trafficReportHint') }}</p>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.notificationDashboardUrl || 'Dashboard URL' }}</label>
                <input type="url" v-model.trim="settings.notification_dashboard_url" class="form-input" placeholder="https://monitor.example.com">
                <p class="form-hint">{{ trans.notificationDashboardUrlTip }}</p>
              </div>
            </div>
            <div class="settings-actions">
              <button type="button" class="btn" :disabled="trafficBaselineRebuilding" @click="$emit('rebuild-traffic-baselines')">
                {{ trafficBaselineRebuilding ? '⏳' : '↻' }}
                {{ trafficBaselineRebuilding ? (trans.rebuildingTrafficBaselines || 'Initializing...') : st('rebuildBaselines') }}
              </button>
              <span class="form-hint">{{ st('rebuildBaselinesHint') }}</span>
            </div>
            <p class="form-hint warning">⚠️ {{ trans.trafficReportRestartWarning }}</p>
          </div>

          <div class="settings-card">
            <div class="settings-card-title-row">
              <h4 class="settings-card-title">{{ trans.resourceAlert }} <span class="resource-alert-count">[{{ resourceAlertRules.length }}]</span></h4>
              <button type="button" class="btn btn-primary btn-sm" @click="addResourceAlertRule">+ {{ trans.resourceAlertAddRule }}</button>
            </div>
            <p class="form-hint">{{ st('resourceAlertHint') }}</p>

            <div v-if="resourceAlertRules.length === 0" class="resource-alert-empty text-muted text-sm">
              {{ trans.resourceAlertEmpty }}
            </div>

            <div v-for="(rule, index) in resourceAlertRules" :key="rule.id || index" class="resource-alert-rule">
              <div class="resource-alert-rule-title">
                <span>{{ trans.resourceAlertRule }} #{{ index + 1 }}</span>
                <button type="button" class="btn btn-red btn-sm" @click="removeResourceAlertRule(index)">{{ trans.delete }}</button>
              </div>

              <div class="form-group">
                <label class="form-label">{{ trans.resourceAlertRuleName }}</label>
                <input :ref="el => setResourceAlertRuleNameInput(rule.id, el)" type="text" v-model="rule.name" class="form-input" :placeholder="trans.resourceAlertRuleNamePlaceholder">
              </div>

              <div class="alert-sentence">
                <span>{{ st('ruleWhen') }}</span>
                <select v-model="rule.metric" class="form-select" @change="normalizeRuleThreshold(rule)">
                  <option v-for="option in resourceAlertMetricOptions" :key="option.value" :value="option.value">{{ option.label }}</option>
                </select>
                <span>{{ st('ruleWithin') }}</span>
                <select v-model="rule.intervalMinutes" class="form-select">
                  <option v-for="option in resourceAlertIntervalOptions" :key="option.value" :value="option.value">{{ option.label }}</option>
                </select>
                <span>{{ st('ruleInside') }}</span>
                <select v-model="rule.mode" class="form-select">
                  <option value="average">{{ st('ruleModeAverage') }}</option>
                  <option value="continuous">{{ st('ruleModeContinuous') }}</option>
                </select>
                <span>{{ st('ruleExceeds') }}</span>
                <div class="input-with-unit">
                  <input type="number" min="0" :max="resourceAlertThresholdMax(rule.metric)" step="1" v-model="rule.threshold" class="form-input alert-threshold" :placeholder="resourceAlertThresholdPlaceholder(rule.metric)">
                  <span class="input-unit">{{ isNetworkAlertMetric(rule.metric) ? 'Mbps' : '%' }}</span>
                </div>
                <span>{{ st('ruleThen') }}</span>
              </div>

              <div class="form-group">
                <label class="form-label">{{ st('ruleServers') }}</label>
                <details class="resource-alert-server-dropdown">
                  <summary class="form-select resource-alert-server-summary">
                    {{ resourceAlertServerSelectLabel(rule) }}
                  </summary>
                  <div class="resource-alert-server-menu">
                    <label v-for="server in resourceAlertServerOptions" :key="server.id" class="resource-alert-server-option">
                      <input type="checkbox" :checked="resourceAlertRuleHasServer(rule, server.id)" :disabled="isLastResourceAlertRuleServer(rule, server.id)" @change="toggleResourceAlertRuleServer(rule, server.id, $event.target.checked)">
                      <span>{{ server.name }}</span>
                    </label>
                  </div>
                </details>
              </div>
            </div>
          </div>
        </section>

        <!-- ============ 站点与显示 ============ -->
        <section v-show="activeGroup === 'site'" class="settings-group">
          <header class="settings-group-header">
            <h3>{{ st('siteTitle') }}</h3>
            <p>{{ st('siteDesc') }}</p>
          </header>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('visibilityTitle') }}</h4>
            <div class="settings-check-grid">
              <label class="checkbox-item"><input type="checkbox" v-model="settings.is_public"><span><b>{{ trans.publicAccess }}</b><small>{{ st('publicHint') }}</small></span></label>
              <label class="checkbox-item"><input type="checkbox" v-model="settings.show_price"><span>{{ trans.showPrice }}</span></label>
              <label class="checkbox-item"><input type="checkbox" v-model="settings.show_expire"><span>{{ trans.showExpire }}</span></label>
              <label class="checkbox-item"><input type="checkbox" v-model="settings.show_tf"><span>{{ trans.showTf }}</span></label>
              <label class="checkbox-item"><input type="checkbox" v-model="settings.show_three_net_details"><span>{{ trans.showThreeNetDetails }}</span></label>
            </div>
          </div>

          <div v-if="currentOrigin === selectedApiBase" class="settings-card">
            <h4 class="settings-card-title">{{ st('appearanceTitle') }}</h4>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.siteTitle }}</label>
                <input type="text" v-model="settings.site_title" class="form-input" :placeholder="'Cloudflare Server Monitor'">
              </div>
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.displayMode }}</label>
                <select v-model="settings.display_mode" class="form-select">
                  <option value="bar">{{ trans.displayModeBar }}</option>
                  <option value="ring">{{ trans.displayModeRing }}</option>
                  <option value="table">{{ trans.displayModeTable }}</option>
                </select>
              </div>
            </div>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.defaultTheme }}</label>
                <select v-model="settings.preferred_theme" class="form-select">
                  <option value="auto">{{ trans.themeAuto }}</option>
                  <option value="dark">{{ trans.themeDark }}</option>
                  <option value="light">{{ trans.themeLight }}</option>
                </select>
              </div>
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.defaultLanguage }}</label>
                <select v-model="settings.default_language" class="form-select">
                  <option value="auto">{{ trans.languageAuto }}</option>
                  <option value="zh">{{ trans.languageChinese }}</option>
                  <option value="en">{{ trans.languageEnglish }}</option>
                </select>
              </div>
            </div>
          </div>

          <div v-if="currentOrigin === selectedApiBase" class="settings-card">
            <h4 class="settings-card-title">{{ st('imagesTitle') }}</h4>
            <p class="form-hint">{{ trans.remoteImageTip }}</p>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.bgImage }}</label>
                <div class="flex" style="gap:8px;">
                  <input type="text" v-model="settings.custom_bg" class="form-input flex-1" placeholder="https://...">
                  <div class="upload-btn-wrapper">
                    <button class="btn btn-margin-0">📁 {{ trans.upload }}</button>
                    <input type="file" accept="image/*" @change="$emit('upload-bg', $event)">
                  </div>
                </div>
                <img v-if="settings.custom_bg" :src="settings.custom_bg" class="bg-preview">
              </div>
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.mobileBgImage }}</label>
                <div class="flex" style="gap:8px;">
                  <input type="text" v-model="settings.custom_bg_mobile" class="form-input flex-1" placeholder="https://...">
                  <div class="upload-btn-wrapper">
                    <button class="btn btn-margin-0">📁 {{ trans.upload }}</button>
                    <input type="file" accept="image/*" @change="$emit('upload-bg-mobile', $event)">
                  </div>
                </div>
                <img v-if="settings.custom_bg_mobile" :src="settings.custom_bg_mobile" class="bg-preview">
              </div>
            </div>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.favicon }}</label>
                <div class="flex" style="gap:8px;">
                  <input type="text" v-model="settings.favicon" class="form-input flex-1" placeholder="https://...">
                  <div class="upload-btn-wrapper">
                    <button class="btn btn-margin-0">📁 {{ trans.upload }}</button>
                    <input type="file" accept="image/*,.ico" @change="$emit('upload-favicon', $event)">
                  </div>
                </div>
                <img v-if="settings.favicon" :src="settings.favicon" class="bg-preview">
              </div>
            </div>
          </div>
        </section>

        <!-- ============ 数据采集 ============ -->
        <section v-show="activeGroup === 'collect'" class="settings-group">
          <header class="settings-group-header">
            <h3>{{ st('collectTitle') }}</h3>
            <p>{{ st('collectDesc') }}</p>
          </header>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('realtimeTitle') }}</h4>
            <div class="form-row">
              <div class="form-group flex-1 checkbox-item">
                <input type="checkbox" id="cfg_wss_report_enabled" v-model="settings.wss_report_enabled">
                <label for="cfg_wss_report_enabled"><b>{{ st('wssEnable') }}</b></label>
              </div>
            </div>
            <p class="form-hint">{{ st('wssHint') }}</p>

            <div v-if="settings.wss_report_enabled" class="wss-schedule">
              <div class="wss-schedule-header">
                <div>
                  <div class="form-label wss-schedule-title">{{ st('wssHours') }}</div>
                  <div class="wss-schedule-meta">
                    {{ localTimezoneLabel }} · {{ wssReportHours.length }}/24 {{ trans.hoursSelected }} · {{ trans.agentWssMinVersion }}
                  </div>
                </div>
                <div class="wss-schedule-actions">
                  <button type="button" class="btn btn-sm" @click="selectAllWssReportHours">{{ trans.selectAll }}</button>
                  <button type="button" class="btn btn-sm" @click="clearWssReportHours">{{ trans.clear }}</button>
                </div>
              </div>
              <div class="wss-hour-grid" role="group" :aria-label="trans.wssReportHours">
                <label v-for="hour in 24" :key="hour - 1" class="wss-hour-option" :title="formatWssHourRange(hour - 1)">
                  <input type="checkbox" :checked="isLocalWssReportHourSelected(hour - 1)" @change="toggleLocalWssReportHour(hour - 1, $event.target.checked)">
                  <span>{{ String(hour - 1).padStart(2, '0') }}</span>
                </label>
              </div>
              <p class="form-hint">{{ st('wssHoursHint') }}</p>
            </div>

            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ st('wsTimeoutLabel') }}</label>
                <select v-model.number="settings.frontend_ws_timeout_minutes" class="form-select">
                  <option v-for="option in wsTimeoutOptions" :key="option.value" :value="option.value">{{ option.label }}</option>
                </select>
                <p class="form-hint">{{ st('wsTimeoutHint') }}</p>
              </div>
              <div class="form-group flex-1">
                <label class="form-label">{{ st('historyPointsLabel') }}</label>
                <select v-model="settings.long_history_points" class="form-select">
                  <option v-for="option in historyPointOptions" :key="option.value" :value="option.value">{{ option.label }}</option>
                </select>
                <p class="form-hint">{{ st('historyPointsHint') }}</p>
              </div>
            </div>
          </div>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ trans.pingNodes }}</h4>
            <p class="form-hint">{{ st('pingHint') }}</p>
            <div class="ping-node-list">
              <div v-for="node in pingNodeFieldList" :key="node.field" class="ping-node-item">
                <div class="ping-node-fields">
                  <input type="text" v-model.trim="settings[`${node.field}_name`]" class="form-input ping-node-name" :placeholder="st('pingName')">
                  <span class="ping-node-arrow">→</span>
                  <input type="text" v-model.trim="settings[node.field]" :class="['form-input', { 'input-invalid': pingNodeErrors[node.field] }]" :placeholder="node.placeholder">
                </div>
                <p v-if="pingNodeErrors[node.field]" class="text-red text-sm mt-1">{{ pingNodeErrors[node.field] }}</p>
              </div>
            </div>
          </div>
        </section>

        <!-- ============ 登录与安全 ============ -->
        <section v-show="activeGroup === 'security'" class="settings-group">
          <header class="settings-group-header">
            <h3>{{ st('securityTitle') }}</h3>
            <p>{{ st('securityDesc') }}</p>
          </header>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('adminTitle') }}</h4>
            <div class="form-group">
              <label class="form-label">{{ trans.username }}</label>
              <input type="text" name="settings_admin_user" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.username" class="form-input" :placeholder="trans.usernamePlaceholder">
            </div>

            <div class="inline-help-action mb-3">
              <button type="button" class="btn btn-sm" @click="$emit('toggle-admin-password-change')">
                {{ changeAdminPassword ? trans.cancelPasswordChange : trans.changePassword }}
              </button>
            </div>
            <p v-if="changeAdminPassword" class="form-hint">{{ st('passwordHint') }}</p>

            <div v-if="changeAdminPassword" class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.password }}</label>
                <div class="password-input-wrapper">
                  <input :type="passwordVisible.password ? 'text' : 'password'" name="settings_admin_passphrase" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.password" class="form-input" :placeholder="settings.password_configured ? '••••••••' : ''">
                  <button type="button" class="password-toggle" @click="$emit('toggle-password', 'password')">
                    {{ passwordVisible.password ? '🙈' : '👁️' }}
                  </button>
                </div>
              </div>
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.confirmPassword }}</label>
                <div class="password-input-wrapper">
                  <input :type="passwordVisible.confirmPassword ? 'text' : 'password'" name="settings_admin_passphrase_confirm" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.confirm_password" class="form-input" :placeholder="settings.password_configured ? '••••••••' : ''">
                  <button type="button" class="password-toggle" @click="$emit('toggle-password', 'confirmPassword')">
                    {{ passwordVisible.confirmPassword ? '🙈' : '👁️' }}
                  </button>
                </div>
              </div>
            </div>
          </div>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('githubTitle') }}</h4>
            <div class="form-group checkbox-item">
              <input type="checkbox" id="cfg_github_oauth_enabled" v-model="settings.github_oauth_enabled">
              <label for="cfg_github_oauth_enabled"><b>{{ trans.enableGithubOAuth }}</b></label>
            </div>
            <p class="form-hint">{{ trans.githubOAuthTip }}</p>

            <template v-if="settings.github_oauth_enabled">
              <div class="form-row">
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.githubClientId }}</label>
                  <input type="text" name="github_client_id" autocomplete="off" v-model.trim="settings.github_client_id" class="form-input" :placeholder="trans.githubClientIdPlaceholder">
                </div>
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.githubClientSecret }}</label>
                  <div class="password-input-wrapper">
                    <input type="text" name="github_client_secret" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.github_client_secret" :class="['form-input', { 'secret-input-masked': !passwordVisible.githubClientSecret }]" :placeholder="settings.github_client_secret_configured ? '••••••••' : ''">
                    <button type="button" class="password-toggle" @click="$emit('toggle-password', 'githubClientSecret')">
                      {{ passwordVisible.githubClientSecret ? '🙈' : '👁️' }}
                    </button>
                  </div>
                </div>
              </div>

              <div class="form-group">
                <label class="form-label">{{ trans.githubBoundAccount }}</label>
                <div class="inline-help-action">
                  <span class="text-sm">{{ settings.github_user_id ? `ID: ${settings.github_user_id}` : trans.githubNotBound }}</span>
                  <button type="button" class="btn btn-sm" :disabled="!canBindGithub || githubBindingLoading" @click="$emit('bind-github-account')">
                    {{ githubBindingLoading ? '⏳' : (settings.github_user_id ? trans.rebindGithubAccount : trans.bindGithubAccount) }}
                  </button>
                  <HelpTooltip :text="canBindGithub ? trans.githubBindingTip : trans.githubSaveBeforeBinding" />
                </div>
              </div>

              <div class="form-group">
                <label class="form-label">{{ trans.githubCallbackUrl }}</label>
                <div class="flex-center-gap-sm">
                  <input type="text" class="form-input flex-1 github-callback-input" :value="githubCallbackUrl" readonly>
                  <button type="button" class="btn btn-sm" :aria-label="githubCallbackCopied ? trans.copied : trans.copy" :title="githubCallbackCopied ? trans.copied : trans.copy" @click="copyGithubCallbackUrl">
                    {{ githubCallbackCopied ? `✅ ${trans.copied}` : `📋 ${trans.copy}` }}
                  </button>
                </div>
                <p class="form-hint">{{ trans.githubCallbackUrlTip }}</p>
              </div>
            </template>
          </div>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('turnstileTitle') }}</h4>
            <p class="form-hint">{{ st('turnstileHint') }}</p>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ st('turnstileMode') }}</label>
                <select v-model="turnstileMode" class="form-select">
                  <option value="off">{{ st('turnstileOff') }}</option>
                  <option value="login">{{ st('turnstileLogin') }}</option>
                  <option value="global">{{ st('turnstileGlobal') }}</option>
                </select>
              </div>
            </div>
            <div v-if="turnstileMode !== 'off'" class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.turnstileSiteKey }}</label>
                <input type="text" name="turnstile_site_key" autocomplete="off" v-model="settings.turnstile_site_key" class="form-input" :placeholder="trans.turnstileSiteKeyPlaceholder">
              </div>
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.turnstileSecretKey }}</label>
                <div class="password-input-wrapper">
                  <input type="text" name="turnstile_secret_key" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.turnstile_secret_key" :class="['form-input', { 'secret-input-masked': !passwordVisible.turnstileSecret }]" :placeholder="trans.turnstileSecretKeyPlaceholder">
                  <button type="button" class="password-toggle" @click="$emit('toggle-password', 'turnstileSecret')">
                    {{ passwordVisible.turnstileSecret ? '🙈' : '👁️' }}
                  </button>
                </div>
              </div>
            </div>
          </div>

        </section>

        <!-- ============ 高级 ============ -->
        <section v-show="activeGroup === 'advanced'" class="settings-group">
          <header class="settings-group-header">
            <h3>{{ st('advancedTitle') }}</h3>
            <p>{{ st('advancedDesc') }}</p>
          </header>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ st('cloudflareTitle') }}</h4>
            <p class="form-hint">{{ st('cloudflareHint') }}</p>
            <div class="form-row">
              <div class="form-group flex-1">
                <label class="form-label">{{ trans.cloudflareAccountId }}</label>
                <input type="text" name="cloudflare_account_id" autocomplete="off" v-model="settings.cloudflare_account_id" class="form-input" :placeholder="trans.cloudflareAccountIdPlaceholder">
              </div>
              <div class="form-group flex-1">
                <label class="form-label">Cloudflare API Token</label>
                <div class="password-input-wrapper">
                  <input type="text" name="cloudflare_token" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.cloudflare_token" :class="['form-input', { 'secret-input-masked': !passwordVisible.cloudflareToken }]" :placeholder="trans.cloudflareTokenPlaceholder">
                  <button type="button" class="password-toggle" @click="$emit('toggle-password', 'cloudflareToken')">
                    {{ passwordVisible.cloudflareToken ? '🙈' : '👁️' }}
                  </button>
                </div>
                <p class="form-hint">{{ trans.cloudflareTokenTip }}</p>
              </div>
            </div>
            <div class="settings-actions">
              <button type="button" @click="$emit('query-d1-usage')" class="btn" :disabled="d1UsageLoading">{{ d1UsageLoading ? '⏳' : '🔍' }} {{ trans.queryD1Quota }}</button>
            </div>
          </div>

          <div class="settings-card">
            <h4 class="settings-card-title">{{ trans.jwtSecret }}</h4>
            <p class="form-hint">{{ trans.jwtSecretTip }}</p>
            <p class="form-hint warning">⚠️ {{ st('jwtWarning') }}</p>
            <div class="password-input-wrapper">
              <input type="text" name="jwt_secret" autocomplete="off" data-lpignore="true" data-1p-ignore="true" data-bwignore="true" data-form-type="other" v-model="settings.jwt_secret" :class="['form-input', { 'secret-input-masked': !passwordVisible.jwtSecret }]" placeholder="••••••••••••••••••••••••••••••••">
              <button type="button" class="password-toggle" @click="$emit('toggle-password', 'jwtSecret')">
                {{ passwordVisible.jwtSecret ? '🙈' : '👁️' }}
              </button>
            </div>
          </div>
          <template v-if="currentOrigin === selectedApiBase">
            <div class="settings-card">
              <h4 class="settings-card-title">{{ st('customCodeTitle') }}</h4>
              <p class="form-hint warning">⚠️ {{ trans.cspWarning }}</p>
              <div class="form-row">
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.customHead }}</label>
                  <textarea v-model="settings.custom_head" class="form-textarea" rows="3" placeholder="<link rel='stylesheet' href='...'"></textarea>
                </div>
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.customScript }}</label>
                  <textarea v-model="settings.custom_script" class="form-textarea" rows="3" placeholder="console.log('Hello');"></textarea>
                </div>
              </div>
              <div class="form-row">
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.cspStatic }}</label>
                  <input type="text" v-model="settings.csp_static" class="form-input" placeholder="https://unpkg.com,https://cdn.jsdelivr.net" @blur="validateCspField('csp_static')">
                  <p v-if="cspErrors.csp_static" class="text-danger text-sm">{{ cspErrors.csp_static }}</p>
                  <p class="form-hint">{{ trans.cspStaticTip }}</p>
                </div>
                <div class="form-group flex-1">
                  <label class="form-label">{{ trans.cspApi }}</label>
                  <input type="text" v-model="settings.csp_api" class="form-input" placeholder="https://api.example.com" @blur="validateCspField('csp_api')">
                  <p v-if="cspErrors.csp_api" class="text-danger text-sm">{{ cspErrors.csp_api }}</p>
                  <p class="form-hint">{{ trans.cspApiTip }}</p>
                </div>
              </div>
            </div>

            <div class="settings-card">
              <h4 class="settings-card-title">{{ trans.themeOptions }}</h4>
              <p class="form-hint">{{ trans.themeOptionsTip }}</p>
              <textarea v-model="settings.theme_options" class="form-textarea" rows="5" placeholder='{"mikus":1}'></textarea>
            </div>
          </template>
        </section>
      </div>
    </div>

    <div class="settings-savebar" :class="{ dirty: isDirty }">
      <span class="settings-savebar-status">
        <template v-if="saving">⏳ {{ trans.saving }}</template>
        <template v-else-if="hasPingNodeErrors">⚠️ {{ st('pingInvalid') }}</template>
        <template v-else-if="isDirty">● {{ st('unsaved') }}</template>
        <template v-else>✓ {{ st('saved') }}</template>
      </span>
      <button @click="$emit('save-settings')" class="btn btn-primary" :disabled="saving || hasPingNodeErrors">{{ saving ? '⏳' : '💾' }} {{ saving ? trans.saving : trans.saveConfig }}</button>
    </div>
  </div>
</template>

<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue'
import HelpTooltip from '../../../components/HelpTooltip.vue'
import { FRONTEND_WS_TIMEOUT_MINUTES_MAX, HISTORY } from '../../../utils/constants.js'
import { copyTextToClipboard } from '../../../utils/clipboard.js'
import { currentLang } from '../../../utils/i18n.js'
import { PING_NODE_FIELDS, validatePingNode } from '../../../utils/pingNode.js'

const props = defineProps({
  trans: { type: Object, required: true },
  settings: { type: Object, required: true },
  servers: { type: Array, default: () => [] },
  passwordVisible: { type: Object, required: true },
  activeTab: { type: String, default: 'settings' },
  selectedApiBase: { type: String, default: '' },
  currentOrigin: { type: String, default: '' },
  saving: { type: Boolean, default: false },
  changeAdminPassword: { type: Boolean, default: false },
  testNotificationLoading: { type: Boolean, default: false },
  d1UsageLoading: { type: Boolean, default: false },
  trafficBaselineRebuilding: { type: Boolean, default: false },
  githubBindingLoading: { type: Boolean, default: false }
})

const githubCallbackUrl = computed(() => {
  try {
    return new URL('/auth/github/callback', props.selectedApiBase || props.currentOrigin).toString()
  } catch (_) {
    return '/auth/github/callback'
  }
})

const githubCallbackCopied = ref(false)
let githubCallbackCopiedTimer

const copyGithubCallbackUrl = async () => {
  const copied = await copyTextToClipboard(githubCallbackUrl.value)
  if (!copied) {
    emit('alert-message', props.trans.httpsRequired)
    return
  }

  githubCallbackCopied.value = true
  clearTimeout(githubCallbackCopiedTimer)
  githubCallbackCopiedTimer = setTimeout(() => {
    githubCallbackCopied.value = false
  }, 1500)
}

const canBindGithub = computed(() => Boolean(
  String(props.settings.github_client_id || '').trim() &&
  props.settings.github_client_secret_configured
))

const emit = defineEmits([
  'toggle-password', 'toggle-admin-password-change',
  'save-settings', 'upload-bg', 'upload-bg-mobile', 'upload-favicon',
  'send-test-notification', 'query-d1-usage', 'rebuild-traffic-baselines', 'bind-github-account',
  'alert-message'
])

const commonNotificationTimezones = [
  'UTC',
  'Asia/Shanghai',
  'Asia/Hong_Kong',
  'Asia/Tokyo',
  'Asia/Singapore',
  'Europe/London',
  'Europe/Berlin',
  'America/New_York',
  'America/Los_Angeles'
]
const EXPIRE_REMINDER_DAYS_MAX = 365
const CUSTOM_EXPIRE_REMINDER_VALUE = '__custom__'
const CUSTOM_NOTIFICATION_TIMEZONE_VALUE = '__custom__'
const manualCustomExpireReminder = ref(false)
const manualCustomNotificationTimezone = ref(false)
const isCommonNotificationTimezone = (value) => commonNotificationTimezones.includes(String(value || '').trim())

const isPresetExpireReminder = (value) => {
  const reminder = String(value ?? '').trim()
  return reminder === '0' || (Number.isInteger(Number(reminder)) && Number(reminder) >= 1 && Number(reminder) <= 7)
}

const selectedExpireReminder = computed({
  get: () => {
    const currentReminder = String(props.settings.expire_reminder ?? '').trim()
    if (manualCustomExpireReminder.value || !isPresetExpireReminder(currentReminder)) {
      return CUSTOM_EXPIRE_REMINDER_VALUE
    }
    return currentReminder || '0'
  },
  set: (value) => {
    if (value === CUSTOM_EXPIRE_REMINDER_VALUE) {
      manualCustomExpireReminder.value = true
      const currentReminder = Number(props.settings.expire_reminder)
      if (!Number.isInteger(currentReminder) || currentReminder < 1 || currentReminder > EXPIRE_REMINDER_DAYS_MAX) {
        props.settings.expire_reminder = '8'
      }
      return
    }
    manualCustomExpireReminder.value = false
    props.settings.expire_reminder = value
  }
})

const showCustomExpireReminder = computed(() => selectedExpireReminder.value === CUSTOM_EXPIRE_REMINDER_VALUE)

const markCustomExpireReminder = () => {
  manualCustomExpireReminder.value = true
}

// Reset the local custom-mode flag only when switching to a different settings
// object. Changes made while typing must not collapse the input after the
// first digit (for example, typing "12" briefly produces "1").
watch(
  () => props.settings,
  () => {
    manualCustomExpireReminder.value = false
  }
)

const selectedNotificationTimezone = computed({
  get: () => {
    const currentTimezone = String(props.settings.notification_timezone || '').trim()
    if (manualCustomNotificationTimezone.value || (currentTimezone && !isCommonNotificationTimezone(currentTimezone))) {
      return CUSTOM_NOTIFICATION_TIMEZONE_VALUE
    }
    return currentTimezone || 'UTC'
  },
  set: (value) => {
    if (value === CUSTOM_NOTIFICATION_TIMEZONE_VALUE) {
      manualCustomNotificationTimezone.value = true
      return
    }
    manualCustomNotificationTimezone.value = false
    props.settings.notification_timezone = value
  }
})

const showCustomNotificationTimezone = computed(() => selectedNotificationTimezone.value === CUSTOM_NOTIFICATION_TIMEZONE_VALUE)

watch(
  () => props.settings.notification_timezone,
  (value) => {
    if (!String(value || '').trim() || isCommonNotificationTimezone(value)) {
      manualCustomNotificationTimezone.value = false
    }
  }
)

const expireNotificationHourOptions = Array.from({ length: 24 }, (_, hour) => String(hour))

const cspErrors = reactive({
  csp_static: '',
  csp_api: ''
})

const offlineNotifyOptions = computed(() => [
  { value: '0', label: st('offlineOff') },
  ...[3, 4, 5, 6, 7, 8, 9, 10, 15, 20, 30].map(minutes => ({
    value: String(minutes),
    label: st('offlineMinutes').replace('{minutes}', minutes)
  }))
])

const expireReminderOptions = computed(() => [
  { value: '0', label: st('expireOff') },
  ...Array.from({ length: 7 }, (_, index) => ({
    value: String(index + 1),
    label: st('expireDays').replace('{days}', index + 1)
  }))
])

const longHistoryPointOptions = computed(() => (
  HISTORY.LONG_RANGE_POINT_OPTIONS.map(points => ({
    value: String(points),
    label: props.trans.historyPointCount
      ? props.trans.historyPointCount.replace('{points}', points)
      : `${points} points`
  }))
))

const wssReportHours = computed(() => {
  const source = Array.isArray(props.settings.wss_report_hours)
    ? props.settings.wss_report_hours
    : Array.from({ length: 24 }, (_, hour) => hour)
  return source
    .map(hour => Number(hour))
    .filter(hour => Number.isInteger(hour) && hour >= 0 && hour <= 23)
    .filter((hour, index, hours) => hours.indexOf(hour) === index)
    .sort((a, b) => a - b)
})

const localHourToUtcHour = hour => new Date(2000, 0, 1, hour, 0, 0, 0).getUTCHours()

const localTimezoneLabel = computed(() => {
  const timezone = Intl.DateTimeFormat().resolvedOptions().timeZone
  const localTime = props.trans.localTime || 'Local time'
  return timezone ? `${localTime} · ${timezone}` : localTime
})

const isLocalWssReportHourSelected = hour => wssReportHours.value.includes(localHourToUtcHour(hour))

const toggleLocalWssReportHour = (hour, checked) => {
  const utcHour = localHourToUtcHour(hour)
  const selected = new Set(wssReportHours.value)
  if (checked) selected.add(utcHour)
  else selected.delete(utcHour)
  props.settings.wss_report_hours = Array.from(selected).sort((a, b) => a - b)
}

const selectAllWssReportHours = () => {
  props.settings.wss_report_hours = Array.from({ length: 24 }, (_, hour) => hour)
}

const clearWssReportHours = () => {
  props.settings.wss_report_hours = []
}

const formatWssHourRange = hour => {
  const utcHour = localHourToUtcHour(hour)
  const localHourText = String(hour).padStart(2, '0')
  const utcHourText = String(utcHour).padStart(2, '0')
  return `${localHourText}:00-${localHourText}:59 ${props.trans.localTime} (${utcHourText}:00-${utcHourText}:59 UTC)`
}

const notificationChannel = computed({
  get: () => props.settings.notification_webhook_enabled ? 'webhook' : 'builtin',
  set: (value) => {
    props.settings.notification_webhook_enabled = value === 'webhook'
  }
})

const ensureResourceAlertRules = () => {
  if (!Array.isArray(props.settings.resource_alert_rules)) {
    props.settings.resource_alert_rules = []
  }
  const serverIds = props.servers.map(server => String(server.id || '').trim()).filter(Boolean)
  if (serverIds.length > 0) {
    for (const rule of props.settings.resource_alert_rules) {
      if (!Array.isArray(rule.servers) || rule.servers.length === 0) {
        rule.servers = [...serverIds]
      }
    }
  }
  return props.settings.resource_alert_rules
}

const resourceAlertRules = computed(() => ensureResourceAlertRules())
const resourceAlertExpanded = ref(false)
const resourceAlertRuleNameInputs = new Map()
const resourceAlertToggleText = computed(() => {
  const isZh = currentLang.value === 'zh'
  return resourceAlertExpanded.value
    ? (isZh ? '收起' : 'Collapse')
    : (isZh ? '展开' : 'Expand')
})

const toggleResourceAlertExpanded = () => {
  resourceAlertExpanded.value = !resourceAlertExpanded.value
}

const setResourceAlertRuleNameInput = (ruleId, input) => {
  if (!ruleId) return
  if (input) {
    resourceAlertRuleNameInputs.set(ruleId, input)
  } else {
    resourceAlertRuleNameInputs.delete(ruleId)
  }
}

const resourceAlertMetricOptions = computed(() => [
  { value: 'cpu', label: st('metricCpu') },
  { value: 'ram', label: st('metricRam') },
  { value: 'disk', label: st('metricDisk') },
  { value: 'netIn', label: st('metricNetIn') },
  { value: 'netOut', label: st('metricNetOut') }
])

const resourceAlertIntervalOptions = computed(() => (
  Array.from({ length: 6 }, (_, index) => ({
    value: String(index + 5),
    label: st('minutesValue').replace('{minutes}', index + 5)
  }))
))

const resourceAlertServerOptions = computed(() => (
  props.servers
    .map(server => ({
      id: String(server.id || '').trim(),
      name: server.name || server.id
    }))
    .filter(server => server.id)
))

const getResourceAlertRuleServerIds = (rule) => (
  Array.isArray(rule.servers)
    ? rule.servers.map(item => String(item || '').trim()).filter(Boolean)
    : []
)

const resourceAlertRuleHasServer = (rule, serverId) => (
  getResourceAlertRuleServerIds(rule).includes(String(serverId))
)

const isLastResourceAlertRuleServer = (rule, serverId) => {
  const selected = getResourceAlertRuleServerIds(rule)
  return selected.length === 1 && selected[0] === String(serverId)
}

const toggleResourceAlertRuleServer = (rule, serverId, checked) => {
  const id = String(serverId || '').trim()
  if (!id) return

  const selected = getResourceAlertRuleServerIds(rule)
  const selectedSet = new Set(selected)
  if (checked) {
    selectedSet.add(id)
  } else if (selectedSet.size > 1) {
    selectedSet.delete(id)
  }
  rule.servers = Array.from(selectedSet)
}

const closeResourceAlertServerDropdowns = (event) => {
  const target = event?.target
  if (!(target instanceof Element)) return
  const currentDropdown = target.closest('.resource-alert-server-dropdown')

  document.querySelectorAll('.resource-alert-server-dropdown[open]').forEach(dropdown => {
    if (dropdown === currentDropdown) return
    dropdown.removeAttribute('open')
  })
}

const resourceAlertServerSelectLabel = (rule) => {
  const options = resourceAlertServerOptions.value
  const total = options.length
  if (total === 0) return props.trans.noServers || 'No servers'

  const optionMap = new Map(options.map(server => [server.id, server.name]))
  const selected = getResourceAlertRuleServerIds(rule).filter(id => optionMap.has(id))
  if (selected.length === total) return `${props.trans.all || 'All'} (${total})`
  if (selected.length === 1) return optionMap.get(selected[0])
  return `${selected.length}/${total} ${props.trans.servers || 'Servers'}`
}

const createRuleId = () => `rule_${Date.now().toString(36)}_${Math.random().toString(36).slice(2, 8)}`
const isNetworkAlertMetric = metric => metric === 'netIn' || metric === 'netOut'
const resourceAlertThresholdPlaceholder = metric => isNetworkAlertMetric(metric) ? '100' : '80'
const resourceAlertThresholdMax = metric => isNetworkAlertMetric(metric) ? 100000 : 100

const normalizeRuleThreshold = (rule) => {
  const threshold = Number(rule.threshold)
  const max = resourceAlertThresholdMax(rule.metric)
  if (!Number.isFinite(threshold) || threshold <= 0 || threshold > max) {
    rule.threshold = resourceAlertThresholdPlaceholder(rule.metric)
  }
}

const addResourceAlertRule = () => {
  resourceAlertExpanded.value = true
  const rules = ensureResourceAlertRules()
  const metric = 'cpu'
  const rule = {
    id: createRuleId(),
    name: `${props.trans.resourceAlertRule || 'Resource Alert'} ${rules.length + 1}`,
    metric,
    threshold: resourceAlertThresholdPlaceholder(metric),
    servers: resourceAlertServerOptions.value.map(server => server.id),
    intervalMinutes: '5',
    mode: 'average'
  }
  rules.push(rule)
  nextTick(() => {
    const input = resourceAlertRuleNameInputs.get(rule.id)
    input?.focus()
    input?.select()
  })
}

const removeResourceAlertRule = (index) => {
  ensureResourceAlertRules().splice(index, 1)
}

const pingNodeErrorMessage = computed(() => (
  props.trans.invalidPingNodeFormat || 'Use domain, IPv4, or host:port. Port must be 1-65535.'
))

const pingNodeErrors = computed(() => Object.fromEntries(
  PING_NODE_FIELDS.map(field => [
    field,
    validatePingNode(props.settings[field]).valid ? '' : pingNodeErrorMessage.value
  ])
))

const hasPingNodeErrors = computed(() => Object.values(pingNodeErrors.value).some(Boolean))

const validatePingNodes = () => !hasPingNodeErrors.value

const isValidCspOrigin = (value) => {
  const raw = String(value || '').trim()
  if (!raw || /[\s;"']/.test(raw)) return false
  try {
    const url = new URL(raw)
    if (url.protocol !== 'https:') return false
    if (url.username || url.password || url.search || url.hash) return false
    if (url.pathname && url.pathname !== '/') return false
    return true
  } catch (_) {
    return false
  }
}

const validateCspField = (field) => {
  const value = props.settings[field] || ''
  if (!value) {
    cspErrors[field] = ''
    return true
  }
  const domains = value.split(',').map(s => s.trim()).filter(Boolean)
  for (const domain of domains) {
    if (!isValidCspOrigin(domain)) {
      cspErrors[field] = props.trans.cspInvalidDomain || 'Each domain must start with https://'
      return false
    }
  }
  cspErrors[field] = ''
  return true
}

onMounted(() => {
  document.addEventListener('click', closeResourceAlertServerDropdowns)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', closeResourceAlertServerDropdowns)
  clearTimeout(githubCallbackCopiedTimer)
})


// ---------- grouped layout ----------

const SETTINGS_TEXT = {
  zh: {
    offlineLabel: '离线通知',
    offlineOff: '不通知',
    offlineMinutes: '离线超过 {minutes} 分钟',
    expireLabel: '到期提醒',
    expireOff: '不提醒',
    expireDays: '提前 {days} 天开始提醒',
    customDays: '自定义天数…',
    daysUnit: '天',
    minutesValue: '{minutes} 分钟',
    resourceAlertHint: '服务器负载持续过高时发送通知。没有规则时不检查；规则之间互不影响。',
    ruleWhen: '当',
    ruleWithin: '在最近',
    ruleInside: '内的',
    ruleModeAverage: '平均值',
    ruleModeContinuous: '每一次采样都',
    ruleExceeds: '超过',
    ruleThen: '时通知',
    ruleServers: '适用服务器',
    metricCpu: 'CPU 使用率',
    metricRam: '内存使用率',
    metricDisk: '磁盘使用率',
    metricNetIn: '下行带宽',
    metricNetOut: '上行带宽',
    wssEnable: '实时推送（WebSocket）',
    wssHint: '开启后 Agent 通过长连接推送数据，页面刷新更及时、Workers 请求更少；代价是占用 Durable Objects 时长，全天开启约用掉免费额度的 85%。',
    wssHours: '开启时段',
    wssHoursHint: '按你的本地时间显示。未选中的时段，Agent 自动改回普通 HTTP 上报。',
    wsTimeoutLabel: '访客页面实时连接时长',
    wsTimeoutHint: '访客打开页面后保持实时刷新的最长时间，超时后断开并提示刷新，用来节省额度。',
    wsTimeoutNone: '不限制',
    wsTimeoutMinutes: '{minutes} 分钟后断开',
    wsTimeoutCustom: '{minutes} 分钟后断开（当前值）',
    historyPointsLabel: '长时间图表精度',
    historyPointsHint: '查看 1 小时以上的图表时每条曲线的点数。点越多越精细，但读取的数据库行数也越多。',
    historyPoints60: '60 点 · 省额度',
    historyPoints120: '120 点 · 推荐',
    historyPoints180: '180 点',
    historyPoints240: '240 点 · 最精细',
    adminTitle: '后台账号',
    passwordHint: '此密码只用于登录后台；Agent 上报使用的 API_SECRET 环境变量不受影响。',
    turnstileHint: '在登录页或整个站点加入 Cloudflare 人机验证，防止暴力破解和爬虫。密钥在 Cloudflare 控制台 → Turnstile 创建。',
    turnstileMode: '验证范围',
    turnstileOff: '关闭',
    turnstileLogin: '仅登录后台时验证',
    turnstileGlobal: '所有访问都验证（含首页）',
    jwtWarning: '修改后所有已登录的会话会立即失效，需要重新登录。',
    navChannel: '通知渠道', navChannelDesc: '消息发到哪里',
    navAlerts: '告警与报告', navAlertsDesc: '什么时候发、发什么',
    navSite: '站点与显示', navSiteDesc: '首页给访客看什么',
    navCollect: '数据采集', navCollectDesc: 'Agent 上报与延迟测试',
    navSecurity: '登录与安全', navSecurityDesc: '账号、第三方登录、人机验证',
    navAdvanced: '高级', navAdvancedDesc: '密钥、自定义代码、Cloudflare',
    channelTitle: '通知渠道',
    channelDesc: '选择接收通知的平台，并填写对应的凭据。所有告警、到期提醒和流量报告都通过这里发送。',
    channelType: '渠道类型',
    channelMismatch: '当前填写的内容看起来是 {type} 的格式，发送时会按 {type} 处理。',
    webhookBodyHint: '可使用 {{event}}、{{message}}、{{notification}}、{{time}} 等变量。',
    testHint: '使用当前页面上（未保存）的配置发送一条测试消息。',
    templateTitle: '消息模板',
    templateDesc: '一般无需修改',
    templateVars: '可用变量：{{emoji}} 图标、{{event}} 事件名、{{message}} 正文、{{client}} 服务器名、{{count}} 数量、{{time}} 时间。',
    alertsTitle: '告警与报告',
    alertsDesc: '设置哪些事件会发通知，以及每日任务的发送时间。',
    scheduleTitle: '发送时间',
    dailyHour: '每日发送时刻',
    dailyHourHint: '到期提醒和每日流量报告都在这个时刻发送，按左侧时区计算。',
    serverAlertsTitle: '服务器告警',
    offlineHint: '服务器连续这么久没有上报时发送离线通知，恢复后再发一条。',
    expireHint: '在服务器到期前的这段时间内，每天提醒一次。',
    trafficReportTitle: '流量报告',
    trafficReportEnable: '发送流量报告',
    trafficReportHint: '每天发送昨日流量和本周期用量，每周一发送上周汇总，到达重置日时发送上一周期结算。',
    rebuildBaselines: '按历史重建基线',
    rebuildBaselinesHint: '首次开启或报告显示“暂无数据”时使用：用历史记录补齐昨日、上周和本月的起始值。',
    siteTitle: '站点与显示',
    siteDesc: '控制首页对访客展示的内容和外观。',
    visibilityTitle: '访客可见内容',
    publicHint: '关闭后首页需要登录才能查看',
    appearanceTitle: '默认外观',
    imagesTitle: '图片',
    collectTitle: '数据采集',
    collectDesc: 'Agent 如何上报数据，以及延迟测试的目标节点。',
    realtimeTitle: '实时上报',
    pingHint: '每台服务器会定时 Ping 这些节点，用于首页的延迟和丢包图表。左边填显示名称，右边填地址；地址留空的节点不测试。',
    pingName: '名称',
    pingInvalid: 'Ping 节点格式有误，请检查“数据采集”',
    securityTitle: '登录与安全',
    securityDesc: '管理后台账号、第三方登录和人机验证。',
    githubTitle: 'GitHub 登录',
    turnstileTitle: 'Cloudflare Turnstile 人机验证',
    advancedTitle: '高级',
    advancedDesc: '一般无需修改。修改错误可能导致页面无法加载。',
    cloudflareTitle: 'Cloudflare 账户（查询 D1 用量）',
    cloudflareHint: '只用于在后台查询 D1 数据库的读写用量，不影响监控功能。',
    customCodeTitle: '自定义代码与安全策略',
    unsaved: '有未保存的修改',
    saved: '所有修改已保存'
  },
  en: {
    offlineLabel: 'Offline alert',
    offlineOff: 'Off',
    offlineMinutes: 'Offline for {minutes} min',
    expireLabel: 'Expiry reminder',
    expireOff: 'Off',
    expireDays: 'Start {days} days before',
    customDays: 'Custom days…',
    daysUnit: 'days',
    minutesValue: '{minutes} min',
    resourceAlertHint: 'Notify when server load stays high. No rules means no checks; rules are independent.',
    ruleWhen: 'When',
    ruleWithin: 'over the last',
    ruleInside: ', the',
    ruleModeAverage: 'average',
    ruleModeContinuous: 'every sample',
    ruleExceeds: 'exceeds',
    ruleThen: ', notify',
    ruleServers: 'Applies to',
    metricCpu: 'CPU usage',
    metricRam: 'Memory usage',
    metricDisk: 'Disk usage',
    metricNetIn: 'Download bandwidth',
    metricNetOut: 'Upload bandwidth',
    wssEnable: 'Real-time push (WebSocket)',
    wssHint: 'Agents push data over a persistent connection: fresher dashboards and fewer Workers requests, at the cost of Durable Objects duration (about 85% of the free quota if on all day).',
    wssHours: 'Active hours',
    wssHoursHint: 'Shown in your local time. Outside the selected hours agents fall back to plain HTTP reports.',
    wsTimeoutLabel: 'Visitor live-connection limit',
    wsTimeoutHint: 'How long a visitor page keeps live updates before disconnecting and asking to refresh, to save quota.',
    wsTimeoutNone: 'No limit',
    wsTimeoutMinutes: 'Disconnect after {minutes} min',
    wsTimeoutCustom: 'Disconnect after {minutes} min (current)',
    historyPointsLabel: 'Long-range chart detail',
    historyPointsHint: 'Points per series for ranges over 1 hour. More points are finer but read more database rows.',
    historyPoints60: '60 points · saves quota',
    historyPoints120: '120 points · recommended',
    historyPoints180: '180 points',
    historyPoints240: '240 points · finest',
    adminTitle: 'Admin account',
    passwordHint: 'Only used for the admin login; the agents\' API_SECRET environment variable is unaffected.',
    turnstileHint: 'Adds a Cloudflare captcha to the login page or the whole site against brute force and scrapers. Create keys in Cloudflare → Turnstile.',
    turnstileMode: 'Protect',
    turnstileOff: 'Off',
    turnstileLogin: 'Admin login only',
    turnstileGlobal: 'Every visit (incl. dashboard)',
    jwtWarning: 'Changing it signs out every session immediately.',
    navChannel: 'Channel', navChannelDesc: 'Where messages go',
    navAlerts: 'Alerts & reports', navAlertsDesc: 'When and what to send',
    navSite: 'Site & display', navSiteDesc: 'What visitors see',
    navCollect: 'Data collection', navCollectDesc: 'Agent reports & ping',
    navSecurity: 'Login & security', navSecurityDesc: 'Account, OAuth, captcha',
    navAdvanced: 'Advanced', navAdvancedDesc: 'Keys, custom code, Cloudflare',
    channelTitle: 'Notification channel',
    channelDesc: 'Choose where notifications are delivered and enter its credentials. Alerts, expiry reminders and traffic reports all use this channel.',
    channelType: 'Channel type',
    channelMismatch: 'The value looks like a {type} credential and will be sent as {type}.',
    webhookBodyHint: 'Variables such as {{event}}, {{message}}, {{notification}} and {{time}} are available.',
    testHint: 'Sends a test message using the (unsaved) values on this page.',
    templateTitle: 'Message template',
    templateDesc: 'Usually no change needed',
    templateVars: 'Variables: {{emoji}}, {{event}}, {{message}}, {{client}}, {{count}}, {{time}}.',
    alertsTitle: 'Alerts & reports',
    alertsDesc: 'Choose which events send notifications and when daily jobs run.',
    scheduleTitle: 'Schedule',
    dailyHour: 'Daily send time',
    dailyHourHint: 'Expiry reminders and daily traffic reports are sent at this hour in the selected timezone.',
    serverAlertsTitle: 'Server alerts',
    offlineHint: 'Notify when a server has not reported for this long, and again when it recovers.',
    expireHint: 'Remind once a day within this window before a server expires.',
    trafficReportTitle: 'Traffic reports',
    trafficReportEnable: 'Send traffic reports',
    trafficReportHint: 'Daily: yesterday and current-cycle usage. Mondays: last week. Reset day: last cycle summary.',
    rebuildBaselines: 'Rebuild baselines from history',
    rebuildBaselinesHint: 'Use when enabling reports or when a report shows "no data": fills the start values for yesterday, last week and this month from history.',
    siteTitle: 'Site & display',
    siteDesc: 'What the public dashboard shows and how it looks.',
    visibilityTitle: 'Visible to visitors',
    publicHint: 'When off, the dashboard requires login',
    appearanceTitle: 'Default appearance',
    imagesTitle: 'Images',
    collectTitle: 'Data collection',
    collectDesc: 'How agents report data and which targets they ping.',
    realtimeTitle: 'Real-time reporting',
    pingHint: 'Every server pings these targets for the latency and packet-loss charts. Left: display name, right: address; nodes with an empty address are skipped.',
    pingName: 'Name',
    pingInvalid: 'Invalid ping node, check Data collection',
    securityTitle: 'Login & security',
    securityDesc: 'Admin account, third-party login and captcha.',
    githubTitle: 'GitHub login',
    turnstileTitle: 'Cloudflare Turnstile captcha',
    advancedTitle: 'Advanced',
    advancedDesc: 'Usually no change needed. Wrong values may break page loading.',
    cloudflareTitle: 'Cloudflare account (D1 usage)',
    cloudflareHint: 'Only used to query D1 read/write usage in the admin panel.',
    customCodeTitle: 'Custom code & security policy',
    unsaved: 'Unsaved changes',
    saved: 'All changes saved'
  }
}
const st = key => (SETTINGS_TEXT[currentLang.value === 'zh' ? 'zh' : 'en'][key] ?? SETTINGS_TEXT.en[key] ?? key)

const settingsGroups = computed(() => [
  { id: 'channel', icon: '📨', label: st('navChannel'), desc: st('navChannelDesc') },
  { id: 'alerts', icon: '🔔', label: st('navAlerts'), desc: st('navAlertsDesc') },
  { id: 'site', icon: '🖥', label: st('navSite'), desc: st('navSiteDesc') },
  { id: 'collect', icon: '📡', label: st('navCollect'), desc: st('navCollectDesc') },
  { id: 'security', icon: '🔐', label: st('navSecurity'), desc: st('navSecurityDesc') },
  { id: 'advanced', icon: '⚙️', label: st('navAdvanced'), desc: st('navAdvancedDesc') }
])
const SETTINGS_GROUP_STORAGE_KEY = 'admin_settings_group'
const readStoredGroup = () => {
  try { return localStorage.getItem(SETTINGS_GROUP_STORAGE_KEY) || 'channel' } catch (_) { return 'channel' }
}
const activeGroup = ref(readStoredGroup())
const selectGroup = (id) => {
  activeGroup.value = id
  try { localStorage.setItem(SETTINGS_GROUP_STORAGE_KEY, id) } catch (_) { /* ignore */ }
}
watch(hasPingNodeErrors, (invalid) => { if (invalid) selectGroup('collect') })

// The backend picks the delivery method from the token's format (see
// sendNotification), so the type selector is derived from the stored value.
const isZh = () => currentLang.value === 'zh'
const CHANNELS = [
  { value: 'telegram', label: 'Telegram', match: () => false },
  { value: 'wecom', label: '企业微信 / WeCom', match: t => t.includes('https://qyapi.weixin.qq.com') },
  { value: 'feishu', label: '飞书 / Feishu', match: t => t.includes('open.feishu.cn') },
  { value: 'dingtalk', label: '钉钉 / DingTalk', match: t => t.includes('oapi.dingtalk.com') || t.includes('api.dingtalk.com') },
  { value: 'bark', label: 'Bark', match: t => t.includes('https://api.day.app/') || t.startsWith('bark:') },
  { value: 'serverchan', label: 'Server酱', match: t => t.includes('https://sctapi.ftqq.com/') || t.startsWith('server:') },
  { value: 'wxpusher', label: 'WxPusher', match: t => t.includes('https://wxpusher.zjiecode.com/api/send/message/SPT_') },
  { value: 'gotify', label: 'Gotify', match: t => t.includes('/message?token=') },
  { value: 'onebot', label: 'OneBot (QQ)', match: t => t.startsWith('onebot:') },
  { value: 'webhook', label: isZh() ? '自定义 Webhook' : 'Custom Webhook', match: () => false }
]
const detectChannel = (token) => {
  const value = String(token || '').trim()
  if (!value) return ''
  // onebot: must be checked before URL-based channels it may contain.
  if (value.startsWith('onebot:')) return 'onebot'
  const found = CHANNELS.find(channel => channel.value !== 'telegram' && channel.value !== 'onebot' && channel.match(value))
  return found ? found.value : 'telegram'
}
const channelOptions = computed(() => CHANNELS.map(({ value, label }) => ({
  value,
  label: value === 'webhook' ? (isZh() ? '自定义 Webhook' : 'Custom Webhook') : label
})))
const manualChannel = ref('')
const channelType = computed({
  get: () => {
    if (props.settings.notification_webhook_enabled) return 'webhook'
    return manualChannel.value || detectChannel(props.settings.tg_bot_token) || 'telegram'
  },
  set: (value) => {
    props.settings.notification_webhook_enabled = value === 'webhook'
    manualChannel.value = value === 'webhook' ? '' : value
  }
})
watch(() => props.settings, () => { manualChannel.value = '' })
const detectedChannel = computed(() => detectChannel(props.settings.tg_bot_token))
const detectedChannelLabel = computed(() => CHANNELS.find(channel => channel.value === detectedChannel.value)?.label || '')
const channelTokenMismatch = computed(() => (
  channelType.value !== 'webhook' && detectedChannel.value && detectedChannel.value !== channelType.value
))
const CHANNEL_FIELDS = {
  telegram: {
    zh: { tokenLabel: 'Bot Token', placeholder: '123456789:AAH...', hint: '在 Telegram 找 @BotFather 创建机器人后获得。', chatIdLabel: 'Chat ID', chatIdPlaceholder: '123456789', chatIdHint: '接收消息的用户或群组 ID，可通过 @userinfobot 获取。' },
    en: { tokenLabel: 'Bot Token', placeholder: '123456789:AAH...', hint: 'Create a bot with @BotFather on Telegram.', chatIdLabel: 'Chat ID', chatIdPlaceholder: '123456789', chatIdHint: 'User or group ID that receives messages (try @userinfobot).' },
    chatId: true
  },
  wecom: {
    zh: { tokenLabel: '群机器人 Webhook 地址', placeholder: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=...', hint: '企业微信群 → 添加群机器人 → 复制 Webhook 地址。' },
    en: { tokenLabel: 'Group bot webhook URL', placeholder: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=...', hint: 'WeCom group → Add group bot → copy the webhook URL.' }
  },
  feishu: {
    zh: { tokenLabel: '自定义机器人 Webhook 地址', placeholder: 'https://open.feishu.cn/open-apis/bot/v2/hook/...', hint: '飞书群设置 → 群机器人 → 添加自定义机器人。' },
    en: { tokenLabel: 'Custom bot webhook URL', placeholder: 'https://open.feishu.cn/open-apis/bot/v2/hook/...', hint: 'Feishu group settings → Bots → Add custom bot.' }
  },
  dingtalk: {
    zh: { tokenLabel: '自定义机器人 Webhook 地址', placeholder: 'https://oapi.dingtalk.com/robot/send?access_token=...', hint: '钉钉群设置 → 机器人 → 添加自定义机器人。' },
    en: { tokenLabel: 'Custom bot webhook URL', placeholder: 'https://oapi.dingtalk.com/robot/send?access_token=...', hint: 'DingTalk group settings → Bots → Add custom bot.' }
  },
  bark: {
    zh: { tokenLabel: 'Bark 推送地址', placeholder: 'https://api.day.app/你的Key/', hint: '打开 Bark App 复制推送地址；自建服务器可写成 bark:https://你的服务器/Key/。' },
    en: { tokenLabel: 'Bark push URL', placeholder: 'https://api.day.app/YOUR_KEY/', hint: 'Copy the URL from the Bark app; for self-hosted use bark:https://your-server/KEY/.' }
  },
  serverchan: {
    zh: { tokenLabel: 'SendKey 地址', placeholder: 'https://sctapi.ftqq.com/SCT....send', hint: '登录 Server酱 获取 SendKey，也可写成 server:你的SendKey。' },
    en: { tokenLabel: 'SendKey URL', placeholder: 'https://sctapi.ftqq.com/SCT....send', hint: 'Get a SendKey from ServerChan, or write server:YOUR_SENDKEY.' }
  },
  wxpusher: {
    zh: { tokenLabel: 'SPT 推送地址', placeholder: 'https://wxpusher.zjiecode.com/api/send/message/SPT_xxx', hint: '关注 WxPusher 公众号获取极简推送 SPT。' },
    en: { tokenLabel: 'SPT push URL', placeholder: 'https://wxpusher.zjiecode.com/api/send/message/SPT_xxx', hint: 'Get a simple-push SPT from WxPusher.' }
  },
  gotify: {
    zh: { tokenLabel: 'Gotify 消息地址', placeholder: 'https://gotify.example.com/message?token=...', hint: '在 Gotify 创建应用后，用服务器地址加 /message?token=应用Token。' },
    en: { tokenLabel: 'Gotify message URL', placeholder: 'https://gotify.example.com/message?token=...', hint: 'Create an application in Gotify and use /message?token=APP_TOKEN.' }
  },
  onebot: {
    zh: { tokenLabel: 'OneBot 接口地址', placeholder: 'onebot:http://127.0.0.1:3000/send_private_msg?access_token=...', hint: '以 onebot: 开头。私聊用 send_private_msg，群聊用 send_group_msg。', chatIdLabel: 'QQ 号 / 群号', chatIdPlaceholder: '123456', chatIdHint: '接收消息的 QQ 号或群号。' },
    en: { tokenLabel: 'OneBot API URL', placeholder: 'onebot:http://127.0.0.1:3000/send_private_msg?access_token=...', hint: 'Starts with onebot:. Use send_private_msg or send_group_msg.', chatIdLabel: 'User / group ID', chatIdPlaceholder: '123456', chatIdHint: 'QQ user or group that receives messages.' },
    chatId: true
  }
}
const currentChannel = computed(() => {
  const entry = CHANNEL_FIELDS[channelType.value] || CHANNEL_FIELDS.telegram
  return { ...(isZh() ? entry.zh : entry.en), chatId: Boolean(entry.chatId) }
})

const WS_TIMEOUT_PRESETS = [0, 10, 30, 60, 120, 360]
const wsTimeoutOptions = computed(() => {
  const current = Number(props.settings.frontend_ws_timeout_minutes) || 0
  const options = WS_TIMEOUT_PRESETS.map(minutes => ({
    value: minutes,
    label: minutes === 0 ? st('wsTimeoutNone') : st('wsTimeoutMinutes').replace('{minutes}', minutes)
  }))
  if (!WS_TIMEOUT_PRESETS.includes(current)) {
    options.push({ value: current, label: st('wsTimeoutCustom').replace('{minutes}', current) })
  }
  return options
})
const historyPointOptions = computed(() => HISTORY.LONG_RANGE_POINT_OPTIONS.map(points => ({
  value: String(points),
  label: st(`historyPoints${points}`) === `historyPoints${points}` ? `${points}` : st(`historyPoints${points}`)
})))

// Global Turnstile already covers the login form, so the two flags form one
// three-way choice.
const turnstileMode = computed({
  get: () => {
    if (props.settings.turnstile_enabled === true || props.settings.turnstile_enabled === 'true') return 'global'
    if (props.settings.turnstile_login_enabled === true || props.settings.turnstile_login_enabled === 'true') return 'login'
    return 'off'
  },
  set: (mode) => {
    props.settings.turnstile_enabled = mode === 'global'
    props.settings.turnstile_login_enabled = mode === 'login'
  }
})

const pingNodeFieldList = [
  { field: 'custom_ct', placeholder: 'gd-ct-dualstack.ip.zstaticcdn.com' },
  { field: 'custom_cu', placeholder: 'gd-cu-dualstack.ip.zstaticcdn.com' },
  { field: 'custom_cm', placeholder: 'gd-cm-dualstack.ip.zstaticcdn.com' },
  { field: 'custom_bd', placeholder: 'ip.zstaticcdn.com' },
  { field: 'node_1', placeholder: 'host[:port] / [IPv6]:port' },
  { field: 'node_2', placeholder: 'host[:port] / [IPv6]:port' },
  { field: 'node_3', placeholder: 'host[:port] / [IPv6]:port' },
  { field: 'node_4', placeholder: 'host[:port] / [IPv6]:port' }
]

// Unsaved-change tracking: snapshot after settings load and after each save.
const serializeSettings = () => {
  try { return JSON.stringify(props.settings) } catch (_) { return '' }
}
const savedSnapshot = ref('')
const captureSnapshot = () => nextTick(() => { savedSnapshot.value = serializeSettings() })
watch(() => props.settings, captureSnapshot, { immediate: true })
watch(() => props.servers.length, captureSnapshot)
watch(() => props.saving, (saving, wasSaving) => { if (wasSaving && !saving) captureSnapshot() })
const isDirty = computed(() => Boolean(savedSnapshot.value) && serializeSettings() !== savedSnapshot.value)

defineExpose({ validateCspField, cspErrors, validatePingNodes, pingNodeErrors })
</script>
