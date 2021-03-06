<template>
  <v-card width="50%" min-height="350px" class="mx-auto my-10 py-7 text-center">
    <div class="text-h5 mb-2">Create An Account</div>
    <div v-if="!loading" class="mb-2">
      <CircularImagePicker
        v-model="imageFile"
        :image="sendImage"
        icon="mdi-image-plus"
        icon-color="#C80000"
        @input="sendImage = $event"
      />
    </div>
    <v-progress-circular
      v-if="loading"
      indeterminate
      color="#313F53"
      class="my-auto mx-0"
    ></v-progress-circular>
    <v-form v-if="!loading" ref="form" class="form-div mx-auto">
      <ul
        v-if="errors.length"
        style="color: red; margin-bottom: 15px; text-align: left"
      >
        <li v-for="(error, i) of errors" :key="i">
          {{ error }}
        </li>
      </ul>
      <ul
        v-if="success"
        style="color: green; margin-bottom: 15px; text-align: left"
      >
        <li>Account Created Successfully</li>
      </ul>
      <v-container>
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="person.name"
              color="primary"
              label="Name"
              placeholder="John Doe"
              :rules="[required]"
              dense
            ></v-text-field>
          </v-col>
          <v-col cols="12">
            <v-text-field
              v-model="person.phone"
              color="primary"
              label="Phone"
              placeholder="+92 123 45678590"
              :rules="[phoneValidator]"
              dense
            ></v-text-field>
          </v-col>
          <v-col cols="12">
            <v-text-field
              v-model="person.username"
              color="primary"
              label="Email"
              :rules="[required, emailValidator]"
              placeholder="johndoe@gmail.com"
              dense
            ></v-text-field>
          </v-col>
          <v-col cols="12" sm="12" md="6" lg="6">
            <v-text-field
              v-model="person.password"
              :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
              :type="showPassword ? 'text' : 'password'"
              :rules="[required, lengthValidator]"
              label="Password"
              dense
              @click:append="showPassword = !showPassword"
            ></v-text-field>
          </v-col>
          <v-col cols="12" sm="12" md="6" lg="6">
            <v-text-field
              v-model="confirmPassword"
              :append-icon="showConfirmPassword ? 'mdi-eye' : 'mdi-eye-off'"
              :type="showConfirmPassword ? 'text' : 'password'"
              :rules="[required]"
              label="Confirm Password"
              dense
              @click:append="showConfirmPassword = !showConfirmPassword"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-btn
          x-large
          block
          class="white--text rounded-md my-2"
          color="primary"
          elevation="2"
          @click="register"
          >Register
        </v-btn>
      </v-container>
    </v-form>
  </v-card>
</template>

<script>
import CircularImagePicker from '@/components/helper/circularImagePicker'
import { Person } from '@/models/person'
import {
  required,
  emailValidator,
  phoneValidator,
  lengthValidator,
} from '@/common/utils/validators'
import { userRole } from '@/common/utils/local-data'

export default {
  name: 'RegisterForm',
  components: { CircularImagePicker },
  data() {
    return {
      loading: false,
      success: false,
      errors: [],
      person: new Person(),
      showPassword: false,
      showConfirmPassword: false,
      confirmPassword: '',
      imageFile: null,
      sendImage: null,
    }
  },
  methods: {
    required,
    emailValidator,
    phoneValidator,
    lengthValidator,
    userRole,
    async register() {
      if (this.$refs.form.validate()) {
        try {
          this.loading = true
          this.errors = []
          if (this.person.password !== this.confirmPassword) {
            this.loading = false
            this.errors.push('Could not confirm password.')
            return
          }
          const formData = new FormData()
          for (const key of Object.keys(this.person)) {
            if (this.person[key] !== undefined) {
              if (
                key === 'interests' ||
                key === 'characters' ||
                key === 'films'
              ) {
                continue
              } else {
                formData.append(key, this.person[key])
              }
            }
          }
          if (this.sendImage) {
            formData.append('image', this.sendImage)
          }
          formData.append('role', userRole('Admin').toString())
          await this.$axios.post('persons', formData)
          this.person = new Person()
          this.confirmPassword = ''
          this.sendImage = null
          this.imageFile = null
          this.loading = false
          this.success = true
        } catch (err) {
          this.loading = false
          if (err.response) {
            this.errors.push(err.response.data.message)
          } else {
            this.errors.push('Unable to process, Try again later')
          }
        }
      }
    },
  },
}
</script>

<style scoped>
.form-div {
  width: 90%;
}
</style>
