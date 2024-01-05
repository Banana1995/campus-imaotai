<template>
  <div class="app-container home">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>绑定用户</span>
      </div>
      <el-form ref="form" :model="form" label-width="120px">
        <el-form-item label="用户名">
          <el-input v-model="form.remark" />
        </el-form-item>
        <el-form-item label="手机号">
          <div style="display: flex;align-items: center; gap: 20px;">
            <el-input v-model="form.mobile" />
            <div>
              <el-button
                type="primary"
                :disabled="state"
                @click="sendCode(form.mobile)"
              >发送验证码<span v-if="state">({{ stateNum }})</span>
              </el-button>
            </div>
          </div>
        </el-form-item>
        <el-form-item label="验证码">
          <el-input v-model="form.code" />
        </el-form-item>
        <el-form-item label="选择门店">
          <el-card class="shop-card">
            <div slot="header" class="clearfix">
              <el-input v-model="shopSearchContent" @input="handleInputDebounce" @focus="showShopList = true" @blur="handleBlur" />
            </div>
            <template v-if="showShopList || !form.shop">
              <div v-for="shop in shopList" :key="shop.ishopId" class="shopListItem" @click="handleSelectShop(shop)">
                <span class="shopName"> {{ shop.tenantName }}  </span>
                <span class="shopAddress"> {{ shop.fullAddress }} </span>
              </div>
            </template>
          </el-card>
        </el-form-item>
        <el-form-item>
          <el-button
            type="primary"
            @click="submit"
          >提交
          </el-button>
        </el-form-item>
      </el-form>

    </el-card>
  </div>
</template>

<script>
import {
  getUser,
  updateUser,
  sendCode,
  login
} from '@/api/imt/user'
import { listShop } from '@/api/imt/shop'
import { debounce } from '@/utils'

export default {
  name: 'Index',
  data() {
    return {
      form: {
        remark: '',
        mobile: '',
        code: '',
        deviceId: '',
        shop: ''
      },
      stateNum: 60,
      state: false,
      shopList: [],
      queryParams: {
        pageNum: 1,
        pageSize: 10
      },
      total: 0,
      shopSearchContent: '',
      showShopList: false,
      shopLoading: false,
      handleInputDebounce: null
    }
  },
  created() {
    this.handleInputDebounce = debounce(this.getList, 500)
  },
  methods: {
    guid() {
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(
        /[xy]/g,
        function(c) {
          var r = (Math.random() * 16) | 0
          var v = c === 'x' ? r : (r & 0x3) | 0x8
          return v.toString(16)
        }
      )
    },
    sendCode(mobile, deviceId) {
      if (deviceId === undefined || deviceId === '') {
        this.form.deviceId = this.guid()
      } else {
        this.form.deviceId = deviceId
      }
      sendCode(mobile, this.form.deviceId).then(() => {
        this.$modal.msgSuccess('发送成功')
        this.state = true
        const timer = setInterval(() => {
          this.stateNum--
          if (this.stateNum === 0) {
            clearInterval(timer)
            this.state = false
            this.stateNum = 60
          }
        }, 1000)
      })
    },
    getList() {
      this.shopLoading = true
      listShop({
        pageNum: 1,
        pageSize: 50,
        cityName: this.shopSearchContent
      }).then((response) => {
        console.log(response.rows)
        this.shopList = response.rows
        this.total = response.total
        this.shopLoading = false
      })
    },
    handleBlur() {
      setTimeout(() => {
        this.showShopList = false
      }, 200)
    },
    handleSelectShop(shop) {
      console.log(shop)
      this.form.shop = shop.ishopId
      this.shopSearchContent = shop.tenantName
    },
    async login() {
      await login(this.form.mobile, this.form.code, this.form.deviceId)
    },
    async submit() {
      await this.login()
      const { data } = await getUser(this.form.mobile)
      updateUser({
        ...data,
        remark: this.form.remark,
        itemCode: 10213,
        ishopId: this.form.shop
      }).then(() => {
        this.$modal.msgSuccess('提交成功')
      })
    }
  }
}
</script>

<style scoped lang="scss">
.home {
  blockquote {
    padding: 10px 20px;
    margin: 0 0 20px;
    font-size: 17.5px;
    border-left: 5px solid #eee;
  }

  hr {
    margin-top: 20px;
    margin-bottom: 20px;
    border: 0;
    border-top: 1px solid #eee;
  }

  .col-item {
    margin-bottom: 20px;
  }

  ul {
    padding: 0;
    margin: 0;
  }

  font-family: "open sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 13px;
  color: #676a6c;
  overflow-x: hidden;

  ul {
    list-style-type: none;
  }

  h4 {
    margin-top: 0px;
  }

  h2 {
    margin-top: 10px;
    font-size: 26px;
    font-weight: 100;
  }

  p {
    margin-top: 10px;

    b {
      font-weight: 700;
    }
  }

  .update-log {
    ol {
      display: block;
      list-style-type: decimal;
      margin-block-start: 1em;
      margin-block-end: 1em;
      margin-inline-start: 0;
      margin-inline-end: 0;
      padding-inline-start: 40px;
    }
  }
}
.shopListItem {
  display: flex;
  justify-content: space-between ;
  padding: 10px 0;
  line-height: 16px;
  cursor: pointer;
  &:hover {
    background-color: #f5f5f5;
  }
  .shopName {
    color: #333;
  }
  .shopAddress {
    color: #999;
  }
}
</style>

<style>
  .shop-card .el-card__body {
      padding: 0px 20px 14px 20px;
  }
</style>

