<template>
  <form ref="form" class="vns-create-form" @submit.prevent="submit">
    <div class="vns-create-form__main">
      <section>
        <h3>Bên nhận</h3>

        <div class="vns-form-group">
          <div class="vns-form-control">
            <label for="shipping_name">Họ tên</label>
            <input type="text" name="name" id="shipping_name" v-model="name">
          </div>

          <div class="vns-form-control">
            <label for="shipping_phone">Điện thoại</label>
            <input type="text" name="name" id="shipping_phone" v-model="phone">
          </div>
        </div>

        <div class="form-group">
          <div class="vns-form-control">
            <label for="shipping_address">Địa Chỉ</label>
            <input type="text" name="name" id="shipping_address" v-model="address">
          </div>
        </div>

        <div class="vns-form-control">
          <label>Quận/Huyện</label>
          <address-field v-model="address_data"/>
        </div>
      </section>

      <section>
        <h3>Hàng hoá</h3>

        <div class="vns-form-group is-3-columns">
          <div class="vns-form-control">
            <label for="length">Dài (cm)</label>

            <input
              v-model.number="length"
              type="number"
              id="length"
              name="length"
              min="0"
              max="200"
              required
            />
          </div>

          <div class="vns-form-control">
            <label for="width">Rộng (cm)</label>

            <input
              v-model.number="width"
              type="number"
              id="width"
              name="width"
              min="0"
              max="200"
              required
            />
          </div>

          <div class="vns-form-control">
            <label for="height">Cao (cm)</label>

            <input
              v-model.number="height"
              type="number"
              id="height"
              name="height"
              min="0"
              max="200"
              required
            />
          </div>
        </div>

        <div class="vns-form-group is-3-columns">
          <div class="vns-form-control">
            <label for="weight">Khối lượng (gram)</label>

            <input
              v-model.number="weight"
              type="number"
              id="weight"
              name="weight"
              min="0"
              max="1600000"
            />
          </div>

          <div class="vns-form-control">
            <label for="insurance">Tổng giá trị hàng hoá (VNĐ)</label>

            <input
              v-model.number="insurance"
              type="number"
              id="insurance"
              name="insurance"
              min="0"
              max="10000000"
            />

            <small class="form-text text-muted">
              <a href="https://viettelpost.com.vn/wp-content/uploads/2024/01/Dieu-khoan-va-dieu-kien-su-dung-dich-vu.pdf"
                 target="_blank">Qui trình</a>
              &nbsp; &amp; &nbsp;
              <a href="https://viettelpost.com.vn/wp-content/uploads/2023/10/PL2.-QUY-ĐỊNH-VỀ-CHÍNH-SÁCH-BỒI-THƯỜNG.pdf"
                 target="_blank">Chính sách xử lý đền bù</a>
            </small>
          </div>
        </div>
      </section>

      <section>
        <h3>Gói cước</h3>

        <notice
          v-if="errors.services"
          :error="errors.services"
        />

        <block-ui
          v-else
          :is-loading="isLoading('getAvailableServices')"
          :is-small="true">
          <div class="vns-form-group is-3-columns" v-if="availableServices && address_data.province && address_data.district">
            <div class="vns-form-control"  v-for="service in availableServices" :key="service.MA_DV_CHINH">
              <label>
                <input
                  v-model="ORDER_SERVICE"
                  :value="service"
                  type="radio"
                  name="MA_DV_CHINH"
                />

                <span>{{ service.TEN_DICHVU }}</span>
                <p>{{ service.GIA_CUOC.toLocaleString('vi-VN') }} VNĐ</p>
              </label>
            </div>
          </div>
          <block-ui
          v-else>
            <p class="vns-component-notice">Vui lòng nhập đầy đủ địa chỉ</p>
          </block-ui>
        </block-ui>
      </section>

      <section>
        <h3>Dịch vụ cộng thêm</h3>
        <notice
          v-if="errors.services"
          :error="errors.services"
        />

        <block-ui
          v-else
          :is-loading="isLoading('getAvailableServices')"
          :is-small="true">
          <div class="vns-form-group is-3-columns" v-if="availableServices && address_data.province && address_data.district && ORDER_SERVICE">
              <div class="vns-form-control"  v-for="extra_service in ORDER_SERVICE.EXTRA_SERVICE" :key="extra_service.SERVICE_CODE">
                <label>
                  <input
                    v-model="ORDER_EXTRA_SERVICE"
                    :value="extra_service.SERVICE_CODE"
                    type="checkbox"
                    name="SERVICE_NAME"
                  />

                  <span>{{ extra_service.SERVICE_NAME }}</span>
                </label>
              </div>
            </div>
          <block-ui
          v-else>
            <p class="vns-component-notice">Vui lòng nhập đầy đủ địa chỉ</p>
          </block-ui>
        </block-ui>
      </section>

      <section>
        <h3>Ghi chú</h3>
        <div class="vns-form-control">
          <label for="note">Ghi chú (không quá 120 ký tự)</label>
          <textarea name="note" id="note" rows="4" v-model="note" maxlength="120"></textarea>
        </div>
      </section>
    </div>

    <div class="vns-create-form__side">
      <div class="vns-create-form__submit">

         <h3>Ước tính cước phí</h3>

        <div class="vns-form-control">
          <label>Phí ship</label>

          <label style="display: inline-block; margin-right: 1.5rem;">
            <input
              v-model="is_freeship"
              :value="false"
              type="radio"
              name="is_freeship"
            />
            <span>Khách trả</span>
          </label>

          <label style="display: inline-block;">
            <input
              v-model="is_freeship"
              :value="true"
              type="radio"
              name="is_freeship"
            />
            <span>Shop trả</span>
          </label>
        </div>

        <div class="vns-form-control">
          <label for="cod">Thu hộ tiền COD (VNĐ)</label>

          <label style="display: inline-block; margin-right: 1.5rem;">
            <input
              v-model="codCheck"
              type="checkbox"
              name="codCheck"
            />
            <span>Thu hộ bằng tiền hàng</span>
          </label>

          <input
            v-model.number="cod"
            type="number"
            id="cod"
            name="cod"
            min="0"
            max="50000000"
            :disabled="codCheck"
          />
        </div>

        <div class="vns-form-control">
          <label style="color: #c1212d; font-weight: bold;">Tiền shop trả</label>

          <label style="display: inline-block; margin-right: 1.5rem;">
            <template v-if="ORDER_SERVICE">
              {{ is_freeship ? (ORDER_SERVICE.GIA_CUOC).toLocaleString('vi-VN') : '0' }} VNĐ
            </template>
            <template v-else>
             <p class="vns-component-notice">Vui lòng chọn gói cước</p>
            </template>
          </label>

        </div>

        <div class="vns-form-control">
          <label style="color: #c1212d; font-weight: bold;">Tiền khách trả</label>

          <label style="display: inline-block; margin-right: 1.5rem;">
            <template v-if="ORDER_SERVICE">
              {{ is_freeship ? cod.toLocaleString('vi-VN') : (ORDER_SERVICE.GIA_CUOC + cod).toLocaleString('vi-VN') }} VNĐ
            </template>
            <template v-else>
              <p class="vns-component-notice">Vui lòng chọn gói cước</p>
            </template>
          </label>

        </div>

        <button type="submit" class="button button-primary" style="background: #c1212d; border-color: #c1212d;" :disabled="!isValid">
          Tạo mã vận đơn
        </button>
      </div>

    </div>
  </form>
