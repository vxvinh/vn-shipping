<template>
  <form ref="form" class="vns-create-form" @submit.prevent="submit">
    <div class="vns-create-form__main">
      <section>
        <h3>Bên nhận</h3>

        <div class="vns-form-group">
          <div class="vns-form-control">
            <label for="shipping_name">Họ tên</label>
            <input type="text" name="shipping_name" id="shipping_name" v-model="name" :class="{ 'error': !name }" :placeholder="!name ? 'Vui lòng nhập họ tên' : ''" required>
          </div>

          <div class="vns-form-control">
            <label for="shipping_phone">Điện thoại</label>
            <input
              type="text"
              name="shipping_phone" 
              id="shipping_phone" 
              v-model="phone" 
              :class="{ 'error': !phone || !isPhoneValid }" 
              :placeholder="!phone ? 'Vui lòng nhập số điện thoại' : ''"  
              required
            />
            <p v-if="!isPhoneValid" class="error-text">Số điện thoại không hợp lệ</p>
          </div>
        </div>

        <div class="form-group">
          <div class="vns-form-control">
            <label for="shipping_address">Địa Chỉ</label>
            <input type="text" name="shipping_address" id="shipping_address" v-model="address" :class="{ 'error': !address }" :placeholder="!address ? 'Vui lòng nhập địa chỉ' : ''" required>
          </div>
        </div>

        <div class="vns-form-control">
          <label>Quận/Huyện</label>
          <address-field v-model="address_data" />
        </div>
      </section>

      <section>
        <h3>Hàng hoá</h3>

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
              :class="{ 'error': !weight }"
              :placeholder="!weight ? 'Vui lòng nhập khối lượng của hàng hoá' : ''"
              required
            />
            <p v-if="weightError" class="error-text">Khối lượng phải là một số hợp lệ</p>
          </div>

          <div class="vns-form-control">
            <label for="insurance">Tổng giá trị hàng hoá (VNĐ)</label>

            <input
              v-model.number="insurance"
              type="number"
              id="insurance"
              name="insurance"
              min="0"
              max="50000000"
              :class="{ 'error': insurance === null || insurance === '' }"
              :placeholder="(insurance === null || insurance === '') ? 'Vui lòng nhập giá trị' : ''"
              required
            />
            <p v-if="insuranceError" class="error-text">Giá trị hàng hoá phải là một số hợp lệ</p>
          </div>
        </div>
      </section>

      <section>
         <h3>Gói cước</h3>

        <div class="vns-form-control">
          <label>Hình thức vận chuyển</label>

          <label style="display: inline-block; margin-right: 1.5rem;">
            <input
              v-model.number="transport"
              type="radio"
              value="road"
              name="transport"
            />
            <span>Đường bộ</span>
          </label>

          <label style="display: inline-block;">
            <input
              v-model.number="transport"
              type="radio"
              value="fly"
              name="transport"
            />
            <span>Đường bay</span>
          </label>

          <label>
            <template v-if="serviceFees && address_data.province && address_data.district">
              {{ formattedServiceFee }} VNĐ
            </template>
            <template v-else>
             <p class="vns-component-notice">Vui lòng nhập đầy đủ địa chỉ</p>
            </template>
          </label>

          <p style="margin: 0; color: #999;">
            <i>Nếu phương thức vận chuyển không hợp lệ thì GHTK sẽ tự động nhảy về PTVC mặc định</i>
          </p>

        </div>
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

          <label style="display: inline-block;  margin-right: 1.5rem;">
            <input
              v-model.number="is_freeship"
              :value=0
              type="radio"
              name="is_freeship"
            />
            <span>Khách trả</span>
          </label>

          <label style="display: inline-block;">
            <input
              v-model.number="is_freeship"
              :value=1
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
            :class="{ 'error': cod === null || cod === '' }"
            :placeholder="(cod === null || cod === '') ? 'Vui lòng nhập số tiền thu hộ' : ''"
          />
          <p v-if="codError" class="error-text">Tiền thu hộ phải là một số hợp lệ</p>
        </div>

        <div class="vns-form-control">
          <label style="color: #00693a; font-weight: bold;">Tiền shop trả</label>

          <label style="display: inline-block; margin-right: 1.5rem;">
            <template v-if="serviceFees && address_data.province && address_data.district">
              {{ is_freeship == 1 ? (serviceFees.fee).toLocaleString('vi-VN') : '0' }} VNĐ
            </template>
            <template v-else>
             <p class="vns-component-notice">Vui lòng chọn gói cước</p>
            </template>
          </label>

        </div>

        <div class="vns-form-control">
          <label style="color: #00693a; font-weight: bold;">Tiền khách trả</label>

          <label style="display: inline-block; margin-right: 1.5rem;">
            <template v-if="serviceFees && address_data.province && address_data.district">
              {{ is_freeship == 1 ? cod.toLocaleString('vi-VN') : (serviceFees.fee + cod).toLocaleString('vi-VN') }} VNĐ
            </template>
            <template v-else>
              <p class="vns-component-notice">Vui lòng chọn gói cước</p>
            </template>
          </label>

        </div>

        <button type="submit" class="button button-primary" style="background: #00693a; border-color: #00693a;" :disabled="!isValid">
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

