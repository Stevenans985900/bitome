<template>
  <div class="action">
    <h3>Bitome Settings</h3>
    <el-form ref="form" label-width="100px" size="mini">
      <el-form-item label="Exchange">
        <el-switch
          v-model="settings.useUSD"
          :active-text="currencyDisplay"
          inactive-text="BTC"
        >
        </el-switch>
      </el-form-item>

      <el-form-item label="Raise color">
        <el-switch
          v-model="settings.useRedUp"
          active-color="#ff4949"
          inactive-color="#13ce66"
        >
        </el-switch>
      </el-form-item>
      <el-form-item label="Marquee">
        <el-switch
          v-model="settings.marquee"
          active-color="#13ce66"
        >
        </el-switch>
      </el-form-item>
      <el-form-item label="Currencys">
        <el-select class="select":xs="7" v-model="settings.currentCurrency.value">
          <el-option
            v-for="item in currencys"
            :key="`currencys-${item.value}`"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
    Version: {{package.version}} {{ env !== 'production' ? 'dev': '' }} by <a :href="mylink" target="_blank">Kelvin Ho</a>
  </div>
</template>
<script>
  import _isEmpty from 'lodash/isEmpty'
  import CONFIG from '../utils/config'
  import store from '../ext/storage'
  import PACKAGE from '../../package.json'

  const SETTING_DB_NAME = 'settings'
  export default {
    data: function () {
      return ({
        timer: [],
        mylink: 'https://kelvinho.js.org',
        settings: {
          currentCurrency: {
            value: 'USD'
          },
          marquee: true,
          useRedUp: false,
          useUSD: true
        },
        inited: false,
        currencys: CONFIG.currencys,
        package: PACKAGE,
        env: process.env.NODE_ENV
      })
    },
    computed: {
      currencyDisplay () {
        return this.settings.currentCurrency.value || 'USD'
      }
    },
    created () {
      this.loadSettings()
      this.timer.loadSetting = setInterval(this.loadSettings, 5 * 1000)
      this.$gtm.trackView('options', '/options.html')
    },
    watch: {
      settings: {
        handler: function (setting, oldSetting) {
          this.saveSettings()
        },
        deep: true
      }
    },
    methods: {
      loadSettings () {
        const that = this
        store.get(SETTING_DB_NAME)
          .then((db) => {
            if (JSON.stringify(this.settings) === JSON.stringify(db[SETTING_DB_NAME])) {
              return
            }
            if (!_isEmpty(db[SETTING_DB_NAME])) {
              if ('useRedup' in db[SETTING_DB_NAME]) {
                that.settings.useRedUp = db[SETTING_DB_NAME].useRedUp
              }
              if ('currentCurrency' in db[SETTING_DB_NAME]) {
                that.settings.currentCurrency = db[SETTING_DB_NAME].currentCurrency
              }
              if ('useUSD' in db[SETTING_DB_NAME]) {
                that.settings.useUSD = db[SETTING_DB_NAME].useUSD
              }
              if ('marquee' in db[SETTING_DB_NAME]) {
                that.settings.marquee = db[SETTING_DB_NAME].marquee
              }
            }
          }).finally(() => {
            that.inited = true
          })
      },
      saveSettings () {
        if (this.inited) {
          store.set(SETTING_DB_NAME, this.settings)
          this.$emit('changeSettings', this.settings)
        }
      }
    }
  }
</script>
<style>
</style>
