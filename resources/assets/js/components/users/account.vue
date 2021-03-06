<template>
  <div class="bg-white rounded shadow py-8 mt-8">
    <div class="flex flex-col py-4">
      <div class="mb-4 px-8">
        <label class="block text-grey-darker text-sm font-bold mb-2" for="username">
          {{ 'Email' | localize }}
        </label>
        <input v-model="user.email" class="shadow appearance-none border rounded w-full py-2 px-3 text-grey-darker" id="email" type="email" placeholder="john@example.com">
        <div v-if="errorMessage.email" class="text-red text-base pt-2">
          {{ errorMessage.email }}
        </div>
      </div>
      <div class="border-half mt-4 mb-8">
      </div>
      <div class="mb-4 px-8">
        <label class="block text-grey-darker text-sm font-bold mb-2" for="username">
          {{ 'Username' | localize }}
          <span v-if="!usernameAvailable" class="text-red-light font-normal text-sm">(Username taken. Please try again)</span>
        </label>
        <div class="relative">
          <input @keyup="checkUsernameAvailability" v-model='user.username' class="shadow appearance-none border rounded w-full py-2 px-3 text-grey-darker" id="username" type="text" placeholder="Nehal Hasnayeen">
          <font-awesome-icon v-if="usernameAvailable" :icon="faCheck" class="absolute pin-y pin-r text-green pt-1 mt-2 mr-2"></font-awesome-icon>
          <font-awesome-icon v-if="!usernameAvailable" :icon="faTimes" class="absolute pin-y pin-r text-red pt-1 mt-2 mr-2"></font-awesome-icon>
        </div>
      </div>
      <div class="border-half mt-4 mb-8">
      </div>
      <div class="mb-8 px-8">
        <label class="block text-grey-darker text-sm font-bold mb-2" for="username">
          {{ 'Current Password' | localize }}
        </label>
        <input v-model="currentPassword" class="shadow appearance-none border rounded w-full py-2 px-3 text-grey-darker" id="current_password" type="password" placeholder="******">
        <div v-if="errorMessage.current_password" class="text-red text-base pt-2">
          {{ errorMessage.current_password }}
        </div>
      </div>
      <div class="mb-8 px-8">
        <label class="block text-grey-darker text-sm font-bold mb-2" for="username">
          {{ 'New Password' | localize }}
        </label>
        <input v-model="newPassword" class="shadow appearance-none border rounded w-full py-2 px-3 text-grey-darker" id="new_password" type="password" placeholder="******">
        <div v-if="errorMessage.new_password" class="text-red text-base pt-2">
          {{ errorMessage.new_password }}
        </div>
      </div>
      <div class="mb-8 px-8">
        <label class="block text-grey-darker text-sm font-bold mb-2" for="username">
          {{ 'Confirm Password' | localize}}
        </label>
        <input v-model="newPasswordConfirmation" class="shadow appearance-none border rounded w-full py-2 px-3 text-grey-darker" id="confirm_password" type="password" placeholder="******">
        <div v-if="errorMessage.new_password_confirmation" class="text-red text-base pt-2">
          {{ errorMessage.new_password_confirmation }}
        </div>
      </div>
      <div class="my-6 px-8">
        <a @click="update" class="font-bold rounded shadow bg-teal-light p-4 text-white hover:shadow-lg hover:font-bold hover:bg-teal-dark no-underline" href="#">
          {{ 'Update' | localize }}
        </a>
      </div>
      <div class="border-half mt-4 mb-8">
      </div>
      <div class="mt-4 px-8">
        <a class="rounded p-4 text-red-light border border-grey-light hover:font-bold hover:border-0 hover:shadow-lg hover:bg-red-light hover:text-white no-underline" href="#">
          {{ 'Delete Account' | localize }}
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import { faCheck, faTimes } from '@fortawesome/free-solid-svg-icons'

export default {
  props: {
    user: {
      required: true,
      type: Object
    }
  },
  data: () => ({
    currentPassword: '',
    newPassword: '',
    newPasswordConfirmation: '',
    errorMessage: {
      email: '',
      current_password: '',
      new_password: '',
      new_password_confirmation: ''
    },
    usernameAvailable: true,
    faCheck,
    faTimes
  }),
  methods: {
    update () {
      var params = {}
      params.email = this.user.email
      params.username = this.user.username
      if (this.newPassword !== '') {
        params.current_password = this.currentPassword
        params.new_password = this.newPassword
        params.new_password_confirmation = this.newPasswordConfirmation
      }
      axios.put('/users/' + this.user.username + '/account', params)
        .then((response) => {
          EventBus.$emit('notification', response.data.message, response.data.status)
          this.email = ''
          this.username = ''
          this.currentPassword = ''
          this.newPassword = ''
          this.newPasswordConfirmation = ''
        })
        .catch((error) => {
          for (const prop in error.response.data.errors) {
            if (error.response.data.errors.hasOwnProperty(prop)) {
              this.$set(this.errorMessage, prop, error.response.data.errors[prop][0])
            }
          }
          setTimeout(() => {
            this.errorMessage = {
              email: '',
              current_password: '',
              new_password: '',
              new_password_confirmation: ''
            }
          }, 2000)
        })
    },
    checkUsernameAvailability () {
      axios.get('/username', {
        params: {
          username: this.user.username
        }
      })
        .then((response) => {
          if (response.data.status === 'success') {
            this.usernameAvailable = true
          }
        })
        .catch((error) => {
          this.usernameAvailable = false
          console.error(error)
        })
    }
  }
}
</script>