const TAGS = {
  '1': 'Dễ vỡ',
  '7': 'Nông sản/thực phẩm khô'
};

export default {
  name: 'CreateGHTKOrder',

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
      pick_option: 'cod', // cod | post
      transport: 'road',
      is_freeship: Number(window.vnOrderConfigGHTK?.is_freeship ?? 0),
      cod: 0,
      codManual: 0,
      codCheck: false,

      deliver_option: 'none',
      tags: [], //

      serviceFees: null,

      errors: {}
    };
  },

  computed: {
    isValid() {
      return (
        this.name &&
        this.phone && this.address &&
        this.address_data?.province &&
        this.address_data?.district &&
        this.weight > 0 &&
        this.transport &&
        typeof this.insurance === 'number' && this.insurance >= 0 &&
        typeof this.cod === 'number' && this.cod >= 0 &&
        this.isPhoneValid &&
        !this.weightError &&
        !this.insuranceError &&
        !this.codError
      );
    },
    formattedServiceFee() {
      return this.serviceFees?.fee?.toLocaleString('vi-VN') || '0';
    },
    isPhoneValid() {
      const phonePattern = /^(0|\+84)[0-9]{9}$/; // Vietnamese phone numbers
      return phonePattern.test(this.phone);
    },
    weightError() {
      return this.weight === null || this.weight === '' || isNaN(this.weight) || this.weight <= 0;
    },
    insuranceError() {
      return ['insurance'].some(field => {
        const value = this[field];
        return value === '' || value === null || isNaN(value) || value < 0;
      });
    },
    codError() {
      return ['cod'].some(field => {
        const value = this[field];
        return value === '' || value === null || isNaN(value) || value < 0;
      });
    }
  },

  created() {
    this.debounceFetchFees = debounce(this.fetchFees, 450);

    this.fetchFees();

    this.$watch('insurance', this.debounceFetchFees);
    this.$watch('transport', this.debounceFetchFees);
    this.$watch('weight', this.debounceFetchFees);
  },

  unmounted() {
    if (this.debounceFetchFees) {
      this.debounceFetchFees.cancel();
    }
  },

  watch: {
    address_data: {
      handler(newVal) {
        if (newVal.province && newVal.district) {
          this.fetchFees();
        } else {
          this.serviceFees = null;
        }
      },
      deep: true
    },
    codCheck(newVal) {
      if (newVal) {
        this.codManual = this.cod;
        this.cod = this.insurance;
      } else {
        this.cod = this.codManual;
      }
    },
    insurance(newVal) {
      if (this.codCheck && this.cod !== newVal) {
        this.cod = newVal;
      }
    },
  },

  methods: {
    async fetchFees() {
      if (!this.isValid) return;

      if (!this.address_data?.province || !this.address_data?.district ) return;

      if (!this.weight) {
        return;
      }

      this.serviceFees = null;

      this.getShippingFee('ghtk', {
        pick_province: window.vnStoreInfoGHTK.province,
        pick_district: window.vnStoreInfoGHTK.district,
        address: this.address_data?.address,
        province: this.address_data?.province,
        district: this.address_data?.district,
        ward: this.address_data?.ward,
        weight: this.weight,
        value: this.insurance,
        transport: this.transport,
        deliver_option: this.deliver_option,
        vue: 1,
      }, true).then(response => {
        if (response && response.fee !== undefined) {
          this.serviceFees = response;
        } else {
          this.serviceFees = null;
          console.warn('GHTK shipping fee not returned as expected', response);
        }
      });
    },
  },
};
</script>