</template>

<script>
import Notice from '../../elements/Notice';
import BlockUi from '../../elements/BlockUi';
import AddressField from '../../elements/AddressField';

import {
  FormattingMixin,
  InteractsWithAPI,
  InteractsWithCreateOrder
} from '../../api';
import { castArray, debounce } from 'lodash';

export default {
  name: 'CreateVTPOrder',

  components: {
    Notice,
    BlockUi,
    AddressField
  },

  mixins: [
    FormattingMixin,
    InteractsWithAPI,
    InteractsWithCreateOrder
  ],

  data() {
    return {
      is_freeship: false,
      cod: 0,
      codManual: 0,
      codCheck: false,
      ORDER_SERVICE: {
        GIA_CUOC: 0,
        ORDER_SERVICE_ADD: [],
      },

      ORDER_EXTRA_SERVICE: [],
      ORDER_SERVICE_ADD: "",

      note: '',

      errors: {},
      availableServices: [],
    };
  },

  watch: {
    address_data: {
      handler(newVal) {
        if (newVal.province && newVal.district) {
          this.fetchServices();
        } else {
          this.availableServices = [];
          this.ORDER_SERVICE = null;
        }
        console.log(newVal.province);
      },
      deep: true
    },
    codCheck(newVal) {
      if (newVal) {
        // Save the manual cod before overriding
        this.codManual = this.cod;
        this.cod = this.insurance;
      } else {
        // Restore previous cod
        this.cod = this.codManual;
      }
    },
    insurance(newVal) {
      if (this.codCheck) {
        this.cod = newVal;
      }
    },
     ORDER_EXTRA_SERVICE: {
      deep: true,
      handler(newVal) {
        this.ORDER_SERVICE_ADD = newVal.join(',');
      },
    },
  },

  created() {
    this.debounceFetchServices = debounce(this.fetchServices, 550);

    this.fetchServices();

    this.$watch(() => this.address_data?.province, this.debounceFetchServices);
    this.$watch(() => this.address_data?.district, this.debounceFetchServices);
    this.$watch(() => this.address_data?.ward, this.debounceFetchServices);

    this.$watch('width', this.debounceFetchServices);
    this.$watch('height', this.debounceFetchServices);
    this.$watch('length', this.debounceFetchServices);
    this.$watch('weight', this.debounceFetchServices);
  },

  unmounted() {
    if (this.debounceFetchServices) {
      this.debounceFetchServices.cancel();
    }
  },

  methods: {

    async fetchServices() {
      if (!this.address_data?.province || !this.address_data?.district) {
        console.warn('Địa chỉ người nhận chưa đầy đủ.');
        return;
      }

      // Reset selection
      this.ORDER_SERVICE = '';
      this.ORDER_EXTRA_SERVICE = [];
      this.ORDER_SERVICE_ADD = "";
      this.errors.services = null;
      this.availableServices = [];

      try {
        const response = await this.getAvailableServices('vtp', {
          MONEY_COLLECTION: Number(this.cod) || 0,
          PRODUCT_PRICE: this.insurance,

          PRODUCT_WEIGHT: Number(this.weight),
          PRODUCT_LENGTH: Number(this.length),
          PRODUCT_WIDTH: Number(this.width),
          PRODUCT_HEIGHT: Number(this.height),

          RECEIVER_PROVINCE: Number(this.address_data.province),
          RECEIVER_DISTRICT: Number(this.address_data.district),
          SENDER_PROVINCE: Number(window.vnStoreInfo.province_code),
          SENDER_DISTRICT: Number(window.vnStoreInfo.district_code),
        });

        const services = Array.isArray(response) ? response : [response];

        this.availableServices = services.filter(
          (service, index, self) =>
            index === self.findIndex(s => s.MA_DV_CHINH === service.MA_DV_CHINH)
        );

        if (services.length > 0) {
          this.ORDER_SERVICE = services[0]; // Auto-select the first service
        } else {
          this.ORDER_SERVICE = null;
        }

        console.log('📦 ViettelPost Services:', services);
      } catch (error) {
        this.availableServices = null;

        const message =
          error?.message || 'ViettelPost Không thể lấy gói cước cho địa chỉ này. Vui lòng thử lại.';

        this.errors.services = message;
        console.error('❌ Lỗi khi lấy dịch vụ ViettelPost:', error);
      }
    },
  },

  computed: {
    isValid() {
      return true;
    },

    cod: {
      get() {
        // Always display as string — fallback to "0" when empty
        return this.cod === '' ? '0' : Number(this.cod);
      },
      set(value) {
        // If user clears it, reset to 0
        this.cod = value === '' ? 0 : Number(value);
      }
    }
  }
};
</script>
