<template>
  <v-container class="d-flex justify-center align-center fill-height">
    <SimpleForm
      :method="isUpdate ? 'patch' : 'post'"
      :data="formData"
      :endpoint="isUpdate ? 'submissions' : 'submissions'"
      return
    >
      <template #header>
        <v-row>
          <v-col
            style="display: flex; align-items: center; justify-content: center"
            cols="1"
          >
            <v-btn icon @click="returnBack">
              <v-icon>mdi-keyboard-backspace</v-icon>
            </v-btn>
          </v-col>
          <v-col cols="10">
            <v-card-title>{{ title }}</v-card-title>
          </v-col>
        </v-row>
      </template>
      <div class="span-2 my-8 px-md-16">
        <label>User</label>
        <v-select
          v-model="
            // eslint-disable-next-line vue/no-mutating-props
            submission.person
          "
          :rules="[required]"
          :readonly="isUpdate"
          single-line
          :items="users"
          item-value="_id"
          item-text="name"
          placeholder="e.g. Assigned User"
          dense
        >
          <template #no-data>
            <p style="margin: 0 5px 0 0; text-align: right">
              <span>
                No Users Found for this products theme.
                <span style="cursor: pointer" @click="getAll">Get All</span>
              </span>
            </p>
          </template>
        </v-select>
        <label>Product</label>
        <v-select
          v-model="
            // eslint-disable-next-line vue/no-mutating-props
            submission.product
          "
          :rules="[required]"
          :readonly="isUpdate || singleProduct"
          :items="products"
          item-text="name"
          item-value="_id"
          single-line
          placeholder="e.g. Assigned Product"
          dense
        ></v-select>
        <label>Service Name</label>
        <v-text-field
          v-model="
            // eslint-disable-next-line vue/no-mutating-props
            submission.service
          "
          :rules="[required]"
          single-line
          placeholder="e.g. FedX"
          dense
        ></v-text-field>
        <label>Tracking ID</label>
        <v-text-field
          v-model="
            // eslint-disable-next-line vue/no-mutating-props
            submission.trackingId
          "
          :rules="[required]"
          single-line
          placeholder="e.g. UD417638005YP"
          dense
        ></v-text-field>
        <label>Tracking URL</label>
        <v-text-field
          v-model="
            // eslint-disable-next-line vue/no-mutating-props
            submission.trackingUrl
          "
          :rules="[required]"
          single-line
          placeholder="e.g. https://tools.usps.com/go/TrackConfirmAction_input?strOrigTrackNum=UD417638005YP"
          dense
        ></v-text-field>
      </div>
    </SimpleForm>
  </v-container>
</template>

<script>
import { required } from '@/common/utils/validators'
import { Submission } from '@/models/submission'
import SimpleForm from '../../../common/ui/widgets/SimpleForm'
export default {
  name: 'SubmissionForm',
  components: {
    SimpleForm,
  },
  props: {
    title: {
      type: String,
      default: 'Title',
    },
    isUpdate: {
      type: Boolean,
      default: false,
    },
    submission: {
      type: Submission,
      default: () => new Submission(),
    },
    singleProduct: {
      type: Boolean,
      default: false,
    },
    productId: {
      type: String,
      default: null,
    },
  },
  data() {
    return {
      users: [],
      products: [],
    }
  },
  mounted() {
    this.getData()
    if (this.singleProduct) {
      this.selectProduct()
    }
  },
  methods: {
    required,
    returnBack() {
      this.$router.back()
    },
    async getData() {
      if (this.singleProduct) {
        this.users = await this.$axios.$get(
          '/persons/match-interests/' + this.productId
        )
      } else {
        this.users = await this.$axios.$get('/persons/customers')
      }
      this.products = await this.$axios.$get('/products')
    },
    async getAll() {
      this.users = await this.$axios.$get('/persons/customers')
    },
    selectProduct() {
      this.submission.product = this.productId
    },
    formData() {
      return this.submission
    },
  },
}
</script>

<style>
.form {
  width: 800px !important;
}
@media screen and (max-width: 600px) {
  .form {
    width: 100% !important;
  }
}
</style>
